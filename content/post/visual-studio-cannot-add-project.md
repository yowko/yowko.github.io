---
title: "Visual Studio 2015 無法新增專案？！"
date: 2017-11-02T23:18:00+08:00
lastmod: 2021-10-04T23:18:46+08:00
draft: false
tags: ["套件","Visual Studio"]
slug: "visual-studio-cannot-add-project"
aliases:
    - /2017/11/visual-studio-cannot-add-project.html
---
## Visual Studio 2015 無法新增專案？！

專案如火如塗地進行中，要加入的 component 正以飛快速度被建立出來，想不到此時卻又遇到人品大爆發，我居然無法在方案 (solution) 中新增專案 (project)，雖然解決方式很簡單，但為了避免下次又在遇到趕專案時驚慌失措，當然要筆記一下囉

## 錯誤訊息

1. 訊息內容

    ```txt
    Could not load file or assembly 
    'Microsoft.VisualStudio.ApplicationInsights.Interfaces, Version=14.0.0.0,
    Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its
    dependencies. The system cannot find the file specified.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32333626-53b7fff0-c023-11e7-9337-2af79e09156b.png)

## 解決方式：重新安裝 `ApplicationInsights`

* 仔細看錯誤訊息，會發現錯誤是由 `Microsoft.VisualStudio.ApplicationInsights.Interfaces` 引起，雖然 local 端的專案不會用到 ApplicationInsights 但專案範本還是預設安裝，不僅如此還造成錯誤，真氣人XD

    ![1error2](https://user-images.githubusercontent.com/3851540/32333629-53e1dce4-c023-11e7-9960-519e4114acc0.png)

* 下載 [Application Insights Tools for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=50730)

* 直接安裝

    ![2install](https://user-images.githubusercontent.com/3851540/32333631-540acb7c-c023-11e7-918c-ae6270eab576.png)

    ![3install](https://user-images.githubusercontent.com/3851540/32333623-5334d58a-c023-11e7-8c64-fb849d635d01.png)

* 安裝完成後就可以正常新增專案

    ![4ok](https://user-images.githubusercontent.com/3851540/32333624-5362b716-c023-11e7-8bba-5f9207aa2e01.png)

## 心得

我想這應該是 Visual Studio 2015 專案範本的 bug ，但也不確定是怎麼造成的，前幾天就都一切正常。

回到問題本身，解決的方式很直覺也很容易，不過就提供了值得省思的設計概念：該不該讓加值功能預設附屬在主功能上、該讓使用者有選擇使用附加功能的空間嗎、附加功能異常該不該造成主功能無法使用，其中最沒有爭議就屬 `附加功能不該影響主功能運作`，但事實上大家在設計上常會忽略這件事，最近剛好就有例子：某個專案在完成交易後會去呼叫外部服務做相關通知，結果剛好外部服務異常沒有回應，造成畫面 hang 到 timeout，最後提示使用者交易逾時，而事實上主要流程的交易已正確完成，逾時的是附屬的外部服務，不過開發者在沒有考慮充份就將呼叫外部服務與主要交易流程綁在一起造成出現錯誤的使用者提示，我想今天這個 Visual Studio 無法新增專案的問題可能也有點未考慮周全的成份在，畢竟使用者又不一定需要附屬的功能

## 參考資訊

1. [Cannot create web projects in visual studio 2015](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/016cf45d-d6d2-4c48-ae62-6b96f75779bf/cannot-create-web-projects-in-visual-studio-2015?forum=ApplicationInsights)
2. [Application Insights Tools for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=50730)
