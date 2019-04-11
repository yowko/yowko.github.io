---
title: "Visual Studio 無法 Debug 專案 (錯誤代碼：CS0006)"
date: 2017-05-24T01:00:00+08:00
lastmod: 2018-09-22T01:00:00+08:00
draft: false
tags: ["Debug","Visual Studio"]
slug: "visual-studio-debug-cs0006"
aliases:
    - /2017/05/visual-studio-debug-cs0006.html
---
# Visual Studio 無法 Debug 專案 (錯誤代碼：CS0006)
剛剛開啟一個月前做的專案，針對新的需求做個小調整，說是小調整其實也改了 flow，所以連帶著也要調整 unit test，但執行結果與預期不相符，立馬按下 Ctrl + R，Ctrl + T 要來 debug 為什麼出錯，結果既然出錯了XD 要不是我確定這個專案只有我做，我一定會暗自咒罵 "到底會不會寫程式" ，以結果來看，不會寫程式的人是我 - 自己做的專案還是搞到自己沒辦法 debug 實在是警訊呀

## 確認 Configuration

專案可能存在不同的 Configuration 的設定，第一時間或是時間久遠，第一個念頭想到可能是 Configuration 造成的問題，所以先確認當下使用的 Configuraion 是 release mode 還是 debug mode (release mode 無法逐步 debug)

![1config](https://cloud.githubusercontent.com/assets/3851540/26347936/b94fec04-3fdd-11e7-9a44-32d23cfec7ca.png)


*  無法偵錯

    ![2cannotdebug](https://cloud.githubusercontent.com/assets/3851540/26347941/b9595726-3fdd-11e7-8aa7-31d86ebcb0ae.png)

## 錯誤資訊

> 確認使用正確 configuration 後，還是出現無法 debug 的錯誤，錯誤訊息還更難理解

*   無法偵錯

    ![3buildfail](https://cloud.githubusercontent.com/assets/3851540/26347938/b954181a-3fdd-11e7-8213-48ba42449624.png)

*   錯誤訊息

    - 訊息內容
        
        ```
        Severity Code Description Project File Line Suppression State
        Error CS0006 Metadata file 'D:\SyncSessionDBRedisService\UK\UK-SyncSessionDBRedisService\SyncSessionDBRedisServiceConsole\bin\Debug\SyncSessionDBRedisServiceConsole.exe' could not be found SyncSessionDBRedisServiceConsoleTests D:\SyncSessionDBRedisService\UK\UK-SyncSessionDBRedisService\SyncSessionDBRedisServiceConsoleTests\CSC 1 Active
        ```
    - 錯誤截圖
        
        ![4errormsg](https://cloud.githubusercontent.com/assets/3851540/26347940/b956e996-3fdd-11e7-8408-7de6e95cd18d.png)

*   錯誤代碼
        
    ```
    CS0006  Test C# Metadata file could not be found
    ```

## 如何解決

1.  solution --> 按右鍵 --> Properties

    ![5properties](https://cloud.githubusercontent.com/assets/3851540/26347937/b9542710-3fdd-11e7-9c3e-71f3799eb603.png)

2.  Configuration Properties --> Configuration --> 勾選 `Build`

    ![6checked](https://cloud.githubusercontent.com/assets/3851540/26347939/b955b828-3fdd-11e7-80ae-9eca3ee0e53d.png)

*   如果本來就是 `勾選`  先取消 `勾選` 存檔之後再重新 `勾選` 一次


## 心得

找到解決方式也確認可以順利解決問題後，我開始反思到底是我流年不利還是根本就是我老愛碰些有的沒的而汙染了我的開發環境，不過生活就是因為這些出乎意料外的挑戰才顯得更有趣呀

# 參考資訊
1.  [Metadata file '.dll' could not be found](https://stackoverflow.com/questions/1421862/metadata-file-dll-could-not-be-found)
