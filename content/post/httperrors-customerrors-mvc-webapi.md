---
title: "httpErrors 與 customErrors 在 ASP.NET MVC 與 ASP.NET WEB API 中的處理方式"
date: 2017-03-10T02:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC","ASP.NET Web API"]
slug: "httperrors-customerrors-mvc-webapi"
aliases:
    - /2017/03/httperrors-customerrors-mvc-webapi.html
---
## httpErrors 與 customErrors 在 ASP.NET MVC 與 ASP.NET WEB API 中的處理方式

這是同事遇到的 production issue，大意是設定了 customErrors 後出現 cpu 釘死在 100% 的重大缺失，聽到這樣的描述立馬想到可以藉機練習黑大最近幾篇文章提到的幾個 debug 工具，但為時已晚 事發在多日前，加上同事已定位實際造成問題的設定，~~只好期待下次機會，開開玩笑，production issue 還是不要有的好~~

問題發生原因是幾個錯綜複雜的寫法跟設定造成的，三言兩語難以解釋得清楚，簡單地說是網站本身就存在數個既有錯誤，最後又加上 customErrors 設定而造成一連串的 redirect 形成無限迴圈，customErrors 也就變成壓倒駱駝的最後一根稻草，成了替死鬼XD

過程中查了些資料也用 ASP.NET MVC 與 ASP.NET WEB API sample project 做了幾個實驗，順手紀錄一下，以供大家參考，如果是 webform 專案請參考 黑大的文章 [customErrors與httpErrors](http://blog.darkthread.net/post-2015-11-10-customerrors-and-httperrors.aspx)

## httpErrors

***<span style="color:red">本機進行測試，記得把 errorMode 設定成  `Custom`</span>***

- 為網站或應用程式設定自定錯誤訊息
- 對應至 IIS - Error Pages 設定

    ![1iiserrorpage](https://cloud.githubusercontent.com/assets/3851540/23617960/cbeb8cd4-02c9-11e7-8063-d3eafd84f08a.png)
- httpErrors 的屬性(attribute)

    attribute|mandatory|type|default|comment
    :---|:---|---|---|---
    allowAbsolutePathsWhenDelegated|optional|boolean|false|-允許是否使用絕對路徑 <br/> -<span style="color:red">只能設定在 ApplicationHost.config 中，網站的 Web.config 不支援</span>
    defaultPath|optional|string|-|- 指定自定錯誤頁面的預設路徑<br/>- 路徑類型由 "defaultResponseMode" 指定：<BR/>`File` - 檔案路徑<BR/>`ExecuteURL` - 相對 URL <br/> `Redirect` - 絕對 URL
    defaultResponseMode|optional|enum|0(File)|- 指定如何處理自訂錯誤 <BR/>- File: 0, 指向靜態檔案<br/> - ExecuteURL : 1, 指向動態內容，必須是 server 中的相對 URL <br/> - Redirect : 2，redirect 其他網址，必須是絕對 URL
    detailedMoreInformationLink|optional|string|[官網](https://go.microsoft.com/fwlink/?LinkID=62293) |用來提供詳細錯誤資訊的網頁連結
    errorMode|optional|enum|0(DetailedLocalOnly)|- 是否啟用 HTTP error<br/> - DetailedLocalOnly:0,如果是 local 就回應詳細錯誤訊息，否則就是自定錯誤<br/> - Custom:1,永遠回傳自訂錯誤<br/> - Detailed:2,永遠回傳詳細錯誤訊息
    existingResponse|optional|enum|0(Auto)|指定收到 http status code 代表錯誤時 (i.e. code>=400)該如何處理<br/>- Auto:0,SetStatus 被設定時就不干涉依 response 處理，否則就依自訂錯誤處理 <br/> - Replace:1,一律依自訂錯誤處理 <br/>- PassThrough:2,一律依 response 處理

  - 關於 existingResponse
    - 我自己做了幾個測試，結果如下
    - httpErrors 設定

            ```xml
            <httpErrors errorMode="Custom" existingResponse="PassThrough">
              <remove statusCode="404" subStatusCode="-1" />
              <error statusCode="404" path="/NotFound" responseMode="ExecuteURL"/>
            </httpErrors>
            ```
    - NotFoundController 的 Index action

            ```cs
            public ActionResult Index()
            {
                return Content("NotFound!!!!");
            }
            ```
    - HomeController 的 Index action

            ```cs
             public ActionResult Index()
            {
                //Response.TrySkipIisCustomErrors = true;//用來指定 SetStatus flag
                return HttpNotFound();
            }
            ```
    - 測試結果截圖
            1. 執行 NotFoundController 的 Index action

                ![2notfound](https://cloud.githubusercontent.com/assets/3851540/23617957/cbc200c6-02c9-11e7-83a0-44b4cf4614da.png)
            2. 直接吐 404

                ![3404](https://cloud.githubusercontent.com/assets/3851540/23617959/cbc80a48-02c9-11e7-9bdb-05e3d3020ea2.png)
    - 測試結果整理

            existingResponse設定\SetStatus|with SetStatus|without SetStatus
            :---|:---|:---
            Auto|結果2|結果1
            Replace|結果1|結果1
            PassThrough|結果2|結果2

  - httpErrors 下的子項目
        1. `<error>`
            - 可以有多組 `<error>`

            Attribute|mandatory|type|Description
            :---|:---|:---|:---
            path|Required |string|- 指定自定錯誤頁面的預設路徑<br/>- 路徑類型由 `responseMode` 指定：<BR/>`File` - 檔案路徑<BR/>`ExecuteURL` - 相對 URL <br/> `Redirect` - 絕對 URL
            prefixLanguageFilePath|optional|string|指定不包含語系的前綴路徑, c:\inetpub\custerr\en-us\404.htm, prefixLanguageFilePath="c:\inetpub\custerr"
            responseMode|optional|enum|指定如何處理自訂錯誤 <BR/>- File: 0, 指向靜態檔案<br/> - ExecuteURL : 1, 指向動態內容，必須是 server 中的相對 URL <br/> - Redirect : 2，redirect 其他網址，必須是絕對 URL
            statusCode|Required |integer|指定處理的 http status code，接受 400 ~ 999
            subStatusCode|optional|integer|指定處理的 http substatus code，接受 -1 ~ 999
        2. `<remove>`
            - 使用 `<remove>` 來移除繼承自上層 IIS 的特定 http error 設定

                >e.g. `<remove statusCode="404" subStatusCode="-1" />`
        3. `<clear>` 則可以一次移除繼承自上層 IIS 的所有 http error 的設定
- 心得
    1. 主要用來處理靜態檔案
    2. 程式指定的回傳內容
    3. ASP.NET WEB API 程式面的錯誤

## customErrors

- ***<span style="color:red">本機進行測試，記得 </span>***
    1. 把 mode 設定成 `on`
    2. 停用 ASP.NET MVC 預設的 ErrorHandler (filter.config)

- 為 ASP.NET 應用程式提供自訂錯誤訊息
- 對應至 ASP.NET - .NET Error Pages 設定

    ![4asperrorpages](https://cloud.githubusercontent.com/assets/3851540/23617958/cbc48ae4-02c9-11e7-990b-7096afca1a6f.png)
- customErrors 的屬性(attribute)

    attribute|mandatory|comment
    :---|:---|:---
    defaultRedirect|optional|- 錯誤發生時 redirect 的 url，未指定會出現 runtime error<br/>- url 可以是絕對及相對路徑
    mode|required|- 指定是否顯示自定錯誤<br/>- On : 啟用自訂錯誤<br/>- Off : 停用自訂錯誤，會顯示完整錯誤訊息<br/>- RemoteOnly : 預設值，只在本機看得到詳細錯誤
    redirectMode|optional|- 指定如何處理原 request 的 url<br/> - ResponseRedirect : URL 會不同<br/> - ResponseRewrite : URL 不會改變，但在 MVC 中使用， path 必須使用實際存在的檔案，詳細原因請見 [ASP.NET MVC 裡redirectMode="ResponseRewrite" 時候無法使用 Controller 來設置特定的錯誤頁面](http://www.cnblogs.com/adandelion/archive/2011/12/28/2304671.html)
- customErrors 下的子項目
  - error
    - optional
    - 可以有多組 `<error>`
    - 指定特定 http status code 來自訂錯誤網頁
- 心得
  - 用來處理 ASP.NET MVC 程式面的錯誤

## 兩者有重複地方 ?

> 兩個設定可以針對相同的 error code 做處理，那會由誰回應？

- 測試情境 1
    1. js call web api 回應 500 或是有 exception
    2. 由 httpErrors 處理
- 測試情境 2
    1. js call MVC controller action 回應 500 或是有 exception
    2. 由 customErros 處理
- 測試情境 3 (預設 - 使用內建 error handler)
    1. 直接 access MVC controller action 回應 500 或是有 exception
    2. 由 httpErrors 處理
- 測試情境 4 (關閉預設 error handler)
    1. 直接 access MVC controller action 回應 500 或是有 exception
    2. 由 customErros 處理
- 結論
    1. 靜態資源問題由 httpErrors 處理
    2. Web API 問題由 httpErrors 處理
    3. 預設情境下 MVC 問題由 httpErrors 處理
    4. 關閉 MVC 內建 error handler 後 MVC 問題由 customErros 處理

## 注意事項

1. IIS 設定會寫入網站的 web.config，需留意部署會被覆蓋，建議直接加入專案的 web.config
2. ASP.NET - .NET Error Pages 需安裝 IIS 的 ASP.NET module
3. 本機測試時，需注意 error 是否正確
4. MVC 專案有預設 error handler

## 參考資料

1. [customErrors與httpErrors](http://blog.darkthread.net/post-2015-11-10-customerrors-and-httperrors.aspx)
2. [HTTP Errors](https://www.iis.net/configreference/system.webserver/httperrors)
3. [error Element for httpErrors [IIS Settings Schema]](https://msdn.microsoft.com/en-us/library/ms689487%28v=vs.110%29.aspx)
4. [How can I use existingResponse=「Auto」 successfully?](http://stackoverflow.com/questions/18276397/how-can-i-use-existingresponse-auto-successfully)
5. [customErrors 項目 (ASP.NET 設定結構描述)](https://msdn.microsoft.com/zh-tw/library/h0hfz6fc%28v=vs.110%29.aspx)
6. [ASP.NET MVC 裡redirectMode="ResponseRewrite" 時候無法使用 Controller 來設置特定的錯誤頁面](http://www.cnblogs.com/adandelion/archive/2011/12/28/2304671.html)
