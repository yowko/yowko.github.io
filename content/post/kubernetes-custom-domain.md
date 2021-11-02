---
title: "在 Kubernetes 中使用自訂 Domain"
date: 2020-06-27T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "kubernetes-custom-domain"
---

## 在 Kubernetes 中使用自訂 Domain

在之前筆記 [在 Windows 環境將特定網址指向不同 IP](/windows-host-file/) 與 [讓 iOS 裝置可以存取自訂 domain](/ios-private-domain/) 中都有提到過自訂域名的用法，大意就是避免因為 server ip 更動就要調整程式，所以連線時都使用自訂域名，但這些自訂域名不一定有向域名組織申請，所以可以透過修改 hosts file 或是使用內部 dns 來進行解析

這樣的需求當然不可能只存在於實體 server 中，在 Kebernetes 的環境中也會遇到，今天就來簡單紀錄一下使用方式

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)

    - Kubernetes v1.16.5

3. images

   - praqma/network-multitool

4. pod.yaml

      ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: hostaliases-pod
      spec:
        restartPolicy: Never
        containers:
        - name: network-tools
          image: praqma/network-multitool
      ```

## 設定方式

在 spec 下加入 `hostAliases` 並指定目標 ip 與對應的 custom domain

```yaml
hostAliases:
- ip: "192.168.50.97"
  hostnames:
  - "yowkotest.com"
  - "testyowko.com"
```

- 完整內容

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hostaliases-pod
    spec:
      restartPolicy: Never
      hostAliases:
      - ip: "192.168.50.97"
        hostnames:
        - "yowkotest.com"
        - "testyowko.com"
      containers:
      - name: network-tools
        image: praqma/network-multitool
    ```

## 實際效果

- 修改前：無法解析自訂 domain

    ![1notfound](https://user-images.githubusercontent.com/3851540/85924443-ebf4a100-b8c4-11ea-810a-76c12e56fbc1.jpg)

- 修改後：正確解析自訂 domain

    ![2ok](https://user-images.githubusercontent.com/3851540/85924444-ed25ce00-b8c4-11ea-9299-6c0d0f79e4d7.jpg)

## 心得

`hostAliases` 會將 ip 與 hostname 的設定直接加至 container 的 `/etc/hosts` 中，行為跟實體 server 上修改 `/etc/hosts` 一樣

雖然是個簡單的設定，但上次我打錯位置 (Helm 的 deployment 有兩層 `spec`)，造成整個 deploy fail，快速紀錄一下提醒自己下次要多看幾眼

## 參考資訊

1. [Adding entries to Pod /etc/hosts with HostAliases](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)
