---
title: "C# 使用 Lua 取得 Redis 自訂複雜型別"
date: 2019-12-09T22:30:00+08:00
lastmod: 2016-12-09T22:30:31+08:00
draft: false
tags: ["csharp","Redis","Lua"]
slug: "csharp-lua-redis-custom-type"
---

## C# 使用 Lua 取得 Redis 自訂複雜型別

之前筆記 [在 Redis 中使用 Lua 的 Dictionary](https://blog.yowko.com/redis-lua-dcitionary/) 紀錄到在 Redis 中使用 Lua 時可以如何模擬並使用 Dictionary，而筆記最後也提到透過這樣的方式處理時，Lua 的回傳值會不如預期，所以今天要來紀錄個人的做法，但我無法確定是不是最好的方法 XD，有錯請大家指教了

## 基本環境說明

1. macOS Catalina 10.15.1
2. .NET Core SDK 3.1.100
3. redis-cli 5.0.5
4. NuGet packages

    - StackExchange.Redis 2.0.601
    - Newtonsoft.Json 12.0.3
    - MessagePack 1.8.80

5. 測試用 model

    ```cs
    public class User
    {
        public int UserId { get; set; }

        public string Name { get; set; }

        public string Email { get; set; }
    }
    ```

## 原始 lua 內容

詳細內容可以參考 [在 Redis 中使用 Lua 的 Dictionary](https://blog.yowko.com/redis-lua-dcitionary/)

```lua
local allUsersKey="users"
local userdataPrefixKey="userdata"
-- 取得所有 user
local allUsers = redis.call("smembers",  allUsersKey)
--逐一處理所有 user
local results = {} for userIndex, userId in ipairs(allUsers) do
    -- 單一 user 的 key
    local userdataKey=userdataPrefixKey .. ":" .. userId
    -- 取得單一 user 的所有屬性值
    local getAll= redis.call("HGETALL", userdataKey)
    -- 暫存欄位名稱
    local tmpSubKey=""
    -- 處理所有欄位屬性
    local tmpSubData = {}
    -- 將 userId 當做 object 的一個屬性
    tmpSubData["userId"]=userId
    for index,value in ipairs (getAll) do
    -- 如果 index 是單數則為欄位名稱，偶數為屬性值
    if index % 2 == 1 then
        tmpSubKey= value
    else
        -- 使用名稱設定 table 
        tmpSubData[tmpSubKey]=value
    end
    end
    -- 將個別 user object 存入 table 中
    table.insert(results, tmpSubData)
end
return results
```

## 回傳 json

將最終結果使用 `cjson.encode` 轉為 json

- lua

    ```lua
    local allUsersKey="users"
    local userdataPrefixKey="userdata"

    -- 取得所有 user
    local allUsers = redis.call("smembers",  allUsersKey)

    --逐一處理所有 user
    local results = {} for userIndex, userId in ipairs(allUsers) do
      -- 單一 user 的 key
      local userdataKey=userdataPrefixKey .. ":" .. userId
      -- 取得單一 user 的所有屬性值
      local getAll= redis.call("HGETALL", userdataKey)
      -- 暫存欄位名稱
      local tmpSubKey=""
      -- 處理所有欄位屬性
      local tmpSubData = {}
      -- 將 userId 當做 object 的一個屬性
      tmpSubData["userId"]=userId
      for index,value in ipairs (getAll) do
        -- 如果 index 是單數則為欄位名稱，偶數為屬性值
        if index % 2 == 1 then
          tmpSubKey= value
        else
          -- 使用名稱設定 table 
          tmpSubData[tmpSubKey]=value
        end
      end
      -- 將個別 user object 存入 table 中
      table.insert(results, tmpSubData)
    end
    return cjson.encode(results)
    ```

- C#

    ```cs
    //準備 redis 連線
    var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379");
    var db = redis.GetDatabase();

    //從專案中取得 lua (lua 設為 EmbeddedResource)
    var script = string.Empty;
    var assembly = Assembly.GetExecutingAssembly();
    const string resourceName = "TestRedisLuaDictionary.user.lua";
    //載入 lua 檔案
    await using (var stream = assembly.GetManifestResourceStream(resourceName))
    {
        using var reader = new StreamReader(stream);
        script = reader.ReadToEnd();
    }
    //將文字內容轉為 lua
    var prepared = LuaScript.Prepare(script);
    //執行 lua
    var redisResult = db.ScriptEvaluate(prepared);
    //將 redis 回傳內容使用 json 轉為物件
    var result = JsonConvert.DeserializeObject<User[]>(redisResult.ToString());

    Console.WriteLine(result);
    ```

- 實際結果

    ```txt
    [{"userId":"1","name":"yowko","email":"yowko@yowko.com"},{"userId":"3","name":"test","email":"test@test.com"},{"userId":"5","name":"poc","email":"poc@poc.com"}]
    ```

## 回傳 MessagePack

- lua

    ```lua
    local allUsersKey="users"
    local userdataPrefixKey="userdata"

    -- 取得所有 user
    local allUsers = redis.call("smembers",  allUsersKey)

    --逐一處理所有 user
    local results = {} for userIndex, userId in ipairs(allUsers) do
    -- 單一 user 的 key
    local userdataKey=userdataPrefixKey .. ":" .. userId
    -- 取得單一 user 的所有屬性值
    local getAll= redis.call("HGETALL", userdataKey)
    -- 暫存欄位名稱
    local tmpSubKey=""
    -- 處理所有欄位屬性
    local tmpSubData = {}
    -- 將 userId 當做 object 的一個屬性
    tmpSubData["userId"]=userId
    for index,value in ipairs (getAll) do
        -- 如果 index 是單數則為欄位名稱，偶數為屬性值
        if index % 2 == 1 then
        tmpSubKey= value
        else
        -- 使用名稱設定 table 
        tmpSubData[tmpSubKey]=value
        end
    end
    -- 將個別 user object 存入 table 中
    table.insert(results, tmpSubData)
    end

    return cmsgpack.pack(results)
    ```

- C#

    - 轉換程式

        > 相關詳細說明可以參考之前筆記 [C# - Property 與 Value 的 Dictionary 轉為 Object](https://blog.yowko.com/property-value-dictionary-to-object/)

        ```cs
        private static T DictionaryToObject<T>(IDictionary<String, Object> dictionary) where T : class, new()
        {
            // 取得 T 所有 property
            var myPropertyInfo = typeof(T).GetProperties();
            // 將 property 的 name 轉為小寫當 key，value 為原始大小寫，讓傳入的資料無論大小寫皆可轉換
            var properties = myPropertyInfo
                .Select(a => new KeyValuePair<string, string>(a.Name.ToLowerInvariant(), a.Name))
                .ToDictionary(a => a.Key, a => a.Value);

            // 建立 T 實體
            var instance = Activator.CreateInstance<T>();

            //處理所有欄位
            foreach (var (key, value) in dictionary)
            {
                var name = key.ToLowerInvariant();

                //欄位名稱不存在就換下一個
                if (!properties.TryGetValue(name, out var property)) continue;

                var prop = typeof(T).GetProperty(property);

                //依據不同型別來做轉換，只意思寫 int 與 string，請自行擴充
                switch (prop.PropertyType)
                {
                    case { } intType when intType == typeof(int):
                        prop.SetValue(instance, Convert.ToInt32(value), null);

                        break;
                    case { } stringType when stringType == typeof(string):
                        prop.SetValue(instance, Convert.ToString(value), null);

                        break;
                }
            }

            return instance;
        }
        ```

    - 實際使用

        ```cs
        //準備 redis 連線
        var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379");
        var db = redis.GetDatabase();

        //從專案中取得 lua (lua 設為 EmbeddedResource)
        var script = string.Empty;
        var assembly = Assembly.GetExecutingAssembly();
        const string resourceName = "TestRedisLuaDictionary.user.lua";
        //載入 lua 檔案
        await using (var stream = assembly.GetManifestResourceStream(resourceName))
        {
            using var reader = new StreamReader(stream);
            script = reader.ReadToEnd();
        }
        //將文字內容轉為 lua
        var prepared = LuaScript.Prepare(script);
        //執行 lua
        var redisResult = db.ScriptEvaluate(prepared);
        //將 redis 回傳內容轉為 byte[] 再透過 MessagePack 反序列化為 Dictionary<string, object>[]
        var result = MessagePackSerializer
                .Deserialize<Dictionary<string, object>[]>((byte[]) redisResult);

        // 將 Dictionary<string, object>[] 轉為 User
        var outputResult = result.Select(DictionaryToObject<User>);

        Console.WriteLine(JsonConvert.SerializeObject(outputResult));
        ```

- 實際結果

    ```txt
    [{"UserId":1,"Name":"yowko","Email":"yowko@yowko.com"},{"UserId":3,"Name":"test","Email":"test@test.com"},{"UserId":5,"Name":"poc","Email":"poc@poc.com"}]
    ```

    - redis 回傳的 MessagePack 原始資料

        ```txt
        ���userId�1�name�yowko�email�yowko@yowko.com��userId�3�name�test�email�test@test.com��userId�5�name�poc�email�poc@poc.com
        ```

## 心得

透過 json 與 MessagePack 都可以將 redis 中 lua 回傳自訂型別的 dictionary 轉換為 object，當然 json 在使用上方便許多：程式碼較少，redis 的回傳內容可讀性也高，但 MessagePack 有壓縮的功用，在傳輸速度上有優勢，這個就看專案需求來選擇囉

## 參考資訊

1. [EVAL script numkeys key [key ...] arg [arg ...]](https://redis.io/commands/eval)
2. [C# - Property 與 Value 的 Dictionary 轉為 Object](https://blog.yowko.com/property-value-dictionary-to-object/)