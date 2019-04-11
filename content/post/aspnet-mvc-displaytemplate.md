---
title: "ASP.NET MVC 統一顯示格式 (使用 DisplayTemplate)"
date: 2017-06-21T23:16:00+08:00
lastmod: 2018-12-09T23:16:03+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-displaytemplate"
aliases:
    - /2017/12/aspnet-mvc-displaytemplate.html
---
# ASP.NET MVC 統一顯示格式 (使用 DisplayTemplate)
在 ASP.NET MVC 中所有資料格式都有預設的顯示方式(ex：DateTime `2017/12/19 下午 11:20:00 `、Deciaml `123457.00`)，有的資料格式顯示方式還會受到地區、文化、語系影響，遇到這類文化語系設定造成的畫面不一致或是預設顯示跟期望有落差時，可以在畫面上直接指定格式來統一輸出，只是畫面一多時日後維護就會遇到麻煩，面對這類需求最好的做法就是修改預設顯示格式

最近專案剛好有個統一全站畫面顯示的需求：日期格式：`MM/dd/yyyy HH:mm:ss`、數值格式：`0,000.00`(千分位，小數點兩位)，我立馬想到透過 
DisplayTemplate 就可以秒殺搞定，結果我忘記語法，新增 template 時 還卡了一下，所以怒紀錄一篇  哈哈

## 直接複寫預設顯示格式
1. 在 Views --> Shared 資料夾中新增 `DisplayTemplates` 資料夾
    
    ![1addDisplayTemplates](https://user-images.githubusercontent.com/3851540/34167974-3eb86ba6-e51e-11e7-9bbc-994b7e71241c.png) 
2. 在 `DisplayTemplates` 資料夾建立以 `目標資料型態` (e.g. `DateTime`、`Decimal`) 為名稱的 View 
    
    > 使用時不需另外編譯 
    
    - DateTime
        - 加入 `DateTime.cshtml`
            
            ![2adddatetime](https://user-images.githubusercontent.com/3851540/34167975-3f03342e-e51e-11e7-923c-c6a705593043.png)

            ![3adddatetime](https://user-images.githubusercontent.com/3851540/34167977-3f6505be-e51e-11e7-9343-07fdfecd9fa3.png)
        - 指定 model 型態(記得處理 null)
            
            ```cs
            @model DateTime?
            ```
            
            - 未處理 null 錯誤
                
                ```
                The model item passed into the dictionary is null, but this dictionary requires a non-null model item of type 'System.DateTime'.
                ```
                
                ![3-1nullerror](https://user-images.githubusercontent.com/3851540/34167976-3f3279dc-e51e-11e7-8782-89691eb869b9.png)
        - 依實際需求客製輸出格式
            
            ```cs
            @if (Model == null)
            {
                <span class="text-danger">No DateTime Data!!</span>
            }
            else
            {
                <span>@(((DateTime)Model).ToString("MM/dd/yyyy HH:mm:ss"))</span>
            }
            ```
        - 完整程式碼
            
            ```cs
            @model DateTime?
            @if (Model == null)
            {
                <span class="text-danger">No DateTime Data!!</span>
            }
            else
            {
                <span>@(((DateTime)Model).ToString("MM/dd/yyyy HH:mm:ss"))</span>
            }
            ```
        - 前後差異
            - 修改前
                
                ![4before](https://user-images.githubusercontent.com/3851540/34167978-3f8eef6e-e51e-11e7-9b09-5860e80a326e.png)
            - 修改後
                
                ![5after](https://user-images.githubusercontent.com/3851540/34167979-3fb9b78a-e51e-11e7-87f6-ee45d53d38b6.png)
    - Decimal
        - 加入 `Decimal.cshtml`
            
            ![6adddecimal](https://user-images.githubusercontent.com/3851540/34167980-3fe3f63a-e51e-11e7-8408-1f58e9998223.png)

            ![7adddecimal](https://user-images.githubusercontent.com/3851540/34167981-401506b2-e51e-11e7-948f-d886d49bf0e9.png)
            
            - 無法加入 `decimal.cshtml`(可以加入 `Decimal.cshtml` 後再 rename)
                
                ![8decimalerror](https://user-images.githubusercontent.com/3851540/34167965-3d7a58bc-e51e-11e7-8e2b-a9d2d78ac96b.png)
        - 指定 model 型態(記得處理 null)
            
            ```cs
            @model decimal?
            ```
            - 未處理 null 錯誤
                
                ```cs
                The model item passed into the dictionary is null, but this dictionary requires a non-null model item of type 'System.Decimal'.
                ```
                
                ![9decimalnullerror](https://user-images.githubusercontent.com/3851540/34167966-3daffbf2-e51e-11e7-9dba-dd4e3725a29b.png)
        - 依實際需求客製輸出格式
            
            ```cs
            @if (Model == null)
            {
                <span class="text-danger">No Decimal Data!!</span>
            }
            else
            {
                <span>@(((decimal)Model).ToString("N"))</span>
            }
            ```
        - 完整程式碼
            
            ```cs
            @model decimal?

            @if (Model == null)
            {
                <span class="text-danger">No Decimal Data!!</span>
            }
            else
            {
                <span>@(((decimal)Model).ToString("N"))</span>
            }
            ```
        - 前後差異
            - 修改前
                
                ![10before](https://user-images.githubusercontent.com/3851540/34167968-3ddf6cca-e51e-11e7-85db-49f37e4e332e.png)
            - 修改後
                
                ![11after](https://user-images.githubusercontent.com/3851540/34167970-3e08b594-e51e-11e7-9f27-89abc22df1d3.png)

3. 使用上檔案名稱不分大小寫
    - `DateTime.cshtml` 等於 `datetime.cshtml`
    - `Decimal.cshtml` 等於 `decimal.cshtml`

## 額外新增顯示格式

> 上述方式可以修改全站預設顯示方式，雖然方便但如果不是全站修改就不適用，以下介紹其他做法

* 新增方式與上述做法相同
    - 在 Views --> Shared 新增 `DisplayTemplates` 資料夾
        
        ![1addDisplayTemplates](https://user-images.githubusercontent.com/3851540/34167974-3eb86ba6-e51e-11e7-9bbc-994b7e71241c.png) 
    - 在 `DisplayTemplates` 資料夾建立 View (View 名稱不能與資料型態相同 - 會直接複寫預設格式)
        
        ![12customview](https://user-images.githubusercontent.com/3851540/34167971-3e32935a-e51e-11e7-8ebf-8bd61b1c9889.png) 

1. 手動指定
    - 格式
        
        ```cs
        @Html.DisplayFor(modelItem => modelItem.{欄位},"{自訂 View 名稱}")
        ```
    - 實例
        
        ```cs
        @Html.DisplayFor(modelItem => item.ShowDateTime,"YowkoDateTime")
        ```
    - 自訂程式碼(`YowkoDateTime.cshtml`)
        
        ```cs
        @model DateTime?
        @if (Model == null)
        {
            <span class="text-danger">Yowko:No DateTime Data!!</span>
        }
        else
        {
            <span>Yowko:@(((DateTime)Model).ToString("MM/dd/yyyy HH:mm:ss"))</span>
        }
        ```
    - 效果
        
        ![13yowkoview](https://user-images.githubusercontent.com/3851540/34167972-3e5b0646-e51e-11e7-8c4d-3192d9754d90.png)
2. 使用 UIHint attibute
    - 格式
        
        ```cs
        [UIHint("{自訂 View 名稱}")]
        public type name { get; set; }
        ```
    - 實例
        
        ```cs
        [UIHint("YowkoDecimal")]
        public Nullable<decimal> ShowDecimal { get; set; }
        ```
    - 自訂程式碼(`YowkoDecimal.cshtml`)
        
        ```cs
        @model decimal?

        @if (Model == null)
        {
            <span class="text-danger">Yowko:No Decimal Data!!</span>
        }
        else
        {
            <span>Yowko:@(((decimal)Model).ToString("N"))</span>
        }
        ```
    - 效果
        
        ![14yowkodecimal](https://user-images.githubusercontent.com/3851540/34167973-3e8a766a-e51e-11e7-9e6a-50f215a635ef.png)
    - 缺點：因為改到 .cs 需要 compile

## 心得
這次使用 DisplayTemplate 的過程讓我徹徹底底領悟到技術的熟悉度主要取決於使用頻率，想當年在專案公司時不用多加思考就可以搞定 DisplayTemplate，經過二、三年時間使用頻率較低現在居然記不起語法XD 

有了這次經驗希望自己不要忘記實在是奢求，只求日後至少要記得可以找到自己的筆記呀 @@"

# 參考資訊
1. [Display templates](http://www.growingwiththeweb.com/2012/12/aspnet-mvc-display-and-editor-templates.html)