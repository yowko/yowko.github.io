---
title: "如何開關 Visual Studio 中編輯視窗的提示底線(小蚯蚓 - squiggles)"
date: 2016-12-03T00:42:34+08:00
lastmod: 2018-09-04T00:42:34+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-squiggles"
aliases:
    - /2016/12/visual-studio-squiggles.html
---
# 如何開關 Visual Studio 中編輯視窗的提示底線(小蚯蚓 - squiggles)
前陣子因為換工作的關係，重新安裝了 Visual Stduio 2013，結果發現工作效率大大地降低，因為 **我的 Visual Studio 為什麼沒有提示小蚯蚓?!**，雖然只是個設定問題還是筆記一下加深印象

Visual Studio 中的 `小蚯蚓 - squiggles` 提示有兩種類型：

1. 語法檢查
2. 錯誤提示

一般情況下顏色會設定成不同，就來看看該如何設定吧

# 1. 開啟或關閉語法檢查
- 設定位置
    
    >Visual Studio `Tools` 主選單 --> `Options` --> `Text Editor` --> `C#` --> `Advanced` --> `Show live semantic errors`

    ![開啟](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/974x563/9dc874f7138c79f33e7d0e7cf7d4c406/1_%E7%BB%93%E6%9E%9C.png)

 - 實際開啟效果
    ![openresult](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/585x207/b52aaf416ed294f261d37c2155a67d4f/4_%E7%BB%93%E6%9E%9C.png)

 - 實際關閉效果
    ![closeresult](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/698x233/c428bc9b8a9b7b6b28798e5929405dad/3_%E7%BB%93%E6%9E%9C.png)

# 2. 開啟或關閉錯誤提示
- 設定位置
    
    >Visual Studio `Tools` 主選單 --> `Options` --> `Text Editor` --> `C#` --> `Advanced` --> `Underline errors in the editor`
    
    ![開啟](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/974x563/3e8a66dc0ff457655d31cef735e126b6/e3_%E7%BB%93%E6%9E%9C.png)
- 實際開啟效果
    ![openresult](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/509x197/fb16faa1fb5c09bf82269c8e24b7ede9/e2_%E7%BB%93%E6%9E%9C.png)
 - 實際關閉效果
    ![closeresult](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/509x197/bebc999b7d32c89a57a2b84e8801c14b/e2-2_%E7%BB%93%E6%9E%9C.png)

# 語法檢查與錯誤提示的差異
1. 顯示時機
    - 語法檢查是即時出現
    - 錯誤提示是在編譯後出現

2. 顏色不同(以我的設定而言)
    - 語法檢查是紅色
    - 錯誤提示是藍色

3. 如何修改顏色
    - 語法檢查
    
        >Visual Studio `Tools` 主選單 --> `Options` --> `Environment` --> `Fonts and Colors` --> `Show settings for: Printer` --> `Syntax Error`

        ![Syntax Error](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/974x563/208f63655b597dd21643b7a06481d959/c2_%E7%BB%93%E6%9E%9C.png)

    - 錯誤提示

        > Visual Studio `Tools` 主選單 --> `Options` --> `Environment` --> `Fonts and Colors` --> `Show settings for: Printer` --> `Compiler Error`

        ![Compiler Error](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/974x563/861e869e292f77a3b359be1d8891f06b/c1_%E7%BB%93%E6%9E%9C.png)


# Visual Studio 2015 *尚未找到* 相同設定
![2015](https://trello-attachments.s3.amazonaws.com/58161cb453e7afc217686635/1053x710/550f60998a7b145976752ef0cdbcfd5c/VS2015_%E7%BB%93%E6%9E%9C.png)

# 參考資料
1. [MSDN Social](https://social.msdn.microsoft.com/Forums/vstudio/en-US/d83134c7-773f-4b87-913e-01de8240568a/visual-studio-2015-to-stop-underlining-errors-and-stuff?forum=visualstudiogeneral)
2. [ADAM PRESCOTT](https://adamprescott.net/2015/02/24/where-my-squiggles-at/)
