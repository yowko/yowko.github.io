---
title: "字串處理速度比較：+ 運算符、string.Format、string.Concat、字串插值(String Interpolation)"
date: 2017-01-05T00:42:34+08:00
lastmod: 2021-10-15T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "stringformat-stringconcat-stringintepolation"
aliases:
    - /2017/01/SpeedCompare-plus-StringFormat-StringConcat-StringIntepolation.html
---
## 字串處理速度比較：+ 運算符、string.Format、string.Concat、字串插值(String Interpolation)

c# 6.0 多了一個方便處理字串的語法糖 - `字串插值(String Interpolation)`, 馬上讓我興起與其他語法的差異，首先就讓我們來看看它的效能如何吧

## 比較方法

1. `+` 運算子

    ```cs
    string stringplus(string name, string method,int i)
    { 
        return name + "用來測試" + method + "的執行效能，這是第" + i + "次迴圈,時間:"+DateTime.Now.ToString();
    }
    ```

2. `string.Format` 方法

    ```cs
    string strinformat(string name,string method,int i)
    { 
        return string.Format("{0}用來測試{1}的執行效能，這是第{2}次迴圈,時間:{3}", name, method, i,DateTime.Now.ToString());
    }
    ```

3. `string.Concat` 方法

    ```cs
    string stringconcat(string name, string method,int i)
    {
        return string.Concat(name, "用來測試", method, "的執行效能，這是第", i, "次迴圈,時間:",DateTime.Now.ToString());
    }
    ```

4. 字串插值(String Interpolation)

    > C# 6.0 新加入的法入，讓字串處理更人性化了

    ```cs
    string stringinterpolation(string name, string method, int i)
    {
        return $"{name}用來測試{method}的執行效能，這是第{i}次迴圈,時間:{DateTime.Now.ToString()}";
    }
    ```

## 比較方式  

> 透過 `Stopwatch` 紀錄每個方法的執行時間

```cs
Stopwatch sw = new Stopwatch();
string name = "yowko";
int times =1000000;//執行次數
#region - string format -
sw.Restart();
sw.Start();
string method = "string format";
for (int i = 0; i < times; i++)
{
    strinformat(name,method,i);
}
sw.Stop();
string.Concat(method, ":", sw.ElapsedMilliseconds.ToString()).Dump();
#endregion
#region - string concat -
sw.Restart();
sw.Start();
method = "string concat";
for (int i = 0; i < times; i++)
{
    stringconcat(name,method,i);
}
sw.Stop();
string.Concat(method, ":", sw.ElapsedMilliseconds.ToString()).Dump();
#endregion
#region - +operation -
sw.Restart();
sw.Start();
method = "string +";
for (int i = 0; i < times; i++)
{
    stringplus(name, method, i);
}
sw.Stop();
string.Concat(method, ":", sw.ElapsedMilliseconds.ToString()).Dump();
#endregion
#region - String Interpolation -
sw.Restart();
sw.Start();
method = "String Interpolation";
for (int i = 0; i < times; i++)
{
    stringinterpolation(name, method, i);
}
sw.Stop();
string.Concat(method, ":", sw.ElapsedMilliseconds.ToString()).Dump();
#endregion
```

## 比較結果

> 我測試約二十次，隨機截錄其中五次如下

1. 第一次

    | method   |  time(ms) |
    |----------|------:|
    | string format | 1215 |
    | string concat | 1135 |
    | string + | 1137 |
    |String Interpolation|    1219|

2. 第二次

    | method   |  time(ms) |
    |----------|------:|
    |string format|    1215|
    |string concat|    1128|
    |string +|1141
    |String Interpolation|1211|

3. 第三次

    | method   |  time(ms) |
    |----------|------:|
    |string format|1215|
    |string concat|1136|
    |string +|1139|
    |String Interpolation|1212|

4. 第四次

    | method   |  time(ms) |
    |----------|------:|
    |string format|1209|
    |string concat|1137|
    |string +|1141|
    |String Interpolation|1214|

5. 第五次

    | method   |  time(ms) |
    |----------|------:|
    |string format|1212|
    |string concat|1136|
    |string +|1140|
    |String Interpolation|1214|

## 結論

速度上 `string.concat` > `+ 運算字` > `string.format` ≒ `字串插值(String Interpolation)`
以結果來看 `string.concat` 快於 `+ 運算字` 快於  `string.format` 與 `字串插值(String Interpolation)`，`string.format` 跟`字串插值(String Interpolation)` 則互有勝負，且差距很小

看起來 c# 6.0 的新語法可以放心用，且次數不多的情況下效能差距絕對是可以忽略不計

## 參考資料

1.[字串插值](https://msdn.microsoft.com/zh-tw/library/dn961160.aspx)
