---
title: "在 Helm 中依不同變數使用不同設定值"
date: 2020-04-08T21:30:00+08:00
lastmod: 2020-04-08T21:30:31+08:00
draft: false
tags: ["Helm"]
slug: "helm-values-by-env"
---

## 在 Helm 中依不同變數使用不同設定值

Helm 使為一個 Kubernetes package 管理工具，在我的理解中它就是一個用來定義 Kubernetes 上的 application 如何進行部署、設定的 template，在不同環境間可以透過參數來決定其中部署與設定的差異，之前使用方式是利用 `helm install` 時加上 `--set` 來指定參數，但如此一來就會讓這些設定值散落在 Helm 的 `values.yaml` 與安裝的 script 中，不僅增加管理難度也讓 debug 變得複雜

為了解決這種狀況，我打算讓 Helm 可以依不同環境變數使用不同 `values.yaml` 內容，嘗試幾種做法後，找到一種不是很漂亮但可以暫時解決問題的方法，先紀錄一下，我再繼續嘗試其他做法

## 基本環境變數

1. macOS Catalina 10.15.4
2. docker desktop community 2.2.0.4(43472)

    - Engine 19.03.8
    - Kubernetes v1.15.5

3. Helm v2.16.1
4. 原始 Helm chart 結構與內容

    > 主要核心概念是新建一個 helm chart (`helm create testenv`)，再移除所有用不到的內容

    - testenv
      - charts
      - templates
        - test.yml

            ```yml
            apiVersion: v1
            kind: Service
            metadata:
                env: {{.Values.env}}
                targeturl: {{.Values.targeturl}}
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
            name: testenv
            version: 0.1.0
            ```

        - values.yml

            ```yml
            env: dev

            targeturl: blog.yowko.com
            ```

## 修改方式

1. 將不同環境的設定值加至 `values.yml`

    ```yml
    env: dev

    targeturl:
      dev: dev.yowko.com
      staging: staging.yowko.com
      uat: uat.yowko.com
      prod: prod.yowko.com
    ```

2. 修改 template 的取值方式

    > 透過 `.Values.env` 至 `.Values.targeturl` 這個 dictionary 中取值 (會取出陣列)，再取第一筆資料

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
        name: {{.Values.env}}
        targeturl: {{pluck .Values.env .Values.targeturl | first}}
    ```

3. 使用方式

    > 這邊透過 `--dry-run --debug` 來輸出 render 的結果

    - 修改前

        > helm install ./ --dry-run --debug

        ![1before](https://user-images.githubusercontent.com/3851540/78741543-2fa5d180-798c-11ea-8338-aa058b4aa0e7.png)

    - 修改後

        > helm install ./ --dry-run --debug --set env=staging

        ![2after](https://user-images.githubusercontent.com/3851540/78741546-316f9500-798c-11ea-985a-6f36564e206c.png)

4. 避免 `env` 不在定義清單中 (預先給定預設值)

    > helm install ./ --dry-run --debug --set env=staging12

    ```yml
    apiVersion: v1
    kind: Service
    metadata:
        name: {{.Values.env}}
        targeturl: {{pluck .Values.env .Values.targeturl | first | default .Values.targeturl.dev}}
    ```

    ![3exception](https://user-images.githubusercontent.com/3851540/78741547-32a0c200-798c-11ea-84b1-64033b7386bb.png)

## 心得

我認為可能有問題的點：

1. Helm 3 也許有更適合的做法

    > 目前團隊還沒使用，暫時不花時間研究

2. 所有的 Value 都放在一起，可能比較亂

    > 有時間會再嘗試拆分檔案，再依變數來 include 對應內容

3. 有其他專家認為應該使用 subchart

    > 這個是我覺得可能是最正統的做法，但也還沒機會嘗試

找資料過程中，發現有不少人會提出 `helm install -f {env}-values.yaml`，我不這麼用的原因是：這樣的做法需要將 `{env}-values.yaml` 複製至 Helm 的 client 端，這麼一來就失去透過透過 Helm 部署的好處 - 除了透過 Helm repository 下載 Helm chart 之外還要額外複製 values 檔，既不好管理，也讓整個部署變得複雜

## 參考資訊

1. [Charts with different set of values for each environment](https://github.com/helm/helm/issues/2620#issuecomment-314719592)
2. [Go语言中的模板引擎](https://blog.gmem.cc/gotpl)
