---
title: "不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle"
date: 2017-11-21T23:11:00+08:00
lastmod: 2021-11-03T23:11:15+08:00
draft: false
tags: ["csharp","Oracle"]
slug: "csharp-oracle-tns-without-client"
aliases:
    - /2017/11/c-sharp-oracle-tns-without-client.html
    - /2017/11/csharp-oracle-tns-without-client
---
## 不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle

之前進公司時報到的第一天依公司前輩給的文件開始架設開發環境，大部份環境都很熟悉不用多久時間就完成安裝及設定，唯獨 Oracle Client 讓我裝了一整天，主要是步驟繁瑣，沒有依前人的文件設定就會失敗，但前人文件版本又太舊，安裝畫面已經不同，搞了好久才成功

直到最近因為想要架設整合測試環境，所以想要把 Oracle client 的安裝給自動化，但重新回想起第一次安裝的慘痛經驗，立馬改變想法：看是否有辦法避開 Oracle Client

經過一輪測試下來，不安裝 Oracle Client 使用 Oracle SQL developer 可以正常連線、.NET 程式(console、ASP.NET MVC) 透過 `Oracle.ManagedDataAccess` 或是 `Oracle.DataAccess.x86` 這兩個套件也可以正常存取資料，這讓我懷疑起 Oracle Client 的用處

據與 DBA 大大請教的結果，因為部份功能只有 Oracle Client 有實作(e.x.：reset password) 加上許多程式都是使用 TNS 進行連線，所以才列為必要安裝項目之一。這樣一來我確定不會在整合測試中操作 reset password ，因此只要解決使用 TNS 連線的問題就好，立馬來看看我的解決方式吧

## 關於 Oracle 連線方式

1. EZCONNECT

    > 使用帳號、密碼並指定 server ip、port 及服務名稱

2. TNSNAMES

    > 透過 tnsnamses.ora 來指定連線位置

3. HOSTNAME

    > 這個我沒用過，不說明了

4. LDAP

    > 類似 windows 驗證

5. NIS

    > 這個我沒用過，不說明了

其中最常見的(可能是我自己最常見)的就是 `EZCONNECT` 與 `TNSNAMES`，而 `EZCONNECT` 使用上比較直覺就是指定相關連線資訊就可以直接連線了，`TNSNAMES` 則是將 server 資訊紀錄在 `tnsnamses.ora` (帳號密碼則仍由程式提供)，類似於 host file 的概念，使用別名進行轉導過去

## 設定使用 tnsnamses.ora 連線

因為既有程式都使用 tns 來連線 oracle，身為進行整合測試的角色實在沒有立場為了測試目標要求改使用 `EZCONNECT` ，所以需要克服 tns 連線問題

1. tnsnamses.ora 預設位置

    > 一般位於 Oracle Client 安裝資料夾中：`C:\oracle\product\10.2.0\db_1\NETWORK\ADMIN`

2. 不安裝 Oracle Client 直接指定 `tnsnamses.ora` 位置

    * 自行準備 `tnsnamses.ora`

        ```config
        yowkooracle =
        (DESCRIPTION =
            (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
            (CONNECT_DATA =
            (SERVICE_NAME = xe)
            )
        )
        ```

    * 設定環境變數 `TNS_ADMIN`
        * 使用 command

            * 語法

                ```cmd
                setx TNS_ADMIN {tnsnamses.ora 所在資料夾} /M
                ```

            * 範例

                ```cmd
                setx TNS_ADMIN c:\oracle /M
                ```

        * 使用 PowerShell
            * 語法

                ```ps1
                [Environment]::SetEnvironmentVariable("TNS_ADMIN", "{tnsnamses.ora 所在資料夾}", "Machine")
                ```

            * 範例

                ```ps1
                [Environment]::SetEnvironmentVariable("TNS_ADMIN", "c:\oraclexe", "Machine")
                ```

## 心得

經由設定環境變數 (`TNS_ADMIN`) 指定 `tnsnamses.ora` 位置就可以讓程式可以使用 tns 連線純粹是誤打誤撞、矇來的，我沒有找到直接文件說明這麼做是可行的，只是看到不少安裝介紹文件有提到這個環境變數 (`TNS_ADMIN`)，測試後想不到真的可行，但在成功之前我也做了許多其他嘗試才找到這個可行方案的

## 參考資訊

1. [HowTo: Set an Environment Variable in Windows - Command Line and Registry](http://www.dowdandassociates.com/blog/content/howto-set-an-environment-variable-in-windows-command-line-and-registry/)
2. [Windows PowerShell Tip of the Week](https://technet.microsoft.com/en-us/library/ff730964.aspx)
3. [Oracle Database - TNS_ADMIN environement variable](https://gerardnico.com/wiki/db/oracle/tns_admin)
4. [Oracle 11g-關於Listener.ora / Sqlnet.ora / Tnsnames.ora](http://biangbiangmichael.blogspot.tw/2013/08/oracle-11g-listenerora-sqlnetora.html)
5. [Oracle : listener.ora, sqlnet.ora, tnsnames.ora](http://rickyju.pixnet.net/blog/post/28183404-oracle-%3A-listener.ora%2C-sqlnet.ora%2C-tnsnames.ora)
6. [Setting an Oracle Connection to Use TNSNames.ora or LDAP.ora](http://kb.tableau.com/articles/howto/setting-an-oracle-connection-to-use-tnsnames-ora-or-ldap-ora)
7. [How to set proper path to TNSNAMES file in C# application?](https://stackoverflow.com/questions/10618512/how-to-set-proper-path-to-tnsnames-file-in-c-sharp-application)
8. [How to connect to Oracle DataBase using TNSName](https://www.codeproject.com/Questions/158011/How-to-connect-to-Oracle-DataBase-using-TNSName)
