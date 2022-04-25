---
title: "GCE 透過 Cloud VPN mount 不同 VPC networks 中的 Filestore"
date: 2022-04-25T00:30:00+08:00
lastmod: 2022-04-25T00:30:31+08:00
draft: false
tags: ["GCP","Network"]
slug: "gce-mount-other-vpc-filestore"
---

## GCE 透過 Cloud VPN mount 不同 VPC networks 中的 Filestore

目前團隊 production 上 GKE 資料都是儲存在 Filestore 上，最近有計劃使用儲存在 Filestore 的 log 內容來做後續處理，大致概念是將 log 集中至某一台 server (預計會使用 GCE VM)上，再由 GCE 上的 application 來處理 log

概念上很簡單，就是將多個 Filestore mount 到同一個 GCE，但因為 Filestore 是依 VPC 切分開的資源，所以實作上就比想像中複雜不少，我試了 2-3 天才成功，趕緊來紀錄一下設定方式，不然怕是過幾天就忘了XD

## 基本環境說明

1. GCE

    > Debian 4.19.235-1 (2022-03-17) x86_64

2. Cloud VPN
3. Cloud Router
4. Filestore
5. 前置作業

    - 建立兩個模擬用 VPC

        > 使用 `vpc-1` 與 `vpc-2` 為例

        - `vpc-1`

            ```gcloud
            gcloud compute networks create vpc-1 --project={project_id} --subnet-mode=custom --mtu=1460 --bgp-routing-mode=global && gcloud compute networks subnets create vpc-1 --project={project_id} --range=10.51.0.0/16 --network=vpc-1 --region=asia-east1 && gcloud compute firewall-rules create vpc-1-allow-icmp --project={project_id} --network=projects/{project_id}/global/networks/vpc-1 --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp && gcloud compute firewall-rules create vpc-1-allow-ssh --project={project_id} --network=projects/{project_id}/global/networks/vpc-1 --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
            ```

        - `vpc-2`

            ```gcloud
            gcloud compute networks create vpc-2 --project={project_id} --subnet-mode=custom --mtu=1460 --bgp-routing-mode=global && gcloud compute networks subnets create vpc-2 --project={project_id} --range=10.52.0.0/16 --network=vpc-2 --region=asia-east1 && gcloud compute firewall-rules create vpc-2-allow-icmp --project={project_id} --network=projects/{project_id}/global/networks/vpc-2 --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp && gcloud compute firewall-rules create vpc-2-allow-ssh --project={project_id} --network=projects/{project_id}/global/networks/vpc-2 --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
            ```

    - 建立兩個 Filestore instance

        > - 分屬於兩個 VPC：
        >   - `test-filestore-1` 在 `vpc-1`
        >   - `test-filestore-2` 在 `vpc-2`
        > - 這個部份 ui 上沒有提供 gcloud shell 建議語法，就不附了

        ![1test-filestore-1](https://user-images.githubusercontent.com/3851540/165066073-a9b160bb-6da1-4597-8895-9378b01321f1.png)

        ![1test-filestore-2](https://user-images.githubusercontent.com/3851540/165066094-3338d94f-70e6-46d9-a3c9-46142b273097.png)

    - 建立一個 GCE VM 並安裝 NFS

        - 建立 GCE：以 `test-filestore-gce-1` 為例

            ```gcloud
            gcloud compute instances create test-filestore-gce-1 --project={project_id} --zone=asia-east1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=vpc-1 --maintenance-policy=MIGRATE --service-account={service_account} --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=tmp-ssh --create-disk=auto-delete=yes,boot=yes,device-name=test-filestore-gce-1,image=projects/debian-cloud/global/images/debian-10-buster-v20220406,mode=rw,size=10,type=projects/{project_id}/zones/us-central1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
            ```

        - GCE instance 上安裝 NFS

            ```gcloud
            sudo apt -y update && sudo apt install -y nfs-common
            ```

        - 可以 mount 相同 VPC 下的 Filestore，但不同 VPC 的 Filestore 無法成功(出現 `mount.nfs: Connection timed out`)

            ```bash
            sudo mount {filestore_ip}:/{file_share_name} {local path}
            ```

            ![timedout](https://user-images.githubusercontent.com/3851540/165066364-790c7b7b-0b0a-4b57-8714-009c1f993c64.png)

## 設定方式

1. 建立 Cloud Router

    > 建立兩個 Cloud Router：
    >
    >   - `test-filestore-router-1`
    >   - `test-filestore-router-2`

    - 一個 VPC 就建立一個 Cloud Router

        > - `test-filestore-router-1` 建立在 `vpc-1`
        > - `test-filestore-router-2` 建立在 `vpc-2`

    - 每個 Cloud Router 的 Google ASN 都需要不同

        > - `test-filestore-router-1` 使用 `65001`
        > - `test-filestore-router-2` 使用 `65002`

        ![2routerasn](https://user-images.githubusercontent.com/3851540/165066115-8582fbfb-53de-445e-a6d7-c3c09756d462.png)

    - 需要設定使用 custom rules 並將 Filestore 的 ip range 進行通告

        > 否則一樣會出現 `mount.nfs: Connection timed out`

        ![3routerfilestoreip](https://user-images.githubusercontent.com/3851540/165066125-44da0077-226b-49bb-a61a-58179e9598f7.png)

        ![4filestorerange1](https://user-images.githubusercontent.com/3851540/165066140-8bfb507f-c6a9-4c78-977f-40095ca68a0c.png)

    - 需要將所有 subnet 對 Cloud Router 通告

        > 我不知道為什麼需要，但不設定 mount filestore 時會失敗 (會出現 `mount.nfs: Connection timed out`)

        ![5routerallsubnet](https://user-images.githubusercontent.com/3851540/165066163-03bd53fa-59ec-43ab-985e-916b9f354cf4.png)

    - gcloud 語法

        - `test-filestore-router-1`

            ```gcloud
            gcloud compute routers create test-filestore-router-1 --project={project_id} --region=asia-east1 --network=vpc-1 --asn=65001 --set-advertisement-mode=custom --set-advertisement-groups=all_subnets --set-advertisement-ranges=10.104.214.56/29
            ```

        - `test-filestore-router-2`

            ```gcloud
            gcloud compute routers create test-filestore-router-2 --project={project_id} --region=asia-east1 --network=vpc-2 --asn=65002 --set-advertisement-mode=custom --set-advertisement-groups=all_subnets --set-advertisement-ranges=10.44.216.48/29
            ```

2. 建立 HA Cloud VPN Gateway

    > 分別在兩個 VPC 上各建立一個 VPN Gateway：
    >
    > - `test-filestore-gateway-1` 建立在 `vpc-1` 中
    > - `test-filestore-gateway-2` 建立在 `vpc-2` 中

    ![6gateway](https://user-images.githubusercontent.com/3851540/165066169-6cdf9852-afb8-43c4-bc4a-1f30c2cacc76.png)

    ![6gateway2](https://user-images.githubusercontent.com/3851540/165066178-70961cf9-3bf8-4f0c-bb0c-2ec9aa9be552.png)

3. 在 Cloud VPN Gateway 中新增 VPN Tunnel

    > 兩個 Cloud VPN Gateway 都要分別設定一次，一個 Cloud VPN Gateway 有兩條 VPN Tunnel 共會建立 `四條` VPN Tunnel

    ![7addtunnel](https://user-images.githubusercontent.com/3851540/165066190-9e5baf41-b88a-4d35-bc41-2e2dae5b00fb.png)

    - 選擇目標 `Google Cloud Project` , `VPN Gateway` 與 `Create a pair of VPN tunnels`

        ![8tunnelproject1](https://user-images.githubusercontent.com/3851540/165066201-723bb4a3-c32e-4c7c-ba0d-0412ea9cff2e.png)

        ![8tunnelproject2](https://user-images.githubusercontent.com/3851540/165066225-04f5ab9f-36aa-4fa0-b91b-9fbe4925bfee.png)

    - 選擇 `Cloud Router` 並設定 VPN tunnel

        > 這邊需要與目標 VPC 的 VPN tunnel 做成對設定

        - peer VPN Gateway 選擇目標正確會自動將本身 Gateway 的 interface 與目標 Gateway interface 做關聯
        - 選擇 IKE version 為 `IKEv1` 並設定 IKE pre-shared key

            > `IKE pre-shared key` 需要與目標 VPN tunnel 相同 (其中一邊產生即可)

        > 以下透過不同顏色的框線來呈現成對設定的部份

        - tunnel1

            ![9tunnel1](https://user-images.githubusercontent.com/3851540/165066236-4fdccac3-6faf-4056-afae-4a4a15fcb740.png)

            ![10tunnel1](https://user-images.githubusercontent.com/3851540/165066253-3212bda7-9542-42c8-9b05-006d6597c4bd.png)

        - tunnel2

            ![9tunnel2](https://user-images.githubusercontent.com/3851540/165066244-46d51c12-89f7-44e2-b6a6-4a9ddfd08d9b.png)

            ![10tunnel2](https://user-images.githubusercontent.com/3851540/165066260-52ae9ef2-561a-40b3-a556-5b4d1371238f.png)

4. 設定 VPN Tunnel 的 BGP session

    > 每個 VPN Tunnel 有各自的 BGP session，所以需要設定 `四個` BGP session

    ![11configbgpsession1](https://user-images.githubusercontent.com/3851540/165066267-56397324-9c13-4036-8280-ac7615f8148d.png)

    ![11configbgpsession2](https://user-images.githubusercontent.com/3851540/165066279-86746913-779b-4143-8064-dc628d45e8fb.png)

    - BGP session 需要成對設定
    - `Peer ASN` 填目標 Cloud Router 的值

        ![12asn](https://user-images.githubusercontent.com/3851540/165066293-e2f55546-58b4-40b7-862b-2a982e2bc24e.png)

        ![13peerasn](https://user-images.githubusercontent.com/3851540/165066311-9bf22a8e-5cc8-4fab-8eed-93b687bdce93.png)

    - `Cloud Router BGP IP`：這個值與目標 VPN Tunnel 的 BGP session 的 `BGP peer IP` 相同

        > - 需要使用 link-local ip (我不知道是什麼)
        > - 兩個 tunnel 的 ip 區段要切開

    - `BGP peer IP`：這個值與目標 VPN Tunnel 的 BGP session 的 `Cloud Router BGP IP` 相同

    > 以下透過不同顏色的框線來呈現成對設定的部份

    - tunnel1 bgp

        ![14bgpip1](https://user-images.githubusercontent.com/3851540/165066321-f18eb3bc-f27d-4838-a1d2-56cee4c49382.png)

        ![15bgpip1](https://user-images.githubusercontent.com/3851540/165066333-59c4a823-ee2f-4665-8ac2-4dc21ecb23cc.png)

    - tunnel2 bgp

        ![14bgpip2](https://user-images.githubusercontent.com/3851540/165066329-2ae3b3e7-6ee4-4c77-ad26-c0390cff9d28.png)

        ![15bgpip2](https://user-images.githubusercontent.com/3851540/165066340-5006ee3e-d766-40b0-93b5-c88a65491209.png)

5. 實際效果

    ![success](https://user-images.githubusercontent.com/3851540/165066348-abb4ed8f-a70d-4f19-8a59-677213fe437c.png)

## 心得

看著 [GCP:Mounting file shares on remote clients](https://cloud.google.com/filestore/docs/remote-mounting) 實在不知道需要設定哪些東西，順著相關連結深入，又覺得很容易迷失，我自己的感覺是相關連結應該是為了模組化說明文件而建立的，所以不一定全然符合 Cloud Filestore 的設定需求，所以是參考 [Youtube:GCP | Google Cloud VPN | GCP to GCP HA VPN using BGP dynamic routing | Google Cloud HA VPN DEMO](https://www.youtube.com/watch?v=Qyu3IjwtOnk) 影片才將 Cloud HA VPN 設定完成，但還不夠，最後找到 [GCP:Cloud Filestore Known issues - Mounting over VPN](https://cloud.google.com/filestore/docs/known-issues#mounting_over_vpn) 才設定成功，可能是這個需要沒有很普遍，或是我關鍵字下得不對，感覺資料不是很好找

另外雖然設定成功，但過程中用到很多東西 (e.g. `BGP session`,`Advertise all subnets visible to the Cloud Router`....) 還是不清楚用途跟實際定義，感覺用起來很虛

先紀錄一下，待後續有機會再深入研究 (據之前經驗，現在花時間研究，經常因為功能迭代速度過快而失去作用，與其浪費時間，倒不如等遇到問題再來看)

## 參考資訊

1. [GCP:Mounting file shares on Compute Engine clients](https://cloud.google.com/filestore/docs/mounting-fileshares)
2. [GCP:Mounting file shares on remote clients](https://cloud.google.com/filestore/docs/remote-mounting)
3. [Youtube:GCP | Google Cloud VPN | GCP to GCP HA VPN using BGP dynamic routing | Google Cloud HA VPN DEMO](https://www.youtube.com/watch?v=Qyu3IjwtOnk)
4. [GCP:Cloud VPN Pricing](https://cloud.google.com/network-connectivity/docs/vpn/pricing)
5. [GCP:Create an HA VPN gateway between Google Cloud networks](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn2)
6. [GCP:Cloud Filestore Known issues - Mounting over VPN](https://cloud.google.com/filestore/docs/known-issues#mounting_over_vpn)
