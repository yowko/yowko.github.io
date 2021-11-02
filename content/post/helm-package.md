---
title: "打包 Helm Chart"
date: 2020-01-27T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Helm","Kubernetes"]
slug: "helm-package"
---

## 打包 Helm Chart

最近專案常需要調整或建立 Helm Chart，但畢竟還不是很熟悉，常常用完一次過沒幾天就又忘記語法，同事都快被我煩死了，趁著假日簡單紀錄一下使用方式

## 基本環境說明

1. macOS Mojave 10.15.2
2. Helm v2.16.1
3. 建立 Chart

    ```bash
    helm create yowkochart
    ```

## 打包方式

* 使用 `helm` cli

    * 語法

        ```bash
        helm package {chart path}
        ```

    * 範例

        ```bash
        helm package yowkochart
        ```

        > 執行後會產生 `yowkochart-0.1.0.tgz` 檔案

        ![1helm-package](https://user-images.githubusercontent.com/3851540/73179549-99189100-414e-11ea-8cf9-5d6bce4c59f5.png)

* 版本控制
  
    1. 預設版本號碼 (以上述為例：`0.1.0`) 由 `Chart.yaml` 的 `version` 控制

        > 每次修改記得調整 `Chart.yaml` 的 `version` 值

    2. 自行控制產出的版本(加上 `--version` 參數)

        * 語法

            ```bash
            helm package {chart path} --version={version}
            ```

        * 範例

            ```bash
            helm package yowkochart --version=1.0.1
            ```

            ![2helm-version](https://user-images.githubusercontent.com/3851540/73179551-99189100-414e-11ea-88d6-7977c4de4d2a.png)

## 心得

紀錄後發現難道我之前都不想寫筆記：實在太沒內容了，不過既然我已經卡關好幾次，我想簡單紀錄一下  加深印象也不錯，也許過陣子我又會被自己拯救了 哈哈

## 參考資訊

1. [How To Create Your First Helm Chart](https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/)
2. [`helm package --version` would be useful for continuous deployment scenarios](https://github.com/helm/helm/issues/1699)
