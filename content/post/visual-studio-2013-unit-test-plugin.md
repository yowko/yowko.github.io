---
title: "Visual Studio 2013 Unit Testing 的好用套件"
date: 2017-02-13T00:42:34+08:00
lastmod: 2018-09-12T00:42:34+08:00
draft: false
tags: ["Visual Studio","Unit Test"]
slug: "visual-studio-2013-unit-test-plugin"
aliases:
    - /2017/02/visual-studio-2013-plugins-for-unit-testing.html
---
# Visual Studio 2013 Unit Testing 的好用套件
公司有些專案仍然使用 Visual Studio 2013，為了有更好的程式品質，慢慢地各專案都需要加上測試，所以就遇到有些好用套件需要另外處理的問題，順手紀錄一下，讓有相同問題的朋友可以少花些時間

## Visual Studio Extensions - Unit Test Generator
> Visual Studio Extension : 讓 Visual Stusio 2013 右鍵選單可以有 Create Unit Test 功能

1. Visual Studio TOOLS --> Extensions and Updates...
    
    ![1tools](https://cloud.githubusercontent.com/assets/3851540/22687116/beb52e44-ed61-11e6-9df2-6268d7551c7e.png) 

2. Online --> Search `Unit Test Generateor`
    
    ![2unitgenerator](https://cloud.githubusercontent.com/assets/3851540/22687114/beb07c6e-ed61-11e6-9c93-892eaed54d20.png)

- 安裝完成後，即可在方法上按右鍵 --> Generate Unit Test
    
    ![3generate](https://cloud.githubusercontent.com/assets/3851540/22687115/beb0b5c6-ed61-11e6-96d4-0b0bf355e828.png) 

    ![4choose](https://cloud.githubusercontent.com/assets/3851540/22687117/beb7aade-ed61-11e6-8587-3cdcc12b945e.png)

## Mock Framework - NSubstitute
> project package : 用來讓測試程式可以隔離外部相依的物件模擬框架，主打容易使用

1. 實測 Visual Studio 2013 的 NuGet Package Manager UI 搜尋不到 NSubstitute
    
    ![5nonsub](https://cloud.githubusercontent.com/assets/3851540/22687118/bec07e20-ed61-11e6-8d1c-658ad3715c41.png) 

2. 請使用 Package Management Console，並注意 Default project 是否選擇正確
    - Install-Package NSubstitute
        
        ![6nsub](https://cloud.githubusercontent.com/assets/3851540/22687111/be757704-ed61-11e6-88c8-06a86bcea1a5.png)

## Fluent Assertions

> project package : 驗證語法讓語意非常容易理解

1. 實測 Visual Studio 2013 的 NuGet Package Manager UI 搜尋不到 Fluent Assertions
    
    ![7nofluent](https://cloud.githubusercontent.com/assets/3851540/22687112/be9aa72c-ed61-11e6-8b5f-0eed27bb19fb.png) 

2. 請使用 Package Management Console，並注意 Default project 是否選擇正確
    - Install-Package FluentAssertions
        
        ![8fluent](https://cloud.githubusercontent.com/assets/3851540/22687113/beafb996-ed61-11e6-9b76-42ebc2e09fc3.png)

# 參考資料
1. [Unit Test Generator](https://marketplace.visualstudio.com/items?itemName=VisualStudioALMRangers.UnitTestGenerator)
2. [NSubstitute](http://nsubstitute.github.io/)
2. [Fluent Assertions](http://www.fluentassertions.com/)