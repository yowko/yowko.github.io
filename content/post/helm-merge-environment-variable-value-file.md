---
title: "Helm 如何讓多個 value file 的 environment 變數合併"
date: 2023-11-13T00:30:00+08:00
lastmod: 2023-11-13T00:30:31+08:00
draft: false
tags: ["helm","kubernetes"]
slug: "helm-merge-environment-variable-value-file"
---

## Helm 如何讓多個 value file 的 environment 變數合併

這是最近在調整 Helm 的時候遇到的問題：團隊為了增加安全性，將原本寫在 config 中的設定值移至 environment variable 中 (也就是從開發人員都可以看到的改為只有可以 access 環境的人能取得，雖說是為了增加安全性，但也沒多安全，建議直上 Vault)，因為環境變數有很多加上有些環境變數是所有 application 使用相同設定值，造成每次需要修改時都要異動幾十甚至上百個檔案又容易改錯，所以想要將共用環境變數設定值拆分至一個 `share-values.yaml` 搭配每個 application 本身的 `application-values.yaml`，但是原本 Helm 的寫法在使用多個 values files 只會套用最後一個 value file 的環境變數，今天就來紀錄一下可以怎麼調整。

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(16297)
3. Helm v3.13.1
4. 使用 `Helm create helmvaluetest` 建立一個測試用的 Helm chart

    - 修改 `helmvaluetest/templates/deployment.yaml` 允許設定環境變數

        ```yaml
        env: 
        {{- range $env :=.Values.envs }}
          {{- range $name,$value :=$env }}
          - name: {{$name}}
            value: {{$value | quote }}
          {{- end }}
        {{- end }}
        ```

    - 修改 `helmvaluetest/values.yaml` 加入預設環境變數

        ```yaml
        envs:
        - ENV: "default"
        ```

## 情境說明

1. share-values.yaml

    ```yaml
    envs:
    - REDISENDPOINT: "127.0.0.1:6379,password=pass.123"
    ```

2. application-values.yaml

    ```yaml
    envs:
    - ENV: "dev"
    - DEV: "true"
    ```

3. 期望結果

    ```yaml
    env:
    - name: ENV
      value: "dev"
    - name: DEV
      value: "true"
    - name: REDISENDPOINT
      value: "127.0.0.1:6379,password=pass.123"
    ```

4. 實際結果

    - share-values.yaml 在前

        > `helm template --debug helmtestvalue ./helmvaluetest/ -f share-values.yaml -f application-values.yaml`

        ```yaml
        env:
          - name: ENV
            value: "dev"
          - name: DEV
            value: "true"
        ```

        ![1sharevalues](https://github.com/yowko/picsbed/assets/3851540/06d23f89-5f9a-4f84-acb1-7d7cc02b8758)

    - applicarion-values.yaml 在前

        > `helm template --debug helmtestvalue ./helmvaluetest/ -f application-values.yaml -f share-values.yaml`

        ```yaml
        env:
          - name: REDISENDPOINT
            value: "127.0.0.1:6379,password=pass.123"
        ```

        ![2applicationvalues](https://github.com/yowko/picsbed/assets/3851540/b1c76b1d-aba6-46df-90ef-f7e8693cdd5f)

## 設定方式

在 Helm 的 GitHub issue 中有人提出了相同的問題 [Helm should preform deep merge on multiple values files](https://github.com/helm/helm/issues/3486)，討論的 thread [Helm should preform deep merge on multiple values files](https://github.com/helm/helm/issues/3486#issuecomment-581619753) 中有提到可以做的的方式：將環境變數改以 `dict` 方式而不是 `list`

1. 修改取用 environment variables 的 `helmvaluetest/templates/deployment.yaml`

    ```yaml
    env: 
      {{- range $key, $value := .Values.envs }}
        - name: {{ $key }}
          value: {{ $value | quote }}
      {{- end }}
    ```

2. 修改 values 的設定方式

    - share-values.yaml

        ```yaml
        envs:
          REDISENDPOINT: "127.0.0.1:6379,password=pass.123"
        ```

    - application-values.yaml

        ```yaml
        envs:
          ENV: "dev"
          DEV: "true"
        ```

3. 實際效果

    share-values.yaml 在前 (`helm template --debug helmtestvalue ./helmvaluetest/ -f share-values.yaml -f application-values.yaml`) 與 applicarion-values.yaml 在前 (`helm template --debug helmtestvalue ./helmvaluetest/ -f application-values.yaml -f share-values.yaml`) 結果相同

    ```yaml
    env:
      - name: DEV
        value: "true"
      - name: ENV
        value: "dev"
      - name: REDISENDPOINT
        value: "127.0.0.1:6379,password=pass.123"
    ```

    ![3envdict](https://github.com/yowko/picsbed/assets/3851540/8821879e-3359-4c48-a08a-57c0cca5230b)

## 心得

之前查資料時，有看到網友推薦使用 `[yq](https://mikefarah.gitbook.io/yq/)` yaml 的 jq，像是之前筆記 [jq 合併多個 json](https://blog.yowko.com/jq-merge-multiple-json/)，我快速看了一下用法，發現跟 jq 使用上還是存在不小的差異，要達成期望的效果，不是直接使用 jq 的寫法就可以達成的，所以就沒有繼續深入研究，後續改用 `dict` 的方式來處理，感覺更乾淨些，完全不需要依賴外部工具。

完成程式碼：[yowko/helm-merge-environment-variable-value-file](https://github.com/yowko/helm-merge-environment-variable-value-file)

- 原始版本可以參考 [commit](https://github.com/yowko/helm-merge-environment-variable-value-file/commit/573a05919989f72a2b10cab4dbcf4198a6c5ec79)
- 修改後版本可以參考 [commit](https://github.com/yowko/helm-merge-environment-variable-value-file/commit/a8e537de26fcc5d97c8d2702902e5af80d893416)

## 參考資訊

1. [Helm should preform deep merge on multiple values files](https://github.com/helm/helm/issues/3486#issuecomment-581619753)
2. [yq](https://mikefarah.gitbook.io/yq/)
3. [jq 合併多個 json](https://blog.yowko.com/jq-merge-multiple-json/)
4. [yowko/helm-merge-environment-variable-value-file](https://github.com/yowko/helm-merge-environment-variable-value-file)
