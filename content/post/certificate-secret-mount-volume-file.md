---
title: "將憑證 secret 以檔案掛戴至 Container 中"
date: 2021-02-09T15:30:00+08:00
lastmod: 2021-02-09T15:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "certificate-secret-mount-volume-file"
---

## 將憑證 secret 以檔案掛戴至 Container 中

之前筆記 [使用 cert-manager 建立憑證](/cert-manager-certificate) 紀錄到如何使用 cert-manager 來建立憑證，只是透過 cert-manager 產生的憑證會以 Kubernetes secret 方式存在，一般 application 無法直接取用，所以今天就來紀錄一下如何將 Kubernetes secret 轉成 file 方式 mount 進 container 中讓 application 使用

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

    - docker engine 20.10.2
    - Kubernetes v1.19.3

3. Helm chart

    - cert-manager v1.1.0

4. docker images

    > 只用來模擬 application ，隨便用任意 container 皆可

    - redis:latest

5. 存有憑證資訊的 secret

    > 這邊使用之前筆記 [使用 cert-manager 建立憑證](/cert-manager-certificate) 產生的 secret 做示範，或是可以參考 [Create a secret containing some ssh keys](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-pod-with-ssh-keys) 來建立

## 設定方式

1. 將 secret `yowko-tls` mount 至 container 的 `/https/` 中

        ```bash
        kubectl apply -f <(echo "
        apiVersion: v1
        kind: Pod
        metadata:
          name: demopod
        spec:
          containers:
          - image: redis
            name: demopod
            volumeMounts:
            - name: certificate-volume
              mountPath: /https
              readOnly: true
            ports:
            - containerPort: 6379
              protocol: TCP
          volumes:
          - name: certificate-volume
            secret:
              secretName: yowko-tls
        ")
        ```

2. 說明

    - cerrt-manager 預設產生的 secret 中的 data 包含三個 key：`ca.crt`，`tls.crt`，`tls.key`

        ![1secretkey](https://user-images.githubusercontent.com/3851540/107331094-63daa900-6aed-11eb-9d9c-0b33e2f87b3f.png)

    - 直接將 secret mount 成資料夾，secret data 下的三組 key 會自行轉換為 file name，value 則會直接進到 file 中

        ```bash
        kubectl exec -it demopod -- ls /https
        ```

        ![2secretfile](https://user-images.githubusercontent.com/3851540/107331116-66d59980-6aed-11eb-9410-0245a6bc3d94.png)

3. 驗證憑證有效性

    - 進到該 container

        ```bash
        kubectl exec -it demopod -- bash
        ```

    - 更新 apt 並安裝 openssl

        ```bash
        apt update && apt install -y openssl
        ```

    - 驗證憑證

        - tls.crt

            ```bash
            openssl x509 -text -noout -in /https/tls.crt
            ```

            ![3checkcrt](https://user-images.githubusercontent.com/3851540/107331119-66d59980-6aed-11eb-9a56-e904cb03f47e.png)

        - tls.key

            ```bash
            openssl rsa -check --in /https/tls.key
            ```

            ![4checkkey](https://user-images.githubusercontent.com/3851540/107331122-6806c680-6aed-11eb-91b5-f4dd6c74c3c6.png)

## 心得

今天筆記對 Kubernetes 專家來說是廢話，但前幾天我真的一直在找方法嘗試將 secret 中的 key 跟 value `手動` 轉換為 file，後來查到 [Mount SSL certificates in the Pod with Kubernetes secret](https://medium.com/faun/mount-ssl-certificates-in-kubernetes-pod-with-secret-8aca220896e6) 才知道原來可以自動轉換呀XD

所以立馬筆記一下，加深自己印象，也許可以讓其他人少花點時間 (還是其實只有我不知道 @@")

## 參考資訊

1. [使用 cert-manager 建立憑證](/cert-manager-certificate)
2. [Create a secret containing some ssh keys](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-pod-with-ssh-keys)
3. [Mount SSL certificates in the Pod with Kubernetes secret](https://medium.com/faun/mount-ssl-certificates-in-kubernetes-pod-with-secret-8aca220896e6)
