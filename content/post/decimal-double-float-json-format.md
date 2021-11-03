---
title: "decimal , double , float 輸出 json 的格式問題"
date: 2018-04-22T17:42:00+08:00
lastmod: 2021-11-03T17:42:45+08:00
draft: false
tags: ["套件","csharp"]
slug: "decimal-double-float-json-format"
aliases:
    - /2018/04/decimal-double-float-json-format.html
---
## decimal , double , float 輸出 json 的格式問題

之前筆記 [decimal 屬性輸出 JSON 時指定的格式問題](/decimal-json-digital) 提到在專案中因為系統介接需要統一 decimal 小數位數，過程中也才發現 json.net 在輸出沒有小數的 decimal 時行為不太一樣(會補上 `.0`：小數點及小數點一位)，最後雖然有解決問題，但解決方式自己卻不甚滿意，加上想要順帶測試 double 及 float 的行為，所以又花了一些時間找其他方法，就來看看過程遇到的問題及最後的解決方式吧

## 前提設定

1. 自訂 model (包含 decimal , double , float 屬性)

    ```cs
    public class TestData
    {
        public decimal Salary { get; set; }
        public double ExRate { get; set; }
        public float TaxRate { get; set; }
    }
    ```

2. 使用 jsonconvert

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
        
        public RoundingJsonConverter(int precision)
        : this(precision, MidpointRounding.AwayFromZero)
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

## 遇到問題及解決方式

1. Math.Round 處理 float 會造成數值不正確

    > Math.Round 沒有 float 版本，使用 Math.Round 會隱含將 float 轉型為 double，而造成數值不正確

    - 原始作法

        ```cs
        var value= 0.225f;
        Math.Round(value,2,MidpointRounding.AwayFromZero).Dump();
        ```

        - 數值不正確

            >![3floatissue](https://user-images.githubusercontent.com/3851540/39093546-b294a4e0-4653-11e8-8470-3a2b84602646.png)

        - Math.Round 傳入 float 會隱含轉型 double

            >![4convertdouble](https://user-images.githubusercontent.com/3851540/39093547-b2bd38c4-4653-11e8-8664-6f17411eea33.png)
            >
            >![5float](https://user-images.githubusercontent.com/3851540/39093548-b2e5c564-4653-11e8-8bff-959f518bfdcd.png)
            >
            >![6double](https://user-images.githubusercontent.com/3851540/39093549-b31080ce-4653-11e8-8b05-b8021530f3fd.png)
    - 新做法：先轉型為 `decimal`

        ```cs
        Math.Round(Convert.ToDecimal(value),2,MidpointRounding.AwayFromZero).Dump();
        ```

        >![7floatresult](https://user-images.githubusercontent.com/3851540/39093550-b33918ae-4653-11e8-9e76-25d4d3e3c999.png)

2. 為了指定輸出格式頻繁轉型 (ToString 再 parse 回 decimal)

    > WriteJson 時移除 decimal.parse 與 Math.Round 並使用 `WriteRawValue` 方法

    - 原始做法

        ```cs
        writer.WriteValue(Math.Round(_value, _precision, _rounding).ToString("0.".PadRight(2 + _precision, '0')));
        ```

        >![1tostring](https://user-images.githubusercontent.com/3851540/39093544-b1024920-4653-11e8-8ef0-b5b5ea141526.png)
    - 新做法

        ```cs
        writer.WriteRawValue((Convert.ToDecimal(value)).ToString("0.".PadRight(2 + _precision, '0')));
        ```

        >![2rawvalue](https://user-images.githubusercontent.com/3851540/39093545-b26cd41a-4653-11e8-87de-25d7b70c84f2.png)

    - 完整 jsonconvert 程式碼

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
        
            public RoundingJsonConverter(int precision)
                : this(precision, MidpointRounding.AwayFromZero)
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
                writer.WriteRawValue((Convert.ToDecimal(value)).ToString("0.".PadRight(2 + _precision, '0')));
            }
        }
        ```

3. 整數不輸出小數點及小數位

    > 加上屬性來控制是否將數字小數位末端的 `0` 移除

    - 原做法

        >![8notruncate](https://user-images.githubusercontent.com/3851540/39093551-b360a0b8-4653-11e8-9ecb-0327aa4a9efe.png)
    - 新做法
        - 加入屬性用來判斷是否要移除

            ```cs
            //用來判斷是否移除最後的 0
            bool _truncate;
            
            public RoundingJsonConverter(int precision, bool truncate) : this(precision, MidpointRounding.AwayFromZero, truncate)
            {
            }
            
            public RoundingJsonConverter(int precision) : this(precision, MidpointRounding.AwayFromZero, false)
            {
            }

            public RoundingJsonConverter(int precision, MidpointRounding rounding, bool truncate)
            {
                _precision = precision;
                _rounding = rounding;
                _truncate = truncate;
            }
            ```

        - 加入 `0` 的處理流程

            ```cs
            if (_truncate)
            {
                var _result = Math.Round(_value, _precision, _rounding) / 1.000000000000000000000000000000000m;//移除0
                if (Int64.TryParse(_result.ToString(), out var longresult))//處理整數
                    writer.WriteValue(longresult);
                else//處理非整數
                    writer.WriteValue(_result);
            }
            ```

        - 在 model 上加入是否移除 `0`

            ```cs
            public class TestData
            {
                [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
                public decimal Salary { get; set; }
                [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
                public double ExRate { get; set; }
                [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
                public float TaxRate { get; set; }
            }
            ```

            >![9truncateresult](https://user-images.githubusercontent.com/3851540/39093552-b3889514-4653-11e8-9950-ec6294d70bc5.png)

    - 完整程式碼

        ```cs
        void Main()
        {
            var test = new TestData() { Salary = 1.995m, ExRate = 2.295D, TaxRate = 1f };
            test.Dump();
            JsonConvert.SerializeObject(test).Dump();
        }
        public class TestData
        {
            [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
            public decimal Salary { get; set; }
            [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
            public double ExRate { get; set; }
            [JsonConverter(typeof(RoundingJsonConverter), 2, true)]
            public float TaxRate { get; set; }
        }
        
        public class RoundingJsonConverter : JsonConverter
        {
            //指定精準度
            int _precision;
            //指定四捨五入的傾向
            MidpointRounding _rounding;
            //用來判斷是否移除最後的 0
            bool _truncate;
        
            //預設精準度小數點下 4 位
            public RoundingJsonConverter() : this(4)
            {
            }
            public RoundingJsonConverter(int precision, bool truncate)
                    : this(precision, MidpointRounding.AwayFromZero, truncate)
            {
            }
            public RoundingJsonConverter(int precision)
                : this(precision, MidpointRounding.AwayFromZero, false)
            {
            }
        
            public RoundingJsonConverter(int precision, MidpointRounding rounding, bool truncate)
            {
                _precision = precision;
                _rounding = rounding;
                _truncate = truncate;
            }
        
            public override bool CanRead
            {
                get { return false; }
            }
        
            public override bool CanConvert(Type objectType)
            {
                return objectType == typeof(decimal) | objectType == typeof(double) | objectType == typeof(float);
                //return objectType == typeof(decimal);
            }
        
            public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
            {
                throw new NotImplementedException();
            }
        
            public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
            {
                var _value = Convert.ToDecimal(value);
                if (_truncate)
                {
                    var _result = Math.Round(_value, _precision, _rounding) / 1.000000000000000000000000000000000m;//移除0
                    if (Int64.TryParse(_result.ToString(), out var longresult))//處理整數
                        writer.WriteValue(longresult);
                    else//處理非整數
                        writer.WriteValue(_result);
                }
                else
                    writer.WriteRawValue(_value.ToString("0.".PadRight(2 + _precision, '0')));
            }
        }
        ```

## 心得

一開始從想要統一 decimal 的輸出格式，到後來持續調整寫法，到額外擴充支援 double 與 float，最終也可以自訂是否輸出可以刪除的小數點及 0，也許再次用到的機會並不高，但為了 coding for fun 帶來的樂趣無價呀

## 參考資訊

1. [decimal 屬性輸出 JSON 時指定的格式問題](/decimal-json-digital)
2. [關於 Decimal 小數尾數零](http://blog.darkthread.net/post-2016-12-11-decimal-trailing-zeros.aspx)
