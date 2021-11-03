---
title: "ASP.NET Identity 2 手動檢查 Token 有效性"
date: 2017-12-16T23:36:00+08:00
lastmod: 2021-11-03T23:36:29+08:00
draft: false
tags: ["ASP.NET Identity"]
slug: "aspnet-identity-2-verify-token"
aliases:
    - /2017/12/aspnet-identity-2-verify-token.html
---
## ASP.NET Identity 2 手動檢查 Token 有效性

之前筆記 [改 ASP.NET Identity 2 的 Token 時效](/aspnet-identity-2-token-lifetime) 紀錄到可以調整 ASP.NET Identity 產生出 Token 的有效時間，而其中也提到可以參考 ForgotPassword 的寫法 (Forgotpasswrod --> 產生含有 token 的 reset password 連結 --> 使用 link 來 reset password)

整體功能是沒有問題的，只是使用體驗個人覺得稍嫌不足：link 包含 token，卻不是在開啟連結時就驗證 token 有效性，而是在填完相關資料才驗證，個人覺得既然 token 無效何必讓 user 多花那個時間填完資料才告訴 user token 無效，所以就來看可以如何調整吧

## 原始流程

1. 填完資料

    ![1inputdata](https://user-images.githubusercontent.com/3851540/34071870-8bccdd9a-e2b8-11e7-8a03-ab2df5c19eb5.png)

2. 送出後才驗證 Token 正確性

    ![2submitverify](https://user-images.githubusercontent.com/3851540/34071871-8c1aa6a6-e2b8-11e7-9bfc-ccf8afd6a159.png)

## 改善後流程

> 避免 Token 無效仍讓 user 浪費時間填寫資料

1. 填寫資料前先驗證 Token

    ![3verifytokenbeforeinput](https://user-images.githubusercontent.com/3851540/34071872-8c467ede-e2b8-11e7-9728-2934502318c8.png)

2. 初次開啟連結時 Token 仍有效

    ![4tokenok](https://user-images.githubusercontent.com/3851540/34071873-8c6f091c-e2b8-11e7-9a9a-7d02112edd7f.png)

3. 輸入完成送出後才失效

    ![5invalidtoken](https://user-images.githubusercontent.com/3851540/34071875-8ca7665e-e2b8-11e7-93be-5d83cdb7b097.png)

## 修改方式

1. 開啟 Controllers --> AccountController.cs

    ![6accountcontroller](https://user-images.githubusercontent.com/3851540/34071876-8cd7b7c8-e2b8-11e7-94b0-edd0f09c932f.png)

2. 修改 ResetPassword 方法
    * 修改前

        ```cs
        [AllowAnonymous]
        public ActionResult ResetPassword(string code)
        {
            return code == null ? View("Error") : View();
        }
        ```

        ![7beforechange](https://user-images.githubusercontent.com/3851540/34071877-8d01d878-e2b8-11e7-9d2e-0a1c3731361c.png)

    * 將 userid 也加入接放參數

        ```cs
        public ActionResult ResetPassword(string code, string userId)
        ```

    * 使用 UserManager.VerifyUserToken 驗證 Token 有效性

        ```cs
        UserManager.VerifyUserToken(userId, "ResetPassword", code)
        ```

    * Token 無效導向自訂頁面(非必要，但可以讓訊息更容易理解)

        ```cs
        return View("InvalidToken");
        ```

3. 完整程式碼

    ```cs
    [AllowAnonymous]
    public ActionResult ResetPassword(string code, string userId)
    {
        if (code == null || !UserManager.VerifyUserToken(userId, "ResetPassword", code))
        {
            return View("InvalidToken");
        }
        return View();
    }
    ```

    ![8afterchange](https://user-images.githubusercontent.com/3851540/34071878-8d3860b4-e2b8-11e7-90bf-1afc4ba6ff47.png)

## 心得

這個需求並不是 user 提出的，而是開發階段我自己測試時覺得使用者體驗不佳，決定自行加上的，站在 user 的角色：我相信沒有人想要填完一堆內容後再告訴我驗證失敗，如果 token 過期是發生在我填寫資料的過程也就算了，但經過修改後至少可以避免掉 token 早就過期還讓 user 填資料的情境

## 參考資訊

1. [改 ASP.NET Identity 2 的 Token 時效](/aspnet-identity-2-token-lifetime)
