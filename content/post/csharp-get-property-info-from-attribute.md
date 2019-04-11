---
title: "C# 如何用特定的 attribute 取得 property 資訊"
date: 2017-02-20T01:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["C#"]
slug: "csharp-get-property-info-from-attribute"
aliases:
    - /2017/02/c-sharp-use-specific-attribute-get-property-info.html
---
# C# 如何用特定的 attribute 取得 property 資訊
今天正在試著如何把自訂 method 寫得更彈性些，不要有太多 magic string 判斷，所以打算利用 attribute 做為媒介，印象中以前也做過這件事，所以筆記一篇對我逝去的記憶表達思念之情XD

## 情境說明
1. 個人資訊的自訂類別
2. 其中一個欄位是 unique key，也準備用來當做 key-value pair 中的 key
3. 剩餘欄位皆封裝進 key-value pair 中的 value

## 自訂類別

```cs
public class PersonInfo
{
	public Guid ID { get; set; }

	public string Name { get; set; }

	public string Tel { get; set; }

	public string EMail { get; set; }
}
```

## 簡易作法
- 我們一眼已經看到 ID 是 key，取得其他 property 就是這樣
    
    ```cs
    var props = GetType().GetProperties().Where(d => d.Name != "ID");
    ```
- 只是這樣 `ID` 就是一個 magic string，有天 ID rename 就會出現問題，所以希望用 attribute 來標示

## 改善作法
1. 把 ID 加上 [KeyAttribute]
    
    ```cs
    [Key]
	public Guid ID { get; set; }
    ```
2. 透過指定 `KeyAttribute` 來找 property name
    
    ```cs
    var key = GetType().GetProperties().FirstOrDefault(gt => gt.GetCustomAttributes().Any(a => ((KeyAttribute)a) != null));
    ```
    - 寫完再看一次都還要回想在寫什麼XD  可見是很不好的寫法
    - 註解如下
        - GetType().GetProperties() 取得類別中所有 property
        - gt.GetCustomAttributes() 取得 property 的所有 attribute
        - gt.GetCustomAttributes().Any(a => ((KeyAttribute)a) != null) 檢查 attribute 是否有 KeyAttribute
        - GetType().GetProperties().FirstOrDefault(gt => gt.GetCustomAttributes().Any(a => ((KeyAttribute)a) != null)) 取得所有 peroperty 中有被標記 KeyAttribute 的第一個
3. 取得其他 property 就變這樣
    
    ```cs
    var props = GetType().GetProperties().Where(d => d.Name != key.Name);
    ```

## 如何簡化
- 上面的第二步驟  LinQ 語法雖然不長，但還是需要多看幾眼才比較清楚在做什麼
- 經過搜尋解法後，找到下面的做法
    
    ```cs
    var key = GetType().GetProperties().FirstOrDefault(prop => prop.IsDefined(typeof(KeyAttribute), false));
    ```
- 直接檢查是否有 KeyAttribute 的定義，而且不是繼承而來的 ( IsDefined 第二個參數 false 表不是繼承)

## 完整程式碼
1. class
    
    ```cs
    public class PersonInfo
    {
    	[Key]
    	public Guid ID { get; set; }
    
    	public string Name { get; set; }
    
    	public string Tel { get; set; }
    
    	public string EMail { get; set; }
    
    	public KeyValuePair<string, List<KeyValuePair<string, string>>> ToDic()
    	{
    		var key = GetType().GetProperties().FirstOrDefault(prop => prop.IsDefined(typeof(KeyAttribute), false));
    		var props = GetType().GetProperties().Where(d => d.Name != key.Name);
    		List<KeyValuePair<string, string>> datas = new List<KeyValuePair<string, string>>();
    		foreach (var item in props)
    		{
    			datas.Add(new KeyValuePair<string, string>(item.Name, item.GetValue(this).ToString()));
    		}
    		return new KeyValuePair<string, List<KeyValuePair<string, string>>>(key.GetValue(this), datas);
    	}
    }
    ```
2. 實際使用
    
    ```cs
    var yowkoinfo = new PersonInfo() { ID = Guid.NewGuid(), Name = "Yowko", Tel = "1234567890", EMail = "yowko@yowko.com" };
	yowkoinfo.ToDic().Dump();
    ```

# 參考連結
1. [Get all properties which marked certain attribute](http://stackoverflow.com/questions/7305787/get-all-properties-which-marked-certain-attribute)