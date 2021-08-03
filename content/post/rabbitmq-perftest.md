---
title: "使用 PerfTest 進行 RabbitMQ 效能測試"
date: 2021-08-03T21:30:00+08:00
lastmod: 2021-08-03T21:30:00+08:00
draft: false
tags: ["RabbitMQ","benchmark"]
slug: "rabbitmq-perftest"
---

## 使用 PerfTest 進行 RabbitMQ 效能測試

之前筆記 [在 CentOS7 上建立 RabbitMQ Cluster](/rabbitmq-cluster-centos7)
 提到打算透過針對 RabbitMQ 的效能測試來確認使用的硬體規格，今天就來紀錄該怎麼 使用 PerfTest 進行 RabbitMQ 效能測試

為了專注於 PerfTest 使用紀錄，所以 RabbitMQ 的部份就透過 docker image 來快速建立，這樣一來可以想見效能比不上實體服務，如果想較忠實呈現效能數據建議依實際格規來建立 RabbitMQ 環境再做測試

## 基本環境說明

1. macOS Big Sur 11.5.1
2. .NET Core SDK 5.0.202
3. docker desktop 3.3.0(26916)
4. docker images

    - rabbitmq:3.8
    - pivotalrabbitmq/perf-test:2.15.0

5. 測試用 RabbitMQ

    > 需要 RabbitMQ cluster，可以參考之前筆記 [在 CentOS7 上建立 RabbitMQ Cluster](/rabbitmq-cluster-centos7)

    ```bash
    docker run -d --rm --name rabbitmq -p 5672:5672 rabbitmq:3.8
    ```

## 使用方式

1. 安裝方式

    - 使用 binary

        > 可以直接從 [GitHub releases](https://github.com/rabbitmq/rabbitmq-perf-test/releases) 下載

    - 使用 docker

        > 可以從 [dockerhub](https://hub.docker.com/r/pivotalrabbitmq/perf-test/) 下載 docker image

2. 參數說明

    > 大多數參數都沒用過，也有不少是看完不知道用法的，但既然都看了順便紀錄一下，也許下次看到時會比較有想法

    參數|說明
    ---|---
    -?,--help                                     |說明文件
    -a,--autoack                                  |自動傳送 ack
    -A,--multi-ack-every `<arg>`                  |指定訊息累積量再一次 ack
    -ad,--auto-delete `<arg>`                     |queue 是否要自動刪除 (預設值： `true`)
    -b,--heartbeat `<arg>`                        |heartbeat 間隔
    -B,--body `<arg>`                             |使用指定的內容做為訊息，用 `逗號` 分隔多個檔案
    -bc,--body-count `<arg>`                      |預先產生的訊息內容數量；與 `--json-body` 一起使用 (預設值： `100`)
    -bfc,--body-field-count `<arg>`               |預先產生內容的欄位與數值；與 `--json-body` 一起使用 (預設值： `100`)
    -c,--confirm `<arg>`                          |未確認訊息的最大發布數量
    -C,--pmessages `<arg>`                        |producer 訊息數量
    -ca,--consumer-args `<arg>`                   |consumer 參數使用 key/values，用 `逗號` 分隔， e.g. x-priority=10
    -cri,--connection-recovery-interval `<arg>`   |連線復原間隔秒數 (預設值：`5`)；支援在兩個設定值間隨機進行嘗試(e.g. 30-60)
    -ct,--confirm-timeout `<arg>`                 |未確認發行訊息的等待時間(以 `秒` 為單位)
    -ctp,--consumers-thread-pools `<arg>`         | 所有 conumer 用的 thread pools 數量 (預設值：每個 consumer 使用一個 thread pool)
    -d,--id `<arg>`                               |test ID
    -D,--cmessages `<arg>`                        |consumer 訊息數量
    -dcr,--disable-connection-recovery            |停用自動連線復原
    -e,--exchange `<arg>`                         |exchange 名稱
    -E,--exclusive                                |排除 server-named queues
    -env,--environment-variables                  |顯示使用的環境變數
    -f,--flag `<arg>`                             | 訊息 flag(s)，允許使用 `persistent` 與 `mandatory`，可以使用多次來指定不同值
    -h,--uri `<arg>`                              |connection URI
    -H,--uris `<arg>`                             |connection URIs (`逗號` 分隔)
    -hst,--heartbeat-sender-threads `<arg>`       |producers and consumers heartbeat 用的 threads 數量
    -i,--interval `<arg>`                         |取樣間隔 (以 `秒` 為單位)
    -jb,--json-body                               |產生隨機的 JSON 做為訊息內容.與 `--size` 一併使用
    -k,--routing-key `<arg>`                      |routing key
    -K,--random-routing-key                       |每則訊息都使用隨機的 routing key
    -l,--legacy-metrics                           |顯示傳統 metrics(min/avg/max latency)
    -L,--consumer-latency `<arg>`                 |consumer latency (以 `毫秒` 為單位)
    -m,--ptxsize `<arg>`                          |producer tx 大小
    -M,--framemax `<arg>`                         |frame max
    -mh,--metrics-help                            |顯示 metrics 用量
    -mp,--message-properties `<arg>`              |key/value 表示的訊息屬性，以 `逗號` 分隔，e.g. priority=5
    -ms,--use-millis                              |latency 是否被蒐集(以 `毫秒` 為單位)，預設值：`false`.一般是 consumer 與 producer 在不同 server 上才啟用
    -n,--ctxsize `<arg>`                          |consumer tx 大小
    -na,--nack                                    |nack 並且 requeue 訊息
    -niot,--nio-threads `<arg>`                   |NIO threads 數量
    -niotp,--nio-thread-pool `<arg>`              |NIO thread pool 數量，應該大於 NIO threads 數量
    -o,--output-file `<arg>`                      |將衡量結果寫至指定檔案
    -p,--predeclared                              |允許使用預先定義的物件
    -P,--publishing-interval `<arg>`              |以秒為單位的發布間隔（與生產者速率限制相反)
    -pi,--polling-interval `<arg>`                |使用 basic.get 輪詢前等待的時間，以毫秒為單位，預設值： `0`
    -po,--polling                                 |使用 basic.get 消費消息，不要在實際應用程式中使用
    -prsd,--producer-random-start-delay `<arg>`   |以秒為單位的最大隨機延遲啟動 producers
    -pst,--producer-scheduler-threads `<arg>`     |在使用 `--publishing-interval` 指定 threads 數量
    -q,--qos `<arg>`                              |consumer prefetch 數量
    -Q,--global-qos `<arg>`                       |channel prefetch 數量
    -qa,--queue-args `<arg>`                      |以 key/value 表示的, 以 `逗號` 分隔，e.g. x-max-length=10
    -qf,--queue-file `<arg>`                      |用來指定 queue name 的檔案
    -qp,--queue-pattern `<arg>`                   |用於建立 queue 的 queue name pattern 順序
    -qpf,--queue-pattern-from `<arg>`             |queue name pattern 範圍開始 (包含)
    -qpt,--queue-pattern-to `<arg>`               |queue name pattern 範圍結束 (包含)
    -qq,--quorum-queue                            |建立 quorum queue(s)
    -r,--rate `<arg>`                             |producer rate 上限
    -R,--consumer-rate `<arg>`                    |consumer rate 上限
    -rkcs,--routing-key-cache-size `<arg>`        |隨機 routing keys cache 的大小，看 `--random-routing-key`
    -S,--slow-start                               |延遲啟動 consumer (每個 consumer 延遲一秒)
    -s,--size `<arg>`                             |以 bytes 為單位的訊息大小
    -sb,--skip-binding-queues                     |不要 bind queue 到 exchange
    -se,--sasl-external                           | 使用 SASL 外部身份驗證，預設值：`false`。如果使用帶有 rabbitmq_auth_mechanism_ssl 套件的客戶端證書身份驗證，則設置為 true。
    -sni,--server-name-indication `<arg>`         |標記 TLS 參數的 server name ，以逗號分隔
    -sst,--servers-startup-timeout `<arg>`        |以秒為單位的啟動超時（以防服務器在運行開始時不可用）。如果服務器不可用，默認是立即失敗。
    -st,--shutdown-timeout `<arg>`                |終止的 timeout，預設值：`5` 秒
    -sul,--servers-up-limit `<arg>`               |開始執行之前所需的可用服務器數。與 `--servers-start-timeout` 結合使用。預設值是從 `--uri` 或 `--uris` 推導出來的。
    -t,--type `<arg>`                             |exchange 類型
    -T,--body-content-type `<arg>`                |內容 content-type
    -u,--queue `<arg>`                            |queue name
    -udsc,--use-default-ssl-context               |使用 JVM 預設 SSL context
    -v,--version                                  |顯示版本資訊
    -vl,--variable-latency `<arg>`                |使用 [MICROSECONDS]:[DURATION] 語法的可變消費者處理延遲，其中 [MICROSECONDS] 整數 >= 0 和 [DURATION] 整數 > 0。多次使用該選項以指定多個值。
    -vr,--variable-rate `<arg>`                   |使用 [RATE]:[DURATION] 語法的可變發布速率，其中 [RATE] 整數 >= 0 和 [DURATION] 整數 > 0。多次使用該選項以指定多個值。
    -vs,--variable-size `<arg>`                   |使用 [SIZE]:[DURATION] 語法的可變消息大小，其中 [SIZE] 整數 > 0 和 [DURATION] 整數 > 0。多次使用該選項以指定多個值。
    -x,--producers `<arg>`                        |producer 數量
    -X,--producer-channel-count `<arg>`           |每個 producer 的 channels 數量
    -y,--consumers `<arg>`                        |consumer 數量
    -Y,--consumer-channel-count `<arg>`           |每個 consumer 的 channels 數量
    -z,--time `<arg>`                             |測試執行時間 (預設值： `永久執行`)

3. 執行測試

    ```bash
    docker run -it --rm pivotalrabbitmq/perf-test:latest -x 2 -y 4 -u "throughput-test-1" -a --id "test 1" -s 1000 -f persistent --uri amqp://192.168.80.3:5672 -z 30
    ```

    > - `-x` : 2 producer
    > - `-y` : 4 consumer
    > - `-u` : queue name 為 `throughput-test-1`
    > - `-a` ： 自動 ack
    > - `--id` ： 測試 id 為 `test 1`
    > - `-s` ： 將 message 大小設為 1000 bytes
    > - `-f` ： 使用 `persistent`
    > - `--uri` ： RabbitMQ 位置為 `amqp://192.168.80.3:5672`
    > - `-z` ：執行 30 秒

4. 實際結果

    ![1result](https://user-images.githubusercontent.com/3851540/127993345-2f8a5f79-cc42-475b-89b6-ea64877a9d06.png)

## 心得

看了幾份文件一直沒有找到比較明確的硬體規格需求計算方式，雖然使用 PerfTest 也沒有辦法回推目標用量所需的規格，但至少有個衡量的基準

工具本身參數很多，不敢說是多餘的，只能說目前所在團隊還沒有這些需求，但不免還是造成雜訊，看參數看半天結果用到的沒幾個XD

## 參考資訊

1. [RabbitMQ PerfTest](https://rabbitmq.github.io/rabbitmq-perf-test/stable/htmlsingle/)
2. [rabbitmq/rabbitmq-perf-test](https://github.com/rabbitmq/rabbitmq-perf-test)
3. [rabbitmq性能测试工具rabbitmq-perf-test（官网阅读笔记）](https://blog.csdn.net/zhuangzi123456/article/details/83858650)
4. [在 CentOS7 上建立 RabbitMQ Cluster](/rabbitmq-cluster-centos7)
