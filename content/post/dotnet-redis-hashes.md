---
title: "如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 2 (使用 Hashes 型別)"
date: 2017-02-21T01:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["C#","Redis"]
slug: "dotnet-redis-hashes"
aliases:
    - /2017/02/dotnet-use-redis-hash.html
---
# 如何在 .NET 程式中使用 Redis 做為 Cache Server - Part 2 (使用 Hashes 型別)
在 [如何在 .NET 程式中使用 Redis 做為 Cache Server](https://blog.yowko.com/2017/02/use-redis-cache-server-for-dotnet.html) 一文中把原本使用 .NET 的 MemoryCache 改為使用 Redis，其中用的是 Redis 最基本的型別 `string` ，今天將會用 `Hash` 型別來改寫，你可能跟我一樣想瞭解 Redis 各型別的 `實際` 差異，雖然官網上把各型別內容都清楚說明還附上 big O - O(1/n) 資訊，但我還是搞不清楚什麼情境要用哪個XD，所以打算來親身體驗一下其中差異，最後有空再來個大比較，今天就從最常被討論到的型別 `Hash` 開始

## 為什麼選擇 Hashes
Rico 大在 [Redis(5)-好用的Hash](https://dotblogs.com.tw/ricochen/2017/01/15/233948) 中提到兩個關鍵

1. Hashes 有經過記憶體優化，效能是比較高的
2. 不用像 string 會 lock 整個 entity ，可針對單一屬性更新

## 基本用法
1. Hashes 的儲存方式是 `key : HashEntry[]`
2. HashEntry 的內容是 `key : value`

如果有個 list ，而 list 中的物件又有好幾個屬性，你會不會跟我一樣懶得慢慢處理，甚至思考著是不是乾脆用 `string` 就好，反正 cache 本來就不是拿來常更新的嘛，但想要成為好工程師的大家怎麼可能真的這麼放棄了

## 如何改良
1. 新增 abstract class
    - 準備讓其他 class 繼承用 
2. 在 abstract class 加上 ToRedisHash 方法
    - 讓其他 class 可以直接呼叫使用，減少重複的 code 
3. ToRedisHash 方法會回傳 `KeyValuePair<string, HashEntry[]>`
    - 使用前一篇文章 [c# 如何用特定的 attribute 取得 property 資訊](https://blog.yowko.com/2017/02/c-sharp-use-specific-attribute-get-property-info.html) 提到的技巧，會將 class object 使用 reflection 組成需要的格式
4. 將需要 Redis Hash 的 class 繼承 abstract class 
5. 在需要 cache 的 class 上套用 Keyattribute 用來標記為 key
    - 需要 key-vlaue 的結構，所以需要有一個 key 

## 範例程式碼
1. abstract class
    
    ```cs
    public abstract class RedisHashExtension
    {
        public KeyValuePair<string, HashEntry[]> ToRedisHash()
        {
            object obj = this;
            var key = GetType().GetProperties().FirstOrDefault(prop => prop.IsDefined(typeof(KeyAttribute), false));
            if (key == null)
                throw new InvalidDataException("miss property with key attibute");
            var keystr = key.GetValue(obj).ToString();
            var props = GetType().GetProperties().Where(d => d.Name != key?.Name);
            List<HashEntry> hashentrys = new List<StackExchange.Redis.HashEntry>();
            foreach (var item in props)
            {
                hashentrys.Add(new HashEntry(item.Name.ToString(), item.GetValue(this)?.ToString()));
            }

            return new KeyValuePair<string, HashEntry[]>(keystr, hashentrys.ToArray());
        }
    }
    ```
2. 需要 cache 的 class
    
    ```cs
    public class PersonInfo : RedisHashExtension
    {
    	[Key]
    	public Guid ID{ get; set; }
    	
    	public string Name { get; set; }
    
    	public string Tel { get; set; }
    }
    ```
3. RedisConnectionFactory
    
    ```cs
    private static readonly Lazy<ConnectionMultiplexer> Connection;
	static RedisConnectionFactory()
	{
		var connectionString = "localhost:6379";
		var options = ConfigurationOptions.Parse(connectionString);
		Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
	}
	public static ConnectionMultiplexer GetConnection => Connection.Value;
	public static IDatabase RedisDB => GetConnection.GetDatabase();
    ```
4. 實際使用
    
    ```cs
    IDatabase _db = RedisConnectionFactory.RedisDB;
    //製造假資料
    List<KeyValuePair<string, HashEntry[]>> people= new List<KeyValuePair<string, HashEntry[]>>();
    for (int i = 0; i < 3; i++)
    {
    		people.Add((new Person { ID = Guid.NewGuid(), Name = $"{i}_yowko", Tel = $"{i}_0123456789"}).ToRedisHash());
    }
    //寫入 redis
    foreach (var item in people)
    {
    	_db.HashSetAsync(item.Key,item.Value);	
    }
    //取資料
    _db.HashGet("{key}","{field}")
    ```

## 還可以更好嗎？
雖然 Hashes 結構比較好，但一筆一筆寫入的作法實在不合理，網路 io 可能就把改善的效能吃光了，這時候可以利用 Redis batch 的功能

1. 建立 batch
    
    ```cs
    var batch = _db.CreateBatch();
    ```
2. 加 cache 資料逐一加入 batch 中
    
    ```cs
    batch.HashSetAsync(item.Key,item.Value);
    ```
3. 執行 batch
    
    ```cs
    batch.Execute();
    ```
4. 範例程式碼
    
    ```cs
    IDatabase _db = RedisConnectionFactory.RedisDB;
    //製造假資料
    List<KeyValuePair<string, HashEntry[]>> people= new List<KeyValuePair<string, HashEntry[]>>();
    for (int i = 0; i < 3; i++)
    {
    		people.Add((new Person { ID = Guid.NewGuid(), Name = $"{i}_yowko", Tel = $"{i}_0123456789"}).ToRedisHash());
    }
    //建立 batch
    var batch = _db.CreateBatch();
    foreach (var item in people)
    {
        //加到 batch 中
    	batch.HashSetAsync(item.Key,item.Value);
    }
    //執行 batch 內容
    batch.Execute();
    ```

## Hashes 指令介紹

指令|參數|對應 StackExchange.Redis 指令 |說明
:---|:---|:---|:---
HDEL| key field [field ...]|HashDelete/HashDeleteAsync|刪除一個或多個 hash 的field
HEXISTS| key field|HashExists/HashExistsAsync|判斷 field 是否存在於 hash 中
HGET| key field|HashGet/HashGetAsync|取得 hash(key) 中 field 的值
HGETALL| key| HashGetAll/HashGetAllAsync|從 hash(key) 中讀取全部的 field 和 value
HINCRBY| key field increment|HashIncrement/HashIncrementAsync|將 hash(key) 中指定 field 的值增加 increment
HINCRBYFLOAT|HashIncrement/HashIncrementAsync| key field increment|將 hash(key) 中指定 field 的值增加 increment
HKEYS| key|HashKeys/HashKeysAsync|取得 hash(key) 的所有 field
HLEN |key|HashLength/HashLengthAsync|取得 hash(key) 裡所有 field 的數量
HMGET| key field [field ...]|HashGetAll/HashGetAllAsync|取得 hash(key) 裡面指定 field 的值
HMSET| key field value [field value ...]|HashSet/HashSetAsync|設定 hash(key) 一個或多個 field 與 value
HSET| key field value|HashSet/HashSetAsync|設定 hash(key) 一個 field 的值為 value
HSETNX |key field value|HashSet/HashSetAsync/<br/>HashSetIfNotExistsAsync|設定 hash(key) 的一個 field 值為 value ，只有當這個 field 不存在時有效
HSTRLEN| key field|-|取得 hash(key) 裡面指定 field 的長度
HVALS |key|HashValues/HashValuesAsync|取得 hash(key) 的所有值
HSCAN |key cursor [MATCH pattern] [COUNT count]|HashScan|逐一列出 hash(key) 裡面的元素


# 參考資料
1. [Redis(5)-好用的Hash](https://dotblogs.com.tw/ricochen/2017/01/15/233948)
2. [StackExchange.Redis/MigratedBookSleeveTestSuite/Batches.cs](https://github.com/StackExchange/StackExchange.Redis/blob/master/MigratedBookSleeveTestSuite/Batches.cs)