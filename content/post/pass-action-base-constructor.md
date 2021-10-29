---
title: "建構式中呼叫基底類別 (base class) 建構式傳入 Action 出現錯誤"
date: 2017-05-22T22:29:00+08:00
lastmod: 2021-10-29T11:19:59+08:00
draft: false
tags: ["csharp","Debug"]
slug: "pass-action-base-constructor"
aliases:
    - /2017/05/pass-action-base-constructor.html
---
## 建構式中呼叫基底類別 (base class) 建構式傳入 Action 出現錯誤

這是最近在重構程式時遇到的狀況，class B 繼承自 class A (class B:A)， class B 的建構式會在呼叫 class A 建構式時傳入 Action 當做 callback 用(publci B():base(Action))，為了在重構的過程中不會搞壞東西造成影響，所以打算先加上測試保護，但在這裡就卡關了，想要抽介面但又無法完全解決，紀錄一下解法，有機會再請教大師正確的做法

## 原始程式碼

```cs
void Main()
{
    var target= new B();
    target.Execute();
}

class A
{
    private readonly Action<string> _callback;
    public A(Action<string> callback)
    {
        this._callback = callback;
    }
 
    public void Execute()
    {
        this._callback("A execute callback");
    }
}


class B : A
{
    public B() : base(action)
    {
    }
    static void action(string str)
    {
        Console.WriteLine(str); 
    }
}
```

## 問題發生原因

會出現需要使用 static 物件的原因是 class B 呼叫 class A 架構式時，如果不用 static ,在那個時間點 action instance 尚未被初始化，就無法正確傳入 instance 而出現錯誤

## 錯誤訊息

- 訊息內容

    ```log
    An object reference is required for the non-static field, method, or property
    ```

    ![1error](https://cloud.githubusercontent.com/assets/3851540/26313475/9fb7e92c-3f3d-11e7-9664-b819446a3d38.png)

## 解決方式

既然已經知道問題是出在 class B 建構式執行時 action 尚未被初始化，直接做法就把 action 搬離 class B 就可以正常運作了

1. 將 action 搬出 class B
2. class B 建構式傳入 action(改由外部注入)
3. 完整程式碼

    ```cs
    void Main()
    {
        var target = new B(action);
        target.Execute();
    }
    class A
    {
        private readonly Action<string> _callback;
        public A(Action<string> callback)
        {
            this._callback = callback;
        }
        public void Execute()
        {
            this._callback("A execute callback");
        }
    }
    
    class B : A
    {
        public B(Action<string> callback) : base(callback)
        {
        }
    }
    void action(string str)
    {
        Console.WriteLine(str);
    }
    ```

## 參考資訊

1. [Can't pass an object to a base class constructor: error CS0120: An object reference is required for the non-static field, method, or property](https://social.msdn.microsoft.com/Forums/en-US/dc9322d8-e6e2-48f7-9755-2b3862df1c9f/cant-pass-an-object-to-a-base-class-constructor-error-cs0120-an-object-reference-is-required-for?forum=csharplanguage)
