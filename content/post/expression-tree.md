---
title: "初探 Expression Tree"
date: 2017-03-02T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "expression-tree"
aliases:
    - /2017/03/expression-tree.html
---
## 初探 Expression Tree

最近在重構取資料的邏輯，希望可以寫得更有彈性，所以開始使用 expression tree，雖然之前就有聽說 expression tree 很厲害，但總覺得好像不是一定要用，花時間就算了，可讀性又不高(~~其實是不會用~~)，這次情境似乎就是非用不可了，藉機來體驗一下其中的魔力

## 情境

1. 輸出 `傳入參數 X 2`
2. 輸出 `傳入參數 平方`

## Lambda 表達式

1. 輸出 `傳入參數 X 2`

    ```cs
    Expression <Func<int, int>> expr1 = n => n*2;// 將運算式轉換成樹狀資料結構
    Func<int, int> fn = expr1.Compile();// 將樹狀結構逆向轉為程式碼，並存入委派物件.
    Console.WriteLine(fn(10));// 印出 20.
    ```

2. 輸出 `傳入參數的平方`
    - <span style="color:red">以下程式碼與說明節錄至 [蔡煥麟老師的 C# 筆記：Expression Trees](http://huan-lin.blogspot.com/2011/08/csharp-expression-trees.html)</span>, 請大家直接參照原文，煥麟老師說明的較詳細
    - statement lambda 只能被轉換成委派物件，
    - expression lambda 可以轉換成委派物件，還可以轉換成運算式樹（expression tree）

    ```cs
    Expression<Func<int, int>> expr = n => n * n;  // 將運算式轉換成樹狀資料結構
    Func<int, int> fn = expr.Compile();  // 將樹狀結構逆向轉為程式碼，並存入委派物件.
    Console.WriteLine(fn(10));    // 印出 100.
    ```

## 純使用 expression

1. 輸出 `傳入參數 X 2`

    ```cs
    ParameterExpression p = Expression.Parameter(typeof(int), "r");//定義傳入參數型別及參數名稱
    ConstantExpression c = Expression.Constant(2);//定義恆數 2
    BinaryExpression b = BinaryExpression.Multiply(c, p);//定義變數間的操作動作 `*`
    LambdaExpression l= Expression.Lambda(b, p);//將多個 expression 組成 lambda expression
    Expression<Func<int, int>> e=(Expression<Func<int, int>>) l;//將 lambda expression 轉成  `Expression<TDelegate>`
    Console.WriteLine(e.Compile()(10));// 印出 20.
    ```

2. 輸出 `傳入參數的平方`

    ```cs
    ParameterExpression p = Expression.Parameter(typeof(int), "r");//參數 `r`
    BinaryExpression b = BinaryExpression.Multiply(p, p);
    LambdaExpression l= Expression.Lambda(b, p);//將多個 expression 組成 lambda expression
    Expression<Func<int, int>> e=(Expression<Func<int, int>>) l;//將 lambda expression 轉成  `Expression<TDelegate>`
    Console.WriteLine(e.Compile()(10));// 印出 100.
    ```

## 心得

看了一些文件後，雖然可以兜出簡單的語法，但還不是很瞭解，如果有寫錯，請大家指教。
接著會再試試其他語法，相信隨著多練習，一定會慢慢熟悉的。

## 參考資料

1. [C# 筆記：Expression Trees](http://huan-lin.blogspot.com/2011/08/csharp-expression-trees.html)
2. [Expression Tree上手指南 （一）](http://www.cnblogs.com/Ninputer/archive/2009/08/28/expression_tree1.html)
3. [How to convert a LambdaExpression to typed Expression<Func\<T, T>>](http://stackoverflow.com/questions/16213005/how-to-convert-a-lambdaexpression-to-typed-expressionfunct-t)
