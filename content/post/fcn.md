---
title: "關於 ASP.NET File Change Notification (FCN) 的一些事"
date: 2018-11-11T00:42:34+08:00
lastmod: 2018-11-11T00:42:34+08:00
draft: false
tags: ["IIS","ASP.NET MVC","ASP.NET WEB API","web.config"]
slug: "fcn"
---
# 關於 ASP.NET File Change Notification (FCN) 的一些事
這幾天在測試如何達成修改 web.config 而不引起 IIS 重啟時看到幾種做法，其中一個是調整 web.config 的 `fcnMode` 設定值，過去不認識這個設定引起我的好奇而查了些資料，剛好趁這個機會多認識個設定，順手做個筆記幫助記憶

## 什麼是 FCN
ASP.NET 應用程式會透過一個 Win32 的 function 稱為 `ReadDirectoryChangesW` 來監控資料夾及檔案的變更，根據變更的內容觸發 FCN (File Change Notification) 來卸載 AppDomain ，而達到重啟的目地

從 ASP.NET WebForm 時期開始，只要修改 ASP.NET 站台下的特定檔案或是異動特定資料夾就會引發 IIS 重啟，這些特殊檔案及資料夾包括(但不限於)下列內容

- 檔案
    1. web.config (也包含父層的 config)
    2. Global.asax
- 資料夾
    1. bin 資料夾
    2. App_Code 資料夾
    3. App_WebReferences 資料夾
    4. App_GlobalResources 資料夾
    5. 所在實體資料夾及虛擬資料夾


## 設定 ASP.NET 站台的 FCN
fcnMode 是 .NET Framework 4.5 開始加入至 web.config 的屬性，可以透過官方預先定義的 enum 來設定單一 website 中的 file change notifications mode 

設定|值|說明
---|---|---
Default|	1	|預設值：每個子資料夾都會建立一個物件來監控
Disabled|	2	|停用檔案變更通知
NotSet|	0	|檔案變更通知未設定，每個子資料夾都會建立一個物件來監控，也就是使用預設行為
Single|	3	|只會建立一個物件來監控所有資料夾

## 心得
之前只知道 ASP.NET 應用程式下的幾個資料夾及檔案不能想改就改，會引起 IIS restart、把線上 user 踢掉，但從來沒有想過是怎麼做或是背後的流程，只把它當做是慣例及規則來遵守，直到同事的問題讓我有機會多了解其中的做法，雖然對於完整流程跟細節還是沒有完整掌握：fcnMode 中 `Default` 及 `NotSet` 有什麼差異(文件上寫得行為完全相同)，`Single` 的目地及好處....等等，但已經算是更接近了核心些，相信日後如果看到相關文件時就不會完全沒有頭緒了


# 參考資訊
1. [All about ASP.Net File Change Notification (FCN)](https://shazwazza.com/post/all-about-aspnet-file-change-notification-fcn/)
2. [ASP.NET File Change Notifications, exactly which files and directories are monitored?](https://blogs.msdn.microsoft.com/tmarq/2007/11/01/asp-net-file-change-notifications-exactly-which-files-and-directories-are-monitored/)
3. [fcnMode](http://imageresizing.net/docs/fcnmode)