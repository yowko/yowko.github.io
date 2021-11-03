---
title: "解決無法刪除 dcoker 在 Windows 下的 image 預設目錄 windowsfilter"
date: 2017-08-24T00:13:00+08:00
lastmod: 2021-10-28T00:13:51+08:00
draft: false
tags: ["Debug","Docker","Windows 10","Windows Server 2016"]
slug: "remove-windowsfilter"
aliases:
    - /2017/08/remove-windowsfilter.html
---
## 解決無法刪除 dcoker 在 Windows 下的 image 預設目錄 windowsfilter

之前文章 [Widnows 環境中修改 Docker image 的儲存位置](/windows-docker-image-location) 介紹如何修改 Windows 環境中的 docker image 儲存位置，讓 image 可以不需佔用系統槽空間

今天同事反應，原本預設的 image 儲存位置 `C:\ProgramData\dcoker\windowsfilter` 因為已有 image，加上 windows 環境的 image size 都不小，一下就把系統槽的空間吃光了，想刪掉還刪不掉

## 無法刪除

1. 錯誤訊息
    * 中文

        ```txt
        您須具有執行此動作的權限
        
        您可以向 Administrators 取得權限來變更此資料夾
        ```

    * 英文

        ```txt
        You need permission to perform this action
        
        You require permission from Administrators to make changes to this folder
        ```

2. 錯誤畫面

    * 中文

        ![1chinese](https://user-images.githubusercontent.com/3851540/29626210-995c3ee0-8860-11e7-80f3-9d62e0e875ed.png)

    * 英文

        ![2english](https://user-images.githubusercontent.com/3851540/29626209-9958f87a-8860-11e7-9a78-e6f8d7986e4a.png)

## 解決方式

詳細內容可以參考：[[Windows] windowsfilter folder impossible to delete](https://github.com/moby/moby/issues/26873)

1. 下載 `docker-ci-zap.exe`

    >下載位置：[https://github.com/jhowardmsft/docker-ci-zap/blob/master/docker-ci-zap.exe](https://github.com/jhowardmsft/docker-ci-zap/blob/master/docker-ci-zap.exe)

2. 使用 `docker-ci-zap.exe` 刪除 `windowsfilter`

    ```cmd
    .\docker-ci-zap.exe -folder "C:\ProgramData\docker\windowsfilter"
    ```

3. 成功刪除

    ![3deleted](https://user-images.githubusercontent.com/3851540/29626211-99609c42-8860-11e7-845a-30b1bfafb8ff.png)

## 心得

這個問題應該是 bug 吧，如果資料夾被 service 咬住還可以理解，但 service 已指向新的路徑，卻仍然無法成功刪除，還需要透過其他工具來處理，感覺還有改善的空間

## 參考資訊

1. [Widnows 環境中修改 Docker image 的儲存位置](/windows-docker-image-location)
2. [[Windows] windowsfilter folder impossible to delete](https://github.com/moby/moby/issues/26873)
3. [docker-ci-zap.exe 下載位置](https://github.com/jhowardmsft/docker-ci-zap/blob/master/docker-ci-zap.exe)
