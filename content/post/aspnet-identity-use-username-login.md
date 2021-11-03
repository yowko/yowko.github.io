---
title: "在 ASP.NET MVC 5 預設專案範本中透過 ASP.NET Identity 由 Email 改用 Username 登入"
date: 2016-12-10T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC","ASP.NET Identity"]
slug: "aspnet-identity-use-username-login"
aliases:
    - /2016/12/aspnet-identity-use-username-replace-email.html
---
## 在 ASP.NET MVC 5 預設專案範本中透過 ASP.NET Identity 由 Email 改用 Username 登入

ASP.NET MVC 5 使用新的會員系統 - `ASP.NET Identity`，預設的專案範本，幫我們建立了基本的會員系統，就是以`ASP.NET Identity`，其中`註冊`及`登入`都是以 E-mail 為帳號，剛好專案的使用者希望用簡單的帳號(不想每次都打 email)，就來看看要修改哪些地方吧

## 一、預設專案範本

- 建立專案--> 預設 MVC 專案範本
    ![create](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x685/3aa1b60d7eb550dc25ab18f9c7543a84/_output_create.png)

    ![defaulttemplate](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1180x920/5260cd2656f01f0fdc8b6ef422ce98d6/_output_defaulttemplate.png)

- 註冊

    ![registerwithemail](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/871x686/bbb0c51a4bff753acafe04ea6cba7f96/_output_registerwithemail.png)

- 登入

    ![loginwithemail](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/727x588/23477ae9d5dc58441d1c3aed7f356c26/_output_loginwithemail.png)

## 二、開始修改

## 1. 註冊

- 1-1. 註冊畫面

    > Views/Account/Register.cshtml

    ![registercshtml](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/371x766/71992c699242ce2fff6aed55b7e3c4f7/_output_registercshtml.png)

  - 將 Email 改為 Username

            ![registercshtml2](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x521/fc8f334b189d7019b57b9f857a776850/_output_registercshtml2.png)

- 1-2. 註冊 Model
  - Models/AccountViewModel.cs 的 RegisterViewModel

        ![registermodel](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x492/2d70be730000853809848b7f05f8a8fd/_output_registermodel.png)

            ![registerdone](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x390/fba91740adeb3b1f24899a0c86ed1da7/_output_registerdone.png)

  - `Email` --> `Username`
  - `Display(Name="Email")` --> `Display(Name="Username")`
  - 移除 `[EmailAddress]`

        >沒有移除`[EmailAddress]` 出現的錯誤

        ![emialattibute](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/802x664/1c52c84c9505de183de1cc29c2e56f28/_output_emialattibute.png)

- 1-3. 註冊功能

    > Contrllers/AccountController.cs 的 Register 方法

    ![register](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x484/9d746ec3df9dc0c8a127f2733bc70af9/_output_register.png)

    ![REGISTERCHANGE](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1078x154/98555c06f9b8b21a6fc54453791ef958/_output_REGISTERCHANGE.png)

    ![REGISTERCHANGEDONE](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1009x110/86c14bacbec05b9880fe6b48f0d67db0/_output_REGISTERCHANGEDONE.png)

  - 把 `Username = model.Email` 改為 `Username = model.Username`

        >由此可知，其實本來就有 Username 這個欄位，只是預設用 email 取代了

  - 把 `,Email = model.Email` 刪除

- 1-4. Identity 設定

    > App_Start/IdentityConfig.cs 的 ApplicationUserManager 屬性

  - `RequireUniqueEmail = true` --> `RequireUniqueEmail = false`
            ![indentitychanged](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x373/0fecdd5cbc2e27e76a96ca41cc741e95/_output_indentitychanged.png)
  - 未修改時會出現錯誤
            ![needemail](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/798x656/cde3967a93c5a650e39326be6812dd15/_output_needemail.png)

## 2. 登入

![login](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/671x656/14d9573959fc349d7066244c161a7be4/_output_login.png)

- 2-1. 登入畫面

    > Views/Login.cshtml

    ![logincshtml](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x503/bf665a6d709d0d2a2b44511bb510efff/_output_logincshtml.png)

  - Email --> Username

        ![logincshtmldone](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x476/da12fc57ff8d3028938d5f3bb2d08329/_output_logincshtmldone.png)

- 2-2. 修改 登入 Model

    >Models/AccountViewModel.cs 的 LoginViewModel 類別

    ![loginviewmodel](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x334/9ae3c862c6cbb44371ef7333bcc7d578/_output_loginviewmodel.png)

    ![loginviewmodeldone](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x280/3626f5007ed543e4b3aec67603becd53/_output_loginviewmodeldone.png)

  - `Email` --> `Username`
  - `Display(Name="Email")` --> `Display(Name="Username")`
  - 移除 `[EmailAddress]`

- 2-3. 登入功能

    >Contrllers/AccountController.cs 的 login 方法

    ![loginfunction](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x371/91ca2bfeead751d2df8854a3faa37637/_output_loginfunction.png)

  - `model.Email` --> `model.Username`

        ![loginfunctiondone](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/1200x382/0e2c74cb8880153e7dac845a234318d4/_output_loginfunctiondone.png)

## 完成修改

1. 註冊

    ![register1](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/786x483/d1d5e7c759e0f98ffbb5709994a6596f/_output_register1.png)

2. 登入

    ![2login](https://trello-attachments.s3.amazonaws.com/5835ab9d70f36ddbf5f15558/704x471/14e38aed7ef3e53ca9ec2085646d6e50/_output_2login.png)

## 心得

登入的帳號從 `Email` 改為 `Username` ，改動範圍並不大，也沒什麼機會瞭解到 `ASP.NET Identity`，接下來再找時間來做一些測試修改，好好瞭解 `ASP.NET Identity` 吧
