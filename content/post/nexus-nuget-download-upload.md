---
title: "如何在 Nexus Repository 的 NuGet server 下載與上傳套件"
date: 2024-10-14T00:30:00+08:00
lastmod: 2024-10-14T00:30:31+08:00
draft: false
tags: ["bash","nuget","dotnet"]
slug: "nexus-nuget-download-upload"
---

## 如何在 Nexus Repository 的 NuGet server 下載與上傳套件

最近團隊因為 Nexus Repository server 的用量太高，造成服務中斷，進而影響到 CI/CD 流程，團隊的開發進度也多少受到影響，所以決定啟用多個 Nexus Repository server 以分散 server 的 loading，所以我們需要將原本的套件上傳到新的 Nexus Repository server，這篇筆記將會紀錄如何在 Nexus Repository 的 NuGet server 下載與上傳套件。

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- OrbStack 1.7.5 (18165)
- dotnet sdk 8.0.401
- container images

     - sonatype/nexus3:3.73.0

- 使用 docker-compose 來建立 Nexus Repository

    - 8081 是 ui port

    {{<gist yowko 1632ea5cf9457d98612b1a5f4c840259 "docker-compose.yml">}}

## 下載與上傳套件

1. 下載

    {{<gist yowko 1632ea5cf9457d98612b1a5f4c840259 "nuget-download.sh">}}

2. 上傳

    - 使用 dotnet cli

        - 取得 NuGet API Key

            ![1apikey](https://github.com/user-attachments/assets/09b0de0d-9cfe-49e2-99bb-f699e941a65b)

        - 設定 Nexus Repository 啟用 NuGet API Key 驗證

            ![2nugetrealm](https://github.com/user-attachments/assets/0b214295-16bf-428d-9112-97e8cb14e6fe)

            - 未設定會出現 401 (Unauthorized) 錯誤

                ![3unauthorized](https://github.com/user-attachments/assets/1dfd2929-4f76-4aa0-8693-36ccffcb3f55)

        {{<gist yowko 1632ea5cf9457d98612b1a5f4c840259 "nuget-dotnet-upload.sh">}}

    - 使用 curl

        {{<gist yowko 1632ea5cf9457d98612b1a5f4c840259 "nuget-curl-upload.sh">}}

## 心得

NEXUS 官方文件不是很清楚，加上 NuGet server 使用 NEXUS 的網路分享也不多，所以花了不少時間才找到正確的方法，簡單紀錄一下用法希望這篇文章能幫助到有需要的人，不過我猜應該是幫助未來的自己的機會大些XD

NuGet CLI 也可以用來上傳 NuGet 套件，但是僅有 exe (windows 版本)， mac 使用需要額外安裝 mono，所以這邊就不紀錄了

## 參考資料

1. [A Guide to Using Nexus Repository for NuGet Package Hosting](https://medium.com/@tharakahalkewelatecs/a-guide-to-using-nexus-repository-for-nuget-package-hosting-e5f14b5ac05a)
2. [File Upload&Download with a Nexus Repository Rest Api](https://cigdemkadakoglu.medium.com/file-upload-download-with-a-nexus-repository-rest-api-f760441bc01c)
3. [upload artifacts to nexus 3.15 with curl](https://groups.google.com/a/glists.sonatype.com/g/nexus-users/c/ZrYFk_-XpSk?pli=1)
4. [Sonatype Nexus Postman collection](https://www.postman.com/njrusmc/public-collections/collection/sw194bz/sonatype-nexus?action=share&creator=22257526)
