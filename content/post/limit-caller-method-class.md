---
title: "將 method 或是 class 標記為 internal 來限定專案使用"
date: 2017-11-19T21:11:00+08:00
lastmod: 2018-09-30T21:11:53+08:00
draft: false
tags: ["C#"]
slug: "limit-caller-method-class"
aliases:
    - /2017/11/limit-caller-method-class.html
---
# 將 method 或是 class 標記為 internal 來限定專案使用

一般專案常常會有部份操作是的前後台行為相同或是極度類似的，如果這些行為雷同的程式分別置於前後台會讓程式碼顯得多餘，但把邏輯統一抽出共用又擔心成員在沒有充份了解 method 功能的情況下而誤用造成資安風險

今天就來紀錄一下我的做法

## 將共用行為抽至 class library

1.  method

    ```cs
    //共用的 class
    public class SharedClass
    {
        //共用 method
        public int MethodA()
        {
            return 0;
        }
        //僅開放給特定專案 method
        internal int MethodB()
        {
            return 9999;
        }
    }
    ```

2.  class

    ```cs
    //僅開放給特定專案案
    internal class LimitClass
    {
        internal int MethodC()
        {
            SharedClass _class=new SharedClass();
            return _class.MethodA() + 123;
        }
    }
    ```

## 將 class library 設定開放給指定專案

1.  開啟專案 `Properties` 下的 `AssemblyInfo.cs`

    ![1asminfo](https://user-images.githubusercontent.com/3851540/32990957-778b112c-cd6d-11e7-98e4-1f59df49d3a8.png)

2.  加入 `InternalsVisibleTo` 設定給指定專案

    *   語法

        ```cs
        [assembly: InternalsVisibleTo("{組件名稱}")]
        ```
        *   取得組件名稱方法

            *   專案上按右鍵 --> Properties

                ![2properties](https://user-images.githubusercontent.com/3851540/32990958-77b957d0-cd6d-11e7-95ba-da81ca3f08e6.png)

            *   Application --> Assembly name

                ![3assemblyname](https://user-images.githubusercontent.com/3851540/32990959-77e53fee-cd6d-11e7-9d93-b04caaa88cdc.png)

    *   實例

        ```cs
        [assembly: InternalsVisibleTo("TestMethodRightB")]
        ```

## 實際使用

```cs
SharedClass _class = new SharedClass();
Console.WriteLine(_class.MethodA());
Console.WriteLine(_class.MethodB());

LimitClass _limitClass = new LimitClass();
Console.WriteLine(_limitClass.MethodC());
```

1.  未設定的專案無法使用 internal object

    ![4noright](https://user-images.githubusercontent.com/3851540/32990960-780e8e62-cd6d-11e7-87af-efe3c176b2d9.png)

2.  已設定專案正常使用

    ![5hasright](https://user-images.githubusercontent.com/3851540/32990961-7838c4ca-cd6d-11e7-8589-1863d6fd9ab0.png)

## 心得

使用存取子來限制特定專案存取是個簡單的做法，只是強制性並不算高，當然這邊的強制性指的不是 C# 語法，而是團隊成員的素質，如果成員看到有個 class 或是 method 他想用，什麼都沒問就不管三七二十一直接改成 public，我想怎樣都白搭

所以對於團隊成員，尤其是新加入的成員，最有效的還是採用 code review，不過還是得看團隊接受度

# 參考資訊
