---
title: "MongoDB cli 中的 try catch"
date: 2022-01-16T00:30:00+08:00
lastmod: 2022-01-16T00:30:31+08:00
draft: false
tags: ["docker"]
slug: "mongo-tr-catch"
---

## MongoDB cli 中的 try catch

這是在調整團隊 MongoDB instance 時遇到的問題：某一隻 js 會在 db init 時執行 create user 的動作，完成後會繼續建立其他 collection 跟 index，但因為這個 user 是跨 db 的 user ，已經被其他 db init 建立過了，所以打算透過 try catch 來處理：create user 失敗就改用 grant 的方式來讓 user 有權限

## 基本環境說明

1. macOS Monterey 12.1
2. docker desktop 4.3.2 (72729)
3. docker images

    - mongo:4.4.11

4. 建立測試環境

    1. init/0.js

        ```js
        use admin;
        db.createUser(
            {
                user: "backupuser",
                pwd: "pass.123",
                roles: [ { role: "readWrite", db: "demo" } ]
            }
        );
        ```

    2. init/1.js

        ```js
        use admin;
        db.createUser(
            {
                user: "backupuser",
                pwd: "pass.123",
                roles: [ { role: "readWrite", db: "test" } ]
            }
        );
        ```

    3. mongo-init.sh

        ```bash
        #! /bin/bash
        init_mongodb(){
        
            for file in /tmp/mongo/*.js; do
                echo $file
                mongo -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD < $file
            done
        }
        
        main(){
            echo "init_data"
            init_mongodb
        }
        
        main "$@"
        ```

    4. docker-compose

        ```yaml
        version: '3.1'
        services:
          mongo:
            image: mongo:4.4.11
            restart: always
            environment:
              MONGO_INITDB_ROOT_USERNAME: root
              MONGO_INITDB_ROOT_PASSWORD: pass.123
            volumes:
              - ${PWD}/init:/tmp/mongo
              - ${PWD}/mongo-init.sh:/docker-entrypoint-initdb.d/mongo-init.sh
        ```

## 錯誤訊息

1. 訊息內容

    ```txt
    mongo_1  | uncaught exception: Error: couldn't add user: User "backupuser@admin" already exists :
    mongo_1  | _getErrorWithCode@src/mongo/shell/utils.js:25:13
    mongo_1  | DB.prototype.createUser@src/mongo/shell/db.js:1386:11
    mongo_1  | @(shell):1:1
    mongo_1  | bye
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/149724467-2764c887-7780-4373-9095-aaec29e786f1.png)

## 解決方式：在 mongo shell 中使用 try catch

1. 調整啟動方式

    - 1-1. init/0.js

        ```js
        db = db.getSiblingDB("admin")
        try {
            db.createUser(
                {
                    user: "backupuser",
                    pwd: "pass.123",
                    roles: [ { role: "readWrite", db: "demo" } ]
                }
            );
        }
        catch (err) {
            print ("backupuser user already created");
            db.grantRolesToUser( "backupuser", [ { role: "readWrite", db: "demo" }] );
        }
        ```

    - 1-2. init/1.js

        ```js
        db = db.getSiblingDB("admin")
        try {
            db.createUser(
                {
                    user: "backupuser",
                    pwd: "pass.123",
                    roles: [ { role: "readWrite", db: "test" } ]
                }
            );
        }
        catch (err) {
            print ("backupuser user already created");
            db.grantRolesToUser( "backupuser", [ { role: "readWrite", db: "test" }] );
        }
        ```

    - 1-3. docker-compose

        ```yaml
        version: '3.1'
        services:
          mongo:
            image: mongo:4.4.11
            restart: always
            environment:
              MONGO_INITDB_ROOT_USERNAME: root
              MONGO_INITDB_ROOT_PASSWORD: pass.123
            volumes:
              - ${PWD}/init:/docker-entrypoint-initdb.d/
        ```

2. 調整 js 寫法

    - 2-1. init/0.js

        ```js
        use admin;
        initFunction= {
            start: function(){
                try {
                    db.createUser(
                        {
                            user: "backupuser",
                            pwd: "pass.123",
                            roles: [ { role: "readWrite", db: "demo" } ]
                        }
                    );
                }
                catch (err) {
                    print(err);
                    db.grantRolesToUser( "backupuser", [ { role: "readWrite", db: "demo" }] );
                }
            }
        }
        
        initFunction.start();
        ```

    - 2-2. init/1.js

        ```js
        use admin;
        initFunction= {
            start: function(){
                try {
                    db.createUser(
                        {
                            user: "backupuser",
                            pwd: "pass.123",
                            roles: [ { role: "readWrite", db: "test" } ]
                        }
                    );
                }
                catch (err) {
                    print(err);
                    db.grantRolesToUser( "backupuser", [ { role: "readWrite", db: "test" }] );
                }
            }
        }
        
        initFunction.start();
        ```

## 心得

如果是 for docker image 的 init 使用 mongodb image 預設的 [MONGO_INITDB_DATABASE](https://hub.docker.com/_/mongo?tab=description) 機制：`調整啟動方式` 會單純許多

但如果想要讓 container 與實體 mongodb instance 行為一致，就可以參考使用 `調整 js 寫法`

- 修改前

    ![2before](https://user-images.githubusercontent.com/3851540/149724486-e730337c-13cd-413a-9974-7ed822ed849f.png)

- 修改後

    ![3after](https://user-images.githubusercontent.com/3851540/149724489-0711ec04-a529-43c0-baa0-d4eb1a04d518.png)

## 參考資訊

1. [MONGO_INITDB_DATABASE](https://hub.docker.com/_/mongo?tab=description)
2. [try..catch errors in mongo shell](https://stackoverflow.com/a/56479750)
3. [Write Scripts for the mongo Shell](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/)
