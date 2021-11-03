---
title: "C# 的跳躍語法( break continue goto 與 return)"
date: 2017-02-14T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "csharp-break-continue-goto-return"
aliases:
    - /2017/02/c-sharp-break-continue-goto-return.html
---
## C# 的跳躍語法( break continue goto 與 return)

最近看了一段程式，感覺跑的順序跟預期的不同，查了 msdn 文件，順手做了個紀錄。
文件請務必看英文版，中文版的錯很大，完全不是同個意思XD

![12ERROR](https://cloud.githubusercontent.com/assets/3851540/22816987/d99b55c2-ef9f-11e6-9d17-d110cc0da15b.png)

## break

> 終止最近一層的 `迴圈`(`while`,`do`,`for`,`foreach`) 或 `switch`，接著執行後續程式碼

- 程式碼範例 1 ：單層迴圈

    ```cs
    for (int i = 1; i <= 100; i++)
    {
        if (i == 5)
        {
            break;
        }
        Console.WriteLine(i);
    }
    Console.WriteLine("end");
    ```

  - 跑第五次時遇到 `break` 跳離最近一層的 `迴圈` 或 `switch` --> 接著執行 `Console.WriteLine("end");`
  - 執行結果

    ![1break](https://cloud.githubusercontent.com/assets/3851540/22816988/d99c8bea-ef9f-11e6-9b9a-c8cc2569df85.png)

- 程式碼範例 2 ：雙層迴圈

    ```cs
    for (int a = 1; a <= 3; a++)
    {
        for (int i = 1; i <= 100; i++)
        {
            if (i == 5)
            {
                break;
            }
            Console.WriteLine($"第二層：{a}_{i}");
        }
        Console.WriteLine($"第一層:{a}");
    }

    Console.WriteLine("end");
    ```

  - 僅會跳離最近一層迴圈(本例來看就是變數 i 這層)，外層 (變數 a 這層不受影響)
  - 執行結果

    ![2break2](https://cloud.githubusercontent.com/assets/3851540/22816989/d99fbad6-ef9f-11e6-8035-3c78c60df7fb.png)

## continue

>會跳離最近一層迴圈該次迴圈(該次迴圈 continue 之後的內容皆不會執行)，執行(`while`,`do`,`for`,`foreach`)的下一個迭代(迴圈內容)

- 程式碼範例 1 ：單層迴圈

    ```cs
    for (int i = 1; i <= 10; i++)
    {
        if (i < 9)
        {
            continue;
        }
        Console.WriteLine(i);
    }

    Console.WriteLine("end");
    ```

  - `i < 9` 就跳離接著執行迴圈下一個內容直到 `i >=9` 才會執行到 `Console.WriteLine(i);`
  - 執行結果

    ![3continue](https://cloud.githubusercontent.com/assets/3851540/22816990/d9aa7a48-ef9f-11e6-8fa9-aee4c0764d8c.png)

- 程式碼範例 2 ：雙層迴圈

    ```cs
    for (int a = 1; a <= 3; a++)
    {
        for (int i = 1; i <= 10; i++)
        {
            if (i < 9)
            {
                continue;
            }
            Console.WriteLine($"第二層：{a}_{i}");
        }
        Console.WriteLine($"第一層:{a}");
    }

    Console.WriteLine("end");
    ```

  - 僅會跳離最近一層迴圈(本例來看就是變數 i 這層)，外層 (變數 a 這層不受影響)
  - 執行結果

    ![4continue2](https://cloud.githubusercontent.com/assets/3851540/22816992/d9bd83f4-ef9f-11e6-855c-b7e8fe66c3dd.png)

## goto

1. 改執行標記程式碼區段位置

    ```cs
    for (int i = 1; i <= 10; i++)
    {
        if (i >= 9)
        {
            goto TestLabel;
        }
        Console.WriteLine(i);
    }

    Console.WriteLine("end");
    
    
    TestLabel:
        Console.WriteLine("Test goto label");
        Console.WriteLine("Test goto label1");
        
        
        
    Console.WriteLine("Test goto labe end");
    ```

    - `i >= 9` 就會跳至 `TestLabel` 重新開始執行
    - 執行結果

        ![6goto1](https://cloud.githubusercontent.com/assets/3851540/22816993/d9be868c-ef9f-11e6-91c0-4eabaab223a3.png)
2. 改執行 switch 中其他 case 區段或是 default 區段

    ```cs
    int n = default(int);
    int cost = default(int);
    n = 2;
    //n = 3;
    switch (n)
    {
        case 1:
            Console.WriteLine("case 1");
            cost += 25;
            break;
        case 2:
            Console.WriteLine("case 2");
            cost += 25;
            goto case 1;
        case 3:
            Console.WriteLine("case 3");
            cost += 50;
            goto default;
        default:
            Console.WriteLine("default");
            break;
    }
    Console.WriteLine($"n={n},end:{cost}");
    ```

    - `n = 2` 執行 `case 2` 後改執行 `case 1`
        - 執行結果

            ![5goto1](https://cloud.githubusercontent.com/assets/3851540/22816991/d9bceeee-ef9f-11e6-9443-22525f293cd5.png)
    - `n = 3` 執行 `case 3` 後改執行 `default`
        - 執行結果

            ![5goto2](https://cloud.githubusercontent.com/assets/3851540/22816994/d9bfb35e-ef9f-11e6-815f-1ddfd23d4b30.png)
3. 用來跳離巢狀迴圈

    ```cs
    for (int a = 1; a <= 3; a++)
    {
        for (int i = 1; i <= 10; i++)
        {
            if ( a ==2 && i == 9)
            {
                goto TestLabel;
            }
            Console.WriteLine($"第二層：{a}_{i}");
        }
        Console.WriteLine($"第一層:{a}");
    }

    TestLabel:
    Console.WriteLine("Test goto label");
    Console.WriteLine("Test goto label1");
    Console.WriteLine("Test goto labe end");
    ```

    - 第一層執行第二次，第二層執行至第九次時，跳至 `TestLabel` 重新開始執行
    - 執行結果

        ![7gotoloop](https://cloud.githubusercontent.com/assets/3851540/22816995/d9c52bfe-ef9f-11e6-945d-77bbedd9a49d.png)

## return

- 會停止所在方法的執行，將程式執行權回歸原呼叫方法並回傳值

    ```cs
    void Main()
    {
        Console.WriteLine("before function");
        Console.WriteLine($"call function result:{Getint()}");
        Console.WriteLine("after function");
    
    }
    
    int Getint()
    {
        Console.WriteLine("call function");
        int result= default(int);
        return result;
    }
    ```

  - 執行結果

    ![8return1](https://cloud.githubusercontent.com/assets/3851540/22816996/d9cdd466-ef9f-11e6-94bc-e4215e36fc2b.png)

- 可用來跳出迴圈，switch
  - 迴圈

    ```cs
    void Main()
    {
        for (int i = 1; i <= 10; i++)
        {
            if (i >= 9)
            {
                return;
            }
            Console.WriteLine(i);
        }
    
        Console.WriteLine("end");
    }
    ```

    - 執行結果

        ![9return2](https://cloud.githubusercontent.com/assets/3851540/22816983/d95f0694-ef9f-11e6-8a9b-5e209cb94b3b.png)

  - switch

    ```cs
    void Main()
    {
        int n = default(int);
        int cost = default(int);
        n = 2;
        //n = 3;
        switch (n)
        {
            case 1:
                Console.WriteLine("case 1");
                cost += 25;
                break;
            case 2:
                Console.WriteLine("case 2");
                return;
                cost += 25;
                goto case 1;
            case 3:
                Console.WriteLine("case 3");
                cost += 50;
                goto default;
            default:
                Console.WriteLine("default");
                break;
        }
        Console.WriteLine($"n={n},end:{cost}");
    }
    ```

    - 執行結果

        ![9return3](https://cloud.githubusercontent.com/assets/3851540/22816984/d98655d2-ef9f-11e6-84ee-ef560e897b25.png)
- void 方法可以不寫 return，也沒回傳值，僅用來轉換程式執行權

    ```cs
    void Main()
    {
        Console.WriteLine("before function");
        test();
        Console.WriteLine("after function");
    }
    
    void test()
    {
        Console.WriteLine("CALL test");
        return ;
        Console.WriteLine("after return");
    }
    ```

  - 執行結果

    ![10return4](https://cloud.githubusercontent.com/assets/3851540/22816985/d999bcc6-ef9f-11e6-83fb-98b4353fe11f.png)

- return 被 try 包著，會先執行 finally 內容

    ```cs
    void Main()
    {
        Console.WriteLine("before function");
        test();
        Console.WriteLine("after function");
    }
    
    void test()
    {
        try
        {            
            Console.WriteLine("function try");
            return;
        }
        catch (Exception ex)
        {
            Console.WriteLine("catch");    
            throw;
        }
        finally
        {
            Console.WriteLine("function finally");
        }
        
    }
    ```

  - 執行結果

    ![11return5](https://cloud.githubusercontent.com/assets/3851540/22816986/d99a299a-ef9f-11e6-9283-2913f5c360d3.png)

## 參考資料

1. [break (C# Reference)](https://msdn.microsoft.com/en-us/library/adbctzc4.aspx)
2. [continue (C# Reference)](https://msdn.microsoft.com/en-us/library/923ahwt1.aspx)
3. [goto (C# Reference)](https://msdn.microsoft.com/en-us/library/13940fs2.aspx)
4. [return (C# Reference)](https://msdn.microsoft.com/en-us/library/1h3swy84.aspx)
