---
title: "[C#] 將 JSON 轉為 Dictionary"
date: 2020-02-01T14:30:00+08:00
lastmod: 2020-02-01T14:30:31+08:00
draft: false
tags: ["csharp","JSON"]
slug: "json-to-dictionary"
---

## [C#] 將 JSON 轉為 Dictionary

使用 JSON 做為避免在程式中寫死固定值的解決方案是種常見做法，不過一旦系統日漸龐大複雜起來時，可能就需要有良好的管理方式或是流程，像是將共用的 config 抽出成另個檔案，專屬於特定系統的 config 則僅存在特定系統中，不過這麼一來就需要留意 config 是否重複的問題，所以我嘗試將 JSON 轉為 Dictionary 預做檢查，避免 config 與預期不一致

## 基本環境說明

1. macOS Catalina 10.15.2
2. .NET Core SDK 3.1.101
3. C# 8.0

## 程式碼與實際效果

1. 程式碼

    ```cs
    private static Dictionary<string, string> ToDictionary(string jsonSrting)
    {
        var jsonObject = JObject.Parse(jsonSrting);
        var jTokens = jsonObject.Descendants().Where(p => !p.Any());
        var tmpKeys = jTokens.Aggregate(new Dictionary<string, string>(),
            (properties, jToken) =>
            {
                properties.Add(jToken.Path, jToken.ToString());
                return properties;
            });
        return tmpKeys;
    }
    ```

2. 使用方式

    ```cs
    ToDictionary(File.ReadAllText(jsonFilePath));
    ```

3. 實際效果

    - 原始 json

        ```json
        {
          "widget": {
            "debug": "on",
            "window": {
              "title": "Sample Konfabulator Widget",
              "name": "main_window",
              "width": 500,
              "height": 500
            },
            "image": {
              "src": "Images/Sun.png",
              "name": "sun1",
              "hOffset": 250,
              "vOffset": 250,
              "alignment": "center"
            },
            "text": {
              "data": "Click Here",
              "size": 36,
              "style": "bold",
              "name": "text1",
              "hOffset": 250,
              "vOffset": 100,
              "alignment": "center",
              "onMouseUp": "sun1.opacity = (sun1.opacity / 100) * 90;"
            }
          }
        }
        ```

    - Dictionary

        ![1dictionary](https://user-images.githubusercontent.com/3851540/73608722-44b75a80-4601-11ea-99d6-f6c2cf3ef9a9.png)

## 心得

不知道做法正不正確，但先試試看避免找不到其他方法，先有個版本出來在討論時或是驗證時至少有個基準點，可能還是我功力不足：先求有再求好，沒辦法一步到位

## 參考資訊

1. [Flatten json object to send within an Azure Hub Notification Template](https://www.bfcamara.com/post/75172803617/flatten-json-object-to-send-within-an-azure-hub)
2. [C# flattening json structure](https://stackoverflow.com/a/35838986/3600583)
