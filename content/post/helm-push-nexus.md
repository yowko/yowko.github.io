---
title: "將打包好的 Helm Chart Push 至 Nexus"
date: 2020-01-28T21:30:00+08:00
lastmod: 2020-01-28T21:30:31+08:00
draft: false
tags: ["Helm","Kubernetes","Tools","Nexus"]
slug: "helm-push-nexus"
---

## 將打包好的 Helm Chart Push 至 Nexus

之前筆記 [打包 Helm Chart](https://blog.yowko.com/helm-package/) 簡單紀錄該如何打包 Helm Chart，但打包後的 Helm Chart 如果無法提供其他 server 使用就失去意義了，所以延伸紀錄將打包後的 Helm Chart 推送至 Nexus 的作法

## 基本環境說明

1. macOS Mojave 10.15.2
2. Helm v2.16.1
3. Nexus 記得使用支援 `Helm` 的版本

    > 我不想多花時間從無到有建置 nexus-repository-helm 所以直接使用 docekr hub 包好的 image

    ```bash
    docker run -d -p 8081:8081 --name nexus nngplatform/nexus-repository-helm
    ```

## Push 方式

1. 使用 `curl`

    - 語法

        ```bash
        curl -v -u {帳號}:{密碼} --upload-file {helm chart file} {nexus helm url}
        ```

    - 範例

        ```bash
        curl -v -u admin:pass.123 --upload-file yowkochart-1.0.1.tgz http://localhost:8081/repository/helm/
        ```

    ![1curl](https://user-images.githubusercontent.com/3851540/73272257-0dba0100-421d-11ea-98cf-4d5d76a41dbd.png)

    ![2updated](https://user-images.githubusercontent.com/3851540/73272258-0dba0100-421d-11ea-8f0c-91d9c70e1f42.png)

2. 使用 plugin - `helm-nexus-push`

    - 安裝 plugin

        ```bash
        helm plugin install --version master https://github.com/sonatype-nexus-community/helm-nexus-push.git
        ```

    - 實際 push

        - 先 login

            - 語法

                ```bash
                helm nexus-push {helm repo 名稱} login  -u {帳號} -p {密碼}
                helm nexus-push {helm repo 名稱} {helm chart}
                ```

            - 範例

                ```bash
                helm nexus-push test login  -u admin -p pass.123
                helm nexus-push test yowkochart-2.0.0.tgz
                ```

            ![3loginpush](https://user-images.githubusercontent.com/3851540/73272259-0dba0100-421d-11ea-897e-42b37eac6e1b.png)

        - 直接 push

            - 語法

                ```bash
                helm nexus-push {helm repo 名稱} {helm chart} -u {帳號} -p
                ```

                > 執行後會要求輸入密碼

            - 範例

                ```bash
                helm nexus-push test yowkochart-2.0.0.tgz -u admin -p
                ```

            ![4push](https://user-images.githubusercontent.com/3851540/73272261-0e529780-421d-11ea-9918-d14b2de483bc.png)

            ![5pushed](https://user-images.githubusercontent.com/3851540/73272262-0e529780-421d-11ea-82ec-d589ea967f4b.png)

## 心得

`helm-nexus-push` 也支援直接 push chart 資料夾，而實際執行時會在真正 push 前直接做一次 helm package，需要留意的是這個動作沒有提供設定版本的參數，所以會依 helm chart 的慣例使用 `Chart.yaml` 的 `version` 值作為 chart version

另外是之前我常卡在 helm nexus-push 時會直接使用 `-p {密碼}` 讓執行時的 username 變為 `''` 造成 `401 Unauthorized` 錯誤，這個我看了原始碼後覺得應該是防呆機制，所以要嘛先 login，要嘛就是 push 只帶 `-u`，再輸入密碼

最後我一開始覺得奇怪的點是：用原生 helm cli 無法成功 push ，只能透過 `curl` 與 `helm-nexus-push`(背後也是使用 `curl`)，後來猜測應該是 nexus 沒有實作 helm repository api 造成的

## 參考資訊

1. [sonatype-nexus-community/nexus-repository-helm](https://github.com/sonatype-nexus-community/nexus-repository-helm)
2. [How can I programmatically upload files into Nexus 3?](https://support.sonatype.com/hc/en-us/articles/115006744008-How-can-I-programmatically-upload-files-into-Nexus-3-)
3. [Helm Nexus Repository Push](https://github.com/sonatype-nexus-community/helm-nexus-push)
