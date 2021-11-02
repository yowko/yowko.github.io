---
title: "Git Push 出現 501 Not Implemented 錯誤"
date: 2017-11-06T23:53:00+08:00
lastmod: 2021-11-02T23:53:10+08:00
draft: false
tags: ["Git"]
slug: "git-push-501-not-implemented"
aliases:
    - /2017/11/git-push-501-not-implemented.html
---
## Git Push 出現 501 Not Implemented 錯誤

這是同事遇到的問題，雖然最後有順利解決問題，但仍然搞不清問題發生的原因。

同事在 Push 某個 repository 時出現 `error: RPC failed; HTTP 501 curl 22 The requested URL returned error: 501 Not Implemented` 的錯誤，感覺像是 Git Server 吐出來的，但使用其他同事電腦卻可以正常 push 相同內容，以結果來看又像是個人設定問題，只是比對了兩人的設定卻找不出差異，甚至是最後解決方式- `http.postBuffer` 的設定，兩人也都未指定，採用預設值，讓我百思不解呀

不過還是先紀錄一下解決方式，避免日後遇到問題又找半天

## 錯誤訊息

* 訊息內容

    ```log
    Counting objects: 33, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (25/25), done.
    Writing objects: 100% (33/33), 3.54 MiB | 33.90 MiB/s, done.
    Total 33 (delta 10), reused 29 (delta 7)
    fatal: The remote end hung up unexpectedly
    fatal: The remote end hung up unexpectedly
    error: RPC failed; HTTP 501 curl 22 The requested URL returned error: 501 Not Implemented
    Everything up-to-date
    ```

* 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32449518-bb2c0790-c34c-11e7-8253-1415512af255.png)

## 解決方式

> **調整 http.postBuffer 大小**

* 先確認 `http.postBuffer` 大小

    ```cmd
    git config --get http.postBuffer
    ```

    ![2setting](https://user-images.githubusercontent.com/3851540/32449516-bafee134-c34c-11e7-8ec8-49506eb5f5bb.png)

* 設定 `http.postBuffer`

  * 如果沒有設定值 (如上圖)，可以設定 500m 試試

    ```cmd
    git config http.postBuffer 524288000
    ```

  * 保哥文章有提到這樣設定可能有問題，**正確大小還是得自行嘗試**，詳細內容請參考保哥文章 - [Git 儲存庫太大導致無法上傳 Visual Studio Online 如���處理](https://blog.miniasp.com/post/2014/09/07/Handle-large-Git-repository-on-Visual-Studio-Online.aspx)

## 心得

解決方式很虛，既不知道發生原因，也不知道為什麼可以解決問題，再來是為什麼其他人就沒遇到問題，明明設定就一樣，總覺得不算是真正解決問題讓我好生困擾呀 @@"

## 參考資訊

1. [Git-TFS-Error: RPC failed; HTTP 501 curl 22 The requested URL returned error: 501 Not Implemented](https://ebia.at/git-tfs-error-rpc-failed-http-501-curl-22-the-requested-url-returned-error-501-not-implemented/)
2. [Git 儲存庫太大導致無法上傳 Visual Studio Online 如���處理](https://blog.miniasp.com/post/2014/09/07/Handle-large-Git-repository-on-Visual-Studio-Online.aspx)
