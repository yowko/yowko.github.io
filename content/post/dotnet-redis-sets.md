---
title: "如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 4 (使用 Sets 型別)"
date: 2017-02-26T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp","Redis"]
slug: "dotnet-redis-sets"
aliases:
    - /2017/02/dotnet-redis-use-sets.html
    - /2017/02/dotnet-redis-sets
---
## 如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 4 (使用 Sets 型別)

今天來看看 Sets 該怎麼使用，建議可與 [如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 3 (使用 Lists 型別)](/dotnet-use-redis-lists) 參照

先說重點：用法與 Lists 幾乎相同，差別是 Sets 的元素不允許重複，如果有興趣多瞭解 StackExchange.Redis 指令對應文中有對應與簡介說明表

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

## 使用 Sets 型別

- 相關指令介紹

    指令|參數|對應 StackExchange.Redis 指令 |說明
    :---|:---|:---|:---
    SADD| key member [member ...]|SetAdd/SetAddAsync|添加一個或者多個元素(member) 到 set(key) 裡
    SCARD| key|SetLength/SetLengthAsync|取得 set(key) 裡面的元素數量
    SDIFF| key [key ...]|SetCombine/SetCombineAsync| 取得一個 set(key) 與其他 set(key) 間的差集元素
    SDIFFSTORE| destination key [key ...]|SetCombineAndStore/SetCombineAndStoreAsync|將一個 set(key) 與其他 set 間的差集元素，儲存至 destination
    SINTER| key [key ...]|SetCombine/SetCombineAsync|取得兩個 set 的交集
    SINTERSTORE| destination key [key ...]|SetCombineAndStore/SetCombineAndStoreAsync|將一個 set(key) 與其他 set 間的交集，儲存至 destination
    SISMEMBER| key member|SetContains/SetContainsAsync|檢查 member 存在於特定 set(key)
    SMEMBERS| key|SetMembers/SetMembersAsync|取得 set(key) 裡面的所有元素
    SMOVE| source destination member|SetMove/SetMoveAsync|移動 set(source) 裡面的 元素(member) 到另一個 set(destination)
    SPOP| key [count]|SetPop/SetPopAsync|刪除並取得 set(key) 裡的 count 個元素
    SRANDMEMBER| key [count]|SetRandomMember/SetRandomMemberAsync<br/>SetRandomMembers/SetRandomMembersAsync|從 set(key) 裡面隨機獲取 count 個元素
    SREM| key member [member ...]|SetRemove/SetRemoveAsync|從 set(key) 裡刪除一個或多個元素(member)
    SUNION| key [key ...]|SetCombine/SetCombineAsync|傳回多個 set(key) 的聯集的所有元素(member)
    SUNIONSTORE| destination key [key ...]|SetCombineAndStore/SetCombineAndStoreAsync|將多個 set(key) 的聯集的所有元素，儲存至 destination
    SSCAN| key cursor [MATCH pattern] [COUNT count]|SetScan|逐一列出 set(key) 中的元素

## 實際使用

- 範例程式碼

    ```cs
    var batch = _db.CreateBatch();
    foreach (var item in peopleDic)
    {
        foreach (var element in item.Value)
        {
            _db.SetAddAsync(item.Key, JsonConvert.SerializeObject(element));
        }
    }
    batch.Execute();
    ```

- 儲存狀況

    ![1result](https://cloud.githubusercontent.com/assets/3851540/23316231/8874ec60-fb04-11e6-8dae-ef860100e3f2.png)

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
            _db.SetAddAsync(item.Key, JsonConvert.SerializeObject(element));
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

1. [Redis - Sets](http://redis.cn/topics/data-types-intro.html#sets)
2. [GitHub - StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/94213e9a16bd651fd20ccade080c3d8cd96e85bb/StackExchange.Redis/StackExchange/Redis/RedisDatabase.cs)
3. [redis set 操作(和list區別是裡面存在的元素不重複)](http://jhao6000.blog.163.com/blog/static/17464738920134342021886/)
