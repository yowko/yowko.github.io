---
title: "應該使用 IsNullOrEmpty 還是 IsNullOrWhiteSpace"
date: 2018-11-17T00:42:34+08:00
lastmod: 2021-10-15T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "string-isnullorempty-isnullorwhitespace"
---
## 應該使用 IsNullOrEmpty 還是 IsNullOrWhiteSpace

前幾天看到 Bruce 分享微軟內部團隊在 C# 寫作上的團隊規範 - [#80 寫程式的參考準測 (coding guideline) - C# 篇](http://www.woolycsnote.tw/2018/11/80-coding-guideline-c.html)，其中一點是 `用 string.IsNullOrWhiteSpace() 來檢查字串是否為 null 或是空白`，雖然我自己一直都這麼做，也是因為過去查過資料才養成這個習慣，但時間久遠不免忘記當時做這個選擇的關鍵原因了，剛好趁這個機會再次複習並紀錄一下  以免日後又花不少時間回想

## 關於 String.IsNullOrEmpty

- 參數：欲檢驗的字串
- 回傳值：Boolean
- 從 .NET Framework 2.0 開始加入
- 用來檢驗傳入參數是否為 `null` 或 `空字串` 的方法

## 關於 String.IsNullOrWhiteSpace

- 參數：欲檢驗的字串
- 回傳值：Boolean
- 從 .NET Framework 4.0 開始加入
- 用來檢驗傳入參數是否為 `null`、`空字串` 或 `只有空白符號` 的方法

## IsNullOrEmpty V.S. IsNullOrWhiteSpace

1. 程式碼涵義

    > 內容來至 MSDN

    - String.IsNullOrEmpty

        ```cs
        return value == null || value == String.Empty;
        ```

    - String.IsNullOrWhiteSpace

        ```cs
        //return String.IsNullOrEmpty(value) || value.Trim().Length == 0;
        return value == null || value == String.Empty || value.Trim().Length == 0;
        ```

2. 實測值比較

    值|IsNullOrEmpty|IsNullOrWhiteSpace
    ---|---|---
    `null`|true|true
    `string.Empty`|true|true
    `""`|true|true
    `" "`|false|true

3. 效能測試

    > 原本對 `String.IsNullOrEmpty` 就效能不抱期待，畢竟做的事較多，但官方文件上提到 `較佳執行效率`，便讓我興起看看效能差距的念頭

    - 基本設定
        - 比較 `string.IsNullOrEmpty`、 `String.IsNullOrEmpty(str) || str.Trim().Length == 0` 與 `string.IsNullOrWhiteSpace` 三種方式
        - 執行一百萬次，再比較總執行時間 (單位：`Milliseconds`)
        - 分別使用 `""` , `null` 與 `" "` 來進行測試
        - 執行程式五次
    - 程式碼

        ```cs
        void Main()
        {
            //分別使用 "", null 與 " " 來進行測試
            string targetstr = "";

            testIsNullOrEmpty(targetstr);
            testIsNullOrEmptyPlus(targetstr);
            testIsNullOrWhiteSpace(targetstr);
        }

        //使用 string.IsNullOrEmpty
        void testIsNullOrEmpty(string str)
        {
            Stopwatch sw = Stopwatch.StartNew();
            for (int i = 0; i < 1000000; i++)
            {
                var test = string.IsNullOrEmpty(str);
            }

            sw.Stop();
            sw.ElapsedMilliseconds.Dump();
        }
        //使用自製的 IsNullOrWhiteSpace
        void testIsNullOrEmptyPlus(string str)
        {
            Stopwatch sw = Stopwatch.StartNew();
            for (int i = 0; i < 1000000; i++)
            {
                var test = string.IsNullOrEmpty(str) || str.Trim().Length == 0;
            }

            sw.Stop();
            sw.ElapsedMilliseconds.Dump();
        }
        //使用 string.IsNullOrWhiteSpace
        void testIsNullOrWhiteSpace(string str)
        {
            Stopwatch sw = Stopwatch.StartNew();
            for (int i = 0; i < 1000000; i++)
            {
                var test = string.IsNullOrWhiteSpace(str);
            }

            sw.Stop();
            sw.ElapsedMilliseconds.Dump();
        }

        ```

    - 結果

        測試值\方法|IsNullOrEmpty|自製 IsNullOrWhiteSpace|IsNullOrWhiteSpace
        ---|---|---|---
        `""`|0,1,1,0,1|1,1,1,1,1|2,1,1,2,2
        `null`|1,0,0,0,0|1,1,1,1,1|2,2,2,3,2
        `" "`|0,0,0,1,0|6,7,10,9,9|3,3,5,4,5

## 心得

經過實際使用程式碼來驗證語法的執行效率後，確認了 IsNullOrWhiteSpace 在執行效率確實比不上 IsNullOrEmpty ，此時再重新檢視 MSDN 文件才發現我會錯意了  MSDN 指的擁有較佳效率是與自製 IsNullOrWhiteSpace 比較 XD，而這點從測試結果確實也印證自製的程式碼在效率上比不使用內建的 IsNullOrWhiteSpace

- `IsNullOrEmpty` 與 `IsNullOrWhiteSpace` 適不適合相提並論：得看 `""` 與 `" "` 在商業邏輯上是否可以視為相同來決定;
- 回到執行效率本身：兩個方法在執行 1,000,000 次的總時間差才不到 5 毫秒之差，多數情境應可以忽略不計

## 參考資訊

1. [#80 寫程式的參考準測 (coding guideline) - C# 篇](http://www.woolycsnote.tw/2018/11/80-coding-guideline-c.html)
2. [String.IsNullOrEmpty(String) Method](https://docs.microsoft.com/en-us/dotnet/api/system.string.isnullorempty?WT.mc_id=DOP-MVP-5002594)
3. [String.IsNullOrWhiteSpace(String) Method](https://docs.microsoft.com/en-us/dotnet/api/system.string.isnullorwhitespace?WT.mc_id=DOP-MVP-5002594)
