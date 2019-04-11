---
title: "找到咬住檔案的程式 - 解決無法刪除或移動檔案"
date: 2017-08-08T23:23:00+08:00
lastmod: 2018-09-24T23:23:27+08:00
draft: false
tags: ["Debug","Tools"]
slug: "find-process-lock-file"
aliases:
    - /2017/08/find-process-lock-file.html
---
# 找到咬住檔案的程式 - 解決無法刪除或移動檔案
大多數人都曾經遇到在 Windows 要刪除或是移動檔案時，出現檔案被另個程式使用中的提示而造成無法完成刪除或是移動的操作

今天剛好在測試 RabbitMQ 時，發現無法完整移除相關檔案，所以紀錄一下該如何處理

## 重現問題

*   提示訊息

    ```
    The action cannot be completed because the file is open in another program
    ```

*   提示畫面

    ![1ALERT](https://user-images.githubusercontent.com/3851540/29079414-b5801038-7c8f-11e7-9f99-b11e814f317d.png)

## 解決方式

1.  外部工具

    > 以前最常用的是個名叫 unlock 的小工具，它可以協助關閉使用特定檔案的程式，讓原本無法執行的刪除或是移動操作可以順利進行，但現在病毒氾濫不免讓人憂心這樣的小工具會不會有木馬，而我漸漸地也不使用這樣的小工具了

2.  官方工具 - [Microsoft/SysInternals Process Explorer](https://docs.microsoft.com/zh-tw/sysinternals/downloads/process-explorer)


    *   程式主選單 Find --> Find Handle or DLL..

        ![2processexplorer](https://user-images.githubusercontent.com/3851540/29079413-b57de498-7c8f-11e7-8cea-a68006f3b0f7.png)

    *   輸入檔案 or 資料夾 --> 搜尋 --> 點擊 Process --> Process Explorer 就會選取該 Process

        ![3search](https://user-images.githubusercontent.com/3851540/29079416-b5991844-7c8f-11e7-8bcb-36851fed4c03.png)

    *   在 Process 上按右鍵即可刪除該 Process

        ![4kill](https://user-images.githubusercontent.com/3851540/29079417-b5a17430-7c8f-11e7-8d3f-dc95419ca20b.png)

3.  Windows 內建工具 - Resource Monitor

    > 微軟提供的官方工具已經非常方便了，但如果環境無法隨意加入這類小工具，可能就只能用 Windows 內建工具來解決了

    *   開啟資源監控器
        *   直接搜尋 `resmon.exe`

            ![5resmon](https://user-images.githubusercontent.com/3851540/29079419-b5a87f46-7c8f-11e7-8f59-3466e2b6b81a.png)

        *   從工作管理員進入

            *   工作管理員 或是 按 `ctrl+alt+del` 進入

                ![6taskmanager](https://user-images.githubusercontent.com/3851540/29079420-b5aee520-7c8f-11e7-9986-cde9009530dd.png)

            *   Performance --> Resource Monitor

                ![7resourcemon](https://user-images.githubusercontent.com/3851540/29079421-b5bc2f14-7c8f-11e7-8fcf-ddeb5963acb3.png)

    *   CPU --> Associated Handles --> 搜尋檔案或是資料夾 --> Process 上按右鍵即可 `End Process`

        ![8endprocess](https://user-images.githubusercontent.com/3851540/29079422-b5bd2c84-7c8f-11e7-9dd1-52fa7bd919d0.png)

## 心得

測試起來 Process Explorer 速度比較快，列出的 Process 也比較完整跟清楚，但 Resource Monitor 的內建特性還是方便許多，如果不是本來就有在使用 Process Explorer，可能會使用 Resource Monitor 為主，提供大家參考

# 參考資訊

1.  [Microsoft/SysInternals Process Explorer](https://docs.microsoft.com/zh-tw/sysinternals/downloads/process-explorer)
2.  [Find out which process is locking a file or folder in Windows](https://superuser.com/questions/117902/find-out-which-process-is-locking-a-file-or-folder-in-windows)
