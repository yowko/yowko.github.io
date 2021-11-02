---
title: "IIS 出現 Service Unavailable HTTP Error 503 - Windows Process Activation Service (WAS) Error"
date: 2018-04-23T22:30:00+08:00
lastmod: 2021-11-02T10:18:55+08:00
draft: false
tags: ["IIS"]
slug: "http-error-503-event-id-5051"
aliases:
    - /2018/04/http-error-503-event-id-5051.html
---
這個問題遇到第二次了，照以往的習慣：問題遇到二次以上就要來筆記一下，避免下次再遇到又要花好多時間追查。之前沒有紀錄的原因：一來是覺得狀況特殊不太有機會再遇到，二來就是牽涉到的功能及設定不好模擬，但這次遇到相同問題又讓我花了好幾個小時才確認問題，所以多花點時間準備環境來完成紀錄

這次遇到的問題，外顯的錯誤訊息很籠統，第一時間並不好定位出問題，但搭配 event log 難度就會降低許多，就來看看實際的狀況為何吧

## 錯誤訊息

1. 訊息內容

    ```text
    Service Unavailable
    HTTP Error 503. The service is unavailable.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/39143392-620734f8-4760-11e8-9b56-e0aaa5939e0c.png)

## 解決方式

> 請務必確認遇到的狀況與下方 `如何確認解決方式適用` 情境相符

1. 取得 IIS Application Pool 所使用的 Identity

    > ![2identity](https://user-images.githubusercontent.com/3851540/39143393-622f7968-4760-11e8-8fd4-d8d4f41fa835.png)

2. 開啟 Local Policies --> User Rights Assignment --> Log on as a batch job

    > ![3logonasbatch](https://user-images.githubusercontent.com/3851540/39143394-62578bc4-4760-11e8-987c-09f46ae52aaf.png)

3. Add User or Group...

    > ![4adduser](https://user-images.githubusercontent.com/3851540/39143395-62812ae2-4760-11e8-9b1d-8fd3cecec471.png)

4. 重新開機(不一定需要)

    > 較新的 Windows 10 及 Windows Server 2016 都可以直接生效，如果較舊的作業系統建議重新開機

## 如何確認解決方式適用

1. IIS 出現 503 Service Unavailable

2. Application Pool 啟動後立即被停止

    > ![5alwaysstop](https://user-images.githubusercontent.com/3851540/39143396-62aede7e-4760-11e8-92d0-6d3992cc6719.png)

3. Event log 中出現 WAS 類型的 Error 及 Warning

    > 以下三個 WAS 類型的 Error 及 Warning 會接續出現

    ![6eventlog](https://user-images.githubusercontent.com/3851540/39143397-62dbd460-4760-11e8-9d2c-e53314ea1ec9.png)

    * `Error` - Event ID 5059

        ```text
        Application pool TestTimeout has been disabled. Windows Process Activation Service (WAS) encountered a failure when it started a worker process to serve the application pool.
        ```

        > ![7eventerror](https://user-images.githubusercontent.com/3851540/39143398-6309cdfc-4760-11e8-84e0-91407e871335.png)

    * `Warning` - Event ID 5057

        ```text
        Application pool TestTimeout has been disabled. Windows Process Activation Service (WAS) did not create a worker process to serve the application pool because the application pool identity is invalid.
        ```

        > ![8eventwarning1](https://user-images.githubusercontent.com/3851540/39143399-63353a5a-4760-11e8-8ce4-a1a3f9f87097.png)

    * `Warning` - Event ID 5051

        ```text
        The identity of application pool TestTimeout is invalid. The user name or password that is specified for the identity may be incorrect, or the user may not have batch logon rights. If the identity is not corrected, the application pool will be disabled when the application pool receives its first request.  If batch logon rights are causing the problem, the identity in the IIS configuration store must be changed after rights have been granted before Windows Process Activation Service (WAS) can retry the logon. If the identity remains invalid after the first request for the application pool is processed, the application pool will be disabled. The data field contains the error number.
        ```

        > ![9eventwarning2](https://user-images.githubusercontent.com/3851540/39143401-636fe178-4760-11e8-8c80-486262dded71.png)

## 需要 Logon as Batch Job 權限的情境

1. 寫入資料至分享資料夾

    > 我遇到的狀況屬於這個

    * 程式使用到分享資料夾(透過網路位置加入)

        > ![10networklocation](https://user-images.githubusercontent.com/3851540/39143402-639c81f6-4760-11e8-86a1-2413b5104c40.png)

    * IIS 使用特定 user 來存取網路硬碟

        > ![11poolidentity](https://user-images.githubusercontent.com/3851540/39143391-61c353dc-4760-11e8-97f2-e43b22b291dc.png)

2. 使用分享印表機

3. 執行互動式程式

## 心得

這次遇到的問題以結果來看不算是困難的狀況，但為了確認解決方案花了較多的心力，主要原因在於實際遇到的 Server 我沒有管理權限，AD 設定也看不到，所以要調整 Group Policies 就變得困難，排除這個因素在定位錯誤原因也花了不少時間：雖然 event log 中有提到缺少 batch logon 的權限，但遲遲沒有意識到是個 Local Policies 的設定，所幸透過重新模擬環境得以完成紀錄，不用擔心下次再遇到問題又要花很多時間才能解決了 哈哈

## 參考資訊

1. [How to See Which Groups Your Windows User Account Belongs To](https://www.howtogeek.com/tips/how-to-see-which-group-your-windows-user-belongs-to/)
2. [Granting "Logon as a batch job"](https://www.brooksnet.com/faq/granting-logon-as-batch-privilege)
