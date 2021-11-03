---
title: "如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 3 (使用 Lists 型別)"
date: 2017-02-23T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp","Redis"]
slug: "dotnet-redis-lists"
aliases:
    - /2017/02/dotnet-use-redis-lists.html
    - /2017/02/dotnet-redis-lists/
---
## 如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 3 (使用 Lists 型別)

先前文章 [如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 2 (使用 Hashes 型別)](http://blog.yowko.com/2017/02/dotnet-use-redis-hash.html) 使用了 Hashed 型別來 cache 資料，但總覺得不太清楚各型別間的差異加上 Redis 型別不多，打算每個來用用看，再好好比較一番

## 基礎建設

1. 處理 redis 連線 factory
    - 使用 StackExchange.Redis 套件，記得 nuget 安裝

        ```cs
        public static class RedisConnectionFactory
        {
            private static readonly Lazy<ConnectionMultiplexer> Connection;
            static RedisConnectionFactory()
            {
                var connectionString = "localhost:6379";
                var options = ConfigurationOptions.Parse(connectionString);
                Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
            }
            public static ConnectionMultiplexer GetConnection => Connection.Value;
            public static IDatabase RedisDB => GetConnection.GetDatabase();
        }
        ```

2. 測試用自訂 class

    ```cs
    public class Person
    {
        public string ID{ get; set; }
        
        public string Name { get; set; }
    
        public string Tel { get; set; }
    }
    ```

3. 存取 redis 用的 object

    ```cs
    IDatabase _db = RedisConnectionFactory.RedisDB;
    ```

4. 製造假資料

    ```cs
    Dictionary<string, List<Person>> peopleDic = new Dictionary<string, List<Person>>();
    for (int i = 0; i < 3; i++)
    {
        var _id = Guid.NewGuid().ToString();
        List<Person> people = new List<UserQuery.Person>();
        for (int j = 0; j < 3; j++)
        {
            people.Add((new Person { ID = $"{i}_{j}_ID", Name = $"{i}_{j}_yowko", Tel = $"{i}_{j}_0123456789" }));
        }
        peopleDic.Add(_id, people);
    }
    ```

5. 使用 batch 來批次處理指令

    ```cs
    var batch = _db.CreateBatch();
    ```

## 使用 Lists 型別

相關指令介紹

指令| 參數 |對應 StackExchange.Redis 指令| 說明
:---|:---|:-------|:---
BLPOP| key [key ...] timeout|-|刪除並取得該 list 中的第一元素，或 blocking 直到有一個可用
BRPOP| key [key ...] timeout|-|刪除並取得該 list 中的最後一個元素，或 blocking 直到有一個可用
BRPOPLPUSH| source destination timeout|-|取出一個list 的值，將它推到另一個 list，並傳回它;或 blocking 直到有一個可用
LINDEX| key index|ListGetByIndex｜ListGetByIndexAsync|使用 index 取得一個元素
LINSERT| key BEFORE｜AFTER pivot value|ListInsertAfter｜ListInsertAfterAsync<br/>｜ListInsertBefore｜ListInsertBeforeAsync|在 list 中的另一個元素之前或之後插入一個元素
LLEN| key|ListLength｜ListLengthAsync|取得 List的長度
LPOP| key|ListLeftPop｜ListLeftPopAsync|從 list 的左邊取出一個元素
LPUSH| key value [value ...]|ListLeftPush｜ListLeftPushAsync|從 list 的左邊插入一個或多個元素
LPUSHX| key value|ListLeftPush｜ListLeftPushAsync|當 list 存在時，從 list 左邊插入一個元素
LRANGE|key start stop|ListRange｜ListRangeAsync|從 list 中獲取指定返回的元素
LREM| key count value|ListRemove｜ListRemoveAsync|從 list 中刪除元素
LSET| key index value|ListSetByIndex｜ListSetByIndexAsync|設置 list 裡面一個元素的值
LTRIM| key start stop|ListTrim｜ListTrimAsync|刪除到指定範圍內的清單
RPOP| key|ListRightPop｜ListRightPopAsync|從 list 的右邊取出一個元
RPOPLPUSH| source destination|ListRightPopLeftPush｜ListRightPopLeftPushAsync|刪除 list 中的最後一個元素，將其加到另一個 list
RPUSH| key value [value ...]|ListRightPush｜ListRightPushAsync|從 list 的右邊插入一個元素
RPUSHX| key value|ListRightPush｜ListRightPushAsync|從 list 的右邊插入一個元素，僅 list 存在時有效

## 實際使用

- 範例程式碼

    ```cs
    var batch = _db.CreateBatch();
    foreach (var item in peopleDic)
    {
        foreach (var element in item.Value)
        {
            _db.ListRightPushAsync(item.Key, JsonConvert.SerializeObject(element));
        }
    }
    batch.Execute();
    ```

- 儲存狀況

    ![1result](https://cloud.githubusercontent.com/assets/3851540/23221107/1693335a-f95f-11e6-8bda-476483591d18.png)

## 完整程式碼

```cs
void Main()
{
    IDatabase _db = RedisConnectionFactory.RedisDB;
    Dictionary<string, List<Person>> peopleDic = new Dictionary<string, List<Person>>();
    for (int i = 0; i < 3; i++)
    {
        var _id = Guid.NewGuid().ToString();
        List<Person> people = new List<UserQuery.Person>();
        for (int j = 0; j < 3; j++)
        {
            people.Add((new Person { ID = $"{i}_{j}_ID", Name = $"{i}_{j}_yowko", Tel = $"{i}_{j}_0123456789" }));
        }
        peopleDic.Add(_id, people);
    }
    var batch = _db.CreateBatch();
    foreach (var item in peopleDic)
    {
        foreach (var element in item.Value)
        {
            _db.ListRightPushAsync(item.Key, JsonConvert.SerializeObject(element));
        }
    }
    batch.Execute();
}

public class Person 
{
    public string ID { get; set; }

    public string Name { get; set; }

    public string Tel { get; set; }
}

public static class RedisConnectionFactory
{
    private static readonly Lazy<ConnectionMultiplexer> Connection;
    static RedisConnectionFactory()
    {
        var connectionString = "localhost:6379";
        var options = ConfigurationOptions.Parse(connectionString);
        Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
    }
    public static ConnectionMultiplexer GetConnection => Connection.Value;
    public static IDatabase RedisDB => GetConnection.GetDatabase();
}
```

## 參考資料

1. [Redis - LIST](http://redis.cn/commands.html#list)
2. [GitHub - StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/94213e9a16bd651fd20ccade080c3d8cd96e85bb/StackExchange.Redis/StackExchange/Redis/RedisDatabase.cs)
