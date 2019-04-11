---
title: "匯出與匯入 Windows Container Image"
date: 2019-02-21T20:40:00+08:00
lastmod: 2019-02-21T20:40:30+08:00
draft: false
tags: ["Docker","Windows"]
slug: "export-import-windows-container-image"
---
# 匯出與匯入 Windows Container Image
同事想將 build 好的 image 在其他 server 測試驗證，但又不想上傳至公開 docker hub ，加上還只是 POC 階段建立 private registry 的效益有限，所以想要先直接 share image 就好

之前的經驗是打包 linux image 網路的做法也是針對 linux image，windows image 相對資源較少，就來試試並紀錄一下囉


## 基本環境說明

1. Windows 10 Version 1803 (OS Build 17134.590)
2. Docker Community 18.09.2 (windows/amd64)

## 語法解釋

- export

    > 將 container 內容匯出為 tar 檔

- save

    > 將 image 匯出為 tar

以同事的需求應該要使用 `save`

## 匯出 Windows Container Image
* 語法

    ```cmd
    docker save -o {輸出檔案位置及名稱} {image}
    ```

    或是

    ```cmd
    docker save {image} -o {輸出檔案位置及名稱}
    ```

* 範例

    ```cmd
    docker save -o c:\windowswithgit.tar windowswithgit:v2
    ```

    或是

     ```cmd
    docker save windowswithgit:v2 -o c:\windowswithgit.tar
    ```

## 匯入 Windows Container Image
- 語法

    ```cmd
    docker load -i {檔案位置}
    ```
- 範例
    
    ```cmd
    docker load -i f:\windowswithgit.tar
    ```
- 實際操作

    - 匯入前：無任何 image

        ![1noimages](https://user-images.githubusercontent.com/3851540/53183058-9d325980-3635-11e9-9405-2d4a06350cf6.png)

    - 匯入後：已新增 image

        ![2imported](https://user-images.githubusercontent.com/3851540/53183059-9dcaf000-3635-11e9-9e9e-ccc14e45d8ec.png)

## 心得
原以為 Windows Container Image 在處理上會有什麼不同，結果與 Linux Contianer Image 操作都一樣嘛，微軟在操作 container 的指令、行為比照 Linux 的做法對於工程師實在相當便利呀，讓不同平台使用者可以降低不少轉換成本

不過基於 Windows Container Image size 的問題，不過是 export 或是 import 需要的時間都不少喔，需要有點耐心

# 參考資訊
1. [How to Export and Import Windows Containers](https://www.deploycontainers.com/2017/09/11/export-import-windows-containers/)
2. [Import and Export Docker images for windows container](https://www.assistanz.com/import-and-export-docker-images/)
3. [比較 save, export 對於映像檔操作差異](https://blog.hinablue.me/docker-bi-jiao-save-export-dui-yu-ying-xiang-dang-cao-zuo-chai-yi/)