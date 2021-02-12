---
title: "Kubernetes 發行憑證給 ASP.NET Core 使用"
date: 2021-02-12T09:30:00+08:00
lastmod: 2021-02-12T09:30:31+08:00
draft: false
tags: ["Kubernetes","ASP.NET Core"]
slug: "kubernetes-certificate-aspdotnet-core"
---

## Kubernetes 發行憑證給 ASP.NET Core 使用

之前筆記 [使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證](/cert-manager-pkcs12-pfx) 紀錄到如何使用 cert-manager 來建立 ASP.NET Core 可以使用的 PKCS12 格式 (.pfx) 憑證，也在 [將憑證 secret 以檔案掛戴至 Container 中](/certificate-secret-mount-volume-file) 紀錄如何將 Kubernetes Secret mount 進 container 的檔案中，今天就來紀錄一對如何搭配使用前二個技巧來設定 ASP.NET Core

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0(51484)

    - docker engine 20.10.2
    - Kubernetes v1.19.3

3. Helm chart

    - cert-manager v1.1.0

4. docker images

    - yowko/healthcheck:healthy

        > 用來模擬 gRPC server，詳細內容可以參考 [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](/kubernetes-liveness-aspdotnet-core-grpc/)

    - redis:latest

        > 只用來模擬 application，隨便用任意 container 皆可

5. 存有憑證資訊的 secret

    > 這邊使用之前筆記 [使用 cert-manager 建立憑證](/cert-manager-certificate) 產生的 secret 做示範，或是可以參考 [Create a secret containing some ssh keys](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-pod-with-ssh-keys) 來建立

## 設定方式

1. Deployment 與 Service yaml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: testcert-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          service: testcert
      template:
        metadata:
          labels:
            service: testcert
        spec:
          containers:
          - name: yowko-testservice
            image: yowko/healthcheck:healthy
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 5001
              protocol: TCP
            env:
            - name: ASPNETCORE_URLS
              value: https://+:5001
            - name: ASPNETCORE_Kestrel__Certificates__Default__Path
              value: /https/keystore.p12
            - name: ASPNETCORE_Kestrel__Certificates__Default__Password
              valueFrom:
                secretKeyRef:
                  name: pfxpwd
                  key: password
            volumeMounts:
            - name: certificate-volume
              mountPath: /https
              readOnly: true
          volumes:
          - name: certificate-volume
            secret:
              secretName: yowko-pkcs12
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: testcert
    spec:
      ports:
        - port: 5001
          targetPort: 5001
          protocol: TCP
      selector:
        service: testcert
    ```

    - 加上 application client 測試用

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: puredis-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              service: puredis
          template:
            metadata:
              labels:
                service: puredis
            spec:
              containers:
              - name: pure-redis
                image: redis
                imagePullPolicy: IfNotPresent
                volumeMounts:
                - name: certificate-volume
                  mountPath: /https
                  readOnly: true
        
              volumes:
              - name: certificate-volume
                secret:
                  secretName: yowko-pkcs12
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: puredis
        spec:
          ports:
            - port: 6379
              targetPort: 6379
              protocol: TCP
          selector:
            service: puredis
        ```

2. 驗證 gRPC 憑證有效性

    > grpcurl 相關使用詳細內容可以參考之前筆記 [使用 grpcurl 呼叫 gRPC Service](/grpcurl/)

    - 更新 apt 並安裝 wget

        ```bash
         apt update && apt install -y wget
        ```

    - 下載 grpcurl

        ```bash
        wget https://github.com/fullstorydev/grpcurl/releases/download/v1.8.0/grpcurl_1.8.0_linux_x86_64.tar.gz
        ```

    - 解壓 grpcurl

        ```bash
        tar -zxvf grpcurl_1.8.0_linux_x86_64.tar.gz
        ```

    - 準備 proto

        ```proto
        syntax = "proto3";
        option csharp_namespace = "GrpcMessages";
        package greet;
        // The greeting service definition.
        service Greeter {
          // Sends a greeting
          rpc SayHello (HelloRequest) returns (HelloReply);
        }
        // The request message containing the user's name.
        message HelloRequest {
          string name = 1;
        }
        // The response message containing the greetings.
        message HelloReply {
          string message = 1;
        }
        ```

    - 使用 grpcurl 呼叫 gRPC service

        > 使用 ca 憑證呼叫 grpc service

        ```bash
        ./grpcurl  -cacert /https/tls.crt -d '{"name":"Yowko"}' -proto greet.proto testcert.default.svc.cluster.local:5001 greet.Greeter/SayHello
        ```

        ![1grpcurlok](https://user-images.githubusercontent.com/3851540/107730843-6f1d1700-6d2f-11eb-9f94-9ffaa2047da1.png)

    - 未提供 ca 憑證呼叫 grpc 錯誤

        - 錯誤訊息

            ```txt
            Failed to dial target host "testcert.default.svc.cluster.local:5001": x509: certificate signed by unknown authority
            ```

        - 錯誤截圖

            ![2error](https://user-images.githubusercontent.com/3851540/107730848-717f7100-6d2f-11eb-86b1-2f13cbc949bc.png)

## 心得

我不知道整個流程是不是正統做法，但至少先紀錄下來  有機會再參考其他人做法調整了

不過我原本以為過透過 Kubernetes 來發行憑證，在 Kubernetes 的服務中在互相呼叫時都可以避開憑證驗證的問題，但目前看來是跟我想的不同，在 client 端還需要手動指定憑證來驗證 ca 有效性才能正確呼叫 gRPC service

## 參考資訊

1. [使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證](/cert-manager-pkcs12-pfx)
2. [將憑證 secret 以檔案掛戴至 Container 中](/certificate-secret-mount-volume-file)
3. [使用 Kubernetes Liveness 來檢查 ASP.NET Core gRPC 回應合乎預期](/kubernetes-liveness-aspdotnet-core-grpc/)
4. [使用 cert-manager 建立憑證](/cert-manager-certificate)
5. [Create a secret containing some ssh keys](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-pod-with-ssh-keys)
6. [在 ASP.NET Core 中設定憑證驗證](https://docs.microsoft.com/zh-tw/aspnet/core/security/authentication/certauth?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
7. [使用 grpcurl 呼叫 gRPC Service](/grpcurl/)
