---
title: "ASP.NET MVC 如何 POST LIST 資料 2"
date: 2017-02-03T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-post-list-2"
aliases:
    - /2017/02/aspnet-mvc-post-list-2.html
---
## ASP.NET MVC 如何 POST LIST 資料 2

先前筆記 [ASP.NET MVC 如何 POST LIST 資料 1](/2017/02/aspnet-mvc-post-list-1) 紀錄著該如何 post 單一資料屬性 list 到後端，而這篇將介紹該如何 post 多種資料屬性 list 到後端

## 定義資料欄位

* 欄位名稱
* 相關屬性設定
* 基本檢核

    ```cs
    public class PstData
    {
        public int Id { get; set; }
        [Required]
        public string name { get; set; }
        [Required]
        public List<WeekDaysEnum> days { get; set; }
        [Required]
        public RegionEnum region { get; set; }
        [Required]
        [Range(0, Int32.MaxValue)]
        public int price { get; set; }
    }
    ```

## Controller

>controller 程式碼相對單純

1. 起始畫面用的 action
2. 準備接受 post 資料的 action
    * 套用 `[HttpPost]`
    * 建議也套用 `[ValidateAntiForgeryToken]` 以增加安全性
    * 接受 list 資料的參數
3. 接受資料的 action 加上基本檢核判斷

    ```cs
    public ActionResult Add()
    {
        List<PstData> data = new List<PstData>();
        foreach (var item in Enumerable.Range(0, 5))
        {
            PstData tmp = new PstData() { Id = item };
            data.Add(tmp);
        }
        return View(data);
    }
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Add(List<PstData> data)
    {
        if (ModelState.IsValid)
        {
            return View();
        }
        return View(data);
    }
    ```

## View

1. 定義 post list 遞增變數 `i`

2. 使用 form
    * `@using (Html.BeginForm())`
    * 如果 post action 名稱與 get 相同，不用另外寫
3. 加上 `Html.AntiForgeryToken` 增加安全性 `@Html.AntiForgeryToken()`
4. 加入迴圈並在迴圈內定義所有回傳資料欄位
    * `name` 為 controller 接受參數名稱加上 `i` 再加上 `欄位名稱`
        * e.g. `data[i].name`
    * 為不同欄位屬性加上不同 input
        * textbox
        * checkbox
        * radiobutton
5. 加上驗證訊息
    * `name` 為 controller 接受參數名稱加上 `i` 再加上 `欄位名稱`
    * e.g. `data[i].name`
6. 完整程式碼

    ```cs
    <div class="row">
    @using (Html.BeginForm())
    {
        @Html.AntiForgeryToken()
        foreach (var item in Model)
        {
            int daysIndex = 0;
            @Html.Hidden($"data[{i}].Id", item.Id)
            <div class="row">
                <label class="col-md-2 control-label">name</label>
                <div class="col-md-10">
                    @Html.TextBox($"data[{i}].name", item.name) @Html.ValidationMessage($"data[{i}].name", new { @class = "text-danger" })
                </div>
            </div>
            <div class="row">
                <label class="col-md-2 control-label">Days</label>
                <div class="col-md-10">
                    @foreach (WeekDaysEnum jtem in Enum.GetValues(typeof(WeekDaysEnum)))
                    {
                        if (jtem != WeekDaysEnum.Default)
                        {
                            string _tmp = string.Concat($"data{i}.days_", daysIndex);
                            string name = $"data[{i}].days";
                            <div class="col-md-3">
                                <input type="checkbox" id="@_tmp" value="@jtem" name="@name" @((item.days != null && item.days.Contains(jtem)) ? "checked" : string.Empty) />
                                <label for="@_tmp">
                                    @jtem.ToString()
                                </label>
                            </div>
                            daysIndex++;
                        }
                    }
                    @Html.ValidationMessage($"data[{i}].days", new { @class = "text-danger" })
                </div>
            </div>
            <div class="row">
                <label class="col-md-2 control-label">Region</label>
                <div class="col-md-10">
                    @Html.EnumDropDownList($"data[{i}].region", typeof(RegionEnum), " - Choice - ")
                    @Html.ValidationMessage($"data[{i}].region", new { @class = "text-danger" })
                </div>
            </div>
            <div class="row">
                <label class="col-md-2 control-label">price</label>
                <div class="col-md-10">
                    @Html.TextBox($"data[{i}].price", item.price)
                    @Html.ValidationMessage($"data[{i}].price", new { @class = "text-danger" })
                </div>
            </div>
            <hr />
            i++;
        }
        <div class="row">
            <div class="col-md-offset-2 col-md-10">
                <input class="btn btn-primary pull-right" type="submit" value="submit" />
            </div>
        </div>

    }
    </div>
    ```

## 參考資訊

1. [ASP.NET MVC 如何 POST LIST 資料 1](/2017/02/aspnet-mvc-post-list-1)
