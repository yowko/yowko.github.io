---
title: "使用 string 建立 instance 及反序列化 json 為 class"
date: 2017-07-13T23:55:00+08:00
lastmod: 2021-10-15T00:09:03+08:00
draft: false
tags: ["csharp"]
slug: "string-create-instance"
aliases:
    - /2017/07/string-create-instance.html
---
## 使用 string 建立 instance 及反序列化 json 為 class

最近的專案在架構拆分時，將可能可以共用的部份獨立設計成開放式的服務，將執行行為也當做輸入參數的一部份，直接交由使用端自行定義。雖然功能開發時遇到不少問題需要逐一的克服，卻可以擁有極大的彈性

今天就針對其中的一小部份語法做個紀錄：如何使用 string 產生 instance 及反序列化 json 為 class

## 基本設定

1. 有個 Model class 為 User，獨立於各個專案之外，其他專案皆引用 dll 來使用 User

    ```cs
    public class User
    {
        public Guid _internalId{ get; set; }
        public string Id { get; set; }
        public string Name { get; set; }
        public string Addr { get; set; }
        public string Phone { get; set; }
        public int BirthMonth { get; set; }
    }
    ```

2. 輸入端需要傳送 class 的 TypeName
3. 接收端使用 class 的 TypeName 將已轉換為 json string 的 User 進行反序列化

## 關於 class 的 TypeName

* 組成

    > 命名空間，加上逗號，後面接著組件的顯示名稱 (`Namespace,AssemblyName`)

* 範例

    > `TestForNuget.User, TestForNuget, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`

* 符號含義

    |分隔符號|意義|
    |--- |--- |
    |反斜線 ()|跳逃字元|
    |逗號 （，）|後面接著組件名稱。|
    |加號 （+）|之前的巢狀的類別。|
    |句號 （.）|代表命名空間識別項。|
    |括號 ([])|類型名稱後面，表示該類型的陣列<br/>-或-<br/>對於泛型類型，封入泛型型別引數清單<br/>-或-<br/>在型別引數清單中，會封裝至組件限定的類型。|

* 取得方式

    > 使用 `AssemblyQualifiedName` 屬性

    ```cs
    var typename=typeof(User).AssemblyQualifiedName;
    ```

* 其他方式？

    1. 先取得 assembly name

        ```cs
        Assembly asm = typeof(User).Assembly;
        asm.FullName.Dump();
        //以下為極簡寫法
        //typeof(User).Assembly.FullName.Dump();
        ```

    2. 取得 class 名稱

        ```cs
        User dto = new User();
        dto.GetType().FullName.Dump();
        //以下為極簡寫法
        //typeof(User).FullName.Dump();
        ```

    3. 將兩者組合

        ```cs
        $"{dto.GetType().FullName}, {asm.FullName}".Dump();
        ```

    4. 結果與直接使用 `AssemblyQualifiedName` 相同，但實際測試下，如果該 class 在建立 instance 前已被使用過就可以成功建立，但如果從未使用不使用 `AssemblyQualifiedName` 會無法建立 instance

## 將 string 轉為 class Type

```cs
Type dataType = Type.GetType(typename);
```

> 這邊的 dataType 如果為 null ，請檢查 typename 是否為合格的 typeName

## 使用 Type 建立 instance

* 寫法 一

    > 使用 assembly name 與 type name 來建立 instance，需多一層轉換

    ```cs
    //使用 assembly name 與 type name 建立 instance
    var tempInstance = Activator.CreateInstance(asm.FullName, dto.GetType().FullName);
    //將封裝的 instance 拆封，並轉為正確 class type
    var realInstance = Convert.ChangeType(tempInstance.Unwrap(), parameterDataType);
    ```

* 寫法 二

    > 直接使用 `AssemblyQualifiedName` 來建立 instance

    ```cs
    var parameterInstance = Activator.CreateInstance(dataType);
    ```

## 將 json string 反序列化為指定 class Type

```cs
JsonConvert.DeserializeObject(jsonstr,dataType)
```

## 完整成果程式碼

1. 發送端

    ```cs
    void Main()
    {
        var typename = typeof(User).AssemblyQualifiedName;
        User dto = new User() { _internalId = Guid.NewGuid(), Name = "Yowko", Addr = "Taipei", BirthMonth = 7, Id = "A123456789", Phone = "09123456789" };
        var jsonstr = JsonConvert.SerializeObject(dto);
        SendObj(typename,jsonstr);
    }
    void SendObj(string typename,string jsonstr)
    {
        //web request
    }
    ```

2. 接收端

    ```cs
    public IHttpActionResult Post([FromBody]PstData value)
    {
        Deserialize(value.typename,value.jsonstr);
    }
    void Deserialize(string typename,string jsonstr)
    {
        Type dataType = Type.GetType(typename);
        var obj = JsonConvert.DeserializeObject(jsonstr, dataType);
        // do something
    }
    public class PstData { 
        public string typename { get; set; }
        public string jsonstr { get; set; }
    }
    ```

## 心得

就完成的程式碼來看，程式碼數量非常少，但卻花了不少時間才找到正確的方法跟 debug。不得不承認對於 C# 語法基礎還是有很多需要好好努力的地方，透過這種專案實作來嘗試不同寫法，雖然會比較辛苦，但的確是學習的好方式，如同 coding 角度的走出舒適圈 XD

## 參考資訊

1. [Type.GetType 方法 (String)](https://msdn.microsoft.com/zh-tw/library/w3f99sx1.aspx)
2. [Type.AssemblyQualifiedName Property](https://msdn.microsoft.com/en-us/library/system.type.assemblyqualifiedname.aspx)
3. [get assembly by class name](https://stackoverflow.com/questions/3657424/get-assembly-by-class-name)
