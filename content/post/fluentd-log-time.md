---
title: "Fluentd 使用自定 Log 時間當做 Timestamp"
date: 2019-05-04T21:30:00+08:00
lastmod: 2019-09-10T21:30:31+08:00
draft: false
tags: ["Fluentd","Log","ELK"]
slug: "fluentd-log-time"
---
## Fluentd 使用自定 Log 時間當做 Timestamp

之前筆記 [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/) 提到最近需要 debug EFK 起因就是發現 Kibana 顯示儲存在 Elasticsearch 中的時間與過濾條件都是 log 的處理時間並非 log 真正的發生時間，想像一下：客戶反應了出問題的時間，工程師在 Kibana 上怎麼查都找不到相關紀錄，翻原始 log 才能找到實際問題.... 想必沒人能接受，導入工具沒有改善流程就算了  還誤導除錯方向這可不行呀

剛好最近手上的功能開發工作有空檔，就由我來調整，不過偶爾碰 ELK 的我很多東西都不記得，東卡西卡的，於是簡單筆記一下

## 基本環境說明

1. macOS Mojave 10.14.4
2. Docker Engine - Community 18.09.2
3. Elastic stack 6.7

    > 我只為了要本機方便開發測試，正式環境請勿這麼做。環境需要 elasticsearch + kibana，我懶得重頭開始架，就拿了 github 上的 docker-compose ([Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk))來用

    ```yaml
    version: '3.2'

    services:
      elasticsearch:
        build:
          context: elasticsearch/
          args:
            ELK_VERSION: $ELK_VERSION
        volumes:
          - type: bind
            source: ./elasticsearch/config/elasticsearch.yml
            target: /usr/share/elasticsearch/config/elasticsearch.  yml
            read_only: true
          - type: volume
            source: elasticsearch
            target: /usr/share/elasticsearch/data
        ports:
          - "9200:9200"
          - "9300:9300"
        environment:
          ES_JAVA_OPTS: "-Xmx256m -Xms256m"
          ELASTIC_PASSWORD: changeme
          # Use single node discovery in order to disable   production mode and avoid bootstrap checks
          # see https://www.elastic.co/guide/en/elasticsearch/  reference/current/bootstrap-checks.html
          discovery.type: single-node
        networks:
          - elk
    
      logstash:
        build:
          context: logstash/
          args:
            ELK_VERSION: $ELK_VERSION
        volumes:
          - type: bind
            source: ./logstash/config/logstash.yml
            target: /usr/share/logstash/config/logstash.yml
            read_only: true
          - type: bind
            source: ./logstash/pipeline
            target: /usr/share/logstash/pipeline
            read_only: true
        ports:
          - "5000:5000/tcp"
          - "5000:5000/udp"
          - "9600:9600"
        environment:
          LS_JAVA_OPTS: "-Xmx256m -Xms256m"
        networks:
          - elk
        depends_on:
          - elasticsearch
    
      kibana:
        build:
          context: kibana/
          args:
            ELK_VERSION: $ELK_VERSION
        volumes:
          - type: bind
            source: ./kibana/config/kibana.yml
            target: /usr/share/kibana/config/kibana.yml
            read_only: true
        ports:
          - "5601:5601"
        networks:
          - elk
        depends_on:
          - elasticsearch
    
    networks:
      elk:
        driver: bridge
    ```

4. Fluentd edge (v1.4.2) - aggregator 採用自製 docker image (詳細內容請參考 [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/) )

    > 這邊使用官方建議方式 (forwarder - aggregator):在需要 parse log 的 server 上安裝一個 forwarder 的 fluentd instance，然後將 log 發送給 aggregator 做後續處理

    ![ha](https://user-images.githubusercontent.com/3851540/64233979-e9851d00-cf27-11e9-8605-32d46ce9d71e.png)

    >圖片出處：https://docs.fluentd.org/v1.0/articles/high-availability

## Fluentd 的原始設定

1. Log Forwarder

    ```txt
    <source>
        @type tail
        path /tmp/fluentd/app/**/All-*.log
        pos_file /tmp/fluentd/pos/app-api.pos
        format /^(?<logdate>[^ ]* [^ ]*) \[(?<application>[^ \]]+)\]\[(?<thread>[\d]+)\]\[(?<type>[A-Za-z_][A-Za-z0-9_]+)\](?<message>\{.+\})/
        tag yowko.app-api
    </source>

    # Log Forwarding
    <match *.**>
        @type forward
        <server>
            host 192.168.50.97
            port 24224
        </server>
        <buffer>
            flush_interval 60s
        </buffer>
    </match>
    ```

2. Log Aggregator

    ```txt
    # Input
    <source>
        @type http
        port 8888
    </source>

    <source>
        @type forward
        port 24224
    </source>

    # Output
    <match *.**>
        @type elasticsearch
        host 192.168.50.97
        port 9200
        logstash_format true
    </match>
    ```

3. 實際問題： kibana 顯示的紀錄時間與 log 不一致

    ![1mismatch](https://user-images.githubusercontent.com/3851540/57190821-63dd6a80-6f51-11e9-86f4-14d6f40dbf49.png)

## 啟動 Fluentd

1. Log Forwarder

    ```bash
    docker run -d  -v /tmp/fluentd/app:/tmp/fluentd/app -v /tmp/fluentd/pos:/tmp/fluentd/pos -v /tmp/fluentd/forwarder:/fluentd/etc -e FLUENTD_CONF=fluentd.conf --name fluentdForwarder fluent/fluentd:edge
    ```

    - 第一個 `-v` mount 需要 parse 的 log 位置 (可以從外部調整 log 方便測試)
    - 第二個 `-v` 用來儲存紀錄上次讀取資訊的 pos 檔
    - 第三個 `-v` 則是將外部的 fluentd config 掛載進去用的
    - `-e` 用來設定環境變數 `FLUENTD_CONF` 的檔名

2. Log Aggregator

     ```bash
    docker run -d -p 8888:8888 -p 24224:24224 -p 24224:24224/udp --name fluentdAggregator -v /tmp/fluentd/aggregator:/fluentd/etc -e FLUENTD_CONF=fluentd.conf fluentdelastic:v1
    ```

    - `-p 8888` 用來開放 http request 方便測試
    - `-p 24224` 則是用來處理 fluentd forward
    - `-v` 則是將外部的 fluentd config 掛載進去用的
    - `-e` 用來設定環境變數 `FLUENTD_CONF` 的檔名
    - `fluentdelastic:v1` 使用自製已安裝 `Elasticsearch Output Plugin` 的 base image (詳細內容請參考 [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/))

## 使用自定時間

1. forwarder 指定 key name (預設使用 `time`)

    > 如果 parser 的 format 中即使用 `time` 變數來儲存 log time，就可以不用額外指定 key name。e.g. `format /^(?<time>[^ ]* [^ ]*) \[(?<application>[^ \]]+)\]\[(?<thread>[\d]+)\]\[(?<type>[A-Za-z_][A-Za-z0-9_]+)\](?<message>\{.+\})/` 就可以省略

    ```
    time_key logdate
    ```

2. forwarder 設定保留 key

    ```
    keep_time_key true
    ```

3. Aggregator 指定 key name

    > 如果時間欄位為 `time` 這邊亦可省略

    ```
    time_key logdate
    ```

4. 實際效果：kibana 已將 log 時間做為顯示及過濾的基準

    ![2correctlogdate](https://user-images.githubusercontent.com/3851540/57190822-63dd6a80-6f51-11e9-9615-e6bc74e4db6d.png)

5. 調整後設定

    - Log Forwarder

        ```txt
        <source>
            @type tail
            path /tmp/fluentd/app/**/All-*.log
            pos_file /tmp/fluentd/pos/app-api.pos
            format /^(?<logdate>[^ ]* [^ ]*) \[(?<application>[^ \]]+)\]\[(?<thread>[\d]+)\]\[(?<type>[A-Za-z_][A-Za-z0-9_]+)\](?<message>\{.+\})/
            time_key logdate
            keep_time_key true
            tag yowko.app-api
        </source>

        # Log Forwarding
        <match *.**>
            @type forward
            <server>
                host 192.168.50.97
                port 24224
            </server>
            <buffer>
                flush_interval 60s
            </buffer>
        </match>
        ```

    - Log Aggregator

        ```txt
        # Input
        <source>
            @type http
            port 8888
        </source>

        <source>
            @type forward
            port 24224
        </source>

        # Output
        <match *.**>
            @type elasticsearch
            host 192.168.50.97
            port 9200
            logstash_format true
            time_key logdate
        </match>
        ```

## 心得

要使用自訂時間當做 timestamp 所需的正確設定很少，但卻很難一次就搞定，我想原因是官網上沒有明確的說明，就連同 `Elasticsearch Output Plugin` 參數列表上也沒有對應選項。經過反覆測試我才終於找出正確設定，不過或許隨著版本遞代  相關參數與設定可能也會跟著調整 (我嘗試了不少網路文章的建議設定都無法正確生效，推測可以也是因為設定調整的關係)

以下列出個人 debug 的好用小技巧

1. 清空 elasticsearch 的所有物件：可以確保打進去的 log 是獨立的，不會受到之前操作的影響

    ```bash
    curl -X DELETE 'http://localhost:9200/_all'
    ```

2. Log Forwarder ： 加入 `stdout` 輸出，方便確認收到的 log 內容

    ```txt
    <source>
        @type tail
        path /tmp/fluentd/app/**/All-*.log
        pos_file /tmp/fluentd/pos/app-api.pos
        format /^(?<logdate>[^ ]* [^ ]*) \[(?<application>[^ \]]+)\]\[(?<thread>[\d]+)\]\[(?<type>[A-Za-z_][A-Za-z0-9_]+)\](?<message>\{.+\})/ 
        time_key logdate
        keep_time_key true
        tag yowko.app-api
    </source>
    # Log Forwarding
    <match *.**>
        @type copy
        <store>
            @type forward
            <server>
                host 192.168.50.97
                port 24224
            </server>
        <buffer>
                flush_interval 60s
            </buffer>
        </store>
        <store>
            @type stdout
        </store>
    </match>
    ```

3. Log Aggregator 加入 `http` 輸入，可以直接透過 http 傳入資料，方便 debug

    ```txt
    # Input
    <source>
        @type http
        port 8888
    </source>
    <source>
        @type forward
        port 24224
    </source>
    # Output
    <match *.**>
        @type copy
        <store>
            @type elasticsearch
            host 192.168.50.97
            port 9200
            logstash_format true
            logstash_prefix fluentd-${tag}
            time_key logdate
        </store>
        <store>
            @type stdout
        </store>
    </match>
    ```

4. Fluentd 間或是與 Elastisearch 的溝通記得使用可以對外服務的 ip，因為是 container，使用 localhosr or 127.0.0.1 都會解析到自己

## 參考資訊

1. [Fluentd 安裝 Elasticsearch Output Plugin 封裝成 Docker image](https://blog.yowko.com/fluentd-elasticsearch-docker/)
2. [Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk)
