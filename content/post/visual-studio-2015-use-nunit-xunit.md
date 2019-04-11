---
title: "Visual Studio 2015 如何產生 NUnit 或 xUnit 的測試專案"
date: 2017-02-06T00:42:34+08:00
lastmod: 2018-09-11T00:42:34+08:00
draft: false
tags: ["Visual Studio 2015","Unit Test","NUnit","xUnit"]
slug: "visual-studio-2015-use-nunit-xunit"
aliases:
    - /2017/02/visual-studio-2015-use-nunit-xunit.html
---
# Visual Studio 2015 如何產生 NUnit 或 xUnit 的測試專案
Visual Studio 2015 預設使用 MSTest 做為預設的 test framework 來產生測試專案，這篇筆記紀錄想要使用 NUnit 或是 xUnit test framework 時該安裝的套件

## 預設僅有 MSTest
1. method 右鍵 --> Create Unit Tests
    
    ![1default](https://cloud.githubusercontent.com/assets/3851540/22578848/ae62884c-ea06-11e6-89e4-8ac374bb1665.png)
2. Test Framework 只有 MSTest
    
    ![2mstest](https://cloud.githubusercontent.com/assets/3851540/22578850/ae749c44-ea06-11e6-9cbe-7be3330a2b99.png)

## NUnit
1. Visual Studio 安裝 NUnit 相關擴充套件
    - Visual Studio 主選單 --> Tools --> Extensions and Updates...
        
        ![3install](https://cloud.githubusercontent.com/assets/3851540/22578849/ae72eb1a-ea06-11e6-982a-edcedee4e3c5.png)

2. Test Generator NUnit extension
    
    ![4tgne](https://cloud.githubusercontent.com/assets/3851540/22578851/ae7666f0-ea06-11e6-9be1-69cda7710628.png) 
    
    - 用來讓測試產生器可以使用 NUnit test framework
    - 會一併安裝 `NUnit` 與 `NUnit 2`
    - NUnit 即代表 NUnit 3 ，詳情可見 [GitHub nunit/nunit-vs-testgenerator](https://github.com/nunit/nunit-vs-testgenerator/wiki)
        
        ![5tgnedone](https://cloud.githubusercontent.com/assets/3851540/22578852/ae87a668-ea06-11e6-8107-25de7be8e608.png) 
3. NUnit Templates for Visual Studio
    
    ![6ntfvs](https://cloud.githubusercontent.com/assets/3851540/22578840/ae0dd504-ea06-11e6-9e4c-4739aa3529a0.png)
    
    - 用來讓測試產生器產生測試專案的範本
    - 未安裝錯誤
        
        ![7ntfvserror](https://cloud.githubusercontent.com/assets/3851540/22578841/ae33bbb6-ea06-11e6-9cd6-ab6775658355.png) 
4. NUnit 3 Test Adapter
    ![8n3ta](https://cloud.githubusercontent.com/assets/3851540/22578842/ae4e2488-ea06-11e6-8f02-5fae6fce1e74.png) 
    
    - 用來在 Visual Studio 中直接執行 NUnit 測試

## 測試專案加入 NUnit 
- 由測試產生器選擇 NUnit 產生測試專案後，預設已安裝 3.0.1 版本
    
    ![9includenuit](https://cloud.githubusercontent.com/assets/3851540/22578843/ae4f36fc-ea06-11e6-93d4-ad59b66d3634.png)

## xUnit
1. Visual Studio 安裝 xUnit
    
    ![10xunit](https://cloud.githubusercontent.com/assets/3851540/22578844/ae50f47e-ea06-11e6-883d-74162b0af119.png)
2. 只需安裝 `xUnit.net Test Extensions`, 相關功能已整合
    
    ![11xunitdoe](https://cloud.githubusercontent.com/assets/3851540/22578846/ae55ab86-ea06-11e6-811f-cf579b922108.png)

## 測試專案加入 xUnit 相關套件
> 由測試產生器選擇 xUnit 產生測試專案後，預設已安裝下列套件

1. xunit
2. xunit.abstractions
3. xunit.assert
4. xunit.core
5. xunit.extensibility.core
6. xunit.runner.visualstudio

    ![13xunitplugin](https://cloud.githubusercontent.com/assets/3851540/22579040/123236c8-ea08-11e6-9097-16be8b17e62d.png)

## 心得
專案 namespace , class , method 名稱產生似乎有 bug：已指定名稱，但實際產生時卻未生效
   
![12xunitbug](https://cloud.githubusercontent.com/assets/3851540/22579167/d83fbbe2-ea08-11e6-8f2f-2d831f6b15d7.png)

# 參考資料
1. [VS2015使用NUNIT 3.0進行測試 - 2016新年快樂](http://blog.kkbruce.net/2016/01/vs2015-use-nunit3-test-and-2016-happy-new-year.htm)