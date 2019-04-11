---
title: "初探容器技術(Container)"
date: 2017-05-01T00:45:00+08:00
lastmod: 2018-09-18T09:57:03+08:00
draft: false
tags: ["Docker"]
slug: "container"
aliases:
    - /2017/05/container.html
---
# 初探容器技術(Container)
近幾年容器(container) 技術發展篷勃，尤其 Docker open source 後，各家大型軟體公司都陸續投入資源加入新一代的虛擬化技術。而從聽到 docker 與 container 相關技術也好幾年了，無奈身為半路出家的 .NET 派工程師對於 linux 實在苦手，雖然在 Windows 上有 boot2docker 的工具可以測試，但執行效能跟穩定性還是相當不足，直到 Windows Server 2016 為了 container 技術重新設計架構推出更完整的支援， 在 Windows 10 上也推出相對應的功能，讓習慣 Windows 環境的技術人員也能與 docker 所帶起的這波新潮流接軌，接下來就介紹個人接觸 container 技術的相關心得與筆記

## 什麼是容器(container) 技術

容器(container) 技術雖然與傳統虛擬化(Virtual Machine) 都是虛擬化技術，但與傳統需要需要安裝作業系統的虛擬化技術不同，容器(container) 技術共用底層 OS 的做法，從 OS 層級的虛擬化轉為應用程式層級的虛擬化，這樣的轉變讓原本所需的資源大為降低、執行速度也快上許多，

VM v.s. Container

|-|VM|Container|
|--- |--- |--- |
|架構圖|![1vm](https://user-images.githubusercontent.com/3851540/39084482-5e56a1d0-45a9-11e8-9978-dc07c7879232.png)|![2container](https://user-images.githubusercontent.com/3851540/39084481-5e148318-45a9-11e8-97d0-56a3f1720a6d.png)|
|虛擬化方式|作業系統層級|應用程式層級|
|支援 OS|所有 OS|僅支援 image 對應的 OS|
|啟動速度|以分鐘計|以秒計|
|佔用資源|多(10-100 vm)|少(100-1000 container)|
|代表作|vSphere,Hyper-V|docker|



## 容器(container) 技術類型

1.  Linux Container

    *   chroot

        > 1982 首見於 Unix

    *   Docker for linux ....etc

        > 由 Docker 公司 open source 的標準化 container 流程

2.  Windows Container


    *   Windows Server Container


        > 首見 Windows Server 2016

    *   Hyper-V Container

        > Windows 10 、 boot2docker、Docker for Windows 都是這個做法，這也能讓 Windows 可以使用 linux container

## docker 技術名詞解釋

這邊簡單介紹一下學習 docker 時，比較常見的幾個名詞：

1.  Image

    > 唯讀映像檔，是一套執行環境的封裝，用來建立 container 用的，可以由網路下載或是自行建立

2.  Container

    > 由 image 建立的環境執行 instance，實際用來部署服務及程式的環境，不同於 image 的唯讀特性， container 可讀可寫，container 也可以被啟動、停止及刪除，各個 container 間是互相隔離的

3.  Repository

    > image 的儲存庫 ，像是 git 的 repository

4.  Registry

    > docker repository 伺服器，像是 GitHub 一樣，Docker Hub 就是 docker 官方提供的 registry，如果有資安疑慮也可以自建

5.  Build

    > 可以將當下的 container 封裝成 image ，供建立其他 container 用

6.  Dockerfile

    > 用來建立 image 的描述 script

# 參考資訊
1.  [10個Q&A快速認識Docker](http://www.ithome.com.tw/news/91847)
2.  [Comparing Containers and Virtual Machines](https://www.docker.com/what-container)
3.  [Docker —— 從入門到實踐(繁中)](https://philipzheng.gitbooks.io/docker_practice/content/)
4.  [Docker — 從入門到實踐(簡中)](https://yeasy.gitbooks.io/docker_practice/content/)
