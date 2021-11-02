---
title: "在 Windows 7、Winodws 10、Windows Server 2016 上安裝 RabbitMQ"
date: 2017-05-16T23:00:00+08:00
lastmod: 2021-11-02T15:02:51+08:00
draft: false
tags: ["RabbitMQ","Windows 10","Windows Server 2016"]
slug: "install-rabbitmq-on-windows7-windows10-windows2016"
aliases:
    - /2017/05/install-rabbitmq-on-windows7-windows10-windows2016.html
---
## 在 Windows 7、Winodws 10、Windows Server 2016 上安裝 RabbitMQ

公司既有專案在實際 production 環境上有效能瓶頸，雖然有 load balance 搭配 cluster 架構，但 load balance 設定方式使得 cluster 中的各 server 間 流量分配不均，所以想透過 message queue 來分配工作的想法，原本屬意效能極佳的 kafka，但同事因為訊息健全性建議使用相對成熟的 RabbitMQ

RabbitMQ 是一款基於 AMQP - Advanced Message Queuing Protocol（訊息佇列協定），使用 Erlang 開發的 open ource 訊息佇列系統。是套優秀的訊息佇列系統，主要由兩部分組成：服務端和客戶端，客戶端支持多種語言，如：.Net、JAVA、Erlang 等。為了要比對兩者差異，當然就得來架設環境囉

## Windows 7

1. 安裝 Erlang

    * [Erlang 官網 download](http://www.erlang.org/downloads)
    * 2017/05/15 是 OTP 19.3 版
    * 注意 32-bit/64bit 版本

        ![1download](https://cloud.githubusercontent.com/assets/3851540/26097959/e486e290-3a58-11e7-848f-bf99671251ff.png)

2. 設定 Erlang 環境變數
    * `Computer` 上按滑鼠右鍵 --> Properties

        ![2properties](https://cloud.githubusercontent.com/assets/3851540/26097960/e4895354-3a58-11e7-9a60-8005344eba0f.png)

    * 點擊 `Advenced system settings` --> 點擊 `Environment Varialbes`

        ![3envvariable](https://cloud.githubusercontent.com/assets/3851540/26097961/e48aa6b4-3a58-11e7-9d64-6b6d7c44e547.png)

    * 新增系統變數 (New..)

        ![4new](https://cloud.githubusercontent.com/assets/3851540/26097962/e49d5b56-3a58-11e7-9cc6-7be40a182e0d.png)

    * Variable name --> `ERL_HOME`; Variable value --> `Erlang 安裝路徑`

        ![5erlpath](https://cloud.githubusercontent.com/assets/3851540/26097963/e4ab8de8-3a58-11e7-8994-6045198086f3.png)

    * 修改 `Path` 變數 (點選 `Path` --> Edit)

        ![6editpath](https://cloud.githubusercontent.com/assets/3851540/26097964/e4b1d43c-3a58-11e7-8d24-336b07ff8b76.png)

    * 將 `%ERL_HOME%\bin;` 加到 `Variable value` 最後

        ![7addpath](https://cloud.githubusercontent.com/assets/3851540/26097977/e5a53ece-3a58-11e7-882d-61bc62dfed38.png)

3. 測試是否成功安裝 Erlang

    * 開啟 command prompt or Windows PowerShell 執行 `erl` (`ctrl+c` 可以跳出)

        ![8erl](https://cloud.githubusercontent.com/assets/3851540/26097965/e4b279c8-3a58-11e7-8def-3f417cca0150.png)

        ![9erl](https://cloud.githubusercontent.com/assets/3851540/26097970/e4f58c72-3a58-11e7-8494-b2ea9d1e64dd.png)

4. 安裝 RabbitMQ

    * [http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/install-windows.html)，可選擇從 RabbitMQ 下載或是從 GitHub 下載
    * [Installing on Windows Guideline](http://www.rabbitmq.com/install-windows.html)
    * 安裝時預設會註冊成為 service

        ![10installrabbitmq](https://cloud.githubusercontent.com/assets/3851540/26097966/e4c1ec1e-3a58-11e7-92cc-aacd8d918c90.png)

5. 確認是否安裝並順利啟動

    * 開啟 command prompt 輸入 `sc query "RabbitMQ"` (state - RUNNING 代表執行中)

        ![11servicestate](https://cloud.githubusercontent.com/assets/3851540/26097980/e74c4b0a-3a58-11e7-8d10-f64784d24f67.png)

6. 安裝 RabbitMQ 管理套件

    * 開啟 `RabbitMQ Command Prompt(sbin dir)`(安裝 RabbitMQ 時順便安裝的 command prompt) 執行 `rabbitmq-plugins.bat enable rabbitmq_management`
    * 也可以開啟一般的 command prompt 進到 `{RabbitMQ 安裝路徑}/sbin` 下執行 `rabbitmq-plugins.bat enable rabbitmq_management`

        ![12mqmanageplugin](https://cloud.githubusercontent.com/assets/3851540/26097967/e4e0a3d4-3a58-11e7-9726-d43758f2cb9f.png)

7. 登入RabbitMQ 管理介面

    * 成功安裝後，可以使用 `http://localhost:15672/` 開啟管理介面

        ![13manageui](https://cloud.githubusercontent.com/assets/3851540/26097968/e4e18ab0-3a58-11e7-9e18-c3dddf31288b.png)

    * 預設帳號密碼是：`guest`/`guest`

        ![14managelogin](https://cloud.githubusercontent.com/assets/3851540/26097969/e4ea9006-3a58-11e7-8f09-1ea439b490de.png)

## Windows 10

1. 安裝 Erlang
    * [Erlang 官網 download](http://www.erlang.org/downloads)
    * 2017/05/15 是 OTP 19.3 版
    * 注意 32-bit/64bit 版本

        ![1download](https://cloud.githubusercontent.com/assets/3851540/26097959/e486e290-3a58-11e7-848f-bf99671251ff.png)

2. 設定 Erlang 環境變數

    * 按下鍵盤 `windows` 圖示鍵 or 放大鏡，搜尋 `environment`

        ![14win10](https://cloud.githubusercontent.com/assets/3851540/26097972/e51bce64-3a58-11e7-8cbc-85e726e7b66c.png)

    * 點擊 `Edit the system environment`

        ![15editenv](https://cloud.githubusercontent.com/assets/3851540/26097971/e5113d82-3a58-11e7-9fdb-8e02d1a3ade5.png)

    * 點擊 `Environment Varialbes`

        ![16envvar](https://cloud.githubusercontent.com/assets/3851540/26097973/e51c99e8-3a58-11e7-9920-ed8bbea2f8c6.png)

    * 新增 System variables (New..)

        ![17newvar](https://cloud.githubusercontent.com/assets/3851540/26097976/e53f84da-3a58-11e7-9a0f-6a82b59fcfb6.png)

    * Variable name --> `ERL_HOME`; Variable value --> `Erlang 安裝路徑`

        ![18varsetting](https://cloud.githubusercontent.com/assets/3851540/26097975/e53839a0-3a58-11e7-8639-b997751006b9.png)

    * 修改 `Path` 變數 (點選 `Path` --> Edit)

        ![19editpath](https://cloud.githubusercontent.com/assets/3851540/26097955/e442a2e2-3a58-11e7-9742-fce575a14264.png)

    * New --> `%ERL_HOME%\bin;`

        ![20addtopath](https://cloud.githubusercontent.com/assets/3851540/26097956/e47439e2-3a58-11e7-89c7-56561c115db6.png)

3. 測試是否成功安裝 Erlang

    * 開啟 command prompt or Windows PowerShell 執行 `erl` (`ctrl+c` 可以跳出)

        ![8erl](https://cloud.githubusercontent.com/assets/3851540/26097965/e4b279c8-3a58-11e7-8def-3f417cca0150.png)

        ![9erl](https://cloud.githubusercontent.com/assets/3851540/26097970/e4f58c72-3a58-11e7-8494-b2ea9d1e64dd.png)

4. 安裝 RabbitMQ

    * [http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/install-windows.html)，可選擇從 RabbitMQ 下載或是從 GitHub 下載
    * [Installing on Windows Guideline](http://www.rabbitmq.com/install-windows.html)
    * 安裝時預設會註冊成為 service

         ![10installrabbitmq](https://cloud.githubusercontent.com/assets/3851540/26097966/e4c1ec1e-3a58-11e7-92cc-aacd8d918c90.png)

5. 確認是否安裝並順利啟動

    * 開啟 command prompt 輸入 `sc query "RabbitMQ"` (state - RUNNING 代表執行中)

        ![11servicestate](https://cloud.githubusercontent.com/assets/3851540/26097980/e74c4b0a-3a58-11e7-8d10-f64784d24f67.png)

6. 安裝 RabbitMQ 管理套件

    * 開啟 `RabbitMQ Command Prompt(sbin dir)`(安裝 RabbitMQ 時順便安裝的 command prompt) 執行 `rabbitmq-plugins.bat enable rabbitmq_management`
    * 也可以開啟一般的 command prompt 進到 `{RabbitMQ 安裝路徑}/sbin` 下執行 `rabbitmq-plugins.bat enable rabbitmq_management`

        ![12mqmanageplugin](https://cloud.githubusercontent.com/assets/3851540/26097967/e4e0a3d4-3a58-11e7-9726-d43758f2cb9f.png)

    * 如果出現錯誤訊息

        * 錯誤訊息

            ```log
            Plugin configuration unchanged.
            
            Applying plugin configuration to rabbit@SC-201607101239... failed.
                * Could not contact node rabbit@SC-201607101239.
                    Changes will take effect at broker restart.
                * Options: --online  - fail if broker cannot be contacted.
                            --offline - do not try to contact broker.
            ```

        * 錯誤截圖

            ![23manageerror](https://user-images.githubusercontent.com/3851540/37697639-e5518f6e-2d18-11e8-8235-bbf412f892a2.png)

        * 解決方式可以參考 [Windows搭建測試RabbitMq遇到的問題](https://my.oschina.net/wenjinglian/blog/729699)、[Run 「rabbitmq-plugins enable rabbitmq_management」 .. failed](http://stackoverflow.com/questions/28258392/rabbitmq-has-nodedown-error/34538688#34538688)

            * 將 `%SystemRoot%\.erlang.cookie` 複製到 `%HOMEDRIVE%%HOMEPATH%\.erlang.cookie` ， 並重新執行 `rabbitmq-plugins.bat enable rabbitmq_management` 即可

7. 登入RabbitMQ 管理介面

    * 成功安裝後，可以使用 `http://localhost:15672/` 開啟管理介面

        ![13manageui](https://cloud.githubusercontent.com/assets/3851540/26097968/e4e18ab0-3a58-11e7-9e18-c3dddf31288b.png)

    * 預設帳號密碼是：`guest`/`guest`

        ![14managelogin](https://cloud.githubusercontent.com/assets/3851540/26097969/e4ea9006-3a58-11e7-8f09-1ea439b490de.png)

## Windows Server 2016

1. 安裝 Erlang
    * [Erlang 官網 download](http://www.erlang.org/downloads)
    * 2017/05/15 是 OTP 19.3 版
    * 注意 32-bit/64bit 版本

        ![1download](https://cloud.githubusercontent.com/assets/3851540/26097959/e486e290-3a58-11e7-848f-bf99671251ff.png)

2. 設定 Erlang 環境變數
    * 按下鍵盤 `windows` 圖示鍵 or 放大鏡，搜尋 `environment`

        ![21env](https://cloud.githubusercontent.com/assets/3851540/26097957/e4851c4e-3a58-11e7-809c-d79b7d83977d.png)

    * 點擊 `Edit the system environment`

        ![22endedit](https://cloud.githubusercontent.com/assets/3851540/26097958/e4863f16-3a58-11e7-97da-741a3ba268f0.png)

    * 點擊 `Environment Varialbes`

        ![16envvar](https://cloud.githubusercontent.com/assets/3851540/26097973/e51c99e8-3a58-11e7-9920-ed8bbea2f8c6.png)

    * 新增 System variables (New..)

        ![17newvar](https://cloud.githubusercontent.com/assets/3851540/26097976/e53f84da-3a58-11e7-9a0f-6a82b59fcfb6.png)

    * Variable name --> `ERL_HOME`; Variable value --> `Erlang 安裝路徑`

        ![18varsetting](https://cloud.githubusercontent.com/assets/3851540/26097975/e53839a0-3a58-11e7-8639-b997751006b9.png)

    * 修改 `Path` 變數 (點選 `Path` --> Edit)

        ![19editpath](https://cloud.githubusercontent.com/assets/3851540/26097955/e442a2e2-3a58-11e7-9742-fce575a14264.png)

    * New --> `%ERL_HOME%\bin;`

        ![20addtopath](https://cloud.githubusercontent.com/assets/3851540/26097956/e47439e2-3a58-11e7-89c7-56561c115db6.png)

3. 測試是否成功安裝 Erlang

    * 開啟 command prompt or Windows PowerShell 執行 `erl` (`ctrl+c` 可以跳出)

        ![8erl](https://cloud.githubusercontent.com/assets/3851540/26097965/e4b279c8-3a58-11e7-8def-3f417cca0150.png)

        ![9erl](https://cloud.githubusercontent.com/assets/3851540/26097970/e4f58c72-3a58-11e7-8494-b2ea9d1e64dd.png)

4. 安裝 RabbitMQ
    * [http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/install-windows.html)，可選擇從 RabbitMQ 下載或是從 GitHub 下載
    * [Installing on Windows Guideline](http://www.rabbitmq.com/install-windows.html)
    * 安裝時預設會註冊成為 service

        ![10installrabbitmq](https://cloud.githubusercontent.com/assets/3851540/26097966/e4c1ec1e-3a58-11e7-92cc-aacd8d918c90.png)

5. 確認是否安裝並順利啟動

    * 開啟 command prompt 輸入 `sc query "RabbitMQ"` (state - RUNNING 代表執行中)

        ![11servicestate](https://cloud.githubusercontent.com/assets/3851540/26097980/e74c4b0a-3a58-11e7-8d10-f64784d24f67.png)

6. 安裝 RabbitMQ 管理套件
    * 開啟 `RabbitMQ Command Prompt(sbin dir)`(安裝 RabbitMQ 時順便安裝的 command prompt) 執行 `rabbitmq-plugins.bat enable rabbitmq_management`
    * 也可以開啟一般的 command prompt 進到 `{RabbitMQ 安裝路徑}/sbin` 下執行 `rabbitmq-plugins.bat enable rabbitmq_management`

        ![12mqmanageplugin](https://cloud.githubusercontent.com/assets/3851540/26097967/e4e0a3d4-3a58-11e7-9726-d43758f2cb9f.png)

7. 登入RabbitMQ 管理介面
    * 成功安裝後，可以使用 `http://localhost:15672/` 開啟管理介面

        ![13manageui](https://cloud.githubusercontent.com/assets/3851540/26097968/e4e18ab0-3a58-11e7-9e18-c3dddf31288b.png)

    * 預設帳號密碼是：`guest`/`guest`

        ![14managelogin](https://cloud.githubusercontent.com/assets/3851540/26097969/e4ea9006-3a58-11e7-8f09-1ea439b490de.png)

## 參考資訊

1. [window7下面rabbitMQ安裝配置過程詳解](http://mp.weixin.qq.com/s/WxdJhVLo6XhgCrYbdHqplg)
2. [Installing on Windows Guideline](http://www.rabbitmq.com/install-windows.html)
3. [Windows搭建測試RabbitMq遇到的問題](https://my.oschina.net/wenjinglian/blog/729699)
4. [Run 「rabbitmq-plugins enable rabbitmq_management」 .. failed](http://stackoverflow.com/questions/28258392/rabbitmq-has-nodedown-error/34538688#34538688)
