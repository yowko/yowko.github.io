---
title: "調整 Visual Studio MSBuild 專案建置輸出詳細等級(Output 視窗輸出資訊詳細等級)"
date: 2017-01-02T00:42:34+08:00
lastmod: 2021-10-04T00:42:34+08:00
draft: false
tags: ["Debug","Visual Studio","MSBuild"]
slug: "visual-studio-build-output-verbosity"
aliases:
    - /2017/01/Visual-Studio-Output-Level.html
---
## 調整 Visual Studio MSBuild 專案建置輸出詳細等級(Output 視窗輸出資訊詳細等級)

有時 Visual Studio 編譯專案失敗，但只有顯示發生錯誤並沒有其他資訊造成偵錯困難或者想要瞭解 MSBuild 到底下了哪些指令，這個時候我們就可以透過調整 output 詳細等級來達到目的

## 預設(最小)

![minimal](https://cloud.githubusercontent.com/assets/3851540/21678152/85b0c23c-d378-11e6-8b1e-e50557a5e581.png)

## 調整方式

1. 開啟 Visual Studio 相關設定頁

    - Visual Studio 主選單 `Tools`--> `Options...`

        ![options](https://cloud.githubusercontent.com/assets/3851540/21678153/85b25bba-d378-11e6-8d73-b7d4f1016b86.png)

2. 打開 `專案及方案`--> `建置及執行`

    - `Projects and Solutions` --> `Build and Run`

        ![buildandrun](https://cloud.githubusercontent.com/assets/3851540/21678154/85b2dc2a-d378-11e6-8774-e45fcc17e28f.png)

3. 調整 `MSBuild 專案建置輸出詳細等級`

    - `MSBuild project build output verbosity`
  
        ![verbosity](https://cloud.githubusercontent.com/assets/3851540/21678155/85b3f29a-d378-11e6-95af-fd8035a90fb1.png)

## 詳細等級

|詳細等級|描述|
|:---|:---|
|無訊息(Quiet)|只會顯示組建的摘要。|
|最小(Minimal)|顯示組建的摘要和錯誤、被分類為高度和重要的警告訊息。|
|一般(Normal)|顯示組建的摘要;錯誤、被分類為高度重要的警告和訊息;建置的主要步驟。  您通常會使用這個詳細資料層級。  |
|詳細(Detailed)|顯示組建的摘要;錯誤、被分類為高度重要的警告和訊息;所有建置步驟;就一般重要性分類的訊息。|
|診斷(Diagnostic)|顯示為組建可用的所有資料。  您可以使用這個詳細資料層級協助偵錯與自訂建置指令碼和其他組建問題的問題。  |

## 測試(一般)

![normal](https://cloud.githubusercontent.com/assets/3851540/21678156/85b9d14c-d378-11e6-82fc-00638643172d.png)

## 參考資料

1. [如何：檢閱、儲存和設定建置記錄檔](https://msdn.microsoft.com/ZH-TW/library/jj651643.aspx)
