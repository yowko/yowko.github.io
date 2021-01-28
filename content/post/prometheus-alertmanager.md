---
title: "Prometheus 搭配 Alertmanager 發送警示"
date: 2021-01-27T21:30:00+08:00
lastmod: 2021-01-27T21:30:31+08:00
draft: false
tags: ["Prometheus","Monitoring"]
slug: "prometheus-alertmanager"
---

## Prometheus 搭配 Alertmanager 發送警示

之前筆記 [使用 ElastAlert 監控 Elasticsearch 發出通知](/elastalert-alert) 提到近期都在進行主動監控的相關設定與測試，之前紀錄了 Elasticsearch 上使用 application 所產生的 log 內容來進行條件偵測與通知，今天要來紀錄如何為 infrastructure 層中 server 及 service 的可用性加上主動通知

今天筆記分別測試了 `email` 與 `webhook` 功能

建議可以先了解架構部份，詳細內容請參考官網 [OVERVIEW](https://prometheus.io/docs/introduction/overview/)

![architecture](https://prometheus.io/assets/architecture.png)

> 圖片出處 [OVERVIEW](https://prometheus.io/docs/introduction/overview/)

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker desktop 3.1.0 (51484)
3. Fiddler Everywhere 1.5.0
4. Helm charts

    - prometheus-community/kube-prometheus-stack 13.2.1

5. docker images

    - quay.io/prometheus-operator/prometheus-config-reloader v0.45.0
    - quay.io/prometheus-operator/prometheus-operator v0.45.0
    - quay.io/prometheus/prometheus v2.24.0
    - quay.io/prometheus/alertmanager v0.21.0

## 設定方式

1. helm repo 設定

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update
    ```

2. 設定 Prometheus 的 alert rule

    > 這個就先略過，使用內建 rule 來觸發就好，以這篇筆記使用的 helm chart 有以下這些 [內建 rule](https://github.com/prometheus-community/helm-charts/blob/3591019764be8998d97cd7de26c2f40a7c31b647/charts/kube-prometheus-stack/values.yaml#L29)

3. 建立 AlertManager 的 alert 發送規則

    > 以下使用 `testalert.yaml` 做為範例

    ```yaml
    alertmanager:
      config:
        global:
          smtp_smarthost: 'smtp.mailtrap.io:2525'
          smtp_auth_username: 'username'
          smtp_auth_password: 'password'
          smtp_from: 'alertmanager@example.org'
        route:
          group_by: ['alertname', 'prometheus'] # 用來將 alert 分群
          receiver: 'webhook' # `webhook` 預設接收者
          routes:
          - receiver: 'null' # 符合 `alertname: Watchdog` 就由 `null` 處理
            match:
              alertname: Watchdog
          - receiver: 'email' # 符合 `severity: critical` 就由 `email` 處理
            match:
              severity: critical
        receivers:
        - name: 'null' # 不做事
        - name: 'email'
          email_configs: # 指定 email config 來處理
          - to: 'xxx@yowko.com'
        - name: 'webhook'
          webhook_configs: # 指定 webhook config 來處理
          - url: 'http://blog.yowko.com'
            http_config:
              proxy_url: 'http://192.168.80.3:8866' # proxy 是個人 debug 用途
    ```

4. 使用 AlertManager 的 alert 發送規則 進行部署

    ```bash
    helm install testalert prometheus-community/kube-prometheus-stack -f testalert.yaml
    ```

5. 實際效果

    - null

        > 不會收到預設 `alertname: Watchdog` 的通知

    - email

        > 因為 `severity: critical` match 會收到 email

        ![1email](https://user-images.githubusercontent.com/3851540/105979953-40683500-60cf-11eb-938c-3fcce1c7977d.png)

    - webhook

        > 因為未 match `severity: critical` 與 `alertname: Watchdog`

        ![2webhook](https://user-images.githubusercontent.com/3851540/105979961-43fbbc00-60cf-11eb-86b2-76a45bc0c053.png)

## 心得

測試及 debug 工具

1. 開啟 alertmanager ui ui port forward

    ```bash
    kubectl --namespace default port-forward alertmanager-testalert-kube-prometheus-alertmanager-0 9093
    ```

2. 開啟 prometheus-server ui port forward

    ```bash
    kubectl --namespace default port-forward prometheus-testalert-kube-prometheus-prometheus-0 9090
    ```

3. 調整 alertmanager log level

    > 修改 `testalert.yaml` , 搭配 `kubectl logs -f alertmanager-testalert-kube-prometheus-alertmanager-0 -c alertmanager` 看 log

    ```yaml
    alertmanager:
      alertmanagerSpec:
        logLevel: debug
    ```

    ![3debuglog](https://user-images.githubusercontent.com/3851540/105979965-452ce900-60cf-11eb-99fa-9d9644ef6340.png)

4. 手動發送 alert event

    ```bash
    curl -H "Content-Type: application/json" -d '[{"labels":{"alertname":"test-email","severity":"critical"}}]' localhost:9093/api/v1/alerts &&  curl -H "Content-Type: application/json" -d '[{"labels":{"alertname":"test-webhook"}}]' localhost:9093/api/v1/alerts
    ```

## 參考資訊

1. [OVERVIEW](https://prometheus.io/docs/introduction/overview/)
2. [CONFIGURATION](https://prometheus.io/docs/alerting/latest/configuration/)
3. [prometheus-community/helm-charts/charts/kube-prometheus-stack/](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
4. [監控要告警啊 AlertManager](https://fufu.gitbook.io/kk8s/task-memory/14.alertmanager-info)
5. [Manual alert for routes and receivers test](https://github.com/prometheus/alertmanager/issues/437#issuecomment-263413632)
