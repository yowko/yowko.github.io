---
title: "[C#] 將 .NET Core 中的 Configuration 輸出為 JSON"
date: 2020-02-08T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["csharp","JSON","dotnet core"]
slug: "configuration-to-json"
---

## [C#] 將 .NET Core 中的 Configuration 輸出為 JSON

之前筆記 [[C#] 將 Dictionary 轉為 JSON](/dictionary-to-json) 紀錄到 將 Dictionary 轉為 JSON 的方式，對 .NET Core 有些認識的朋友馬上就想到是為了處理 .NET Core Configuration，主要需求就是將搭配不同環境 build 完的最終版 Configuration 存回 JSON 之中，因為沒找到方法所以只好硬來，同事看完後覺得可以試試 Configuration.GetSection 遞迴方式來處理可能會簡單些，既然有方向就來試試囉

## 基本環境說明

1. macOS Catalina 10.15.2
2. .NET Core SDK 3.1.101
3. C# 8.0
4. 原始 JSON

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

## 程式碼與實際效果

1. 程式碼

    ```cs
    private static JObject ToJson(IEnumerable<IConfigurationSection> configurationSections)
    {
        JContainer result = null;
        var setting = new JsonMergeSettings {MergeArrayHandling = MergeArrayHandling.Merge};

        var enumerable = configurationSections as IConfigurationSection[] ?? configurationSections.ToArray();

        // 檢查是否是 array：value 不為 null 表示為最底層; array 的元素 key 會是 int (可能出現誤判);最後是 path 扣掉 int 的部份如果都是相同值就進一步確認為 array
        if (enumerable.All(a => a.Value != null
                                && int.TryParse(a.Key, out var _))
            && enumerable.Select(a => a.Path.Split(":").Reverse().Skip(1).First()).Distinct().Count() == 1)
        {
            var tmpList = new List<string>();
            tmpList.AddRange(enumerable.Select(a => a.Value));
            //取得 array 名稱
            var key = enumerable.Select(a => a.Path.Split(":").Reverse().Skip(1).First()).First();
            result = new JObject {{key, JToken.FromObject(tmpList)}};

            return (JObject) result;
        }

        foreach (var child in enumerable)
        {
            var obj = new JObject();
            if (string.IsNullOrWhiteSpace(child.Value))
            {
                var toJson = ToJson(child.GetChildren());
                // 如果是 array 的話，就換掉整個 object，避免多一層相同 property name
                if (toJson.ContainsKey(child.Key))
                {
                    obj = toJson;
                }
                else
                {
                    obj.Add(child.Key, ToJson(child.GetChildren()));
                }
            }
            else
            {
                //處理轉型，這邊只寫了 int 與 bool
                if (int.TryParse(child.Value, out var intValue))
                {
                    obj.Add(child.Key, intValue);
                }
                else if (bool.TryParse(child.Value, out var boolValue))
                {
                    obj.Add(child.Key, boolValue);
                }
                else
                {
                    obj.Add(child.Key, child.Value);
                }
            }

            if (result == null)
            {
                result = obj;
            }
            else
            {
                result.Merge(obj, setting);
            }
        }

        return (JObject) result;
    }
    ```

2. 實際使用方式

    > 將 ConfigurationBuilder build 完的 config 使用 `GetChildren` 之前傳入

    ```cs
     ToJson(configRoot.GetChildren()).ToString()
    ```

3. 實際效果

    - 原始 Dictionary

        ![1dictionary](https://user-images.githubusercontent.com/3851540/74087729-12f33780-4aca-11ea-8b56-174d76f3991e.png)

    - json string

        ![2json](https://user-images.githubusercontent.com/3851540/74087740-17b7eb80-4aca-11ea-8804-1e4f2601c471.png)

## 心得

原本傻傻地要直接上手工轉換的版本，幸虧強大的同事提點才可以學到更適合的方式，有強大的同事真好，不過這也再次驗證了對於 .NET Core 我的掌握度還是很不足 XD

## 參考資訊

1. [ConfigurationRoot.GetChildren](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationroot.getchildren?view=dotnet-plat-ext-3.1&WT.mc_id=DOP-MVP-5002594)
2. [ConfigurationSection.GetChildren](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationsection.getchildren?view=dotnet-plat-ext-3.1&WT.mc_id=DOP-MVP-5002594)
3. [[C#] 將 Dictionary 轉為 JSON](/dictionary-to-json)
