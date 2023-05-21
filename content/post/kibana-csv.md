---
title: "允許 Kibana 可以匯出並下載 CSV"
date: 2023-05-20T00:30:00+08:00
lastmod: 2023-05-20T00:30:31+08:00
draft: false
tags: ["efk","elasticsearch"]
slug: "kibana-csv"
---

## 允許 Kibana 可以匯出並下載 CSV

最近在為外部團隊建立 EFK 讓部份查詢不用開發人員介入，保留開發能量為更有價值的功能做準備

團隊在 Production 環境中，有嚴格的權限管理，基本上遵循最小知識原則：依據權責給予對應的權限，不會多給，所以這次外部團隊也只會給到 readonly 或是建立個人查詢相關 visualize 與 dashboard 的權限，原本可以透過內建的 role：`reporting_user` 來賦予匯出 csv 的權限，但在 Kibana 8 中 `reporting_user` 已被註記為棄用，今天就來紀錄一下個人設定上的一些理解誤區

![11deprecated](https://github.com/yowko/picsbed/assets/3851540/8d805383-497c-43e5-8bdb-e8685a86cac3)

## 基本環境說明

1. macOS Ventura 13.2
2. OrbStack Version 0.8.1 (1331)
3. images

    - docker.elastic.co/elasticsearch/elasticsearch:8.7.1
    - docker.elastic.co/kibana/kibana:8.7.1

4. 環境建置可以參考之前筆記：[使用 Docker Compose 建立啟用 X-Pack 的 Elasticsearch 與 Kibana](/docker-compose-elasticsearch-kibana-x-pack)

    - `.env`

        ```config
        # Password for the 'elastic' user (at least 6 characters)
        ELASTIC_PASSWORD=pass.123
        
        # Password for the 'kibana_system' user (at least 6 characters)
        KIBANA_PASSWORD=pass.123
        
        # Version of Elastic products
        STACK_VERSION=8.7.1
        
        # Set the cluster name
        CLUSTER_NAME=docker-cluster
        
        # Set to 'basic' or 'trial' to automatically start the 30-day trial
        LICENSE=basic
        #LICENSE=trial
        
        # Port to expose Elasticsearch HTTP API to the host
        ES_PORT=9200
        #ES_PORT=127.0.0.1:9200
        
        # Port to expose Kibana to the host
        KIBANA_PORT=5601
        #KIBANA_PORT=80
        
        # Increase or decrease based on the available host memory (in bytes)
        MEM_LIMIT=1073741824
        
        # Project namespace (defaults to the current folder name if not set)
        #COMPOSE_PROJECT_NAME=myproject
        ```

    - 建立 `docker-compose.yml`

        ```yaml
        version: '3'

        services:
        
          setup:
            image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
            volumes:
              - certs:/usr/share/elasticsearch/config/certs
            user: "0"
            command: >
              bash -c '
                if [ x${ELASTIC_PASSWORD} == x ]; then
                  echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
                  exit 1;
                elif [ x${KIBANA_PASSWORD} == x ]; then
                  echo "Set the KIBANA_PASSWORD environment variable in the .env file";
                  exit 1;
                fi;
                if [ ! -f config/certs/ca.zip ]; then
                  echo "Creating CA";
                  bin/elasticsearch-certutil ca --silent --pem -out config/certs/ca.zip;
                  unzip config/certs/ca.zip -d config/certs;
                fi;
                if [ ! -f config/certs/certs.zip ]; then
                  echo "Creating certs";
                  echo -ne \
                  "instances:\n"\
                  "  - name: elasticsearch\n"\
                  "    dns:\n"\
                  "      - elasticsearch\n"\
                  "      - localhost\n"\
                  "    ip:\n"\
                  "      - 127.0.0.1\n"\
                  > config/certs/instances.yml;
                  bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
                  unzip config/certs/certs.zip -d config/certs;
                fi;
                echo "Setting file permissions"
                chown -R root:root config/certs;
                find . -type d -exec chmod 750 \{\} \;;
                find . -type f -exec chmod 640 \{\} \;;
                echo "Waiting for Elasticsearch availability";
                until curl -s --cacert config/certs/ca/ca.crt https://elasticsearch:9200 | grep -q "missing authentication credentials"; do sleep 30; done;
                echo "Setting kibana_system password";
                until curl -s -X POST --cacert config/certs/ca/ca.crt -u "elastic:${ELASTIC_PASSWORD}" -H "Content-Type: application/json" https://elasticsearch:9200/_security/user/kibana_system/_password -d "{\"password\":\"${KIBANA_PASSWORD}\"}" | grep -q "^{}"; do sleep 10; done;
                echo "All done!";
              '
            healthcheck:
              test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
              interval: 1s
              timeout: 5s
              retries: 120
        
          elasticsearch:
            image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
            volumes:
              - certs:/usr/share/elasticsearch/config/certs
            ports:
              - ${ES_PORT}:9200
            environment:
              - node.name=elasticsearch
              - cluster.name=${CLUSTER_NAME}
              - cluster.initial_master_nodes=elasticsearch
              - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
              - bootstrap.memory_lock=true
              - xpack.security.enabled=true
              - xpack.security.http.ssl.enabled=true
              - xpack.security.http.ssl.key=certs/elasticsearch/elasticsearch.key
              - xpack.security.http.ssl.certificate=certs/elasticsearch/elasticsearch.crt
              - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
              - xpack.security.transport.ssl.enabled=true
              - xpack.security.transport.ssl.key=certs/elasticsearch/elasticsearch.key
              - xpack.security.transport.ssl.certificate=certs/elasticsearch/elasticsearch.crt
              - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
              - xpack.security.transport.ssl.verification_mode=certificate
              - xpack.license.self_generated.type=${LICENSE}
            mem_limit: ${MEM_LIMIT}
            ulimits:
              memlock:
                soft: -1
                hard: -1
            healthcheck:
              test:
                [
                  "CMD-SHELL",
                  "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
                ]
              interval: 10s
              timeout: 10s
              retries: 120
        
          kibana:
            depends_on:
              elasticsearch:
                condition: service_healthy
            image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
            volumes:
              - certs:/usr/share/kibana/config/certs
            ports:
              - ${KIBANA_PORT}:5601
            environment:
              - SERVERNAME=kibana
              - ELASTICSEARCH_HOSTS=https://elasticsearch:9200
              - ELASTICSEARCH_USERNAME=kibana_system
              - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
              - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
            mem_limit: ${MEM_LIMIT}
            healthcheck:
              test:
                [
                  "CMD-SHELL",
                  "curl -s -I http://localhost:5601 | grep -q 'HTTP/1.1 302 Found'",
                ]
              interval: 10s
              timeout: 10s
              retries: 120
        volumes:
          certs:
            driver: local
        ```

## 設定方式

1. 設定 kibana.yml

    > 加上 `xpack.reporting.roles.enabled: false` 這個設定是讓 Kibana 不要再使用 `reporting_user` 這個角色來管理 Reporting 相關功能 (CSV 包含在 Reporting 中)，其他設定都是預設產生的

    ```yml
    server.host: "0.0.0.0"
    server.shutdownTimeout: "5s"
    elasticsearch.hosts: [ "http://elasticsearch:9200" ]
    monitoring.ui.container.elasticsearch.enabled: true
    xpack.reporting.roles.enabled: false
    ```

2. 設定 role 權限

    - 建立 reporting 用的 role

        ![1createrole](https://github.com/yowko/picsbed/assets/3851540/c517cf0b-8eea-4612-9c2b-8c0db2b67548)

    - 設定 elaticsearch 權限

        - 1.指定 role name
        - 2.指定 indices
        - 3.加上 indices 對應權限

            > Reporting 需要 `read` 與 `view_index_metadata`

        ![2indexprivilege](https://github.com/yowko/picsbed/assets/3851540/b6b8debe-4491-4134-b841-82d05fb15e96)

    - 設定 Kibana 權限

        - 1.指定 Space
        - 2.選擇自訂權限
        - 3.根據不同 applicaiton 給予權限

            > 基本授權(free) 不提供單一子功能設定，需要 Reporting 功能只能選 `All` 或是透過 role API 調整

        ![3kibanaprivilege](https://github.com/yowko/picsbed/assets/3851540/708517c7-eca7-4746-a961-c9107a7fc16c)

3. 使用上述建立的 role 來建立 user

    ![4createuser](https://github.com/yowko/picsbed/assets/3851540/81755937-6f05-43fb-9a14-b2d816f8cf49)

    - 1. 指定 username
    - 2. 設定 password
    - 3. 指定使用上述建立的 role

    ![5userwithrole](https://github.com/yowko/picsbed/assets/3851540/26f30c36-74ee-4628-a439-c98004dc6a65)

4. 實際效果

    - 設定前

        - 無法匯出

            ![7before1](https://github.com/yowko/picsbed/assets/3851540/bce35e26-78f6-4113-8386-f3918f31718b)

            ![8before2](https://github.com/yowko/picsbed/assets/3851540/83332fe0-4f67-4f00-b02e-c846d94c4afa)

        - UI

            ![9before3](https://github.com/yowko/picsbed/assets/3851540/ea765b47-e452-423b-8a1a-d76f2fc3baa8)

    - 設定後

        - UI

            ![10after1](https://github.com/yowko/picsbed/assets/3851540/08746721-d5a1-42b7-8dab-a868c11e2799)

## 心得

[Kibana 官方文件 ： Configure reporting in Kibana](https://www.elastic.co/guide/en/kibana/current/secure-reporting.html) 有說明如何設定，只是文件說明讓我誤會了

1. `xpack.reporting.roles.enabled: false` 字面定義

    > 讓我誤以為是不啟用 reporting 相關功能，這只是個人腦補造成的，就是我自以為我要開啟 export csv 功能，直覺認為應該要將某個設定改為 `true`；殊不知官方的思維是另個角度：關閉 `reporting_user` 的管理權限改由 x-pack 設定

2. role API 的版面配置：[Configure reporting in Kibana - Grant access with the role API](https://www.elastic.co/guide/en/kibana/current/secure-reporting.html#reporting-roles-user-api)

    > 與上面的設定步驟：[Grant users access to reporting](https://www.elastic.co/guide/en/kibana/current/secure-reporting.html#grant-user-access) 是同一層，讓我以為 `Grant access with the role API` 也是一種設定方式，但實際上只是用來調整權限用的

3. role API 內容有缺誤

    - url 錯誤 (這個是出現在 Copy as curl 時，文件本身是正確的)

        > 應為 `localhost:5601/api/security/role/{role name}` 卻得到 `localhost:9200/<kibana host>:<port>/api/_security/role/{role name}?pretty`

        - domain 應該是 `<kibana host>:<port>` 卻多了 `localhost:9200`
        - path 應為 `security` 而非 `_security`
        - 不該有 `?pretty`

            ```log
            {"statusCode":400,"error":"Bad Request","message":"[request query.pretty]: definition for this key is missing"}% 
            ```

    - POST 一直遇到 404
    - 沒有 auth 無法執行

        ```log
        {"statusCode":401,"error":"Unauthorized","message":"Unauthorized"}% 
        ```

    - 缺 header

        ```log
        {"statusCode":400,"error":"Bad Request","message":"Request must contain a kbn-xsrf header."}% 
        ```

    - 權限不足：`generate_report` 與 `download_csv_report`

        ![6missingprivige](https://github.com/yowko/picsbed/assets/3851540/df69229d-2eb3-4e2c-bb73-6dd8931faa05)

    > 這邊我提供一下我可以使用的版本

    ```bash
    curl -u {username}:{password} -X PUT "{kibana host}:{kibana port}/api/security/role/{role name}" -H 'Content-Type: application/json' -H 'kbn-version:8.7.1' -d'
    {
            "elasticsearch": {
                    "cluster": [],
                    "indices": [],
                    "run_as": []
            },
            "kibana": [{
                    "spaces": ["*"],
                    "base": [],
                    "feature": {
                            "dashboard": ["generate_report",  
        "download_csv_report"], 
        "discover": ["generate_report"], 
                            "canvas": ["generate_report"], 
                            "visualize": ["generate_report"] 
                    }
            }]
    }
    '
    ```

## 參考資訊

1. [Role reporting_user is deprecated. Please use Kibana feature privileges instead](https://discuss.elastic.co/t/role-reporting-user-is-deprecated-please-use-kibana-feature-privileges-instead/294593)
2. [Configure reporting in Kibana](https://www.elastic.co/guide/en/kibana/current/secure-reporting.html)
3. [使用 Docker Compose 建立啟用 X-Pack 的 Elasticsearch 與 Kibana](/docker-compose-elasticsearch-kibana-x-pack)
