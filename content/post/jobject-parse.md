---
title: "從 Json String 中取得物件的屬性值"
date: 2017-07-30T23:55:00+08:00
lastmod: 2018-09-24T00:41:54+08:00
draft: false
tags: ["套件","C#"]
slug: "jobject-parse"
aliases:
    - /2017/07/jobject-parse.html
---
# 從 Json String 中取得物件的屬性值
之前文章 [使用 string 建立 instance 及反序列化 json 為 class](https://blog.yowko.com/2017/07/string-create-instance.html) 介紹到如何使用 string 產生 instance 及反序列化 json 為 class，後來同事看到寫法建議可以用 json.net 的 `JObject.Parse` 語法

後來試了一下，覺得滿方便的，但用途不太一樣，順手紀錄一下

## 基本設定

本文會延續 [使用 string 建立 instance 及反序列化 json 為 class](https://blog.yowko.com/2017/07/string-create-instance.html) 來進行比較，詳細資訊可以參考 使用 [string 建立 instance 及反序列化 json 為 class](https://blog.yowko.com/2017/07/string-create-instance.html) 內容

1.  共用的 model class

    ```cs
    public class User
    {
        public Guid _internalId { get; set; }
        public string Id { get; set; }
        public string Name { get; set; }
        public string Addr { get; set; }
        public string Phone { get; set; }
        public int BirthMonth { get; set; }
    }
    ```

2.  呼叫遠端方法時，一併提供


    *   class 的 `AssemblyQualifiedName`
    *   model instance 的 json string

3.  遠端需要取得 json string 的屬性值進行操作


## 將 json string 反序列化指定 class

```cs
void Deserialize(string typename, string jsonstr)
{
    Type dataType = Type.GetType(typename);
    var obj = JsonConvert.DeserializeObject(jsonstr, dataType);
    // do something
}
```

## 改使用 JObject.Parse

1.  發送端程式碼

    ```cs
    void Main()
    {
        var typename = typeof(User).AssemblyQualifiedName;
        User dto = new User() { _internalId = Guid.NewGuid(), Name = "Yowko", Addr = "Taipei", BirthMonth = 7, Id = "A123456789", Phone = "09123456789" };
        var jsonstr = JsonConvert.SerializeObject(dto);
        SendObj(typename, jsonstr);
    }
    void SendObj(string typename, string jsonstr)
    {
        //web request
    }
    public class PstData
    {
        public string typename { get; set; }
        public string jsonstr { get; set; }
    }
    ```

2.  接收端程式碼

    ```cs
    public IHttpActionResult Post([FromBody]PstData value)
    {
        Deserialize(value.typename, value.jsonstr);
    }
    void Deserialize(string typename, string jsonstr)
    {
        JObject.Parse(jsonstr).Dump();
        // do something
    }
    ```
3.  實際效果

    ![1jobjectparse](https://user-images.githubusercontent.com/3851540/28755226-93bbf434-7588-11e7-815b-d8056c7c210f.png)

## 兩者效果比較

![2compare](https://user-images.githubusercontent.com/3851540/28755227-93e78fae-7588-11e7-84aa-0cd21aeee3fd.png)

*   結果型別不同
    *   JsonConvert.DeserializeObject 會在執行時期將 json string 轉型為目標型別
    *   JObject.Parse 會將 json string 轉為 [JObject](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm)

*   特定屬性操作
    *   JsonConvert.DeserializeObject 在開發時間僅擁有 object 的相關 api 可用


        > 只能使用 reflection
        > 
        >     *   obj.GetType().GetProperties().Skip(1).Take(1).FirstOrDefault().GetValue(obj);

    *   JObject.Parse 可以使用 [JObject](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm) 功能
        
        > 可以指定屬性名稱或是屬性位置取得實際值
        > 
        >     *   obj.Property("Name")
        >     *   obj.Properties().AsJEnumerable().Skip(1).Take(1)

## 心得

如果需要完整將 json string 還原為特定 class，就使用 `JsonConvert.DeserializeObject`，可以取得完整的屬性名稱及值

如果只需取得值來進行操作，就使用 `JObject.Parse` 相對方便，也不用擔心，class 名稱錯誤或是轉型失敗的問題

# 參考資訊

1.  [使用 string 建立 instance 及反序列化 json 為 class](https://blog.yowko.com/2017/07/string-create-instance.html)
2.  [JObject](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm)
3.  [JObject.Parse Method](http://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_Linq_JObject_Parse.htm)
