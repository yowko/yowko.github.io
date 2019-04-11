---
title: "如何讓 ASP.NET MVC View 中的錯誤可以在編譯時期被揭露"
date: 2017-03-17T02:43:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-build-view"
aliases:
    - /2017/03/asp-dot-net-mvc-build-view.html
---
# 如何讓 ASP.NET MVC View 中的錯誤可以在編譯時期被揭露
有些時候可能因為手誤或是 code merge 的異常而造成 View 被改壞，雖然 View 壞掉很容易修改，但給人的觀感還是很差，畢竟畫面一開起來就壞，感覺像是完全沒檢查就上線，加上如果專案已有嚴謹的上線流程，就會形成改 View 三秒，處理上線的流程超過 三十分鐘的冏狀，所以今天就來看看我們可以怎麼設定，讓 Visual Studio 在 compile 時就抓出基本的異常

## 情境說明
1. 使用 ASP.NET MVC 預設 project template 建立專案
2. 使用 .NET Framework 4.6.2
3. 使用預設專案範本的登入功能(AccountController 的 login action)
4. 模擬 view 改壞
    
    ![1error](https://cloud.githubusercontent.com/assets/3851540/23912652/58e4912e-091b-11e7-947c-106be2c47d20.png)

## 預設情境
- View 有錯誤但 build success
    
    ![2default](https://cloud.githubusercontent.com/assets/3851540/23912655/58e7e5fe-091b-11e7-8378-028d06d99980.png)

## 如何修改
- 編譯 View 的功能需手動修改 .csproj 檔
    1. 將專案卸載
        - 專案上按右鍵 --> Unload Project
            
            ![3unload](https://cloud.githubusercontent.com/assets/3851540/23912653/58e6be22-091b-11e7-9d7c-dca293d68371.png) 
    2. 編輯 csproj
        - 已卸載專案上按右鍵 --> Edit *.csproj
            
            ![4edit](https://cloud.githubusercontent.com/assets/3851540/23912656/58ecf49a-091b-11e7-9d2a-164b24ce19da.png) 
    3. 修改設定
        - 搜尋 `MvcBuildViews` 
            
            ![5mvcbuildview](https://cloud.githubusercontent.com/assets/3851540/23912654/58e70c92-091b-11e7-9309-c60a2510d1d6.png) 
        - 將值改為 `true`
            
            ![6true](https://cloud.githubusercontent.com/assets/3851540/23912659/590e3f6a-091b-11e7-90b7-d39438f59e25.png)
- 重新載入專案
    - 已卸載專案上按右鍵 --> Reload Project
        
        ![7reload](https://cloud.githubusercontent.com/assets/3851540/23912658/590d8e26-091b-11e7-8a6f-fa76ca187387.png)

## 效果
- View 有錯誤且已出現 Build error
    
    ![8buildfail](https://cloud.githubusercontent.com/assets/3851540/23912660/590f772c-091b-11e7-80ea-3bcacccd1383.png) 

## 提醒
啟用 MvcBuildViews 可以在 compile 時期檢查 View 的異常，但因為 Build View 會讓 build 的工作增加，如果 view 的數量眾多就會嚴重拖慢效能，建議可以設定 release mode 再啟用
