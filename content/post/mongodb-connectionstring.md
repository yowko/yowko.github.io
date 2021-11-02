---
title: "C# 搭配 MongoDB 的連線寫法"
date: 2018-06-10T20:26:00+08:00
lastmod: 2021-11-02T17:33:50+08:00
draft: false
tags: ["csharp","MongoDB"]
slug: "mongodb-connectionstring"
aliases:
    - /2018/06/mongodb-connectionstring.html
---
## C# 搭配 MongoDB 的連線寫法

最近有個新專案需要儲存 json 格式的資料，MongoDB 是考慮的選項之一，評估的過程中才發現我沒有 C# 連線 MongoDB 的使用筆記，雖然專案時程非常急迫需求假日額外加班處理，原本很認份地要開工但仔細想想工作畢竟是工作就算再急也不該為了工作失去熱誠，雖說我自己常常額外加班不過都是出於對程式熱誠，一旦只是為了對專案有所交待，我愈來愈說服不了自己，所以決定在休假期間放下時程的考量做自己覺得開心的事

立馬就來紀錄一下 MongoDB 的連線寫法吧

## 安裝套件 `MongoDB.Driver`

```cmd
Install-Package MongoDB.Driver
```

![1install](https://user-images.githubusercontent.com/3851540/41201507-f5df2afe-6ceb-11e8-96fe-c6889adcd89d.png)

## MongoDB 的連線寫法

1. 使用預設連線 (localhost:27017)

    ```cs
    MongoClient client = new MongoClient();
    ```

2. 使用連線字串

    ```cs
    MongoClient client = new MongoClient("mongodb://localhost:27017");
    ```

3. 使用 MongoClientSettings

    ```cs
    MongoClient client = new MongoClient(
        new MongoClientSettings
        {
            Server = new MongoServerAddress("localhost",27017)
        });
    ```

4. 使用 MongoUrl

    ```cs
    MongoClient client = new MongoClient(
        MongoUrl.Create("mongodb://localhost:27017"));
    ```

## 有密碼保護的 MongoDB 連線寫法

1. 使用預設連線 (localhost:27017)

    >預設連線方式無法指定帳號密碼

2. 使用連線字串
    - 語法

        ```cs
        MongoClient client= new MongoClient("mongodb://{username}:{password}@{host}:{port}/{Database}");
        ```

    - 實際範例

        ```cs
        MongoClient client= new MongoClient("mongodb://yowkoTest:passwordTest@127.0.0.1:27017/test");
        ```

3. 使用 MongoClientSettings
    - 寫法一
        - 語法

            ```cs
            var credential = MongoCredential.CreateCredential("{Database}", "{username}", "{password}");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = credential,
                Server = new MongoServerAddress("{host}", {port}),
            });
            ```  

        - 實際範例

            ```cs
            var credential = MongoCredential.CreateCredential("test", "yowkoTest", "passwordTest");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = credential,
                Server = new MongoServerAddress("localhost", 27017),
            });
            ```

    - 寫法二
        - 語法

            ```cs
            MongoIdentity identity = new MongoInternalIdentity("{Database}",  "{username}");
            MongoIdentityEvidence evidence = new PasswordEvidence("{password}");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = new MongoCredential(null, identity, evidence),
                Server = new MongoServerAddress("{host}", {port}),
            });
            ```

        - 實際範例

            ```cs
            MongoIdentity identity = new MongoInternalIdentity("test", "yowkoTest");
            MongoIdentityEvidence evidence = new PasswordEvidence("passwordTest");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = new MongoCredential(null, identity, evidence),
                Server = new MongoServerAddress("localhost", 27017),
            });
            ```

4. 使用 MongoUrl
    - 語法

        ```cs
        MongoClient client = new MongoClient(
                MongoUrl.Create("mongodb://{username}:{password}@{host}:{port}/{Database}"));
        ```

    - 實際範例

        ```cs
        MongoClient client = new MongoClient(
                MongoUrl.Create("mongodb://yowkoTest:passwordTest@localhost:27017/test"));
        ```

## 有密碼保護的 MongoDB ReplicaSet 連線寫法

1. 使用預設連線 (localhost:27017)

    >預設連線方式無法指定帳號密碼及 ReplicaSet
2. 使用連線字串
    - 語法

        ```cs
        MongoClient client= new MongoClient("mongodb://{username}:{password}@{host1}:{port1},{host2}:{port2},{host3}:{port3}/?replicaSet={replicaSet Name}");
        ```

    - 實際範例

        ```cs
        MongoClient client= new MongoClient("mongodb://yowkoTest:passwordTest@127.0.0.1:27017,127.0.0.1:27027,127.0.0.1:27037/?replicaSet=TestReplicaSet");
        ```

3. 使用 MongoClientSettings
    - 寫法一
        - 語法

            ```cs
            var credential = MongoCredential.CreateCredential(null, "{username}", "{password}");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = credential,
                Servers = new List<MongoServerAddress>(){
                    new MongoServerAddress("{host1}", {port1}),
                    new MongoServerAddress("{host2}", {port2}),
                    new MongoServerAddress("{host3}", {port3})
                },
                ReplicaSetName="{replicaSet Name}"
            });
            ```  

        - 實際範例

            ```cs
            var credential = MongoCredential.CreateCredential(null, "yowkoTest", "passwordTest");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = credential,
                Servers = new List<MongoServerAddress>(){
                    new MongoServerAddress("localhost", 27017),
                    new MongoServerAddress("localhost", 27027),
                    new MongoServerAddress("localhost", 27037)
                },
                ReplicaSetName="TestReplicaSet"
            });
            ```

    - 寫法二
        - 語法

            ```cs
            MongoIdentity identity = new MongoInternalIdentity("{Database}",  "{username}");
            MongoIdentityEvidence evidence = new PasswordEvidence("{password}");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = new MongoCredential(null, identity, evidence),
                Servers = new List<MongoServerAddress>(){
                    new MongoServerAddress("{host1}", {port1}),
                    new MongoServerAddress("{host2}", {port2}),
                    new MongoServerAddress("{host3}", {port3})
                },
                ReplicaSetName="{replicaSet Name}"
            });
            ```

        - 實際範例

            ```cs
            MongoIdentity identity = new MongoInternalIdentity("test", "yowkoTest");
            MongoIdentityEvidence evidence = new PasswordEvidence("passwordTest");
            MongoClient client = new MongoClient(
            new MongoClientSettings
            {
                Credential = new MongoCredential(null, identity, evidence),
                Servers = new List<MongoServerAddress>(){
                    new MongoServerAddress("localhost", 27017),
                    new MongoServerAddress("localhost", 27027),
                    new MongoServerAddress("localhost", 27037)
                },
                ReplicaSetName="TestReplicaSet"
            });
            ```

4. 使用 MongoUrl
    - 語法

        ```cs
        MongoClient client = new MongoClient(
                MongoUrl.Create("mongodb://{username}:{password}@{host1}:{port1},{host2}:{port2},{host3}:{port3}/?replicaSet={replicaSet Name}"));
        ```

    - 實際範例

        ```cs
        MongoClient client = new MongoClient(
                MongoUrl.Create("mongodb://yowko:pass.123@127.0.0.1:27017,127.0.0.1:27027,127.0.0.1:27037/?replicaSet=TestReplicaSet"));
        ```

## 心得

原本只是沒營養地紀錄一下原本預估一 ~ 二小時可以搞定，出乎意料地花了一整天的時間，之前我就有一樣的感覺：對於 MongoDB 常常覺得不用紀錄的東西，但下次真的要用時又要花不少時間摸索，希望再加上這次經驗後 下次使用起來會順手些

## 參考資訊

1. [How to MongoDB in C# – Part 2](https://blog.oz-code.com/how-to-mongodb-in-c-part-2/)
2. [Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/)
3. [Authenticate to MongoDB with the C# Driver](https://mongodb-documentation.readthedocs.io/en/latest/ecosystem/tutorial/authenticate-with-csharp-driver.html)
