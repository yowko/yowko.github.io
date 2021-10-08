---
title: "在 Windows 中使用 Command 安裝 Oracle XE (Silent Installation)"
date: 2017-11-08T01:48:00+08:00
lastmod: 2021-10-08T01:48:20+08:00
draft: false
tags: ["Oracle"]
slug: "windows-oracle-xe-silent-install"
aliases:
    - /2017/11/windows-oracle-xe-silent-install.html
---
## 在 Windows 中使用 Command 安裝 Oracle XE (Silent Installation)

想要將自動化測試範圍擴大，其中一部份就是將 db 語法也納入自動化測試中，為了避免跨 OS 遇到更多麻煩，所以打算先透過 Windows Oracle 來進行

第一步就是將 Oracle 安裝自動化，用法有些不同，特別紀錄一下

## 關於 Silent Installation

* 不需透過 GUI 的互動模式來執行安裝
* 相關安裝參數使用 Response File 來指定

## 安裝 Oracle XE

1. 使用管理者權限執行安裝
2. 下載 oracle xe 安裝檔
    * [Oracle Database Express Edition 11g Release 2](http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html)
    * Downloads --> Accept License Agreement --> 選擇適合版本

        ![1download](https://user-images.githubusercontent.com/3851540/32508738-40a8a3d6-c426-11e7-989b-b6bcc374c268.png)

3. 解壓縮 oracle xe 安裝檔

    > 完整資料夾結構如下，其中 `*.iss` 即為 Response File，也就是安裝參數設定值儲存檔

    * response
        * OracleXE-install.iss
        * OracleXE-remove.iss
        * OracleXE-repair.iss

    * upgrade

        * gen_inst.sql

    * setup.exe

4. 檢查參數設定值
    * szDir: 有效路徑
        * 預設值：`C:\oraclexe\`
        * 執行安裝時就必需確保其中 `szDir` 是存在的，否則會出現 `ResultCode=-3` 的問題

    * TNSPort: listener port ，用來連線至 Oracle XE
        * 預設值：`1521`

    * MTSPort: MTS port
        * 預設值：`2031`

    * HTTPPort: http port

        * 預設值：`8080`

    * SYSPassword:`sys` 管理者密碼

        * 預設值：`oraclexe`

5. 執行指令

    * pattern

        ```cmd
        setup.exe /s /f1"{Response File 位置}" /f2"{安裝 log 位置}"
        ```

        * `/s`：silent mode
        * `/f1`：指定 response file
        * `/f2`：指定 log file

    * 實例

        ```cmd
        "C:\oraclexeinstall\setup.exe" /s /f1"C:\oraclexeinstall\response\OracleXE-Install.iss" /f2"C:\oraclexeinstall\setup.log"
        ```

## 安裝成功

> 安裝時間比較長，可能要個數十分鐘，可以透過 `setup.log` 建立來判斷是否完成安裝

1. 安裝成功時 log 會出現 `ResultCode=0`
2. `Services.msc` 會加入數個 service

    > 其中下列 service 的狀態會是 `執行中`

    * OracleXETNSListener
    * OracleServiceXE

        ![2services](https://user-images.githubusercontent.com/3851540/32508739-40d4792a-c426-11e7-9ecc-059b037f5938.png)

3. 測試成功

    ![3success](https://user-images.githubusercontent.com/3851540/32508740-40fe2996-c426-11e7-9431-3876bbc2dbe3.png)

## 心得

本來透過 msi silent install 的語法，結果就出現錯誤。找了好一下資料才真正解決問題、安裝成功，主要就是對 Oracle 安裝的機制不了解，這點不由得想要抱怨一下 Oracle，有自己的機制就算了，資料還不好找，使用者體驗不太好

## 參考資訊

1. [Oracle® Database Express Edition](https://docs.oracle.com/cd/E17781_01/install.112/e18803/toc.htm#XEINW125)
2. [Customizing and Creating Response Files](https://docs.oracle.com/cd/B10501_01/em.920/a96697/rsp.htm)
3. [Unattended Installation](http://www.oracle.com/technetwork/tutorials/tutorials-1725259.html)
