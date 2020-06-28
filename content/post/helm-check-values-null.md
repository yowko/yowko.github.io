---
title: "在 Helm (Go Template) 中檢查 Value 是否有值"
date: 2020-06-28T21:30:00+08:00
lastmod: 2020-06-28T21:30:31+08:00
draft: false
tags: ["Kubernetes","Helm"]
slug: "helm-check-values-null"
---

## 在 Helm (Go Template) 中檢查 Value 是否有值

這個需求是延續之前筆記 [在 Kubernetes 中使用自訂 Domain](https://blog.yowko.com/kubernetes-custom-domain) 而來的，其中提到某些服務會掛在自訂 domain 下，但內網測試環境可能因為無法對外連線 (連不到外部 dns) 或是成本考慮，不一定會註冊 domain，但在外網環境時為了方便測試，註冊 domain 就是常見的

所以就出現這個需求：在內網環境部署時需要加上 hostalias 才能正確存取服務；但外網環境加上 hostalias 可能反而出現未即時更新 ip 而造成無法存取服務的狀況，所以今天就來快速紀錄一下在 Helm (Go Template) 中檢查 Value 是否有值，以 render 出不同內容

## 基本環境說明

1. macOS Catalina 10.15.5
2. docker desktop community 2.3.0.3(45519)

    - Kubernetes v1.16.5
3. Helm v3.2.1
4. images

   - praqma/network-multitool

5. 原始 Helm chart 結構與內容

    - checkvalue
        - charts
        - templates
            - deployment.yml

                ```yml
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                  name: checkvalue
                spec:
                  selector:
                    matchLabels:
                      name: checkvalue
                  template:
                    metadata:
                      labels:
                        name: checkvalue
                    spec:
                      containers:
                        - name: checkvalue
                          image: praqma/network-multitool
                          imagePullPolicy: Always
                ```

        - .helmignore

            ```yml
            # Patterns to ignore when building packages.
            # This supports shell glob matching, relative path matching, and
            # negation (prefixed with !). Only one pattern per line.
            .DS_Store
            # Common VCS dirs
            .git/
            .gitignore
            .bzr/
            .bzrignore
            .hg/
            .hgignore
            .svn/
            # Common backup files
            *.swp
            *.bak
            *.tmp
            *~
            # Various IDEs
            .project
            .idea/
            *.tmproj
            .vscode/
            ```

        - Chart.yml

            ```yml
            apiVersion: v1
            appVersion: "1.0"
            description: A Helm chart for Kubernetes
            name: checkvalue
            version: 0.1.0
            ```

        - values.yml (空檔案)

            > 用來模擬外網環境 (有註冊 domain)，也是預設值

            ```yml
            ```

        - withvalues.yml

            > 用來模擬內網環境 (沒有註冊 domain)

            ```yaml
              hostaliases:
                ip: "192.168.50.97"
                hostnames:
                - "yowkotest.com"
                - "testyowko.com"
            ```

## 修改方式

使用 `sprig` 語法中的 `kindIs` function 來進行檢查

> 下列兩者方式皆可行，請依實際情境調整

1. 檢查是否為 `null`

    > 我嘗試使用 `!` 會有問題

    - 檢查語法範例

          ```yaml
            {{- if kindIs "invalid" .Values.hostaliases }}
            {{else}}
            hostAliases:
            - ip: {{ .Values.application.hostaliases.ip | quote}}
              hostnames:
              {{- range $hostname := .Values.application.hostaliases.hostnames }}
              - {{$hostname | quote}}
              {{- end }}
            {{- end }}
          ```

    - 完整 yaml

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: checkvalue
        spec:
          selector:
            matchLabels:
              name: checkvalue
          template:
            metadata:
              labels:
                name: checkvalue
            spec:
              {{- if kindIs "invalid" .Values.hostaliases }}
              {{else}}
              hostAliases:
              - ip: {{ .Values.hostaliases.ip | quote}}
                hostnames:
                {{- range $hostname := .Values.hostaliases.hostnames }}
                - {{$hostname | quote}}
                {{- end }}
              {{- end }}
              containers:
                - name: checkvalue
                  image: praqma/network-multitool
                  imagePullPolicy: Always
        ```

    - 實際效果

        - 預設值 (values.yaml)

            ```bash
            helm template checkvalue ./checkvalue
            ```

            ![1invaliddefault](https://user-images.githubusercontent.com/3851540/85950364-e2d00680-b98e-11ea-83af-11889b1fb737.jpg)

        - 給值 (withvalues.yaml)

            ```bash
            helm template -f  checkvalue/withvalues.yaml checkvalue ./checkvalue
            ```

            ![1invaliddefault](https://user-images.githubusercontent.com/3851540/85950366-e499ca00-b98e-11ea-85bd-a1263aa950d8.jpg)

2. 檢查是否為 `特定格式`

    > 這邊以 `map` 為例

    - 檢查語法範例

          ```yaml
            {{- if kindIs "map" .Values.hostaliases}}
            hostAliases:
            - ip: {{ .Values.hostaliases.ip | quote}}
              hostnames:
              {{- range $hostname := .Values.hostaliases.hostnames }}
              - {{$hostname | quote}}
              {{- end }}
            {{- end }}
          ```

    - 完整 yaml

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: checkvalue
        spec:
          selector:
            matchLabels:
              name: checkvalue
          template:
            metadata:
              labels:
                name: checkvalue
            spec:
              {{- if kindIs "map" .Values.hostaliases}}
              hostAliases:
              - ip: {{ .Values.hostaliases.ip | quote}}
                hostnames:
                {{- range $hostname := .Values.hostaliases.hostnames }}
                - {{$hostname | quote}}
                {{- end }}
              {{- end }}
              containers:
                - name: checkvalue
                  image: praqma/network-multitool
                  imagePullPolicy: Always
        ```

    - 實際效果

        - 預設值 (values.yaml)

            ```bash
            helm template checkvalue ./checkvalue
            ```

            ![3mapdefault](https://user-images.githubusercontent.com/3851540/85950367-e5326080-b98e-11ea-92d0-ce9802793e7a.jpg)

        - 給值 (withvalues.yaml)

            ```bash
            helm template -f  checkvalue/withvalues.yaml checkvalue ./checkvalue
            ```

            ![4mapvalue](https://user-images.githubusercontent.com/3851540/85950369-e5caf700-b98e-11ea-9aa1-72f219ad0ca2.jpg)

## 心得

原則上兩種作法差不多，硬要說的話 `kindIs "invalid"` 因為沒辦法使用 `not` (`!`) 所以在沒有給值時會多一行空白；除此之外直接使用 `kindIs "map"` 在程式碼數量上比較少，所以一般情境下我會選用這個方式

另外如果一開始對 values 的格式沒把握，可以透過 `kindOf .Values.hostaliases` 先取得 values 格式後再來寫判斷式

## 參考資訊

1. [在 Kubernetes 中使用自訂 Domain](https://blog.yowko.com/kubernetes-custom-domain)
2. [Clarification: nil vs empty string](https://github.com/Masterminds/sprig/issues/53#issuecomment-483414063)
3. [Reflection Functions](http://masterminds.github.io/sprig/reflection.html)
