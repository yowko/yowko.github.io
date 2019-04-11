---
title: "使用 Dapper 將 json string 轉換為 object"
date: 2018-12-26T23:45:00+08:00
lastmod: 2018-12-26T23:44:30+08:00
draft: false
tags: ["C#","Dapper"]
slug: "dapper-json-string-to-object"
---
# 使用 Dapper 將 json string 轉換為 object
同事設計物件儲存在 DB 的 schema 時將非核心功能屬性 (e.g. 畫面顯示用或是狀態表示用) 轉為 json 放在單一欄位中而不是一一建立欄位。我覺得超酷的呀，雖然過去使用 PostgreSQL 時會刻意將非固定欄位的關連資料用 json type 儲放，但我從來沒想過可以極簡到把其他非核心的屬性都放至 json 中，同事真是強大呀

儘管在 DB 中可以是 json string 但對 application 依然是一般正常的 property，偶然間看到同事透過 JObject 手動 mapping，雖然高效但我還是擔心日後維護時可能會因為 fat finger 而出錯，於是就興起做些嘗試的想法，就來看看如何透過 Dapper 的 TypeHandler 來自動 mapping 吧

## 前提說明
1. 將 User 非主要資訊(過去經驗)以 json 存至 Titles 中

    ![1data](https://user-images.githubusercontent.com/3851540/50452987-8c889e80-0978-11e9-9154-6939051f4e89.png)
2. application 使用物件時可直接取用
    
    ![2entity](https://user-images.githubusercontent.com/3851540/50452986-8bf00800-0978-11e9-9d8c-a5c73615fca9.png)

## 1. 調整 model

- 將原為 string 的 Titles 改用實際 class - `TitleModel` 並改為 private
- 在原 class 中將 Titles property 一一列入並將 value 指向 Titles 內容

    ```cs
    class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime Birthday { get; set; }
        public string Addr { get; set; }
        private TitleModel Titles { get; set; }

        #region - 以下為 json string 中的屬性 -
        public string OrgName
        {
            get { return Titles.OrgName; }
            set { Titles.OrgName = value; }
        }
        public string Title
        {
            get { return Titles.Title; }
            set { Titles.Title = value; }
        }
        public string Award
        {
            get { return Titles.Award; }
            set { Titles.Award = value; }
        }
        public decimal Salary
        {
            get { return Titles.Salary; }
            set { Titles.Salary = value; }
        }
        #endregion
    }
    class TitleModel
    {
        public string OrgName { get; set; }
        public string Title { get; set; }
        public string Award { get; set; }
        public decimal Salary { get; set; }
    }
    ```

## 2. 客製 Dapper 的 TypeHandler 

- 繼承 `SqlMapper.ITypeHandler` 並實作 `Parse` 與 `SetValue` 方法
    
    ```cs
    class UserTypeHandler : SqlMapper.ITypeHandler
    {
        //將 json string 從 db 取出時做轉型
        public object Parse(Type destinationType, object value)
        {
            //將 DB 的 json string 內容轉為目標的型別
            return JsonConvert.DeserializeObject(value.ToString(), destinationType);
        }
        //將值存回 db 時由 object 轉為 json
        public void SetValue(IDbDataParameter parameter, object value)
        {
            parameter.Value = (value == null) ? (object)DBNull.Value : JsonConvert.SerializeObject(value);
            parameter.DbType = DbType.String;
        }
    }
    ```

## 3. 方式使用

* 重點在於透過 `SqlMapper.AddTypeHandler()` 先行指定需要轉換的型別與對應使用的 TypeHandler

    ```cs
    //指定需要轉換的 Type 與對應使用的 TypeHandler
    SqlMapper.AddTypeHandler(typeof(TitleModel), new UserTypeHandler());

    var users = new List<User>();
    // db 連線
    using (var conn = new SqlConnection("Data Source=.;database=YowkoTest;Integrated Security=SSPI;app=LINQPad"))
    {
        // dapper 取得資料
        users = conn.Query<User>("SELECT * FROM dbo.[User]").ToList();
    }
    // linqpad 輸出資料結果
    users.Dump();
    ```

## 4. 實際產出

![2entity](https://user-images.githubusercontent.com/3851540/50452986-8bf00800-0978-11e9-9d8c-a5c73615fca9.png)

## 心得
以結果來看透過 Dapper 來轉換的方法與概念都滿簡單的，但過程中我也想過其他做法，最後決定的重點是較少的客製程式碼就可以降低出錯的機會。過了第一關接著下一步就得面臨同事們對執行效能上嚴苛的要求了，如果效能數據不好看我再來紀錄其他寫法XD


# 參考資料
1. [DAPPER – JSON TYPE CUSTOM MAPPER](https://radblog.pl/2018/01/22/dapper-json-type-custom-mapper/)
2. [Dapper小技巧：以資料表保存集合物件JSON](https://blog.darkthread.net/blog/dapper-typehandler/)