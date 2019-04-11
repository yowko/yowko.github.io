---
title: "decimal 屬性輸出 JSON 時指定的格式問題"
date: 2018-04-21T00:14:00+08:00
lastmod: 2018-10-04T00:14:25+08:00
draft: false
tags: ["套件","C#"]
slug: "decimal-json-digital"
aliases:
    - /2018/04/decimal-json-digital.html
---
# decimal 屬性輸出 JSON 時指定的格式問題
這是之前專案遇到的狀況：輸出 `金額` 時只需處理到小數點下二位。既然是 `金額`，為了避免精準度造成的誤差都會選用 `deciaml` 資料類型，而在 db 中使用 `money` 儲存(因為業務需求面沒有運算需求可以使用，如果會有運算 `money` 有失真的風險，詳細內容請參考 [欄位開立(2) - decimal, numeric, float, real, money 的抉擇](https://dotblogs.com.tw/henryli/2015/06/13/151557))，預設精準度為小數點下四位，為了符合目前系統的要求(小數點下二位)，就需要調整輸出，來看看可以怎麼做吧

## 前提設定

1.  自訂 model 中的 decimal 屬性

    ```cs
    public class TestData
    {
        public decimal Salary { get; set; }
    }
    ```

2.  可能存在不同小數點位數的值

    ```cs
    var test = new TestData() { Salary = 1.03355M };
    var test2 = new TestData() { Salary = 2.0000M };
    var test3 = new TestData() { Salary = 3M };
    var test4 = new TestData() { Salary = 4.115M };
    ```

3.  原始輸出

    ```cs
    #region - data1 -
    var test = new TestData() { Salary = 1.03355M };
    test.Dump();
    JsonConvert.SerializeObject(test).Dump();
    #endregion
             
    #region - data2 -
    var test2 = new TestData() { Salary = 2.0000M };
    test2.Dump();
    JsonConvert.SerializeObject(test2).Dump();
    #endregion
    
    #region - data3 -
    var test3 = new TestData() { Salary = 3M };
    test3.Dump();
    JsonConvert.SerializeObject(test3).Dump();
    #endregion
             
    #region - data4 -
    var test4 = new TestData() { Salary = 4.115M };
    test4.Dump();
    JsonConvert.SerializeObject(test4).Dump();
    #endregion
    ```

    > ![1original](https://user-images.githubusercontent.com/3851540/39060398-d777fb88-44f3-11e8-8c84-43cd2dc09721.png)

    可以看到除了整數(3)被加上一個小數位(3.0)之外，其他數值都會完整輸出

## 使用私有欄位儲存，在 get 時格式化

*   加上私有欄位並格式化輸出

    ```cs
    public class TestData
    {
        private decimal _salary;
        public decimal Salary
        {
            get { return Math.Round(_salary, 2); }
            set { _salary = value; }
        }
    }
    ```
*   結果

    > ![2fieldformat](https://user-images.githubusercontent.com/3851540/39060399-d79e3820-44f3-11e8-85f4-a5eb6457b3f0.png)

*   缺點

    `可以看到整數儲存與輸出 json 不符合，這個是 json.net 的特性，另外不一定可以完全符合指定小數位數`

    
    >![4fieldissue](https://user-images.githubusercontent.com/3851540/39060401-d7ed1b66-44f3-11e8-9d51-d9017b392cb9.png)

*   改善

    > 如果有強烈的需求還是可以強制調整特定輸出，但這樣會有執行效率不佳的問題要特別留意

    
    ```cs
    public class TestData
    {
        private decimal _salary;
        public decimal Salary
        {
            get { return decimal.Parse(Math.Round((double)_salary, 2).ToString("0.00")); }
            set { _salary = value; }
        }
    }
    ```

    > ![3fieldboxing](https://user-images.githubusercontent.com/3851540/39060400-d7c4f5c8-44f3-11e8-8292-f4599b4d7bb2.png)

## 客製 Json.Net 的 JsonConverter

1.  加入自訂 JsonConverter 並繼承 JsonConverter

    ```cs
    public class RoundingJsonConverter : JsonConverter
    {
        //指定精準度
        int _precision;
        //指定四捨五入的傾向
        MidpointRounding _rounding;
        
        //預設精準度小數點下 4 位
        public RoundingJsonConverter(): this(4)
        {
        }
        
        public RoundingJsonConverter(int precision) : this(precision, MidpointRounding.AwayFromZero)
        {
        }
                    
        public RoundingJsonConverter(int precision, MidpointRounding rounding)
        {
            _precision = precision;
            _rounding = rounding;
        }
                    
        public override bool CanRead
        {
            get { return false; }
        }
                    
        public override bool CanConvert(Type objectType)
        {
            return objectType == typeof(decimal);
        }
                    
        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            throw new NotImplementedException();
        }
                    
        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            decimal _value=(decimal)value;
            writer.WriteValue(Math.Round(_value, _precision, _rounding));
        }
    }
    ```

2.  在欲指定精準度的屬性上加入 attribute

    ```cs
    public class TestData
    {
        [JsonConverter(typeof(RoundingJsonConverter),2)]
        public decimal Salary { get; set; }
    }
    ```

*   結果

    > ![4jsonconverter](https://user-images.githubusercontent.com/3851540/39060402-d814b5ae-44f3-11e8-854d-ddc66ffd2553.png)

*   缺點

    `一樣有指定小數位數未生效的狀況，以下指定小數四位為例`

    > ![5jsonissue](https://user-images.githubusercontent.com/3851540/39060403-d8454aa2-44f3-11e8-89c8-8fdd1ab7bdc8.png)

*   改善

    > 一樣會有效能問題

    ```cs
    public class RoundingJsonConverter : JsonConverter
    {
        //指定精準度
        int _precision;
        //指定四捨五入的傾向
        MidpointRounding _rounding;
        
        //預設精準度小數點下 4 位
        public RoundingJsonConverter() : this(4)
        {
        }
        
        public RoundingJsonConverter(int precision) : this(precision, MidpointRounding.AwayFromZero)
        {
        }
                    
        public RoundingJsonConverter(int precision, MidpointRounding rounding)
        {
            _precision = precision;
            _rounding = rounding;
        }
        
        public override bool CanRead
        {
            get { return false; }
        }
        
        public override bool CanConvert(Type objectType)
        {
            return objectType == typeof(decimal);
        }
        
        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            throw new NotImplementedException();
        }
        
        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            decimal _value = (decimal)value;
            //為值補 0
            writer.WriteValue(decimal.Parse(Math.Round(_value, _precision, _rounding).ToString("0.".PadRight(2 + _precision, '0'))));
        }
    }
    ```

    > ![6josnconvertboxing](https://user-images.githubusercontent.com/3851540/39060404-d86de20a-44f3-11e8-9a29-6c08c60cfc29.png)

## 心得

事實上專案中的介接目標系統沒有強制要求所有有數字都需精準至小數下二位，只是調整過程中潔癖發作，一直想要調整到人眼看也是很整齊，不過實在很難，最後還是選了個不漂亮的做法，當然主因就是找不到更好的方式XD，學藝不精，只好先紀錄一下日後有能力或是緣份到了再來改寫

# 參考資訊

1.  [欄位開立(2) - decimal, numeric, float, real, money 的抉擇](https://dotblogs.com.tw/henryli/2015/06/13/151557)
2.  [Json.NET serializing float/double with minimal decimal places, i.e. no redundant “.0”?](https://stackoverflow.com/questions/21153381/json-net-serializing-float-double-with-minimal-decimal-places-i-e-no-redundant)
3.  [JSON轉換時去除小數字尾零](http://blog.darkthread.net/post-2014-06-13-trim-json-trail-zero.aspx)
