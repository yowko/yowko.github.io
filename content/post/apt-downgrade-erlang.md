---
title: "erlang 降版的步驟"
date: 2023-12-26T00:30:00+08:00
lastmod: 2023-12-26T00:30:31+08:00
draft: false
tags: ["erlang","apt","csharp","linux","ubuntu","rabbitmq"]
slug: "apt-downgrade-erlang"
---

## erlang 降版的步驟

這幾天有台與 partner 介接用的 server 異常，造成 rabbitmq 無法連線，團隊在這類的 service 安裝腳本一直透過 ansible 管理，安裝上很順利，只是安裝後 .NET application 卻無法成功連線 (提示可能是 auth fail)，但透過 rabbitmqadmin 測試又一切正常，原本以為是帳號權限、vhost 或是 topic permission 設定錯誤，最後使用 amidn 仍無法成功，最後才發現是 erlang 版本的問題，這時候才想起來之前也遇到相同問題：[APT 安裝指定版本套件](apt-specific-version/)

檢查了 ansible，看似沒問題，推測可能是 ansible 原本設定 `preferences` 中的版本已經不存在，所以才會安裝最新版

有幾組 server 都有相同問題，所以快速紀錄一下降版的步驟，以免下次再遇到相同問題時，又要花時間回憶XD

## 基本環境說明

1. Linux (ubuntu 22.04)
2. erlang-nox 1:25.3.2.2-1
3. rabbitmq-server 3.11.28-1
4. .NET 錯誤訊息

    ```log
    RabbitMQ.Client.Exceptions.OperationInterruptedException: The AMQP operation was interrupted: AMQP close-reason, initiated by Library, code=0, text='End of stream', classId=0, methodId=0, cause=System.IO.EndOfStreamException: Reached the end of the stream. Possible authentication failure.
        at RabbitMQ.Client.Impl.InboundFrame.ReadFrom(Stream reader, Byte[] frameHeaderBuffer, ArrayPool`1 pool, UInt32 maxMessageSize)
        at RabbitMQ.Client.Impl.SocketFrameHandler.ReadFrame()
        at RabbitMQ.Client.Framing.Impl.Connection.MainLoopIteration()
        at RabbitMQ.Client.Framing.Impl.Connection.MainLoop()
        at RabbitMQ.Client.Impl.SimpleBlockingRpcContinuation.GetReply(TimeSpan timeout)
        at RabbitMQ.Client.Impl.ModelBase.ModelRpc(MethodBase method, ContentHeaderBase header, Byte[] body)
        at RabbitMQ.Client.Framing.Impl.Model._Private_ChannelOpen(String outOfBand)
        at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.CreateNonRecoveringModel()
        at RabbitMQ.Client.Framing.Impl.AutorecoveringConnection.CreateModel()
   ```

## 設定步驟

1. 修改 preferences：`/etc/apt/preferences.d/erlang`

    ```txt
    Package: erlang*
    Pin: version 1:25.3.2.8-1
    Pin-Priority: 1000
    ```

2. 套用 preferences (option)

    > 我沒套用，依然生效，提供給有需要的人參考

    ```bash
    sudo apt policy
    ```

3. 移除安裝錯誤的 26 版 erlang-*

    ```bash
    sudo apt remove -y erlang-*
    ```

    - 可以透過下面語法先確認安裝的 erlang 版本

        ```bash
        apt list --install | grep erlang
        ```

4. 更新套件清單

    ```bash
    sudo apt update -y
    ```

5. 安裝指定版本 erlang

    > 這邊以 `erlang-nox 25.3.2.28` 為例

   ```bash
   sudo apt install erlang-nox=1:25.3.2.8-1 -y 
   ```

    - 先列所有可用版本，避免又選到已不存在的
  
        ```bash
        sudo apt list erlang-nox -a
        ```

6. 重新安裝 rabbitmq-server

    > 因為前面 remove erlang 時， rabbitmq-server 有相依，所以會一併被移隱；這邊以 `rabbitmq-server 3.11.28-1` 為例

    ```bash
    sudo apt install rabbitmq-server=3.11.28-1 -y
    ```

## 心得

.NET library 拋出的訊息有些誤導，造成一開始花了不少時間在檢查 rabbitmq 相關設定，最後才發現是 erlang 版本的問題

另外我也搞不懂 .NET library 為什麼不支援 26 版的 erlang，我看 release note RC 已經是 `February 15, 2023` 的事了，或者可能是我沒有正確設定造成的XD  有空再來確認相關 issue 囉

## 參考資訊

1. [APT 安裝指定版本套件](apt-specific-version/)
