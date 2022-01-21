---
title: "ASP.NET Core Configuration 中的 array 沒有正確覆寫"
date: 2022-01-21T00:30:00+08:00
lastmod: 2022-01-21T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp"]
slug: "aspdotnet-core-config-array-not-override"
---

## ASP.NET Core Configuration 中的 array 沒有正確覆寫

之前筆記 [在 ASP.NET Core Configuration 中使用 array](/aspdotnet-core-config-array) 中紀錄到如何在 config 中使用 array，不過在使用後發現有些限制，趁著這個機會紀錄一下

## 基本環境說明

1. macOS Monterey 12.1
2. .NET SDK 6.0.100
3. 範例 config

    - appsettings.json

        ```json
        {
            "ConfigArray": ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]
        }
        ```

    - appsettings.Development.json

        ```json
        {
            "ConfigArray": ["Mon","Tue","Wed","Thu","Fri"]
        }
        ```

## 預期結果與實際落差

- 預期結果

    ```txt
    Mon
    Tue
    Wed
    Thu
    Fri
    ```

- 實際結果

    ![1actual](https://user-images.githubusercontent.com/3851540/150483824-39f72cff-2224-43d4-9a50-b44330c10320.png)

## 問題發生原因與調整方式

- 問題發生原因

    [Microsoft Docs: Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#bind-an-array) 提到 `ConfigurationBinder` 在處理 array 時是透過 array 指標處理的，也就是說 array 不是單看 config key 而是 `config-key:[0.....n]`

- 調整方式

    1. 不要依賴 base config (`appsettings.json`)，每個 envrinment 的 config 都分開寫，避免覆寫 array 時出現意外
    2. 用 string 來儲存 (e.g. `"Mon,Tue,Wed,Thu,Fri"`，再用 `,` 切分存入 array)

## 心得

我個人比較傾向使用不依賴 base config 將每個 envrinment config 分開寫，讓 config 比較一目瞭然，不過現在這種類型的 config 多是工具 generate 出來的，需要人眼去看的機會有限，除此之外設定較繁瑣也是其他同事選擇使用 string 的關鍵因素

兩個方法各有擁護者，就依團隊規範來選擇囉

## 參考資訊

1. [在 ASP.NET Core Configuration 中使用 array](/aspdotnet-core-config-array)
2. [Overriding an Array Configuration Object](https://levelup.gitconnected.com/overriding-an-array-configuration-object-4d93470e97d0)
3. [Microsoft Docs: Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#bind-an-array)
