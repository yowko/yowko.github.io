---
title: "如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 5 (使用 Sorted Sets 型別)"
date: 2017-02-27T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp","Redis"]
slug: "dotnet-redis-sorted-sets"
aliases:
    - /2017/02/dotnet-redis-use-sorted-set.html
---
## 如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 5 (使用 Sorted Sets 型別)

今天來看看 Sorted Sets 該怎麼使用，建議可與 [如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 4 (使用 Sets 型別)](/dotnet-redis-use-sets) 參照

一樣先說重點：用法與 Sets 幾乎相同，差別是 Sorted Sets 多了一個分數值可以用來自訂優先順序，如果有興趣多瞭解 StackExchange.Redis 指令對應文中有對應與簡介說明表

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

4. 使用 batch 來批次處理指令

    ```cs
    var batch = _db.CreateBatch();
    ```

5. 製造假資料

    ```cs
    Dictionary<string, List<Person>> peopleDic = new Dictionary<string, List<Person>>();
    for (int i = 0; i < 3; i++)
    {
        var _id = Guid.NewGuid().ToString();
        List<Person> people = new List<UserQuery.Person>();
        for (int j = 0; j < 3; j++)
        {
            var element = new Person { ID = $"{i}_{j}_ID", Name = $"{i}_{j}_yowko", Tel = $"{i}_{j}_0123456789" };
            _db.SortedSetAddAsync(_id, JsonConvert.SerializeObject(element), i * 10 + j);
        }
    }
    ```

## 使用 Sorted Sets 型別

- 相關指令介紹

    指令|參數|對應 StackExchange.Redis 指令 |說明
    :---|:---|:---|:---
    ZADD| key [NX｜XX] [CH] [INCR] score member [score member ...]|SortedSetAdd/SortedSetAddAsync|加一個或多個成員到 sorted set (如果 key 已經存在則更新的分數)
    ZCARD| key|SortedSetLength/SortedSetLengthAsync|傳回特定 key 的元素數量
    ZCOUNT| key min max|SortedSetLength/SortedSetLengthAsync|傳回分數 min 至 max 範圍內的元素數量
    ZINCRBY| key increment member|SortedSetIncrement<br/>SortedSetIncrementAsync|將 key 元素 member 的分數值加上 increment
    ZINTERSTORE| destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM｜MIN｜MAX]|SortedSetCombineAndStore<br/>SortedSetCombineAndStoreAsync|將指定數量(numkeys) 的 sorted set(key) 交集結果，儲存至 destination
    ZREVRANGEZLEXCOUNT| key min max|-|傳回分數介於 min - max 間的元素數量
    ZRANGE| key start stop [WITHSCORES]|SortedSetRangeByRank<br/>SortedSetRangeByRankAsync<br/>SortedSetRangeByRankWithScores<br/>SortedSetRangeByRankWithScoresAsync|取得 key 中從 start 至 stop 的元素
    ZRANGEBYLEX| key min max [LIMIT offset count]|SortedSetRangeByValue<br/>SortedSetRangeByValueAsync|取得 key 中依名稱正向排序後從 min 至 max 間分數相同的元素
    ZREVRANGEBYLEX| key max min [LIMIT offset count]|-|取得 key 中依名稱逆向排序後從 max 至 min 間分數相同的元素
    ZRANGEBYSCORE| key min max [WITHSCORES] [LIMIT offset count]|SortedSetRangeByScore<br/>SortedSetRangeByScoreAsync<br/>SortedSetRangeByScoreWithScores<br/>SortedSetRangeByScoreWithScoresAsync|取得 key 中分數從低至高排序後從 min 至 max 的所有元素
    ZRANK| key member|SortedSetRank<br/>SortedSetRankAsync|取得 member 元素在 key 中的排序位置
    ZREM| key member [member ...]|SortedSetRemove<br/>SortedSetRemoveAsync|刪除 key 中一個或多個 member 元素
    ZREMRANGEBYLEX| key min max|SortedSetRemoveRangeByValue<br/>SortedSetRemoveRangeByValueAsync|刪除 key 中名稱按由低到高排序後 從 min 至 max 元素間所有元素
    ZREMRANGEBYRANK| key start stop|SortedSetRemoveRangeByRank<br/>SortedSetRemoveRangeByRankAsync|在排序設置的所有成員在給定的索引中刪除
    ZREMRANGEBYSCORE| key min max|SortedSetRemoveRangeByScore<br/>SortedSetRemoveRangeByScoreAsync|刪除 key 中依分數由低至高排序後，從 min 至 max 元素間所有元素
    ZREVRANGE| key start stop [WITHSCORES]|SortedSetRangeByRank<br/>SortedSetRangeByRankAsync<br/>SortedSetRangeByRankWithScores<br/>SortedSetRangeByRankWithScoresAsync|取得 key 中在分數由高至低排序後，從位置 start 至 stop 的所有元素
    ZREVRANGEBYSCORE| key max min [WITHSCORES] [LIMIT offset count]|SortedSetRangeByRank<br/>SortedSetRangeByRankAsync<br/>SortedSetRangeByRankWithScores<br/>SortedSetRangeByRankWithScoresAsync|取得 key 中分數由高至低排序後 從分數 max 至 min 的所有元素
    ZREVRANK| key member|SortedSetRank<br/>SortedSetRankAsync|取得 key 中分數由高至低排序後 member 的排序位置
    ZSCORE| key member|SortedSetScore<br/>SortedSetScoreAsync|取得 key 中 member 的分數
    ZUNIONSTORE| destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM｜MIN｜MAX]|SortedSetCombineAndStore<br/>SortedSetCombineAndStoreAsync|將指定數量(numkeys) 的 sorted set (key) 聯集結果，儲存至 destination
    ZSCAN| key cursor [MATCH pattern] [COUNT count]|SortedSetScan|逐一列出 sorted set (key) 中的元素

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

    ![1result](https://cloud.githubusercontent.com/assets/3851540/23340518/8edef54a-fc72-11e6-9b33-d316ed1acc69.png)

## 完整程式碼

```cs
void Main()
{
    IDatabase _db = RedisConnectionFactory.RedisDB;
    Dictionary<string, List<Person>> peopleDic = new Dictionary<string, List<Person>>();
    var batch = _db.CreateBatch();
    for (int i = 0; i < 3; i++)
    {
        var _id = Guid.NewGuid().ToString();
        List<Person> people = new List<UserQuery.Person>();
        for (int j = 0; j < 3; j++)
        {
            var element = new Person { ID = $"{i}_{j}_ID", Name = $"{i}_{j}_yowko", Tel = $"{i}_{j}_0123456789" };
            _db.SortedSetAddAsync(_id, JsonConvert.SerializeObject(element), i * 10 + j);
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
```

## 參考資料

1. [Redis - Sorted Sets](http://redis.cn/topics/data-types-intro.html#sorted-sets)
2. [GitHub - StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/94213e9a16bd651fd20ccade080c3d8cd96e85bb/StackExchange.Redis/StackExchange/Redis/RedisDatabase.cs)
