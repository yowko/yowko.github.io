---
title: "使用 Entity Framework Insert 大量資料"
date: 2018-05-19T00:38:00+08:00
lastmod: 2021-11-03T21:56:36+08:00
draft: false
tags: ["EntityFramework","Performance Tuning"]
slug: "entityframework-bulk-insert"
aliases:
    - /2018/05/entityframework-bulk-insert.html
---
## 使用 Entity Framework Insert 大量資料

這是參加 黃忠成老師的 Entity Framework 全線開發 課程時他提出讓學員思考的問題：如何使用 Entity Framework Insert 大量資料，我當下立馬想起印象中以前同事也被這個問題困擾過，而我自己本身則沒有遇到相同困擾，剛好透過這個機會來瞭解其中的差異也順便嘗試看看不同解法

立馬動手模擬看看，親身體驗實際情況吧

## 基本環境及情境說明

1. 使用 Entity Framework 6.1.3 新增資料至 SQL Server 2017
2. SQL Server 與 EntityFramework Client 都在本機電腦上 ，減少 network io
3. 測試新增 `10`,`100`,`1000`,`10000`,`100000` 筆資料至資料庫所需時間
4. 測試的 EF 使用方式有：
    - 逐筆 Add 並逐筆 SaveChanges
    - 逐筆 Add 及一次 SaveChanges
    - 批次 Add 及 SaveChanges
    - 批次 Add 及 SaveChanges 並 dispose context

## 測試內容

1. 逐筆 Add 並逐筆發動 SaveChanges
    - 程式碼

        ```cs
        TestNestedEF.Models.TestEntities db = new TestEntities();

        var count = 100000;
        Stopwatch sw = new Stopwatch();
        sw.Reset();
        sw.Start();
        for (int i = 0; i < count; i++)
        {
            Parent parent = new Parent() { Name = i.ToString() };
            parent.Child = new List<Child>() { new Child() { Name = $"{i}_Son" } };

            db.Parent.Add(parent);
            db.SaveChanges();
        }
        sw.Stop();
        sw.ElapsedMilliseconds.Dump();
        ```

    - 測試筆數
        - 10：`11 ms`

            ![1saveeverytime10](https://user-images.githubusercontent.com/3851540/40090392-0d4bf9cc-58e4-11e8-8dd7-acbc08e079a0.png)
        - 100：`187 ms`

            ![2saveeverytime100](https://user-images.githubusercontent.com/3851540/40090393-0d77e79e-58e4-11e8-926c-91c31c095b3e.png)
        - 1000：`4282 ms`

            ![3saveeverytime1000](https://user-images.githubusercontent.com/3851540/40090394-0da1e562-58e4-11e8-856e-470e742a6f04.png)
        - 10000：`401177 ms`

            ![4saveeverytime10000](https://user-images.githubusercontent.com/3851540/40090395-0dc981ee-58e4-11e8-9ca5-f4bd6520f1d0.png)
        - 100000：`44827184 ms`

            ![5saveeverytime100000](https://user-images.githubusercontent.com/3851540/40090396-0df63072-58e4-11e8-90da-d9033a50de9c.png)

2. 逐筆 Add 及一次 SaveChanges
    - 程式碼

        ```cs
        TestNestedEF.Models.TestEntities db = new TestEntities();

        var count = 10000;
        Stopwatch sw = new Stopwatch();
        sw.Reset();
        sw.Start();
        for (int i = 0; i < count; i++)
        {
            Parent parent = new Parent() { Name = i.ToString() };
            parent.Child = new List<Child>() { new Child() { Name = $"{i}_Son" } };

            db.Parent.Add(parent);
        }
        db.SaveChanges();
        sw.Stop();
        sw.ElapsedMilliseconds.Dump();
        ```

    - 測試筆數
        - 10：`7 ms`

            ![6batchsave10](https://user-images.githubusercontent.com/3851540/40090397-0e24007e-58e4-11e8-9269-fba28089b049.png)
        - 100：`79 ms`

            ![7batchsave100](https://user-images.githubusercontent.com/3851540/40090398-0e51d8a0-58e4-11e8-84fa-31654d4bc7e1.png)
        - 1000：`2309 ms`

            ![8batchsave1000](https://user-images.githubusercontent.com/3851540/40090399-0e7ec9aa-58e4-11e8-837c-ddc596b37ba1.png)
        - 10000：`195678 ms`

            ![9batchsave10000](https://user-images.githubusercontent.com/3851540/40090400-0eaba5d8-58e4-11e8-88b9-484b34f4df65.png)
        - 100000：`19916508 ms`

            ![10batchsave100000](https://user-images.githubusercontent.com/3851540/40090401-0ed89980-58e4-11e8-9732-52ade4c28443.png)
3. 批次 Add 及 SaveChanges
    - 程式碼

        ```cs
        TestNestedEF.Models.TestEntities db = new TestEntities();

        var count = 100000;
        Stopwatch sw = new Stopwatch();
        sw.Reset();
        sw.Start();
        var batchcount = 100;
        for (int i = 0; i < (count / batchcount); i++)
        {
            for (int j = 0; j < batchcount; j++)
            {
                Parent parent = new Parent() { Name = $"{i * batchcount + j}" };
                parent.Child = new List<Child>() { new Child() { Name = $"{i * batchcount + j}_Son" } };

                db.Parent.Add(parent);
            }
            db.SaveChanges();
        }
        sw.Stop();
        sw.ElapsedMilliseconds.Dump();
        ```

    - 測試筆數
        - 10：`9 ms`

            ![11batchsave10](https://user-images.githubusercontent.com/3851540/40090402-0f01573a-58e4-11e8-96a4-4b03909cba79.png)
        - 100：`77 ms`

            ![12batchsave100](https://user-images.githubusercontent.com/3851540/40090383-0ba8c9ce-58e4-11e8-9f62-78091f8bee08.png)
        - 1000：`1955 ms`

            ![13batchsave1000](https://user-images.githubusercontent.com/3851540/40090384-0be486da-58e4-11e8-9ca4-4fba0de7965f.png)
        - 10000：`179073 ms`

            ![14batchsave10000](https://user-images.githubusercontent.com/3851540/40090385-0c16fb7e-58e4-11e8-86da-0b117c333872.png)
        - 100000：`16710546 ms`

            ![15batchsave100000](https://user-images.githubusercontent.com/3851540/40090386-0c417944-58e4-11e8-8d2e-26b72805ac79.png)
4. 批次 Add 及 SaveChanges 並 dispose context
    - 程式碼

        ```cs
        var count = 100000;
        Stopwatch sw = new Stopwatch();
        sw.Reset();
        sw.Start();
        var batchcount = 100;

        for (int i = 0; i < (count / batchcount); i++)
        {
            using (TestNestedEF.Models.TestEntities db = new TestEntities())
            {
                for (int j = 0; j < batchcount; j++)
                {
                    Parent parent = new Parent() { Name = $"{i * batchcount + j}" };
                    parent.Child = new List<Child>() { new Child() { Name = $"{i * batchcount + j}_Son" } };

                    db.Parent.Add(parent);
                }
                db.SaveChanges();
            }
        }
        
        sw.Stop();
        sw.ElapsedMilliseconds.Dump();
        ```

    - 測試筆數
        - 10：`12 ms`

            ![16dispose10](https://user-images.githubusercontent.com/3851540/40090387-0c6e5a0e-58e4-11e8-9cce-2d3b15096eb2.png)
        - 100：`91 ms`

            ![17dispose100](https://user-images.githubusercontent.com/3851540/40090388-0c9fb6ee-58e4-11e8-8c1a-31ef0bb6e68c.png)
        - 1000：`717 ms`

            ![18dispose1000](https://user-images.githubusercontent.com/3851540/40090389-0cca9274-58e4-11e8-933e-a4c13027ad23.png)
        - 10000：`9401 ms`

            ![19dispose10000](https://user-images.githubusercontent.com/3851540/40090390-0cf5b8aa-58e4-11e8-8672-83cd8ad8703a.png)
        - 100000：`81462 ms`

            ![20dispose100000](https://user-images.githubusercontent.com/3851540/40246328-ed709072-5afa-11e8-9acc-4777a0910531.png)

## 測試結果

處理方式\筆數|10|100|1,000|10,000|10,000
:---|---:|---:|---:|---:|---:
逐筆 Add 並逐筆 SaveChanges|11|187|4,282|401,177|44,827,184
逐筆 Add 及一次 SaveChanges|7|79|2,309|195,678|19,916,508
批次 Add 及 SaveChanges|9|77|1,955|179,073|16,710,546
批次 Add 及 SaveChanges <br/>並 dispose context|12|91|717|9,401|81,462

> 單位 `ms`

## 心得

實驗時間超長的，有些情境的執行時間遠超出我的想像 (使用 `逐筆 Add 並逐筆發動 SaveChanges` 處理 10 萬筆資料時竟花超過 12 個小時)，為了避免造成實驗數據誤差，還不能同時執行其他情境，真的很花時間。
原本打算各個情境都測個三、五次取平均值，讓數字相對準確點，無奈某些情境實在太耗時，不得不放棄，實際執行時間的數字部份就勉強當做趨勢比較參考吧

實際測試前，原本就料想到一定是 `逐筆 Add 並逐筆 SaveChanges` 速度最慢，原以為 `逐筆 Add 及一次 SaveChanges` 會有巨大的效能優化，想不到還是極度緩慢，`批次 Add 及 SaveChanges` 已獲得倍數級改善，而 `批次 Add 及 SaveChanges 並 dispose context` 竟然出現令人咋舌的速度優化，還好有來參加忠成老師的課程，學到 EntityFramework 這等調校手法就值回票價了

假設沒有這次學習經驗，我想我應該會直接透過 dapper 或是 ado.net 來處理，在效能與便利性上的取捨我個人意見跟忠成老師很接近：沒有放諸四海皆適合的技術及架構，依不同情境選擇最合適的技術才是正確做法，不該一昧堅持特定做法而畫地自限。

## 參考資訊
