---
title: "使用 Dockerfile 建立 Windows Container 版本 Oracle XE"
date: 2017-11-14T00:43:00+08:00
lastmod: 2021-11-03T22:27:43+08:00
draft: false
tags: ["Docker","Oracle"]
slug: "dockfile-windows-container-oracle-xe"
aliases:
    - /2017/11/dockfile-windows-container-oracle-xe.html
---
## 使用 Dockerfile 建立 Windows Container 版本 Oracle XE

之前曾經在筆記 [在 Windows 中使用 Command 安裝 Oracle XE (Silent Installation)](/2017/11/windows-oracle-xe-silent-install.html) 提到打算將 db 語法也納入自動化測試，所以想要透過 script 來安裝 Oracle XE，既然已經確定 Oracle XE 可以使用 silent installation，那當然不可以錯過使用 container 技術來建立囉

原本的如意算盤是透過 dockerfile 將安裝指令改成 Oracle XE 的 silent installation，大概半天就可以解決收工，想不到跟同事東試西試就是搞不定，各式各樣的方法用了不下數十種，最後在 DBA 同事強而有力的幫助下終於找到可行方式，一定要紀錄一下其中的心酸血淚呀

首先發自內心的感想：Oracle 還是比較適合 linux，在 windows 環境中文件跟資源都不是那麼充裕，常常會遇到靈異現象，如果不是真的有強烈需求，建議不要找自己麻煩

## 準備 Oracle XE 安裝檔

> 詳細內容可以參考之前筆記 [在 Windows 中使用 Command 安裝 Oracle XE (Silent Installation)](/2017/11/windows-oracle-xe-silent-install.html)，有更完整的說明

1. 下載 oracle xe 安裝檔

    * [Oracle Database Express Edition 11g Release 2](http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html)
    * Downloads --> Accept License Agreement --> 選擇適合版本

        ![1download](https://user-images.githubusercontent.com/3851540/32508738-40a8a3d6-c426-11e7-989b-b6bcc374c268.png)

2. 解壓縮 oracle xe 安裝檔

    > 完整資料夾結構如下，其中 `*.iss` 即為 Response File，也就是安裝參數設定值儲存檔

    * response
        * OracleXE-install.iss
        * OracleXE-remove.iss
        * OracleXE-repair.iss

    * upgrade
        * gen_inst.sql

    * setup.exe

## 使用 dockerfile 建立 image

1. 建立 `C:\oraclexeinstall`
2. 將解壓後的 oracle 安裝檔置於 `C:\oraclexeinstall\install` 中

    ![1oraclefolder](https://user-images.githubusercontent.com/3851540/32737003-dd19f8da-c8d3-11e7-9fb8-f299b282444a.png)

3. 在 `C:\oraclexeinstall` 中建立 `dockerfile`

    ```dockerfile
    #指定基礎 os image
    FROM microsoft/windowsservercore
    
    # image 維護者資訊
    MAINTAINER yowko@yowko.com
    
    #將 oracle xe 安裝檔複製到 container 中
    ADD ./install c:/oracle
    
    #建立 oracle xe 的安裝目錄
    RUN mkdir oraclexe
    
    #安裝 oraclexe
    RUN c:\oracle\setup.exe /s /f1"C:\oracle\response\OracleXE-Install.iss" /f2"C:\oracle\setup.log"
    
    #移除 container 中的安裝檔(讓 image 保持乾淨) 並移除 listener.ora 使用動態 binding
    RUN ["powershell","remove-item", "c:/oracle , C:/oraclexe/app/oracle/product/11.2.0/server/network/ADMIN/listener.ora","-Recurse"]
    
    #對外使用 1521 port
    EXPOSE 1521
    
    # 預設執行動作，可用來避免 container 自動停止
    CMD [ "ping localhost -t" ]
    ```

## 建置 image

```cmd
docker build -t yowko/winoraclexe C:\oraclexeinstall
```

![2buildimage](https://user-images.githubusercontent.com/3851540/32737005-dd6aeda8-c8d3-11e7-8c44-2402e9153a40.png)

## 啟動 container

```cmd
docker run -d -p 1521:1521 yowko/winoraclexe
```

![3createcontainer](https://user-images.githubusercontent.com/3851540/32737006-dd988eac-c8d3-11e7-824b-a8165891a9de.png)

## 成功連線

![4success](https://user-images.githubusercontent.com/3851540/32737008-ddc5ec08-c8d3-11e7-88b3-bf36edfaf034.png)

## 心得

原本以為很容易的事，結果超複雜，還請教了 Oracle DBA 大大才搞定，原理我不是很清楚，只能簡單說明一下自己遇到的問題

1. 使用 dockerfile 安裝 oracle 後無法正確連線
    * 問題原因是 Service `OracleXETNSListener` 未正確啟動
    * 可透過在 container 內中執行 `sc query OracleXETNSListener` 確認 service 狀態

2. 從 windowsservercore image 建立基本 container 後，手動安裝可正常連線，但 commit 成 image 後所建立的 container 又無法連線
    * 問題原因是 Service `OracleXETNSListener` 未正確啟動，問題與上述相同
    * 可透過在 container 內中執行 `sc query OracleXETNSListener` 確認 service 狀態

3. Service OracleXETNSListener 無法啟動
    * 問題原因是 `C:\oraclexe\app\oracle\product\11.2.0\server\network\ADMIN\listener.ora` 中設定的 host 不正確，應該使用當前主機的 hostname
    * 推測可以是建立 image 就寫入的，並非建立 container 才寫入，造成錯誤
    * 解決方式：可以手動修改 host 即可，可以使用當下的真實 hostname 或是 ip 皆可

4. ORA-12505, TNS:listener does not currently know of SID given in connect descriptor
    * 問題原因是 listener 未正確啟動 (service 正常啟動，但未綁定正確名稱)
    * DBA 大大說一般情況下不需自行綁定，會動態繫結
    * 解決方式：

        * 手動綁定正確名稱

            * 加上以下設定

                ```config
                (SID_DESC =
                (SID_NAME = XE)
                (ORACLE_HOME = C:\oraclexe\app\oracle\product\11.2.0\server)
                )
                ```

            * 完整 listener.ora

                ```config
                SID_LIST_LISTENER =
                (SID_LIST =
                    (SID_DESC =
                    (SID_NAME = PLSExtProc)
                    (ORACLE_HOME = C:\oraclexe\app\oracle\product\11.2.0\server)
                    (PROGRAM = extproc)
                    )
                    (SID_DESC =
                    (SID_NAME = CLRExtProc)
                    (ORACLE_HOME = C:\oraclexe\app\oracle\product\11.2.0\server)
                    (PROGRAM = extproc)
                    )
                    (SID_DESC =
                    (SID_NAME = XE)
                    (ORACLE_HOME = C:\oraclexe\app\oracle\product\11.2.0\server)
                    )
                )
                    
                LISTENER =
                (DESCRIPTION_LIST =
                    (DESCRIPTION =
                    (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1))
                    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
                    )
                )
                
                DEFAULT_SERVICE_LISTENER = (XE)
                ```

        * 移除相關綁定(上面的 dockerfile 做法就是這個概念)

Windows container 版本的 oracle 問題卡了好幾天，經過上百次的嘗試後認定應該是 Oracle 設定問題，果然請教專業的 DBA 是正確的決定，經過一番奮戰跟調整後終於成功建立了 Windows Container 版本的 Oracle XE 真是可喜可賀(泣)，再次感謝強大的 DBA 大大協助

## 參考資訊

1. [Windows 上的 Dockerfile](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile?WT.mc_id=DOP-MVP-5002594)
2. [Powershell - How to use 「remove-item」 with multiple selectors and wildcards?](https://superuser.com/questions/1045045/powershell-how-to-use-remove-item-with-multiple-selectors-and-wildcards)
3. [How to Fix 'ORA-12505, TNS:listener does not currently know of SID given in connect descriptor'](https://chartio.com/resources/tutorials/how-to-fix-ora-12505-tns-listener-does-not-currently-know-of-sid-given-in-connect-descriptor/)
