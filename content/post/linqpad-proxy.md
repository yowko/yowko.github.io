---
title: "為 LINQPad 設定需驗證的 proxy"
date: 2016-12-16T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "linqpad-proxy"
aliases:
    - /2016/12/LINQPadbehideproxywithauthentication.html
---
## 為 LINQPad 設定需驗證的 proxy

一開始接觸 LINQPad 是為了快速驗證 LINQ 語法的正確性，隨著版本更新 LINQPad 的功能已強大到驗證 LinQ 語法功能只是一小部份了，現在很多 C# 的驗證我自己都改由 LINQPad 進行不再開啟 Visual Studio，甚至有些小型程式或是單次使用的功能也都轉由 LINQPad 開發(推薦一定要買 License!!)，**列為 .net 工程師的必備工具之一**，想必不為過！！

## 需要 proxy 而未設定時的錯誤

![error](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/384x381/93a3ee8d7ba40304a35b66c301f467c2/noproxy_%E7%BB%93%E6%9E%9C.png)

## 設定方式

1. Help -->  Check For Updates

    ![setting1](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/554x475/97636f639a7437c5cf2118f941027b46/Linq1_%E7%BB%93%E6%9E%9C.png)

2. Specify Web Proxy Server...

    ![setting2](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/382x377/18a91818b5b5668347c081025361ef54/Linq2_%E7%BB%93%E6%9E%9C.png)

3. Manually Specify Proxy
    - 3-1. Proxy Address

        >填 proxyserver

    - 3-2. Proxy Port

        >填 proxy port

    - 3-3. Domain(if required)

        >填 AD

    - 3-4. Username(if required)

        >填 username

    - 3-5. Password(if required)

        >填 username

        ![setting3](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/436x385/fc975ce436c9b7394de5dbbd2ccdf33c/linq3_%E7%BB%93%E6%9E%9C.png)

4. 出現 Error 417

    ![error417](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/438x385/8b2f0a2b09d51b27c5f637a179d159a8/417error_%E7%BB%93%E6%9E%9C.png)

    - **勾選 `Disable Expect-100-Continue`**

        ![tick](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/438x385/01a1072aa9289c19a670aaf2c2b8245a/tickexpect100_%E7%BB%93%E6%9E%9C.png)

5. Success

    ![success](https://trello-attachments.s3.amazonaws.com/581164b63ac3d19f70de989f/437x387/81cf6267ed23b20d723ca8f09373fe05/linq4_%E7%BB%93%E6%9E%9C.png)

## 參考資料

1. [LINQPad 官網](https://www.linqpad.net/)
