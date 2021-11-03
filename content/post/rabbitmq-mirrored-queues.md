---
title: "設定 RabbitMQ 的 Mirrored Queues - 讓 Queue 內容可以在多組 RabbitMQ 同步"
date: 2017-08-11T23:35:00+08:00
lastmod: 2021-10-28T23:35:08+08:00
draft: false
tags: ["RabbitMQ"]
slug: "rabbitmq-mirrored-queues"
aliases:
    - /2017/08/rabbitmq-mirrored-queues.html
---
## 設定 RabbitMQ 的 Mirrored Queues - 讓 Queue 內容可以在多組 RabbitMQ 同步

之前文章 [如何在 Windwos 上設定 RabbitMQ Cluster](/windwos-rabbitmq-cluster) 提到：一個完善的系統一定需要確保系統中的各個 component 是具備 HA - High availability 的特性，而 message queue 本身因為角色特性的關係， HA 的重要性又系統中其他 component 來得重要許多，畢竟所有資訊都會有經過 message queue，如果 message queue 掛了，也代表著整個系統都掛了，所以透過 RabbitMQ 的 Cluster 機制來確保 RabbitMQ 可以有多個 node 進行服務

但只有 cluster 還不夠，因為 cluster 只能確保 RabbitMQ 不會因為只有一個 node 無法提供服務就造成整個 RabbitMQ 都失效，不過因為個別 queue 及相關的 message 還是存於個別的 node 之上，如果其中一個 node 出現問題，存在這個 node 上的 queue 跟內容都會遺失。這不是可靠系統該出現的，所以就有今天主題 - Mirrored Queues 的出現

Mirrored Queues 會在多個 node 中互相複製及儲存 queue 及其內容，運作方式是由一個 master 搭配一個(以上)的 slave 組成，除了 publish 之外的所有動作都會交給 master 處理，並由 master 通知 slave 進行資料同步。在 master 發生異常，將由 RabbitMQ 將年資最高的 slave 提昇為 master 接替服務

為了提供 RabbitMQ 資料面的 HA，我們就來看看該如何設定 Mirrored Queues 吧

## 基本環境

最重要的前提就是必需要先設定好 RabbitMQ Cluster ，詳細設定可以參考 [如何在 Windwos 上設定 RabbitMQ Cluster](/windwos-rabbitmq-cluster)，這邊就不重複贅述

## 設定 Mirrored Queues

設定 Mirrored Queues 有三種方式，可以依需求選擇

* 參數說明
  * `policy_name`

        > 識別用名稱

  * `queue_name_pattern`

        > 需要 mirror 的 queue name，格式是 regular expression

    * "^Yowko_" 代表 queue 名稱以 `Yowko_` 開頭就進行 mirror

1. 使用 rabbitmqctl 指令

    > 指令在 linux 與 windows 行為不同，以下會列出 linux 指令寫法，但我只使用過 windows 版本

    * 指令 pattern
        * linux

            > `rabbitmqctl set_policy {policy_name} "{queue_name_pattern}" '{"ha-mode":"all"}'`

        * Windows

            > `rabbitmqctl set_policy {policy_name} "{queue_name_pattern}" "{""ha-mode"":""all""}"`

    * 範例

        > `rabbitmqctl set_policy Yowko_all "^Yowko_" "{""ha-mode"":""all""}"`

    * 實際使用

        ![1rabbit](https://user-images.githubusercontent.com/3851540/29209604-fa48508e-7ec1-11e7-82c2-3fc580d40c27.png)

2. 使用 HTTP API
    * 使用 `put` 方法
    * policy 名稱在 url 指定
        * URL 格式

            > `http://{username}:{password}@{RabbitMQ_Host}:{RabbitMQ_port}/api/policies/%2f/{policy_Name}`

        * 範例

            > `http://yowko:pass.123@localhost:15672/api/policies/%2f/Yowko_all`

    * policy 細節使用 json 格式放在 request body 中
        * body 格式

                {"pattern":"{queue_name_pattern}", "definition":{"ha-mode":"all"}}

        * 範例

                {"pattern":"^Yowko_", "definition":{"ha-mode":"all"}}

    * 實際使用

        ![2api](https://user-images.githubusercontent.com/3851540/29209603-fa45a35c-7ec1-11e7-9d2f-bed429f52d79.png)

3. 使用 Web UI
    * 登入 Web UI --> Admin --> Policies --> 填寫相關資訊

        ![3webui](https://user-images.githubusercontent.com/3851540/29209599-fa3576a8-7ec1-11e7-81f3-3175d6876e88.png)

* 完成設定 policy， Web UI 就會看到相關資訊

    ![4WEBUIRESULT](https://user-images.githubusercontent.com/3851540/29209602-fa430fac-7ec1-11e7-9fa5-1b183a43c8db.png)

## 實際效果

1. 未 mirror

    ![5non-mirror](https://user-images.githubusercontent.com/3851540/29209600-fa3fbd8e-7ec1-11e7-9f8e-8367db7ea07c.png)

2. mirror 後

    ![6mirrored](https://user-images.githubusercontent.com/3851540/29209601-fa4075ee-7ec1-11e7-82a6-d44c2c86e571.png)

## 心得

經過設定 Mirrored Queues 後就可以進一步將 queue 保存至多個 node，避免單一 RabbitMQ node 出現問題而造成資料遺失，讓系統的 HA 往上提高了一個等級

RabbitMQ server 端的設定大致已完備，但 client 的 publisher 與 consumer 也需要針對 RabbitMQ Cluster 來處理才真的能讓系統可以在 RabbitMQ 執行 failover 時無痛轉換，這個部份就留待下篇來分享

## 參考資訊

1. [如何在 Windwos 上設定 RabbitMQ Cluster](/windwos-rabbitmq-cluster)
2. [RabbitMQ官網文檔翻譯 -- Highly Available Queue](https://my.oschina.net/moooofly/blog/94113)
3. [Highly Available (Mirrored) Queues](https://www.rabbitmq.com/ha.html)
4. [Rabbitmq HTTP API request UnAuthorized](https://stackoverflow.com/questions/10647631/rabbitmq-http-api-request-unauthorized)
