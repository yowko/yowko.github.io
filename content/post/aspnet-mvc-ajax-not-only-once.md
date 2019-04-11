---
title: "ASP.NET MVC 上的 Ajax 動作都被觸發多次？！"
date: 2018-01-07T20:01:00+08:00
lastmod: 2018-10-02T20:01:17+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-ajax-not-only-once"
aliases:
    - /2018/01/aspnet-mvc-ajax-not-only-once.html
---
# ASP.NET MVC 上的 Ajax 動作都被觸發多次？！
幫同事調整程式的過程中發現，View 的 Ajax 功能都會被觸發不止一次，本來以為是手殘按了兩次，特別留意後發現問題仍然存在，改懷疑起是不是軌跡球出現連點XD，最後證實只是人為造成的 bug

## 特別提醒

如果在 ASP.NET MVC 5 中有使用內建的 Ajax helper 但是一直沒有發揮 Ajax 效果，請檢查是否已安裝 Microsoft jQuery Unobtrusive Ajax 套件，原本 ASP.NET MVC 4 內建的套件在 MVC 5 時不再內建安裝，需要自行手動安裝，詳情請參考保哥的文章 [ASP.NET MVC 5 遺失的 Microsoft jQuery Unobtrusive Ajax 函式庫](https://blog.miniasp.com/post/2014/11/10/ASPNET-MVC-5-Microsoft-jQuery-Unobtrusive-Ajax-lost-and-found.aspx)

*   以分頁(使用 PagedList.Mvc) 為例

    1.  index.cshtml

        ```cs
        @model PagedList.IPagedList<TestUnobtrusive.Models.Customers>
        @{
            ViewBag.Title = "Index";
        }
        <h2>Index</h2>
        <p>
            @DateTime.Now
        </p>
        <div id="indexdata">
            @Html.Partial("_indexdata",Model)
        </div>
        @section scripts{
            <script src="~/Scripts/jquery.unobtrusive-ajax.min.js"></script>
        }
        ```

    2.  _indexdata.cshtml

        ```cs
        @using PagedList.Mvc
        @model PagedList.IPagedList<TestUnobtrusive.Models.Customers>
        @{
            Layout = null;
            var index = (Model.PageNumber - 1) * Model.PageSize + 1;
        }
        @Html.PagedListPager(Model, page => Url.Action("Index",
            new { PageNumber = page })
            , PagedListRenderOptions.EnableUnobtrusiveAjaxReplacing(
                new AjaxOptions() { HttpMethod = "POST", UpdateTargetId = "indexdata" }))
        <table class="table">
            <thead>
                <tr>
                    <th>
                        No.
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().CompanyName)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().ContactName)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().ContactTitle)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().Address)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().City)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().Region)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().PostalCode)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().Country)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().Phone)
                    </th>
                    <th>
                        @Html.DisplayNameFor(model => model.FirstOrDefault().Fax)
                    </th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                @foreach (var item in Model)
                {
                    <tr>
                        <td>@index</td>
                        <td>
                            @Html.DisplayFor(modelItem => item.CompanyName)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.ContactName)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.ContactTitle)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Address)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.City)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Region)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.PostalCode)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Country)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Phone)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Fax)
                        </td>
                        <td>
                            @Html.ActionLink("Edit", "Edit", new { id = item.CustomerID }) |
                            @Html.ActionLink("Details", "Details", new { id = item.CustomerID }) |
                            @Html.ActionLink("Delete", "Delete", new { id = item.CustomerID })
                        </td>
                    </tr>
                    index++;
                }
                        </tbody>
        </table>
        @Html.PagedListPager(Model, page => Url.Action("Index",
            new { PageNumber = page })
            , PagedListRenderOptions.EnableUnobtrusiveAjaxReplacing(
                new AjaxOptions() { HttpMethod = "POST", UpdateTargetId = "indexdata" }))
        ```

*   預期效果

    1.  透過 Ajax 執行換頁
    2.  畫面上的時間不會異動

*   未安裝套件時的實際狀況

    > 使用 GET 方法，重新 render 整個頁面

    ![1get1](https://user-images.githubusercontent.com/3851540/34649188-041af4e8-f3e5-11e7-860c-794ae03032ee.png)

## Ajax 觸發多次

1.  使用 URL 直接開啟頁面僅出現一次 request

    ![2getone](https://user-images.githubusercontent.com/3851540/34649190-046d51b6-f3e5-11e7-9f5c-46b5c9076983.png)

2.  透過 MVC pager 換頁卻同時出現兩次 request

    ![3post2](https://user-images.githubusercontent.com/3851540/34649191-04977cc0-f3e5-11e7-851d-7093349d9bc6.png)

3.  原因：重複載入 `jquery.unobtrusive-ajax`

    > 在 _layout 已載入一次，而在特定頁面的又重複載入，造成 Ajax helper 相關功能都會被執行多次

4.  解決方式：`移除重複載入檔案即可`


## 心得

這是意外發現的缺失，原本只是調整頁面，但因為頁面調整幅度過大，所以才使用 F5 進行偵錯，也才發現重複載入 `jquery.unobtrusive-ajax` 的問題

單就功能面來看並沒有異常，只是無形中造成 CPU 甚至 DB 資源浪費，當然我相信同事不是刻意寫錯，只是對於 MVC 行為還沒有了解透徹造成的

# 參考資訊

1.  [ASP.NET MVC 5 遺失的 Microsoft jQuery Unobtrusive Ajax 函式庫](https://blog.miniasp.com/post/2014/11/10/ASPNET-MVC-5-Microsoft-jQuery-Unobtrusive-Ajax-lost-and-found.aspx)
