---
title: "decimal 在 C# 中的隱含轉換建構式"
date: 2016-12-24T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["C#"]
slug: "decimal-constructor-with-char"
aliases:
    - /2016/12/Decimal-constructor-with-char.html
---
# decimal 在 C# 中的隱含轉換建構式
最近有個工作項目是將原本 jsp 的金流相關功能，搬遷到 C# 上 重要性不言可喻，加下小弟寫 java 的時間並不長，還是寫 android，jsp 只有聽過的程度 ^_^||

本來想趁機好好觀摩 java 的架構跟設計方式，剛動手沒多久就狂卡關

為了避免耽誤時程 於是立馬改變策略 --> <span style="color:#008000;">單純進行一個人工翻譯的動作</span>

在不瞭解流程面跟 java 語法的情況下，可以想見問題一定是少不了，

# 問題描述
客戶填的金額是 <span style="color:#FF8C00;">500000</span>，結果程式竟然只紀錄了 <span style="color:#FF0000;">**53**</span>

# 問題解析
1. 程式碼
    
    ```cs
    string a="500000";
    var b =new Decimal(a[0]);
    Console.WriteLine(b);//53
    ```

2. 程式碼說明
    - a[0] 會將 `a` 這個 string 轉為 char[]，接著取第 0 個位子也就是第一個數字 `5`
    - `5` 可以理解，但 `53` 哪來的?
        
        >  ***果然就是自己對 decimal 瞭解太少造成的***

3. decimal 的建構子有下列幾種 (可參考  [MSDN](https://msdn.microsoft.com/zh-tw/library/system.decimal.decimal.aspx))
    - a. Decimal(Double)  
    - b. Decimal(Int32)  
    - C. Decimal(Int32, Int32, Int32, Boolean, Byte)  
    - d. Decimal(Int32[])  
    - e. Decimal(Int64)  
    - f. Decimal(Single)  
    - g. Decimal(UInt32)  
    - h. Decimal(UInt64)

    **發現了嗎？沒有一個是接受 `char` 的呀？！**

4. 魔鬼就在細節裡，`Decimal 隱含轉換`
    - 5 的 ascii code 就是 53

        ![ascii](https://trello-attachments.s3.amazonaws.com/5801867431e3c7c7a302c2a4/572x291/4f41ccd4775b98b8e7f96b8aa0d751e9/_output_ascii.png)
 
    - 詳細資料請看 [Decimal Implicit 轉換 運算子](https://msdn.microsoft.com/zh-tw/library/system.decimal.op_implicit.aspx)
    - 整理如下
    
        傳入參數    |   輸出值
        :---|:---
        [Byte](https://msdn.microsoft.com/zh-tw/library/ms131058.aspx) |8 位元不帶正負號整數 
        [Char](https://msdn.microsoft.com/zh-tw/library/9wc2025x.aspx) |Unicode 字元 
        [Int16](https://msdn.microsoft.com/zh-tw/library/ms131059.aspx) |16 位元帶正負號的整數 
        [Int32](https://msdn.microsoft.com/zh-tw/library/ms131060.aspx) |32 位元帶正負號的整數 
        [Int64](https://msdn.microsoft.com/zh-tw/library/ms131061.aspx) |64 位元帶正負號的整數 
        [SByte](https://msdn.microsoft.com/zh-tw/library/ms131062.aspx) |8 位元帶正負號的整數 
        [UInt16](https://msdn.microsoft.com/zh-tw/library/wa5tsxhe.aspx) |16 位元不帶正負號的整數 
        [UInt32](https://msdn.microsoft.com/zh-tw/library/5b2t64td.aspx) |32 位元不帶正負號的整數 
        [UInt64](https://msdn.microsoft.com/zh-tw/library/txcbabh1.aspx) |64 位元不帶正負號整數 


- 原始程式碼在這
    
    ![sourcecode](https://az787680.vo.msecnd.net/user/yowko/e9e0247b-6b97-4d90-a034-27d4cb4a7005/1476376529_81666.png)

# 參考資料
1. [Decimal 建構函式](https://msdn.microsoft.com/zh-tw/library/system.decimal.decimal.aspx)
2. [Decimal Implicit 轉換 運算子](https://msdn.microsoft.com/zh-tw/library/system.decimal.op_implicit.aspx)