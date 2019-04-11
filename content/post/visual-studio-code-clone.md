---
title: "Visual Studio 的 Code Clone 重複程式碼分析工具"
date: 2017-04-12T00:10:00+08:00
lastmod: 2018-09-16T00:10:24+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-code-clone"
aliases:
    - /2017/04/visual-studio-code-clone.html
---
# Visual Studio 的 Code Clone 重複程式碼分析工具
最近的工作是針對一個大型既有專案做程式碼品質改善，當時接到這個任務時，對於這個專案我心裡大概就有幾個想法：

1. 重要性高 
2. 複雜度很高 
3. 可維護性低 
4. 可測試性低 
5. 功能及需求文件不全 
6. user 不一定清楚所有功能用途

照這個概念去發想，也能得出專案由來已久，經手人數眾多，coding style 不一致的現象，很多人都想要打掉重練，無奈成本太高，很難有這麼資源可以重寫一套，所以便退而求其次地進行重構及品質改善計劃

在實際對程式碼進行重構前，一定要先了解對手的實力XD，在之前文章 [Visual Studio 2017 如何使用 Code Map 功能](//blog.yowko.com/2017/04/visual-studio-2017-code-map.html) 有粗略提到 Code Map 可以幫助我們快速了解專案的關聯性以及實際執行流程。針對程式碼品質，今天就看看 Visual Studio 的另個工具：Code Clone 重複程式碼分析工具，透過 Code Clone 的分析，我們可以快速取得專案內複製貼上的程式碼位置與數量

需要留意的是 Code Clone 功能僅在 Visual Studio Premium 以上版本才有

## 分析專案重複程式碼

*  Visual Studio 主選單 Analyze --> Analyze Solution for Code Clones

    ![1analyze](https://cloud.githubusercontent.com/assets/3851540/24918471/20bf2666-1f13-11e7-9e07-4c8dbce7ec8e.png)

*   分析結果

    ![2result](https://cloud.githubusercontent.com/assets/3851540/24918474/20c4402e-1f13-11e7-98e4-22f4632dea85.png)

    *   Clone Group
        *   Match Type
            
            > 依程式碼相似程度分為以下四種 match 類型

            *   Exact Match
            *   Strong Match
            *   Medium Match
            *   Weak Match

        *   Files

            > 重複程式碼出現的檔案數量

    *   Clone Count
        
        >  程式碼重複次數

*   快顯效果

    > 滑鼠移至檔案名稱上，會出現重複程式碼的快顯視窗

    ![3hover](https://cloud.githubusercontent.com/assets/3851540/24918473/20c44b64-1f13-11e7-8a59-40cd2610addd.png)

*   比較檔案

    > 可以在分析結果視窗中選取多個檔案進行重複程式區塊比對

    - `shift+滑鼠左鍵(多選檔案)-->滑鼠右鍵--> Compare`

        ![4compare](https://cloud.githubusercontent.com/assets/3851540/24918472/20c3c1ee-1f13-11e7-8a66-551af7defb84.png)

        ![5compareresult](https://cloud.githubusercontent.com/assets/3851540/24918467/209bb5e6-1f13-11e7-8734-9c0b374953e8.png)

## 針對特定程式碼尋找是否重複

*   選取特定程式碼-->滑鼠右鍵--> Find Matching Clones in Solution
    
    ![6findclones](https://cloud.githubusercontent.com/assets/3851540/24918468/209d778c-1f13-11e7-92b2-384ce75719e1.png)

*   尋找結果
    
    ![7findresult](https://cloud.githubusercontent.com/assets/3851540/24918469/20a3b296-1f13-11e7-87c0-f8901dc05a84.png)

## Visual Studio 2017 如何使用 Code Clone 功能

加入方法與 [Visual Studio 2017 如何使用 Code Map 功能](//blog.yowko.com/2017/04/visual-studio-2017-code-map.html) 所提方式一致，vs 2017 為了提升安裝速度，將多數工具改列可選擇性的安裝項目，Code Clone 便是其一

1.  重新執行 Visual Studio 2017 安裝檔 --> Modify
    
    ![1update](https://cloud.githubusercontent.com/assets/3851540/24850307/1d226250-1e02-11e7-8330-0d1bfe33964b.png)

2.  Individual components --> Code tools --> 勾選 `Code Clone` --> Modify

    ![8vs2017](https://cloud.githubusercontent.com/assets/3851540/24918470/20a40ec6-1f13-11e7-9860-21edc9d1d7e4.png)

## 其他更多功能

*   如何排除特定檔案
*   關於 code clone 的搜尋標準


請參考 [Visual Studio 2012 New Features: Code Clone Analysis](https://blogs.msdn.microsoft.com/zainnab/2012/06/28/visual-studio-2012-new-features-code-clone-analysis/)

# 參考資訊

1.  [Visual Studio 2012 New Features: Code Clone Analysis](https://blogs.msdn.microsoft.com/zainnab/2012/06/28/visual-studio-2012-new-features-code-clone-analysis/)
2.  [Visual Studio 2017 如何使用 Code Map 功能](//blog.yowko.com/2017/04/visual-studio-2017-code-map.html)
