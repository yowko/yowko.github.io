---
title: "Visual Studio 無法執行 xUnit 單元測試，Rider 卻可以？！"
date: 2018-08-30T02:40:00+08:00
lastmod: 2018-08-30T02:40:52+08:00
draft: false
tags: ["Tools","xUnit","Visual Studio","Debug"]
slug: "visual-studio-cannot-run-xunit"
aliases:
    - /2018/08/visual-studio-cannot-run-xunit/
---
# Visual Studio 無法執行 xUnit 單元測試，Rider 卻可以？！
近期正在進行一個測試性專案的 POC，主要是想要導入某個 Open Source 的工具，過程中包含 C# 元件的整合，元件在實際使用上沒什麼重大問題，但因為該元件已經有二年沒有更新，所以擔心某些功能可能有落差，於是想執行看看其中的 unit tests 及 integration tests，了解一下元件的健康狀況，順便透過 tests 來學習元件的使用方式，測試結果完全出乎預期：連測試通過率低都碰不到邊，測試完全無法執行，原以為是專案年久失修，但仔細追查才發現狀況不單純：原來只有 Visual Studio (包含  Visual Studio 2015 、 Visual Studio 2017)無法執行該專案中的單元測試，Rider 可以正確運作，就來看看該如何解決問題吧


## 初步執行結果
1. Visual Studio 2015
    
    > 完全沒有錯誤訊息，也沒找到任何的 unit test

    ![2vs2015test](https://user-images.githubusercontent.com/3851540/46792751-a867a580-cd76-11e8-85fd-bf29ec2b3024.png)

    ![3vs2015notests](https://user-images.githubusercontent.com/3851540/46792753-a9003c00-cd76-11e8-9cda-4168b866745d.png)
2. Visual Studio 2017
    
    > 出現找不到 test adapter 錯誤，已列出 unit test

    ![4vs2017tests](https://user-images.githubusercontent.com/3851540/46792755-a9003c00-cd76-11e8-9748-bb214ff20d27.png)

    - 錯誤截圖
        
        ![1norunner](https://user-images.githubusercontent.com/3851540/46792750-a867a580-cd76-11e8-8ff5-4330cf4f5c39.png) 
    -  訊息內容
        
        ```
        [8/28/2018 4:56:11 PM Informational] Executing test method 'subscription_should_contain_info'
        [8/28/2018 4:56:11 PM Informational] Executing test method 'subscription_should_contain_info'
        [8/28/2018 4:56:11 PM Informational] ------ Run test started ------
        [8/28/2018 4:56:13 PM Informational] Test project Net.Tests does not reference any .NET NuGet adapter. Test discovery or execution might not work for this project.
        It's recommended to reference NuGet test adapters in each test project in the solution.
        [8/28/2018 4:56:13 PM Informational] ========== Run test finished: 0 run (0:00:01.6720082) ==========
        ``` 

3. Rider
    
    > 正常執行 unit tests

    ![5rider](https://user-images.githubusercontent.com/3851540/46792756-a9003c00-cd76-11e8-9128-e478c9b87957.png)

## 解決方案：安裝 `xunit.runner.visualstudio`

> 擇一即可

1. Package Manager Console
    
    ```
    Install-Package xunit.runner.visualstudio
    ``` 
2. Package Manager UI
    
    ![6install](https://user-images.githubusercontent.com/3851540/46792757-a998d280-cd76-11e8-94ad-2d0e9cc801cb.png)
3. 執行結果：正常執行測試
    
    ![7result](https://user-images.githubusercontent.com/3851540/46792758-a998d280-cd76-11e8-9f90-1eee8341034d.png)

## 心得
因為過去使用 xUnit 的經驗影響判斷，以為是 Visual Studio 2017 搭配 xUnit 上有什麼 breaking change 所造成的，但後來使用 Visual Studio 2015 進行測試後才發現原來是專案原本就不是透過 Visual Studio 進行測試造成的(以相依套件看應該都是透過 console 執行的)，差點就錯怪 Visual Studio 2017 了

不過這也讓我留意到 Visual Studio 與 Rider 行為不同：Visual Studio 需要 xunit.runner.visualstudio 才能正確執行 xUnit 而 Rider 不需要，日後除錯時也需要兩者差異考量進去，避免造成其他問題

# 參考資料
1. [Getting Started with xUnit.net (desktop)](https://xunit.github.io/docs/getting-started-desktop.html)