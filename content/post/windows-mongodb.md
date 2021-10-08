---
title: "如何在 Windows 環境安裝及設定 MongoDB"
date: 2017-08-23T00:35:00+08:00
lastmod: 2021-10-08T00:35:31+08:00
draft: false
tags: ["MongoDB","Windows Service"]
slug: "windows-mongodb"
aliases:
    - /2017/08/windows-mongodb.html
---
## 如何在 Windows 環境安裝及設定 MongoDB

最近專案因為資料變異幅度較大，如果使用傳統關聯式資料庫來儲存資料，table schema 很難設計，所以打算利用 MongoDB schema-free 的特性來儲存資料

原本在開發環境中架設 MongoDB 時 並沒有遇到太多障礙，加上專案時程較趕，就沒有刻意留下筆記，結果在架設其他測試環境時卻卡了好久，反而是當時花不少時間查資料才搞定的 RabbitMQ 憑著詳細筆記快速搞定，所以立馬就來補筆記了 XD

## 安裝 MongoDB

MongoDB 有 `Windows`、`Linux`、`OS X` 和 `Solaris` 四個版本，可以滿足不同環境的需求，安裝檔可以以參考 [下載位置](https://www.mongodb.com/download-center#community)，今天內容是針對 Windows OS 的安裝及設定

* 版本 (2017/08/22)

    1. Windows Server 2008 R2 64-bit and later, with SSL support x64
    2. Windows Server 2008 R2 64-bit and later, without SSL support x64
    3. Windows Server 2008 64-bit, without SSL support x64

    ![1download](https://user-images.githubusercontent.com/3851540/29576183-85f52c4e-8799-11e7-9c92-524b938797b7.png)

* 直接點擊 `.msi` 檔執行安裝

    > 原則一直按 `Next` 就可以完成安裝，如果需要自訂安裝位置，需要使用 `Custom`，預設安裝至 `C:\Program Files\MongoDB\Server\3.4\`

    ![2install1](https://user-images.githubusercontent.com/3851540/29576185-85f6137a-8799-11e7-82fd-3bf164168773.png)

    ![3install2](https://user-images.githubusercontent.com/3851540/29576186-85f75942-8799-11e7-91c6-e6e95abb4e6d.png)

    ![3install3](https://user-images.githubusercontent.com/3851540/29576187-85f93f96-8799-11e7-9f7e-f35778bf801a.png)

    ![4insall4](https://user-images.githubusercontent.com/3851540/29576184-85f56d4e-8799-11e7-9b94-d0c4c24cceef.png)

## 設定並啟動 MongoDB

MongoDB 安裝完成後，僅具備 MongoDB 的核心功能，需要設定 MongoDB 執行環境參數並啟動 MongoDB instance 才能真正讓 MongoDB 擁有實際功能，設定方式有多種，可以依情境所需挑選

* 直接啟動 MongoDB instance

    1. 純 command line 指令設定

        > 必要的參數只有 `dbpath` - 用來存放資料的位置，其他參數皆是可選擇參數

        * 指定 db 存放位置

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "C:\mongodb\db1"
            ```

        * 指定 db 存放位置、 log存放位置、 port 、對外連線 ip、db 儲存模式

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "C:\mongodb\db1" --logpath "C:\mongodb\db1\log.log" --port 30000 --bind_ip 127.0.0.1 --directoryperdb
            ```

        > `mongod` 指令參數可以參考 [mongod.exe](https://docs.mongodb.com/manual/reference/program/mongod.exe)

    2. 使用設定檔

        > 雖然必要參數只有一個，但要完整正確的指定所有參數也不是件容易的事。此時可以透過將參數設定檔來啟動 MongoDB

        * 格式

            > MongoDB 2.6 開始使用 YAML，並支援舊版格式；MongoDB 2.4 之前則沒有特定格式，只要符合 `{參數}={設定值}` 即可

            * MongoDB 2.4 之前版本，詳細內容可以參考 [Configuration File Options](https://docs.mongodb.com/v2.4/reference/configuration-options/)

                ```cmd
                port=27017
                dbpath=c:\mongodb\db3
                directoryperdb=true 
                logpath=c:\mongodb\db3\log.log 
                logappend=true 
                bind_ip=127.0.0.1
                ```

            * MongoDB 2.6 YAML 版本，詳細內容可以參考 [Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options)

                ```yml
                systemLog:
                    destination: file
                    path: "C:\\mongodb\\db3\\log.log"
                    logAppend: true
                storage:
                   dbPath: "C:\\mongodb\\db3"
                   directoryPerDB: true
                net:
                   port: 27017
                   bindIp: 127.0.0.1
                ```

        * 指令

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\db3\mongo.conf"
            ```

* Windows Service

    > 透過直接建立 MongoDB instance 的方式，會讓執行指令的 command prompt 一直處於執行中的狀態，一旦關閉， MongoDB instance 也會被關閉，不適合長久型服務，而透過安裝成 Windows Service 就沒有這個問題了

    1. 純 command line 指令設定

        ```cmd
        "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "C:\mongodb\db1" --logpath "C:\mongodb\db1\log.log" --port 30000 --bind_ip 127.0.0.1 --directoryperdb --serviceName MongoDB --serviceDisplayName MongoDB30000 --install
        ```

        > 加上 `serviceName`、`serviceDisplayName`、及 `install`

    2. mongod 指令搭配設定檔

        * MongoDB 2.4 之前版本

            ```config
            port=27017
            dbpath=c:\mongodb\db3
            directoryperdb=true 
            logpath=c:\mongodb\db3\log.log 
            logappend=true 
            bind_ip=127.0.0.1
            ```

            > 這個版本無法直接透過 config 檔案指定 service 相關設定，需與指令搭配使用

            ```cmd
            C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\db3\mongo.conf" --serviceName MongoDB27017 --serviceDisplayName MongoDB27017 --install
            ```

        * MongoDB 2.6 YAML 版本

            ```yaml
            systemLog:
                destination: file
                path: "C:\\mongodb\\db3\\log.log"
                logAppend: true
            storage:
                dbPath: "C:\\mongodb\\db3"
                directoryPerDB: true
            net:
                port: 27017
                bindIp: 127.0.0.1
            processManagement:
                windowsService:
                    serviceName: MongoDB27017
                    displayName: MongoDB27017
            ```

            ```cmd
            "C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\mongodb\db3\mongo.conf" --install
            ```

            > `serviceName`、`displayName` 由設定檔指定

    3. 使用 sc 指令

        ```cmd
        sc.exe create MongoDB binPath= "\"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe\" --service --config=\"C:\mongodb\db3\mongo.conf\"" DisplayName= "MongoDB" start= "auto"
        ```

        * 啟動

            > `net start MongoDB`

        * 停止

            > `net stop MongoDB`

        * 刪除

            > `sc.exe delete MongoDB`

## 心得

仔細想想好像每種安裝方式之前都用過，當然不是為了測試，主要是因為不暸解、不熟悉而沒有固定的套路才會這樣

經過這次忘記安裝步驟的慘痛經驗，再次提醒著我應該要勤作筆記，不為別的就為了拯救記憶力退化的自己

回到 MongoDB 本身，設定方式滿多元的，安裝流程跟設定都算簡單(也就是這樣才覺得不用筆記)，設定的相容性也做得不錯，果然不愧是牌子老口碑好

## 參考資訊

1. [MongoDB 下載位置](https://www.mongodb.com/download-center#community)
2. [mongod.exe](https://docs.mongodb.com/manual/reference/program/mongod.exe)
3. [MongoDB 2.4 Configuration File[ Options](https://docs.mongodb.com/v2.4/reference/configuration-options/)
4. [MongoDB 2.6 Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options)
5. [Install MongoDB Community Edition on Windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
