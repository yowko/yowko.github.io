---
title: "使用 cert-manager 建立憑證"
date: 2021-02-08T21:30:00+08:00
lastmod: 2021-02-08T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "cert-manager-certificate"
---

## 使用 cert-manager 建立憑證

之前筆記 [在 Kubernetes 上安裝 cert-manager](/cert-manager) 提到為什麼要使用 cert-manager 以及如何安裝 cert-manager，今天就來紀錄一下如何使用 cert-manager 來建立憑證

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

    - docker engine 20.10.2
    - Kubernetes v1.19.3

3. Helm chart

    - cert-manager v1.1.0

## 憑證建立方式

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

2. 使用自簽憑證 Issuer 來建立憑證

    > 要注意 Issue 的 Ready 變成 true 再執行建立憑證，否則憑證 Type 會變成 `Opaque` 而不是 `kubernetes.io/tls`

        ```bash
        kubectl apply -f <(echo '
        apiVersion: cert-manager.io/v1alpha2
        kind: Certificate
        metadata:
        name: yowko-tls
        spec:
        secretName: yowko-tls
        dnsNames:
        - "*.default.svc.cluster.local"
        - "*.testyowko.com"
        issuerRef:
            name: selfsigned-issuer
        ')
        ```

    ![2secret](https://user-images.githubusercontent.com/3851540/107238250-98555300-6a62-11eb-90a6-5fc912539f2d.png)

3. 檢查憑證有效性

    - 檢視憑證

        > 確認 `.crt` 與 `.key` 都正確產生

        ```bash
        kubectl get secret yowko-tls -o yaml
        ```

        ![3describe](https://user-images.githubusercontent.com/3851540/107238256-99868000-6a62-11eb-836d-708790dfff74.png)

    - 確認 `.crt` 有效性

        ```bash
        kubectl get secrets/yowko-tls -o "jsonpath={.data['tls\.crt']}" | base64 -D | openssl x509 -text -noout
        ```

        ![4checkcrt](https://user-images.githubusercontent.com/3851540/107238264-9b504380-6a62-11eb-87af-02373e837afb.png)

    - 確認 `.key` 有效性

        ```bash
        kubectl get secrets/yowko-tls -o "jsonpath={.data['tls\.key']}" | base64 -D | openssl rsa -check
        ```

        ![5checkkey](https://user-images.githubusercontent.com/3851540/107238265-9be8da00-6a62-11eb-9d05-0860a413c7be.png)

## 心得

不知道是不是因為 Self Signed 不是正統用法，官網上並沒有完整的使用方式，或者是因為太簡單，所以人家覺得不用寫XD

但除此之外，就我個人使用情境而言覺得官網的文件不是很好用，功能好像都有，文件就提那一、兩句，實際用法都要自己兜，覺得不是很方便，但或許這就是 open source 的常態吧

## 參考資訊

1. [SelfSigned](https://cert-manager.io/docs/configuration/selfsigned/)
2. [Creating Self Signed Certificates on Kubernetes](https://tech.paulcz.net/blog/creating-self-signed-certs-on-kubernetes/)
3. [在 Kubernetes 上安裝 cert-manager](/cert-manager)
4. [The Most Common OpenSSL Commands](https://www.sslshopper.com/article-most-common-openssl-commands.html)
5. [How To Check SSL Certificate Expiration with OpenSSL](https://computingforgeeks.com/how-to-check-ssl-certificate-expiration-with-openssl/)
6. [JSONpath fails to return keys containing dots in a map](https://github.com/kubernetes/kubernetes/issues/23386#issuecomment-305348170)
