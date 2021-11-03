---
title: "無法參考 App_Code 資料夾下的 class"
date: 2018-11-05T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Debug","ASP.NET MVC"]
slug: "access-class-in-app-code"
---
## 無法參考 App_Code 資料夾下的 class

這幾天為了之前同事的問題進而聯想到的情境在做測試，大意就是希望在 web 啟動時紀錄時間讓開發人員可以得知 web 啟動至實際存取特定頁面的時間差

所以參考了 [ASP.NET Global Variables Example - Dot Net Perls](https://www.dotnetperls.com/global-variables-aspnet) 在 App_Code 資料夾中加入一個 static class 與 property 要來驗證，只是過程有點出乎意料，來看看發生了什麼事吧

## 前置準備

1. 建立 App_Code 資料夾

    > project 上按右鍵 --> Add --> Add ASP.NET Folder --> Add_Code

    ![1addappcode](https://user-images.githubusercontent.com/3851540/48008657-0c527380-e155-11e8-871d-5f22220e1137.png)

2. 加入 `InitialData.cs`

    ```cs
    public static class InitialData
    {
        public static DateTime IniTime { get; set; } = DateTime.Now;
    }
    ```

## 在 MVC 的 Razor View 上一切正常

1. 在 cshtml 頁面上引用 App_Code 的 namespace

    ```razor
    @using WebApplication3.App_Code
    ```

2. 透過 Razor 語法將資料顯示出來

    ```razor
    @InitialData.IniTime.ToString()
    ```

    ![2razorview](https://user-images.githubusercontent.com/3851540/48008658-0ceb0a00-e155-11e8-9779-54f308e37766.png)

3. 實際結果

    ![3result](https://user-images.githubusercontent.com/3851540/48008660-0ceb0a00-e155-11e8-9f73-89e1ff8de510.png)

## 問題

1. 根據 ASP.NET MVC 的 lifecycle 應該將紀錄的動作寫在 Global.asax 或是 Startup 中
2. 無論是何者皆無法正確參考 namespace 及 class

    ![4global](https://user-images.githubusercontent.com/3851540/48008652-0bb9dd00-e155-11e8-87c9-bbe1d99d4b02.png)

    ![5startup](https://user-images.githubusercontent.com/3851540/48008653-0bb9dd00-e155-11e8-8021-c41791b7f6a1.png)

## 解決方式

* 將需要被其他 class 參考使用的檔案屬性 - Build Action 設定為 Compile

    ![6property](https://user-images.githubusercontent.com/3851540/48008654-0bb9dd00-e155-11e8-8e36-e55462204074.png)

    ![7compile](https://user-images.githubusercontent.com/3851540/48008655-0c527380-e155-11e8-8492-732194d08153.png)

  * compile 已不會出錯誤訊息，但 visual studio 編輯視窗仍會出現紅色錯誤

* Razor View 使用則反而出現問題：type 出現在兩個 dll 中

    ![8twodll](https://user-images.githubusercontent.com/3851540/48008656-0c527380-e155-11e8-9f86-dbea6c375823.png)

  * 解決方式一：將程式碼移出 `App_Code`
  * 解決方式二：將使用 `App_Code` 的程式碼改由 Controller 存取

## 心得

仔細想想似乎自從開始寫 MVC 後就不常使用 `App_Code` 了，雖然偶爾會寫到 Razor function ，但整體來說頻率還是低很多了

`App_Code` 與 Razor View 相似都是即時動態編譯，所以也就出現今天遇到的 View 與其他 class 在存取指定 class 時行為不一致的現象，久久回味一下溫故知新也不賴XD

## 參考資訊

1. [Classes residing in App_Code is not accessible](https://stackoverflow.com/questions/1222281/classes-residing-in-app-code-is-not-accessible)
2. [ASP.NET 網站中的共用程式碼資料夾](https://msdn.microsoft.com/zh-tw/library/t990ks23%28v=vs.100%29.aspx)
