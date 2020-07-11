---
title: "使用 Elastic Stack (ELK) 來監控 MongoDB"
date: 2018-05-12T00:23:00+08:00
lastmod: 2020-07-11T00:23:54+08:00
draft: false
tags: ["ELK","MongoDB","Monitoring"]
slug: "elk-mongodb-monitor"
aliases:
    - /2018/05/elk-mongodb-monitor.html
---
# 使用 Elastic Stack (ELK) 來監控 MongoDB
之前筆記 [Windows 平台上安裝 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana)](https://blog.yowko.com/2018/05/elastic-stack-elk-windows.html) 提到因為想要將部份系統資料餵進 ELK 用來監控及除錯，順手紀錄 Windows 平台上架設 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana) 的步驟，也提及 ELK 較適合執行於 Linux 環境，只是身為 Microssoft 派工程師，Windows 環境相對好取得用來開發測試

首先第一個目標就是為原本沒有監控機制的 MongoDB  加上基本的效能監控。在同事強力的協助下，終於排除困難將 MongoDB 效能資訊餵進 ELK，過程況狀不斷，如果不紀錄相信一週後我就再也做不出來了，就來看該如何設定吧

## 安裝 Metricbeat

1. 下載 [Metricbeat](https://www.elastic.co/downloads/beats/metricbeat) 並解壓縮
2. 準備 Metricbeat 的 yaml 檔 ：`mongodb.yml` 檔名可自訂

    > 下載的 Metricbeat 中有一份 `metricbeat.reference.yml` 包含許多不同 module 的設定值可以用來參考

    ```yml
        metricbeat.modules:
        - module: mongodb
            metricsets: ["dbstats", "status"]
            # 指定輪詢間隔
            period: 10s
        
            # 連線方式有兩種
            # 連線的帳號需要有 admin database 的權限
            # arbiter 沒有實際 db ，無法使用預設 field 設定匯入 index
            # 第一種：mongo 的 host 與 port 跟 username/ passwork 分開放
            hosts: ["localhost:27017","localhost:27027"]
            username: admin
            password: pass.123
            
            # 第二種：使用 [mongodb://][user:pass@]host[:port]
            # hosts: ["mongodb://admin:pass.123@127.0.0.1:27017","mongodb://admin:pass.    123@127.0.0.1:27027"]
    
        # 將結果輸出至 elasticsearch
        output.elasticsearch:
            # elasticsearch url
            hosts: ["localhost:9200"]
            # 指定進 elasticsearch 的 index 名稱 (%{+yyyy.ww} 是 yaml 的時間格式)
            index: "localhost-mongodb-%{+yyyy.ww}"
    
        # template 是用來在 Elasticsearch 中設定 mapping ，預設就是啟用的
        # 指定 template 名稱
        setup.template.name: "localhost-mongodb"
        # 設定 template mapping 的 pattern 
        setup.template.pattern: "localhost-mongodb-*"
        # 指定產生的欄位
        setup.template.fields: "${path.config}/fields.yml"
        # 複寫已存在的 template
        setup.template.overwrite: true
    ```

    > 2020/07/11 更新

    ```yml
        metricbeat.modules:
        - module: mongodb
          metricsets: ["dbstats", "status", "collstats", "metrics", "replstatus"]
          # 指定輪詢間隔
          period: 10s
          enabled: true
        
          # 連線方式有兩種
          # 連線的帳號需要有 admin database 的權限
          # arbiter 沒有實際 db ，無法使用預設 field 設定匯入 index
          # 第一種：mongo 的 host 與 port 跟 username/ passwork 分開放
          hosts: ["localhost:27017","localhost:27027"]
          username: admin
          password: pass.123
          
          # 第二種：使用 [mongodb://][user:pass@]host[:port]
          # hosts: ["mongodb://admin:pass.123@127.0.0.1:27017","mongodb://admin:pass.123@127.0.0.1:27027"]
        
        # 將結果輸出至 elasticsearch
        output.elasticsearch:
          # elasticsearch url
          hosts: ["localhost:9200"]
          # 指定進 elasticsearch 的 index 名稱 (%{+yyyy.ww} 是 yaml 的時間格式)
          index: "localhost-mongodb-%{+yyyy.ww}"
        # template 是用來在 Elasticsearch 中設定 mapping ，預設就是啟用的
        # 指定 template 名稱
        setup.template.name: "localhost-mongodb"
        # 設定 template mapping 的 pattern 
        setup.template.pattern: "localhost-mongodb-*"
        # 指定產生的欄位
        setup.template.fields: "${path.config}/fields.yml"
        # 複寫已存在的 template
        setup.template.overwrite: true
    ```

3. 啟動 Metricbeat
    - 語法 
        
        ```
        {metricbeat.exe path} -e -c {mongodb yaml path}
        ``` 
    - 範例
        
        ```
        metricbeat.exe -e -c mongodb.yml
        ``` 
    - 成功啟動並開始餵資料至 Elasticsearch
        
        ![1metricbeatstart](https://user-images.githubusercontent.com/3851540/39933636-229e2776-5576-11e8-82f0-9f254ff233a1.png) 

## 修改 Metricbeat 中預設的 index template 與 dashboard

> 更名部份就視實際需要，只要名稱保持與 Metricbeat template 相同就可，不一定需要更名，但更名可以讓管理比較清楚

> 下載的 Metricbeat 的 `kibana` 下 `index-pattern` 與 `dashboard` 的資料夾有既有的 json

1. 修改 index template ：`metricbeat.json`
    - 預設路徑 ：`{metricbeat folder}\kibana\6\index-pattern\metricbeat.json`
    - 將 `title` 與 `id` 的改為之前的 template pattern
        - 修改前
            
            ![3templatebefore](https://user-images.githubusercontent.com/3851540/39933638-22f060a4-5576-11e8-9c26-7c4f169c831f.png)
        - 修改後 
            
            ![4templateafter](https://user-images.githubusercontent.com/3851540/39933639-2317e1d8-5576-11e8-8628-217b09ceb74a.png) 
2. 修改 dashboard ：`Metricbeat-mongodb-overview.json`
    - 預設路徑：`{metricbeat folder}\kibana\6\dashboard\Metricbeat-mongodb-overview.json`
    - 修改 `searchSourceJSON` 的 index 名稱
        - 修改前
            
            ![5dashboardbefore](https://user-images.githubusercontent.com/3851540/39933640-233f98fe-5576-11e8-8d1b-f4cd520d2ec4.png)
        - 修改後
            
            ![6dashnoardafter](https://user-images.githubusercontent.com/3851540/39933641-236c7e46-5576-11e8-9b44-ae3679a149b2.png)
* 如果用不到其他 modudule 建議可以刪除其他 dashboard


## 匯入 Metricbeat 的 Kibana dashboards
* 使用 `setup` 指令匯入 index template 與 dashboards
    
    ```
    metricbeat.exe setup
    ```
    
    ![2metricbeatsetup](https://user-images.githubusercontent.com/3851540/39933637-22c81a36-5576-11e8-93cb-9922f1ac5048.png)

## 完成設定
1. 將匯入的 index pattern 設為預設 index pattern
    
    >這個步驟可略過，但沒設定會一直有 warning

    ![7indexpattern1](https://user-images.githubusercontent.com/3851540/39933642-2393a534-5576-11e8-8e58-04067be679fa.png)

    ![8indexpattern2](https://user-images.githubusercontent.com/3851540/39933644-23bdb2fc-5576-11e8-9c41-f15317d53cf9.png) 
2. 開啟 Dashboard
    
    ![9dashborad1](https://user-images.githubusercontent.com/3851540/39933645-23e889d2-5576-11e8-8221-5b89994881cf.png)

    ![10dashboard2](https://user-images.githubusercontent.com/3851540/39933646-2413bae4-5576-11e8-9967-9967ff9a5bd6.png)

## 心得
之前跟朋友聊到 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana) 的架設大概是使用 Elastic Stac 時最輕鬆的一個步驟，架設完成後實際要使用或是餵資料進去，都有不少眉眉角角要解決，這次要不是有同事協助可能沒辦法那麼順利搞定

這次很幸運還不需要設定 Logstash，單單處理 Elasticsearch 跟 Kibana 就讓我人仰馬翻了，設定跟文件不僅多也比較雜亂，加入 Metricbeat 後讓我對於整體概念一直懞懞懂懂不是很清楚，另外我覺得比較難搞的一點就是不好除錯，雖然有錯誤訊息，只是成因可能很多，造成解決方法更多，當然我自己不熟悉是主要因素，就初學者的立場來看門檻較高，不過也許就是它的彈性與擴充性才讓它受到許多關注與也才得以發揚光大

# 參考資訊
1. [Windows 平台上安裝 Elastic Stack (ELK - Elasticsearch , Logstash , Kibana)](https://blog.yowko.com/2018/05/elastic-stack-elk-windows.html)
2. [Metricbeat](https://www.elastic.co/downloads/beats/metricbeat)
3. [Importing Existing Beat Dashboards](https://www.elastic.co/guide/en/beats/devguide/current/import-dashboards.htm)
4. [MongoDB Performance Monitoring Using the ELK Stack](https://dzone.com/articles/mongodb-performance-monitoring-using-the-elk-stack)
5. [Config file data types](https://www.elastic.co/guide/en/beats/libbeat/current/config-file-format-type.html)