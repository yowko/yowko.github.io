---
title: "Enum To List<SelectListItem> 及 Enum To SelectList"
date: 2018-01-30T02:53:00+08:00
lastmod: 2021-11-03T02:53:41+08:00
draft: false
tags: ["csharp"]
slug: "enum-to-selectlist"
aliases:
    - /2018/01/enum-to-selectlist.html
---
## Enum To List&lt;SelectListItem&gt; 及 Enum To SelectList

無意間看到專案中的一段程式碼，讓我停頓了一下，一時之間好幾個念頭閃過卻不知道該選擇哪個做法來改善

大意是 View 中有個欄位資料型別是一個 enum，預設範本直接透過 `@Html.EnumDropDownListFor` 進行綁定，但這樣的輸出有個缺點： `enum 的預設值也會被輸出`，為了避免預設值也被輸出成選項，第一個念頭就是透過自行組裝 select 及 option 相關 html 的做法

只是靜心想想，是不是有其他更好的做法，想到三個做法筆記一下

## 前提說明

1. enum

    ```cs
    public enum StatusEnum
    {
        Default = 0,
        Active,
        Verified,
        Closed,
        Stop
    }
    ```

2. 使用 `@Html.EnumDropDownListFor`

    ```cs
    @Html.LabelFor(model => model.Status, htmlAttributes: new { @class = "control-label col-md-2" })
    @Html.EnumDropDownListFor(model => model.Status, htmlAttributes: new { @class = "form-control" })
    ```

3. 希望做到可以過濾掉指定選項(Default)

## 自行組裝 html

```cs
@Html.LabelFor(model => model.Status, htmlAttributes: new { @class = "control-label col-md-2" })
<select name="Status">
    @foreach (StatusEnum enumItem in Enum.GetValues(typeof(StatusEnum)))
    {
        //僅在 model 本身為 default 時才會出現 default 選項
        if (enumItem != StatusEnum.Default || Model.Status == StatusEnum.Default)
        {
            <option @((enumItem == Model.Status) ? "selected='selected'" : String.Empty) value="@enumItem">@enumItem.ToString()</option>
        }                        
    }
</select>
```

## 調整 DropDownList 的資料源

* Enum To List＜SelectListItem＞
    1. LINQ - Query syntax

        ```cs
        from StatusEnum status in Enum.GetValues(typeof(StatusEnum))
        where status != StatusEnum.Default
        select new SelectListItem
        {
            Text = status.ToString(),
            Value = ((int)status).ToString()
        };
        ```

        ![2querysyntax](https://user-images.githubusercontent.com/3851540/35528013-74bd794e-0567-11e8-8039-c6bf581bb31d.png)

    2. LINQ - Method syntax

        ```cs
        Enum.GetValues(typeof(StatusEnum)).Cast<StatusEnum>()
        .Where(se => se != StatusEnum.Default).Select(se => new SelectListItem
        {
        Text = se.ToString(),
        Value = ((int)se).ToString()
        });
        ```

        ![1methodsyntax](https://user-images.githubusercontent.com/3851540/35528012-7490c85e-0567-11e8-9546-1c7aca4cec7f.png)

* Enum To SelectList

    ```cs
    var status = Enum.GetValues(typeof(StatusEnum)).Cast<StatusEnum>()
    .Where(se => se != StatusEnum.Default).Select(se => new 
    {
        value = (int)se, text = se.ToString()
    });;
    var result= new SelectList(status, "value", "text");
    ```

    ![3selectlist](https://user-images.githubusercontent.com/3851540/35528014-74ea1a30-0567-11e8-9f6c-f029839e616a.png)

## 心得

`Enum to SelectList` 其實與 `LINQ - Method syntax` 本質相同，只是將過濾後的資料轉為 SelectList 而已，`LINQ - Query syntax` 也可以比照辦理轉為 SelectList

需求很明確、程式很淺顯，但卻讓我想到不少：如果要快，第一個念頭就會使用 html 開工了，既直覺又不用擔心其他地方被影響，不過一昧求快是很難更上一層樓的，應該要對所有程式碼抱著持續調校優化的企圖心，督促自己思考才不會被淘汰

## 參考資訊

1. [How can I convert an enumeration into a List\<selectlistitem>?\</selectlistitem>](https://stackoverflow.com/questions/3489453/how-can-i-convert-an-enumeration-into-a-listselectlistitem)
2. [How to get the values of an enum into a SelectList](https://stackoverflow.com/questions/1110070/how-to-get-the-values-of-an-enum-into-a-selectlist)
3. [有 EnumDropDownListFor 為什麼還要客製 Enum to Dropdownlist ？！](/2017/02/customize-asp-dot-net-mvc-enum-to-dropdownlist.html)
4. [Can you loop through all enum values?](https://stackoverflow.com/questions/972307/can-you-loop-through-all-enum-values)
5. [Query Syntax and Method Syntax in LINQ (C#)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq?WT.mc_id=DOP-MVP-5002594)
