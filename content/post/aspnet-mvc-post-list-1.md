---
title: "ASP.NET MVC 如何 POST LIST 資料 1"
date: 2017-02-01T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-post-list-1"
aliases:
    - /2017/02/aspnet-mvc-post-list-1.html
---
## ASP.NET MVC 如何 POST LIST 資料 1

開發 ASP.NET MVC 網站表單時，不時會遇到需要 post list 資料的需求，功能雖然很容易，但每次遇到就重寫也不是辦法，順手紀錄一下也讓大家看看有沒有其他更好的方法，我一直覺得自己用的方法很笨，但可以解決問題就是了

## Controller

>controller 程式碼相對單純

1. 起始畫面用的 action

2. 準備接受 post 資料的 action
    - 套用 `[HttpPost]`
    - 建議也套用 `[ValidateAntiForgeryToken]` 以增加安全性
    - 加上接受 list 資料的參數
3. 接受資料的 action 加上邏輯及基本檢核

    ```cs
    public ActionResult Index()
    {
        return View();
    }
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Index(List<WeekDaysEnum> days)
    {
        //邏輯檢核
        if (days == null)
        {
            ModelState.AddModelError("days", "must choose one at least!!!");
        }
        if (ModelState.IsValid)
        {
            //實際處理邏輯
            return View();
        }
        return View(days);
    }
    ```

## View

1. 定義用來遞增的變數給 label for 用 `int daysIndex = 0`;
2. 使用 form
    - @using (Html.BeginForm())
    - 如果 post action 名稱與 get 相同，不用另外寫
3. 加上 Html.AntiForgeryToken 增加安全性 @Html.AntiForgeryToken()
4. 定義一個全選按鈕

    ```html
    <input type="checkbox" id="Allday" @((Model == null) ? "checked" : string.Empty) />
    <label for="Allday">
        Any
    </label>
    ```

5. foreach 選項並為選項加上 label for 功能以增加 ux
    - input 的 name 需與 controller 參數名稱一致， model binding 才會正確

        ```cs
        @foreach (WeekDaysEnum item in Enum.GetValues(typeof(WeekDaysEnum)))
        {
            if (item != WeekDaysEnum.Default)
            {
                string _tmp = string.Concat("days_", daysIndex);
                <div class="col-sm-3">
                    <input class="days" type="checkbox" id="@_tmp" value="@item" name="days" @((Model == null || Model.Contains(item)) ? "checked" : string.Empty) />
                    <label for="@_tmp">
                        @item.ToString()
                    </label>
                </div>
                daysIndex++;
            }
        }
        ```

6. 加上驗證訊息的 helper

    ```cs
    @Html.ValidationMessage("days", new { @class = "text-danger" })
    ```

## script

1. 定義 `全選` 按鈕行為
2. 勾選選項檢查是否勾選 `全選` 按鈕

    ```cs
    @section scripts{
    <script type="text/javascript">
        var arraylen = $('.days').length;

        function checkAllday() {
            if ($('.days:checked').length === arraylen) {
                $('#Allday').prop("checked", true);
            } else {
                $('#Allday').prop("checked", false);
            }
        }

        $(function() {
            checkAllday();
            $('#Allday')
                .change(function() {
                    //console.log(this.checked);
                    if (this.checked) {
                        $('.days')
                            .each(function() {
                                $(this).prop("checked", true);
                            });
                    } else {
                        $('.days')
                            .each(function() {
                                $(this).prop("checked", false);
                            });
                    }

                });
            $('.days')
                .change(function() {
                    checkAllday();
                });
        });
    </script>
    }
    ```

## 參考資訊
