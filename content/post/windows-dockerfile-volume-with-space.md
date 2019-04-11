---
title: "Windows Dockerfile 如何指定 VOLUME - 更新版"
date: 2017-09-28T00:04:00+08:00
lastmod: 2018-09-26T00:04:24+08:00
draft: false
tags: ["Docker"]
slug: "windows-dockerfile-volume-with-space"
aliases:
    - /2017/09/windows-dockerfile-volume-with-space.html
---
# Windows Dockerfile 如何指定 VOLUME - 更新版
前幾天同事打算將 Windows container 中的 Jenkins 中的 job 跟 log 保留在 container 外部，避免因為 jenkins container 更新而造成設定資料遺失，經過一番測試後出現了 [Windows Dockerfile 如何指定 VOLUME](https://blog.yowko.com/2017/09/windows-dockerfile-volume.html) 一文，結果同事今天跟著實作時卻發現有問題，所以我又來了XD 非常感謝同事的測試，果然不愧是 King of QA，一直抓出我的錯誤

## 問題描述

[Windows Dockerfile 如何指定 VOLUME](https://blog.yowko.com/2017/09/windows-dockerfile-volume.html) 提到在 dockerfile 中指定 volume 的方法有兩種(因為兩種作法都是錯的，這邊就略過不提)，錯誤的不是語法本身而是邏輯 - 文中提到的兩種 volume 指定語法都可以通過編譯，但最後結果卻是不正確的

*   預期結果

    > `c:\Program Files (x86)`

*   實際結果

    > `c:\program%20files%20%28x86%29`

    ![1error](https://user-images.githubusercontent.com/3851540/30923800-78f6164a-a3df-11e7-9ad8-750718451cfb.png)

## 如何修正

dockerfile 中指定 volume 的方式有兩種：(詳細說明請參考官網 [Dockerfile reference ](https://docs.docker.com/engine/reference/builder/#volume))

1.  使用 json 陣列表示法

    > `VOLUME ["/var/log/"]`

2.  使用純文字表示並用空白符號當區隔

    > `VOLUME /var/log /var/db`

因為 Jenkins 預設安裝在 `c:\Program Files (x86)` 中，透過純文字指定路徑時因為路徑中有空白符號造成會讓編譯失敗才改用 ascii code 修改卻也使得最後結果不如預期，加上在 dockerfile 中 <code>`</code> 與 <code>\\</code> 都是跳脫符號讓 volume 的路徑設定更為麻煩

*   修改方式
    *   使用 json array 表示法
    *   路徑 `\` 改用 `/`

        > 在 windows 環境中兩種符號行為相同

    ```dockerfile
    #指定基礎 os image
    FROM microsoft/windowsservercore

    # image 維護者資訊
    MAINTAINER yowko@yowko.com

    #將 jenkins 安裝檔複製到 container 中
    ADD ./setup c:/jenkins

    #安裝 jenkins
    RUN ["msiexec.exe", "/i", "C:\\jenkins\\jenkins.msi", "/qn"]

    #移除 container 中的安裝檔(讓 image 保持乾淨)
    RUN Powershell.exe -Command remove-item c:/jenkins -Recurse

    # 指定 volume
    VOLUME ["C:/Program Files (x86)/Jenkins/jobs","C:/Program Files (x86)/Jenkins/logs"]

    #對外使用 8080 port
    EXPOSE 8080  
    #如果有 slave 需加上 50000 port
    EXPOSE 50000

    # 預設執行動作，可用來避免 container 自動停止
    CMD [ "ping localhost -t" ]
    ```

以下步驟可以參考 [使用 dockerfile 建立 Windows Container 版 Jenkins](https://blog.yowko.com/2017/08/dockerfile-windows-container-jenkins.html)

*   build 成 image

    ```
    docker build -t yowko/newwinjenkins:latest c:\jenkins
    ```

*   使用 image 建立 container 並指定 volume 對應

    ```
    docker run -it -v C:\jenkinsvolume\jobs:"C:/Program Files (x86)/Jenkins/jobs" -v C:\jenkinsvolume\logs:"C:/Program Files (x86)/Jenkins/logs" -p 8080:8080 yowko/newwinjenkins:latest
    ```

## 心得

前幾天在測試 docker volume 語法，卡關好久一直無法正確通過編譯，直到通過編譯卻沒有檢查正確性所以造成最後結果不如預期，幸虧同事協助驗證才沒讓我帶著錯誤的理解太久，非常感謝同事的幫忙

另外關於同事的一個問題：dockerfile volume 與 docker run -v 的差異，我總覺得的似懂非懂，後來找到一篇文章讓我清楚知道兩種的不同之處： [Dockerfile VOLUME 和 -v 的區別](http://elickzhao.github.io/2016/04/Dockerfile%20VOLUME%20%E5%92%8C%20-v%20%E7%9A%84%E5%8C%BA%E5%88%AB/)，提供給有相同疑問的朋友參考

# 參考資訊

1.  [Windows Dockerfile 如何指定 VOLUME](https://blog.yowko.com/2017/09/windows-dockerfile-volume.html)
2.  [Dockerfile reference ](https://docs.docker.com/engine/reference/builder/#volume)
3.  [使用 dockerfile 建立 Windows Container 版 Jenkins](https://blog.yowko.com/2017/08/dockerfile-windows-container-jenkins.html)
4.  [Dockerfile VOLUME 和 -v 的區別](http://elickzhao.github.io/2016/04/Dockerfile%20VOLUME%20%E5%92%8C%20-v%20%E7%9A%84%E5%8C%BA%E5%88%AB/)
