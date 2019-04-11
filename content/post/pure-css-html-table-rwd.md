---
title: "利用 Pure CSS 讓 HTML Table 也能有 RWD 效果"
date: 2018-01-09T01:35:00+08:00
lastmod: 2018-01-09T01:35:06+08:00
draft: false
tags: ["Frontend"]
slug: "pure-css-html-table-rwd"
aliases:
    - /2018/01/pure-css-html-table-rwd.html
---
這是最近專案 user 提出的需求：所有對外服務的頁面都要有 RWD 顯示。這是現代化網站基本條件並不算特殊需求。雖然透過 grid system 的排版可以快速達成 RWD 的效果，但資料列表的顯示還是 table 最好用，加上 user 無法決定哪些欄位在 mobile 上是可以被隱藏的，所以退而求其次希望可以保留 table 的使用

其間討論過幾種方式：

1.  table 等比例縮少

    > 缺點：實在太小

2.  table 加上橫向捲軸

    > 缺點：使用者體驗稍差

3.  最後經部門 designer 提點，透過 css 來客製 mobile RWD 顯示效果
  

## 修改前後差異

1.  修改前
    *   原始畫面

        ![1normalsize](https://user-images.githubusercontent.com/3851540/34683217-56f44cc6-f4dc-11e7-80e0-513288328e8b.png)

    *   mobile 版本

        ![2mobile](https://user-images.githubusercontent.com/3851540/34683218-571ea962-f4dc-11e7-80cf-d134db1eff74.png)

2.  修改後
    *   原始畫面(完全相同)

        ![1normalsize](https://user-images.githubusercontent.com/3851540/34683217-56f44cc6-f4dc-11e7-80e0-513288328e8b.png)

    *   mobile 版本

        ![3aftermobile](https://user-images.githubusercontent.com/3851540/34683219-574ab96c-f4dc-11e7-92ab-8ec1f0d692cd.png)

## 開始修改

1.  CSS
    *   SCSS

        ```scss
        @media screen and (max-width: 600px) {
            .rwdtable {
                border: 0;
                            
                caption {
                    font-size: 1.3em;
                }
                            
                thead {
                    border: none;
                    clip: rect(0 0 0 0);
                    height: 1px;
                    margin: -1px;
                    overflow: hidden;
                    padding: 0;
                    position: absolute;
                    width: 1px;
                }
                            
                tr {
                    background-color: lightgrey;
                    border-bottom: 3px solid #ddd;
                    display: block;
                    margin-bottom: .625em;
                }
                            
                td {
                    color: #D20B2A;
                    border-bottom: 1px solid #ddd;
                    display: block;
                    font-size: .8em;
                    text-align: right;
                }
                            
                td:before {
                    color: black;
                    content: attr(data-label);
                    float: left;
                    font-weight: bold;
                    text-transform: uppercase;
                }
                            
                td:last-child {
                    border-bottom: 0;
                }
            }
        }
        ```

    *   編譯後的 CSS

        ```css
        @media screen and (max-width: 600px) {
            .rwdtable {
                border: 0;
            }
                            
            .rwdtable caption {
                    font-size: 1.3em;
                }
            
            .rwdtable thead {
                    border: none;
                    clip: rect(0 0 0 0);
                    height: 1px;
                    margin: -1px;
                    overflow: hidden;
                    padding: 0;
                    position: absolute;
                    width: 1px;
                }
                            
            .rwdtable tr {
                    background-color: lightgrey;
                    border-bottom: 3px solid #ddd;
                    display: block;
                    margin-bottom: .625em;
                }
                            
            .rwdtable td {
                    color: #D20B2A;
                    border-bottom: 1px solid #ddd;
                    display: block;
                    font-size: .8em;
                    text-align: right;
                }
                                
            .rwdtable td:before {
                        color: black;
                        content: attr(data-label);
                        float: left;
                        font-weight: bold;
                        text-transform: uppercase;
                    }
                                
            .rwdtable td:last-child {
                        border-bottom: 0;
                    }
        }
        ```

2.  Html 調整


    *   table 加入 `rwdtable` 的 class

        ```html
        <table class="rwdtable">
        </table>
        ```

    *   在 table tbody 的每個 td 上加入 `data-label="{顯示欄位名稱}"`

        ```html
        <td data-label="No.">@index</td>
        ```

        ![4columnname](https://user-images.githubusercontent.com/3851540/34683220-57795e70-f4dc-11e7-9d12-9b82df8eb2d4.png)

    *   完整 html

        ```html
        <table class="table rwdtable">
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
                        <td data-label="No.">@index</td>
                        <td data-label="CompanyName">
                            @Html.DisplayFor(modelItem => item.CompanyName)
                        </td>
                        <td data-label="ContactName">
                            @Html.DisplayFor(modelItem => item.ContactName)
                        </td>
                        <td data-label="ContactTitle">
                            @Html.DisplayFor(modelItem => item.ContactTitle)
                        </td>
                        <td data-label="Address">
                            @Html.DisplayFor(modelItem => item.Address)
                        </td>
                        <td data-label="City">
                            @Html.DisplayFor(modelItem => item.City)
                        </td>
                        <td data-label="PostalCode">
                            @Html.DisplayFor(modelItem => item.PostalCode)
                        </td>
                        <td data-label="Country">
                            @Html.DisplayFor(modelItem => item.Country)
                        </td>
                        <td data-label="Phone">
                            @Html.DisplayFor(modelItem => item.Phone)
                        </td>
                        <td data-label="Fax">
                            @Html.DisplayFor(modelItem => item.Fax)
                        </td>
                        <td data-label="Action">
                            @Html.ActionLink("Edit", "Edit", new { id = item.CustomerID }) |
                            @Html.ActionLink("Details", "Details", new { id = item.CustomerID }) |
                            @Html.ActionLink("Delete", "Delete", new { id = item.CustomerID })
                        </td>
                    </tr>
                    index++;
                }
                        
                </tbody>
        </table>
        ```

## 心得

CSS 本身並不複雜(但我就是沒想到XD)，況且已經寫好在那直接用完全沒有難度，唯一的工就是為 table 的欄位指定顯示名稱，雖然畫面、欄位一多還是滿繁瑣的

不過 user 的反應很好，也比重新使用 div 切版來得快，以總體效益來看這樣的調整還滿成功的，再次感謝部門 designer 協助

# 參考資訊

1.  [Simple Responsive Table in CSS](https://codepen.io/AllThingsSmitty/pen/MyqmdM)
2.  [Bootstrap教學－實現Table表格也支援RWD自適應效果](https://www.minwt.com/webdesign-dev/html/14066.html)
