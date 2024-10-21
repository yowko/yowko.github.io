---
title: "如何在 Nexus Repository 的 NPM server 下載與上傳套件"
date: 2024-10-15T00:30:00+08:00
lastmod: 2024-10-15T00:30:31+08:00
draft: false
tags: ["bash","npm"]
slug: "nexus-npm-download-upload"
---

## 如何在 Nexus Repository 的 NPM server 下載與上傳套件

之前筆記 [如何在 Nexus Repository 的 NuGet server 下載與上傳套件](/nexus-nuget-download-upload) 提到因為 Nexus Repository server 的用量太高，造成服務中斷，進而影響到 CI/CD 流程，團隊的開發進度也多少受到影響，所以決定啟用多個 Nexus Repository server 以分散 server 的 loading，之前筆記已經紀錄如何在 Nexus Repository 的 NuGet server 下載與上傳套件，今天就來紀錄 NPM 的部分。

## 基本環境說明

- macOS Sequoia 15.0.1 (Apple M2 Pro)
- OrbStack 1.7.5 (18165)
- NPM 10.8.2
- container images

     - sonatype/nexus3:3.73.0

- 使用 docker-compose 來建立 Nexus Repository

    - 8081 是 ui port

    {{<gist yowko 1632ea5cf9457d98612b1a5f4c840259 "docker-compose.yml">}}

- NEXUS NPM 設定

    ![1npmrepo](https://github.com/user-attachments/assets/0bf6b44a-5e97-4692-b300-823f8cb89de9)

## 下載與上傳套件

1. 下載套件

    {{<gist yowko 508cee99bff5d85ea94e1b3f8e78c8cf "npm-download.sh">}}

2. 上傳套件

    - 使用 npm

        {{<gist yowko 508cee99bff5d85ea94e1b3f8e78c8cf "npm-upload.sh">}}

    - 使用 curl

        {{<gist yowko 508cee99bff5d85ea94e1b3f8e78c8cf "npm-curl-upload.sh">}}

## 心得

相較於 NuGet，npm 在 NEXUS 官方文件上資源完整許多，網路上的資源也多了不少

完整程式碼請參考 [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)

## 參考資料

1. [如何在 Nexus Repository 的 NuGet server 下載與上傳套件](/nexus-nuget-download-upload)
2. [使用自建Nexus server作為NPM的Registry 倉庫](https://cftang0827.medium.com/%E4%BD%BF%E7%94%A8%E8%87%AA%E5%BB%BAnexus-server%E4%BD%9C%E7%82%BAnpm%E7%9A%84registry-%E5%80%89%E5%BA%AB-e48327d47309)
3. [npm Security](https://help.sonatype.com/en/npm-security.html)
4. [How to set npm credentials using `npm login` without reading from stdin?](https://stackoverflow.com/questions/23460980/how-to-set-npm-credentials-using-npm-login-without-reading-from-stdin)
5. [How to upload npm tgz to nexus3](https://stackoverflow.com/questions/54399393/how-to-upload-npm-tgz-to-nexus3)
6. [Publishing npm Packages](https://help.sonatype.com/en/publishing-npm-packages.html)
7. [GitHub:yowko/nexus-migrate](https://github.com/yowko/nexus-migrate)
