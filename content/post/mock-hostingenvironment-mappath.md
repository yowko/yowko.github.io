---
title: "如何 Mock System.Web.Hosting.HostingEnvironment.MapPath 虛擬路徑"
date: 2017-02-24T01:42:34+08:00
lastmod: 2020-12-11T00:42:34+08:00
draft: false
tags: ["Unit Test"]
slug: "mock-hostingenvironment-mappath"
aliases:
    - /2017/02/mock-hostingenvironment-mappath.html
---
# 如何 Mock System.Web.Hosting.HostingEnvironment.MapPath 虛擬路徑
同事想要為負責的專案加上測試保護，讓之後的修改可以降低風險，但很快地就遇上問題：專案在開發階段時並沒有加上任何測試，專案的部份語法是相對較難測試的，同事遇到的問題就是其一：特定方法使用了 `System.Web.Hosting.HostingEnvironment.MapPath` 去讀取檔案內容。

## 使用 fakes
1. 加入 Fakes assembly
    - 開啟測試專案的 References 資料夾
    - `System.Web.Http` 上按右鍵 --> Add Fakes Assembly
        
        ![1addfake](https://cloud.githubusercontent.com/assets/3851540/23245449/8881b316-f9c7-11e6-9bd9-e3f178a7a9ca.png)  
2. 專案加入 dll 與 config
    - References 資料夾多了 `System.Web.Http.{版本}.Fakes` dll
        
        ![7success](https://cloud.githubusercontent.com/assets/3851540/23245454/88c7f3f8-f9c7-11e6-8735-54dab8e9ee4d.png)
    - Fakes 資料夾已產生 `System.Web.Http.fakes` 的設定檔
    - References 資料夾則沒有出現對應版本的 `System.Web.Http.{版本}.Fakes` dll 請參考這篇 [Fake Assembly 無法自動產生 *.Fakes dll 及出現 build fail](/2017/02/add-fake-assembly-issue.html)
    

## 程式碼 mock
1. create ShimsContext
    - namespace `Microsoft.QualityTools.Testing.Fakes`
        
        ```cs
        using Microsoft.QualityTools.Testing.Fakes;
        ```
    - ShimsContext Create
        
        ```cs
        using (ShimsContext.Create())
        {
        }
        ```
2. 指定 `HostingEnvironment.MapPath`
    
    ```cs
    ShimHostingEnvironment.MapPathString = (s => @"D:\yowkoTest\App_Data");
    ```
3. 完整程式碼
    - api 
        
        ```cs
        public class HomeController : Controller
        {
            [HttpGet]
            public string MapPath()
            {
                return HostingEnvironment.MapPath("~/App_Data");
            }
        }
        ```
    - test 
        
        ```cs
        using (ShimsContext.Create())
        {
            string path = @"D:\yowkoTest\App_Data";
            ShimHostingEnvironment.MapPathString = (s => path);
            HomeController controller = new HomeController();
            var actual= controller.MapPath();
            Assert.AreEqual(path,actual);
        }
        ```


# 參考資料
1. [Fake Assembly 無法自動產生 *.Fakes dll 及出現 build fail](http://blog.yowko.com/2017/02/add-fake-assembly-issue.html)