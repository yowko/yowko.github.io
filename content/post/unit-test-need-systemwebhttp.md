---
title: "ASP.NET Web API Unit Test 出現需要加入 `System.Web.Http` 參考錯誤"
date: 2018-04-08T10:48:00+08:00
lastmod: 2020-09-01T10:48:18+08:00
draft: false
tags: ["ASP.NET Web API","Unit Test"]
slug: "unit-test-need-systemwebhttp"
aliases:
    - /2018/04/unit-test-need-systemwebhttp.html
---
# ASP.NET Web API Unit Test 出現需要加入 `System.Web.Http` 參考錯誤
為 ASP.NET Web API 加上 unit test 時，在加入 action 實際動作後 Visual Studio 就提示需要加入 `System.Web.Http` 參考，仔細回想這個問題也不是第一次遇到了，只是過去都是直接手動加入 `System.Web.Http` 參考先解決問題為主，但因為直接加入 `System.Web.Http` 參考會直接相依個人電腦上的設定容易造成其他團隊成員無法取得 `System.Web.Http` 或是 CI server 無法順利執行測試

所以趁這個機會紀錄一下正較的做法，以供日後參考


## 錯誤訊息
1. 訊息內容
    
    ```
    Severity	Code	Description	Project	File	Line	Suppression State
    Error	CS0012	The type 'ApiController' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Web.Http, Version=5.2.3.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'.	WebAPITestTests	C:\Users\yowko\source\repos\WebAPITest\WebAPITestTests\Controllers\ValuesControllerTests.cs	23	Active
    ```
2. 錯誤截圖
    
    >![1error](https://user-images.githubusercontent.com/3851540/38285929-3769a3e0-37f5-11e8-9a10-8b723cd38fb5.png) 

## 解決方式

> 擇一即可

1. 直接手動加入 (直接相依特定位置，不推薦)
    
    >![2addref](https://user-images.githubusercontent.com/3851540/38285930-3791f0c0-37f5-11e8-944e-7e22aa209969.png)

    >![3refpath](https://user-images.githubusercontent.com/3851540/38285931-37bc11de-37f5-11e8-8690-18329e443dd9.png)

2. NuGet 安裝 `System.Web.Http`
    
    >![4nugethttp](https://user-images.githubusercontent.com/3851540/38285932-37eea4fa-37f5-11e8-9080-181f4267f0ef.png) 

3. NuGet 安裝 `Microsoft.AspNet.WebApi.Core` (根據 Microsoft docs 上文章 - [Unit Testing ASP.NET Web API 2](https://docs.microsoft.com/en-us/aspnet/web-api/overview/testing-and-debugging/unit-testing-with-aspnet-web-api?WT.mc_id=DOP-MVP-5002594))
    
    >![5webapicore](https://user-images.githubusercontent.com/3851540/38285933-38196d3e-37f5-11e8-9754-89daafc9dee4.png)

## 心得
在 [Unit Testing ASP.NET Web API 2](https://docs.microsoft.com/en-us/aspnet/web-api/overview/testing-and-debugging/unit-testing-with-aspnet-web-api?WT.mc_id=DOP-MVP-5002594) 文章下方對於加入 `Microsoft.AspNet.WebApi.Core` 有許多不同意見，至於到底該不該採用這個方式相信大家都有個自看法，如果不喜歡這個方式也可以考慮只加入缺的 `System.Web.Http`，但我個人是傾向使用  `Microsoft.AspNet.WebApi.Core` 原因是這次遇到 `System.Web.Http` 問題，不同情境可能會有其他參考的需要

# 參考資訊
1. [Unit Testing ASP.NET Web API 2](https://docs.microsoft.com/en-us/aspnet/web-api/overview/testing-and-debugging/unit-testing-with-aspnet-web-api?WT.mc_id=DOP-MVP-5002594)