---
title: "如何擴充 enum ？"
date: 2018-11-25T23:45:00+08:00
lastmod: 2021-11-03T23:44:30+08:00
draft: false
tags: ["csharp"]
slug: "extend-enum"
---
## 如何擴充 enum ？

同事問到可不可以擴充 enum ？！我的第一個反應：為什麼不行，就接著上個設定往下加不就好了？！ 不過立馬回過神來，如果這麼容易搞定，同事應該不會跑來問吧？！

果真如此：同事在維護一個舊系統，系統上用到一個跨部門的底層 model dll，因為沒有辦法直接修改這個 dll 但又想增加 enum 選項。針對這個問題我想了一陣子，後來查資料後發現我把問題複雜化了XD，就來看看到底有多簡單吧

## 前提說明

1. SharedModel 簡易示意

    ```cs
    namespace SharedModel
    {
        public enum StatusEnum
        {
            NoSet,
            Create,
            Update,
            Delete
        }
    }
    ```

2. 管理系統參考該 dll 並使用其中的 StatusEnum 做為 Order 的狀態

    ```cs
    namespace ManagementTool.Models
    {
        public class Order
        {
            public int Id { get; set; }
            public StatusEnum Status { get; set; }
        }
    }
    ```

3. 使用者針對管理系統內部想要加上其他 status

    > 增加審核的狀態碼 (該狀態碼僅該管理系統會用到)

    - CreateToApprove
    - UpdateToApprove
    - DeleteToApprove

## 如何擴充 enum

1. enum 是 value type 不能用做繼承
2. enum 內部為數值：只要新的 enum 包含舊 enum 內容即可

    ```cs
    namespace ManagementTool.Models
    {
        public enum StatusExtendEnum
        {
            NoSet,
            Create,
            Update,
            Delete,
            CreateToApprove,
            UpdateToApprove,
            DeleteToApprove
        }
    }
    ```

## 如何使用

1. 兩個 enum 皆存在的值

    > 直接轉型即可對應

    ```cs
    StatusExtendEnum status1 = (StatusExtendEnum)StatusEnum.Create;

    StatusEnum status2 = (StatusEnum)StatusExtendEnum.Create;
    ```

    ![1samecontent](https://user-images.githubusercontent.com/3851540/49029105-d138e280-f1de-11e8-8388-e9f96ca793a7.png)

2. 僅存在新 enum

    >  沒有對應到的 enum 內容會改用數值呈現

    ```cs
    StatusExtendEnum status1 = (StatusExtendEnum)StatusEnum.Create;

    StatusEnum status2 = (StatusEnum)StatusExtendEnum.CreateToApprove;
    ```

    ![2diffcontent](https://user-images.githubusercontent.com/3851540/49029106-d138e280-f1de-11e8-996a-c6e3e8893152.png)

3. 在參數傳遞或是物件指派上也沒有問題
    - 參數傳遞

        ```cs
        void Main()
        {
            Order order = new Order { Id = 1, Status = (StatusEnum)StatusExtendEnum.DeleteToApprove };
            DumpStatus(order.Status);
            
        }

        void DumpStatus(StatusEnum status)
        {
            status.Dump();
        }
        ```

        ![3passparam](https://user-images.githubusercontent.com/3851540/49029107-d1d17900-f1de-11e8-87e5-d0bc9e55f22f.png)
    - 物件指派

        ```cs
        Order order = new Order { Id = 1, Status = (StatusEnum)StatusExtendEnum.UpdateToApprove };
    
        order.Dump();
        ```

        ![4objectinit](https://user-images.githubusercontent.com/3851540/49029108-d1d17900-f1de-11e8-9796-c68e481f77b2.png)

## 心得

經過實驗後才發現原來沒有想像中複雜，一開始聽到同事的需求時我一度以為除了全面拔除舊 enum 的相依關係才能徹底解決問題，查了資料才知道學藝不精，幸虧同事的問題讓我多上了一課

重新回頭看程式碼，簡單的觀念、更沒有複雜語法，原本還在考慮是不是該紀錄，不過再簡單的用法之前沒想到留個學習紀錄提醒自己也滿好的

## 參考資訊

1. [How to extend existing enum (or what to do when I can't cast?)](https://social.msdn.microsoft.com/Forums/en-US/b8a3b217-21a8-4f1c-9fc6-0cf4243d6958/how-to-extend-existing-enum-or-what-to-do-when-i-cant-cast?forum=csharplanguage)
