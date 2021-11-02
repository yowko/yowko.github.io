---
title: "找不到 dumpbin.exe (Visual Studio 2015)"
date: 2017-01-05T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Visual Studio","Debug","Tools"]
slug: "install-dumpbin"
aliases:
    - /2017/01/install-dumpbin.html
---
## 找不到 dumpbin.exe (Visual Studio 2015)

關於 `dumpbin` 可以參考 MSDN：以下內容節錄至 MSDN

```txt
Microsoft COFF 二進位檔案傾印工具 (DUMPBIN.EXE) 會顯示關於通用物件檔案格式 (Common Object File Format，COFF) 之二進位檔案 (Binary File) 的資訊。  
您可使用 DUMPBIN 來檢查 COFF 目的檔 (Object File)、COFF 物件的標準程式庫、可執行檔，和動態連結程式庫 (DLL)。
```

dumpbin 是個用來檢視 binary 檔案的工具，看了阿尼關於.pdb的文章([.pdb 的用處](http://anferneehardaway.pixnet.net/blog/post/3994168-.pdb-%E7%9A%84%E7%94%A8%E8%99%95))，立馬來嘗試看看, 結果指令一打就找不到 dumpbin @@"，來看看該如何解決吧

## 找不到 dumpbin

- 錯誤訊息

    ```log
    'dumpbin' is not recognized as an internal or external command,
    operable program or batch file.
    ```

- 錯誤截圖
    ![NOTFOUND](https://cloud.githubusercontent.com/assets/3851540/21677306/8e9a0db2-d374-11e6-9520-74ee7e599c9e.png)

- everything 也找不到，但 Visual Studio 2013 跟 Visual Studio 2012 卻有
    ![EVERYTHING12](https://cloud.githubusercontent.com/assets/3851540/21677304/8e993e0a-d374-11e6-8003-560ada2c280c.png)

## Visual Studio 2015 安裝 dumpbin

> 搜尋了一輪才發現原來 dumpbin 是 `Visual C++` 的附屬相關工具

- 進到 Visual Studio 2015 的安裝介面
  - `Programming Languages`--> `Visual C++` --> `Common Tools for Visual C++ 2015`

    ![trig](https://cloud.githubusercontent.com/assets/3851540/21677303/8e962152-d374-11e6-8423-646ac02854a8.png)

    ![installing](https://cloud.githubusercontent.com/assets/3851540/21677308/8eb74e36-d374-11e6-9771-79dd4f7ddeed.png)

    ![success](https://cloud.githubusercontent.com/assets/3851540/21677305/8e998054-d374-11e6-871d-dac0bf16388a.png)

## 正常使用

![installed](https://cloud.githubusercontent.com/assets/3851540/21677302/8e9318fe-d374-11e6-9cf8-1fee69a87584.png)

everthing 也出現了
>![everything](https://cloud.githubusercontent.com/assets/3851540/21677307/8e9b213e-d374-11e6-841a-ec606a5a5689.png)

## 參考資料

1. [DUMPBIN 參考](https://msdn.microsoft.com/zh-tw/library/c1h23y6c.aspx)
2. [.pdb 的用處](http://anferneehardaway.pixnet.net/blog/post/3994168-.pdb-%E7%9A%84%E7%94%A8%E8%99%95)
