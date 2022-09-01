---
title: "[C#] 將 Dictionary 轉為 JSON"
date: 2020-02-02T14:30:00+08:00
lastmod: 2022-09-01T14:30:31+08:00
draft: false
tags: ["csharp","JSON"]
slug: "dictionary-to-json"
---

## [C#] 將 Dictionary 轉為 JSON

之前筆記 [[C#] 將 JSON 轉為 Dictionary](/json-to-dictionary) 紀錄到如果將 JSON file 轉為 C# 的 Dictionary 來做後續加工處理，當然有 JSON to Dictionary 就需要有 Dictionary to JSON 囉

## 基本環境說明

1. macOS Catalina 10.15.2
2. .NET Core SDK 3.1.101
3. C# 8.0

## 程式碼與實際效果

1. 程式碼

    ```cs
    public static JObject ToJson(Dictionary<string, string> keyValues)
    {
        JContainer result = null;
        var setting = new JsonMergeSettings {MergeArrayHandling = MergeArrayHandling.Merge};
        foreach (var pathValue in keyValues)
        {
            if (result == null)
            {
                result = UnflattenSingle(pathValue);
            }
            else
            {
                result.Merge(UnflattenSingle(pathValue), setting);
            }
        }
        return result as JObject;
    }

    private static JContainer UnflattenSingle(KeyValuePair<string, string> keyValue)
    {
        var path = keyValue.Key;
        var value = keyValue.Value;
        var pathSegments = SplitPath(path);

        JContainer lastItem = null;
        //build from leaf to root
        foreach (var pathSegment in pathSegments.Reverse())
        {
            var type = GetJsonType(pathSegment);
            switch (type)
            {
                case JsonType.Object:
                    var obj = new JObject();
                    if (null == lastItem)
                    {
                        obj.Add(pathSegment,value);
                    }
                    else
                    {
                        obj.Add(pathSegment,lastItem);
                    }
                    lastItem = obj;
                    break;
                case JsonType.Array:
                    var array = new JArray();
                    var index = GetArrayIndex(pathSegment);
                    array = FillEmpty(array, index);
                    if (lastItem == null)
                    {
                        array[index] = value;
                    }
                    else
                    {
                        array[index] = lastItem;
                    }
                    lastItem = array;
                    break;
            }
        }
        return lastItem;
    }

    public static IList<string> SplitPath(string path){
        IList<string> result = new List<string>();
        var reg = new Regex(@"(?!\.)([^. ^\[\]]+)|(?!\[)(\d+)(?=\])");
        foreach (Match match in reg.Matches(path))
        {
            result.Add(match.Value);
        }
        return result;
    }

    private static JArray FillEmpty(JArray array, int index)
    {
        for (var i = 0; i <= index; i++)
        {
            array.Add(null);
        }
        return array;
    }

    private enum JsonType{
        Object, Array
    }
    private static JsonType GetJsonType(string pathSegment)
    {
        return int.TryParse(pathSegment, out var _) ? JsonType.Array : JsonType.Object;
    }

    private static int GetArrayIndex(string pathSegment)
    {
        if (int.TryParse(pathSegment, out var result))
        {
            return result;
        }
        throw new Exception("Unable to parse array index: " + pathSegment);
    }
    ```

2. 使用方式

    ```cs
    ToJson(jsonDictionary)).ToString();
    ```

3. 實際效果

    - 原始 Dictionary

        ![1dictionary](https://user-images.githubusercontent.com/3851540/73608722-44b75a80-4601-11ea-99d6-f6c2cf3ef9a9.png)

    - json string

        ![1json](https://user-images.githubusercontent.com/3851540/73608715-281b2280-4601-11ea-97db-ce2c1b7d69a1.png)

## 心得

程式碼數量看來把 dictionary 轉回 json 比起把 json 攤平成 dictionary 容易許多，另外的問題是資料型態有可能跑掉：`int`、`bool` 都變成了 `string`，這點要特別留意，除此之外初步滿足功能需求

## 參考資訊

1. [[C#] 將 JSON 轉為 Dictionary](/json-to-dictionary)
2. [How to unflatten flattened json in C#](https://stackoverflow.com/questions/40541842/how-to-unflatten-flattened-json-in-c-sharp)
