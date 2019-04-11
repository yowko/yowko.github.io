---
title: "使用 delagate 來進行多個條件驗證"
date: 2017-11-20T21:57:00+08:00
lastmod: 2018-09-30T21:57:47+08:00
draft: false
tags: ["C#"]
slug: "delegate-rule-check"
aliases:
    - /2017/11/delegate-rule-check.html
---
# 使用 delagate 來進行多個條件驗證
今天跟同事討論到某個功能在實際執行前需要做一些商業邏輯檢查，確認符合所有規則才能繼續執行後面動作，同事本來使用多層 `if` 來處理，後來覺得不好閱讀所以改在執行方法時檢查到不符合條件就丟出自訂 exception

依我個人觀點，多層 `if` 當然不好，但我覺得不符商業邏輯丟出 exception 也不好，一來是 exception 處理會損耗效能，二來這是商業邏輯驗證的結果不該歸為程式面 exception

以下筆記個人做法供參考

## 情境說明

1.  驗證一個值是否符合以下條件
    *   正數
    *   大於 10
    *   大於 100
    *   大於 1000

2.  上述任一條件驗證未過則顯示 `RuleCheck fail!!`
3.  全部通過則顯示 `RuleCheck pass!!`


## 做法一：多層 if (程式碼波動拳)

```cs
void Main()
{
    var data = 10001;
    bool result = false;
    if (CheckPositive(data))
    {
        if (CheckMoreTen(data))
        {
            if (CheckMoreHundred(data))
            {
                if (CheckMoreThousand(data))
                {
                result = true;
                }
            }
        }
    }
    if (result)
        "RuleCheck pass!!".Dump();
    else
        "RuleCheck fail!!".Dump();
}
bool CheckPositive(int a)
{
    if (a <= 0)
        return false;
    else
        return true;
}
bool CheckMoreTen(int a)
{
    if (a <= 10)
        return false;
    else
        return true;
}
bool CheckMoreHundred(int a)
{
    if (a <= 100)
        return false;
    else
        return true;
}
bool CheckMoreThousand(int a)
{
    if (a <= 1000)
        return false;
    else
        return true;
}
```

> ![](://i.imgur.com/BtjZedW.jpg)

*   優點：很直觀
*   缺點：程式碼不好閱讀


## 做法二：規則檢查未通過丟出 exception

```cs
void Main()
{
    var data = 10001;
    try
    {         
        CheckPositive(data);
        CheckMoreTen(data);
        CheckMoreHundred(data);
        CheckMoreThousand(data);
    }
    catch (Exception ex)
    {
        "RuleCheck fail!!".Dump();
        throw;
    }
    "RuleCheck pass!!".Dump();
}
void CheckPositive(int a)
{
    if (a <= 0)
        throw new Exception("CheckPositive fail");
}
void CheckMoreTen(int a)
{
    if (a <= 10)
        throw new Exception("CheckMoreTen fail");
}
void CheckMoreHundred(int a)
{
    if (a <= 100)
        throw new Exception("CheckMoreHundred fail");
}
void CheckMoreThousand(int a)
{
    if (a <= 1000)
        throw new Exception("CheckMoreThousand fail");
}
```

*   優點：程式碼較多層 `if` 好閱讀
*   缺點：有程式碼效能損耗、商業邏輯不該歸為程式面的 exception、不容易定位 exception 來源(可測試性較低)


## 做法三：使用 delegate 進行檢查

```cs
void Main()
{
    var data = 101;
    bool result = true;

    List<Func<int, bool>> RuleCheck = new List<System.Func<int, bool>>() {
        CheckPositive,
        CheckMoreTen,
        CheckMoreHundred,
        CheckMoreThousand
    };

    foreach (var item in RuleCheck)
    {
        if (item.Invoke(data))
        {
            //$"{data}pass {item.Method.Name}".Dump();
            continue;
        }
        else
        {
            result = false;
            //$"{data}fail {item.Method.Name}".Dump();
            break;
        }

    }
    if (result)
        "RuleCheck pass!!".Dump();
    else
        "RuleCheck fail!!".Dump();

}

bool CheckPositive(int a)
{
    if (a <= 0)
        return false;
    else
        return true;
}
bool CheckMoreTen(int a)
{
    if (a <= 10)
        return false;
    else
        return true;
}
bool CheckMoreHundred(int a)
{
    if (a <= 100)
        return false;
    else
        return true;
}
bool CheckMoreThousand(int a)
{
    if (a <= 1000)
        return false;
    else
        return true;
}
```

*   優點：較多層 `if` 好閱讀，也避免因為商業邏輯丟出 exception


## 心得

事實上我不認為同事的做法是錯的，我的立場一向是可以完成需求的程式就及格，要不要將程式寫得更好理解、更容易維護、是否符合軟體設計原則就是個人選擇了

在程式的世界裡要滿足需求是很容易的一件事，是否要將程式寫得漂亮、容易理解、便於維護就得靠自己精雕細琢，精益求精

# 參考資訊

1.  [List<t>.ForEach 方法 (Action<t>)</t></t>](https://msdn.microsoft.com/zh-tw/library/bwabdf9z%28v=vs.110%29.aspx)
