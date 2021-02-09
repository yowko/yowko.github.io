---
title: "使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證"
date: 2021-02-09T10:30:00+08:00
lastmod: 2021-02-09T10:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "cert-manager-pkcs12-pfx"
---

## 使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證

之前筆記 [使用 cert-manager 建立憑證](/cert-manager-certificate) 紀錄到如何使用 cert-manager 來建立憑證，在預設情境下只會產生 .crt + .key 的憑證與 key，但 ASP.NET Core 使用的憑證格式為 `PKCS12` (`.PFX`) ，當然也可以透過 openssl 來將 .crt+.key 轉換為 .pfx ，但這樣的轉換不利於自動化部署，所以還是希望透過 cert-manager 自行產生 .pfx，就來看看該如何設定 cert-manager 來產生 `PKCS12` (`.PFX`) 憑證吧

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

    - docker engine 20.10.2
    - Kubernetes v1.19.3

3. Helm chart

    - cert-manager v1.1.0

## 設定方式

1. 建立自簽憑證的 Issuer

    ```bash
    kubectl apply -f <(echo "
    apiVersion: cert-manager.io/v1
    kind: Issuer
    metadata:
      name: selfsigned-issuer
    spec:
      selfSigned: {}
    ")
    ```

    ![1issuer](https://user-images.githubusercontent.com/3851540/107238248-968b8f80-6a62-11eb-8845-b9ce2355deed.png)

2. 建立 PKCS12 用的密碼

    > 這個 secret 也可以不用建立，到時建立 PKCS12 時用明碼

    ```bash
    kubectl create secret generic pfxpwd --from-literal password=pass.123
    ```

3. 建立 PKCS12 格式 (.pfx) 憑證

    ```bash
    kubectl apply -f <(echo '
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: yowko-pkcs12
    spec:
      secretName: yowko-pkcs12
      keystores:
       pkcs12:
         create: true
         passwordSecretRef:
           name: pfxpwd
           key: password
      dnsNames:
      - "*.default.svc.cluster.local"
      - "*.testyowko.com"
      issuerRef:
        name: selfsigned-issuer
        kind: Issuer
    ')
    ```

    ![1p12](https://user-images.githubusercontent.com/3851540/107316711-333a4580-6ad4-11eb-9fb5-22ee990dab6d.png)

4. 驗證 p12 有效性

    ```bash
    kubectl get secrets/yowko-pkcs12 -o "jsonpath={.data['keystore\.p12']}" | base64 -D |openssl pkcs12 -info
    ```

    ![2verify](https://user-images.githubusercontent.com/3851540/107316719-36cdcc80-6ad4-11eb-8d58-a1d20afbd519.png)

## 心得

之前還沒確定做法前，我查了很多文件跟 GitHub issue，但總是無法順利產生 `PKCS12` (`.PFX`) 憑證，還一直覺得為什麼官網上沒有完整一點的範例，只在 relase note 上放個簡單的 yaml，但後來確定做法後經過反覆檢查  發現是我在 `passwordSecretRef` 這把 key 跟 name 寫反了XD

這邊提供大家基本確認的方式：`TYPE` 必需要是 `kubernetes.io/tls`，如果產生出來的是 `Opaque` 可能會有憑證未成功建立的狀況，後續綁定也可能有問題

![3type](https://user-images.githubusercontent.com/3851540/107316722-37666300-6ad4-11eb-97ba-385ea15c51d6.png)

## 參考資訊

1. [使用 cert-manager 建立憑證](/cert-manager-certificate)
2. [在 ASP.NET Core 中設定憑證驗證](https://docs.microsoft.com/zh-tw/aspnet/core/security/authentication/certauth?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
3. [General Availability of JKS and PKCS#12 keystores](https://cert-manager.io/docs/release-notes/release-notes-0.15/#general-availability-of-jks-and-pkcs-12-keystores)
