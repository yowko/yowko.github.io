---
title: "使用 Docker 建立 Nexus3 的 Image Registry"
date: 2020-01-07T21:30:00+08:00
lastmod: 2020-01-07T21:30:31+08:00
draft: false
tags: ["Container","Docker","macOS"]
slug: "nexus-docker-image-rergistry"
---

## 使用 Docker 建立 Nexus3 的 Image Registry

Nexus 全名是 `nexus repository oss` 是套免費 (有付費的 `pro` 版，差異請參考 [COMPARE to PRO VERSION](https://sonatype.drift.click/oss-vs-pro)) 產出物倉庫，支援多種類型產出物(摘錄至 [Manage these formats](https://www.sonatype.com/download-oss-sonatype))

- Bower
- Docker
- Git LFS
- Maven
- npm
- NuGet
- PyPI
- Ruby Gems
- Yum
- APT
- Conan
- R
- CPAN
- Raw (Universal)
- P2
- Helm
- ELPA
- Go
- CocoaPods

雖然之前為了處理 CI/CD 相關要已經安裝過好幾次，雖然安裝過程簡單，也就是因為簡單讓我每次想紀錄時都覺得沒什麼內容，但每次安裝在設定時還是不免會忘東忘西，而這次安裝的間隔更久，忘得更徹底，於是興起了筆記的念頭

## 基本環境說明

1. macOS Catalina 10.15.2
2. docker community 19.03.5
3. nexus 3.20.1

## 使用 Docker 安裝

我的出發點是為了測試，所以沒有將實際的 artifact 儲存位置透過 volume 保留下來，這點要注意，詳細內容可以參考 docker hub 上的使用說明 [Sonatype Nexus3 Docker: sonatype/nexus3](https://hub.docker.com/r/sonatype/nexus3/)

```bash
docker run -d -p 8081:8081 -p 8082:8082 --name nexus sonatype/nexus3
```

上面的指令，與 docker hub 上的使用說明多開了 `8082` port，這個部份留在後面設定 image registry 時再詳細說明

## 建立並設定 Image Registry

1. 首次登入需使用先取得密碼

    > 預設帳號：`admin`，成功登入後會要求修改密碼

    ```bash
    docker exec -it nexus more /nexus-data/admin.password
    ```

    ![1login](https://user-images.githubusercontent.com/3851540/71873590-89fb8000-315a-11ea-8b32-9759bee4ff98.png)

2. 建立 Image Registry

    ![2creattterepository](https://user-images.githubusercontent.com/3851540/71873592-8a941680-315a-11ea-9b03-9e66db8afffd.png)

    ![3dockerhosted](https://user-images.githubusercontent.com/3851540/71873593-8a941680-315a-11ea-8aac-f531ba802d53.png)

    repository 有三種模式

    - hosted：建立 private Docker registry
    - proxy：可以指向其他外部的 Docker registry
    - group：(官方建議模式) 將 `hosted` 與 `proxy` 綁在一起，讓 client 透過 group 來連線

3. 設定 Image Registry

    ![4setup](https://user-images.githubusercontent.com/3851540/71873595-8a941680-315a-11ea-8746-3af94c610ecc.png)

    - 設定 repository name
    - 設定 HTTP connector

        > 這個就是 docker 多個一個 8082 port 的原因，如果沒有設定 http connector，我在執行 docker login 時一直出錯

        ![5loginerror](https://user-images.githubusercontent.com/3851540/71873597-8b2cad00-315a-11ea-873b-22a0a80b0e83.png)

## 心得

果然實際紀錄起來沒什麼內容，希望改天還有相關需求用到時可以節省一些環境建置的時間囉

不過 Nexus 我個人覺得應該是存在著某些 bug 或是我沒有找到正確的使用方法，因為 UI 沒有強制需要設定 connector，但不設定 connector ，就無法正確執行 docker login，這樣根本不能算是個可用的 docker registrry 呀

## 參考資訊

1. [COMPARE to PRO VERSION](https://sonatype.drift.click/oss-vs-pro)
2. [Manage these formats](https://www.sonatype.com/download-oss-sonatype)[Manage these formats](https://www.sonatype.com/download-oss-sonatype)
3. [Sonatype Nexus3 Docker: sonatype/nexus3](https://hub.docker.com/r/sonatype/nexus3/)
4. [Hosted Repository for Docker (Private Registry for Docker)](https://help.sonatype.com/repomanager3/formats/docker-registry/hosted-repository-for-docker-%28private-registry-for-docker%29?_ga=2.70488336.1037933046.1578291838-2065980777.1578291838)
5. [Proxy Repository for Docker](https://help.sonatype.com/repomanager3/formats/docker-registry/proxy-repository-for-docker)
6. [Grouping Docker Repositories](https://help.sonatype.com/repomanager3/formats/docker-registry/grouping-docker-repositories)
