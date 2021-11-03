---
title: "如何在 ASP.NET MVC 加上簡易表單驗證"
date: 2017-03-19T02:43:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-simple-auth"
aliases:
    - /2017/03/aspnet-mvc.html
    - /2017/03/aspnet-mvc-simple-auth/
---
## 如何在 ASP.NET MVC 加上簡易表單驗證

以前寫 ASP.NET WebForm 的時，有時候專案規模不需要建立整套的 Membership 功能，有時候幾個開發者小工具 DBA 也不想讓你建那些 table，所以~~有時~~(好吧，我承認是常常)就會偷懶利用在 web.config 指定 crendential 搭配 form authentication(表單驗證) 的方式來為網站加上簡易驗證

隨著時間演進，技術不斷更新，不變的依然是我那總是想偷懶的心，所以延續 ASP.NET WebForm 的做法，也可以將在 web.config 指定 crendential 搭配 form authentication(表單驗證) 的簡易驗證功能套用在 ASP.NET MVC 上，剛好最近又想偷懶，就順手紀錄

><h1><span style="color:red">這功能只能防君子，不要用在重要功能上<span></h1>

## 設定 web.config

1. 驗證模式改為 Forms

    ```xml
    <system.web>
        <authentication mode="Forms">
    </system.web>
    ```

2. 避免移除 FormsAuthentication modules

    >ASP.NET MVC 5 開始預設使用 ASP.NET Identity，因此專案範本預設會移除 FormsAuthentication modules

    ```xml
    <system.webServer>
        <!--<modules>
        <remove name="FormsAuthentication" />
        </modules>-->
    </system.webServer>
    ```

3. 指定預設登入頁面

    >loginUrl="/Home/login" 用來指定如果需要驗證時該導向何處進行登入

    ```xml
    <system.web>
        <authentication mode="Forms">
        <forms loginUrl="/Home/login">
        </forms>
        </authentication>
    </system.web>
    ```

4. 加上 crendential 登入資訊

    - 指定 crendential 的 passwordFormat

        Value|    Description
        ---|---
        Clear|使用明碼。好處是直接使用(少了加密，速度較快)，缺點就是安全性低
        MD5|使用 MD5 (Message Digest 5) 加密。程式會自動將使用者輸入密碼以 MD5 加密後才進行驗證(無須人為介入)，可以確保原始密碼不會被儲存且效能優於 SHA1
        SHA1|    使用 SHA1 加碼。為了驗證認證，程式會自動將使用者輸入密碼以 SHA1 加密才進行驗證(無須人為介入)，可以確保原始密碼不會被儲存。安全性優於 MD5，預設值
    - 以 user 指定登入的帳號及密碼

        ```xml
        <system.web>
            <authentication mode="Forms">
            <forms loginUrl="/Home/login">
                <credentials passwordFormat="Clear">
                <user
                    name="yowko"
                    password="cdm4FA2wHQ3S"/>
                </credentials>
            </forms>
            </authentication>
            <compilation debug="true" targetFramework="4.6.2" />
            <httpRuntime targetFramework="4.6.2" />
        </system.web>
        ```

## 加上登入 action

    ```cs
    public ActionResult Login()
    {
        return View();
    }
    ```

## 加上登入頁面

這個當然就是看大家對畫面要要求程度了，重點就是要提供輸入帳號及密碼，以下提供個人不要求美觀版本供大家參考

```cs
@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    <div>帳號:@Html.TextBox("username")</div>
    <div>密碼:@Html.Password("password")</div>
    <input type="submit" value="Log in" class="btn btn-primary" />
}
```

## 加上登入驗證 action

- 需引用 System.Web.Security NameSpace

    ```cs
    using System.Web.Security;
    ```

- 使用 FormsAuthentication.Authenticate 驗證

    ```cs
    FormsAuthentication.Authenticate(user.username, user.password)
    ```

- 將驗證資訊寫入 cookie ，讓 ASP.NET MVC 也認得已通過驗證

    ```cs
    FormsAuthentication.SetAuthCookie(user.username, false);
    ```

- 範例

    ```cs
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Login(UserLogin user)
    {

        if (FormsAuthentication.Authenticate(user.username, user.password))
        {
            FormsAuthentication.SetAuthCookie(user.username, false);
            return RedirectToAction("About");
        }
        return View();
    }
    ```

## 心得

這個簡易驗證功能附屬於 FormsAuthentication module 中，因為已有後繼版本 - ASP.NET Idnetity ，我想更新的機會不高，功能應該滿穩定的但有個大問題是密碼儲存的加密方式，就算是使用支援的最安全加密方式：SHA1，也已證實被攻破不再曠日費時，所以建議不要把重要的功能套上這樣簡易的驗證方式

## 參考資訊

1. [Forms Authentication Credentials](https://msdn.microsoft.com/en-us/library/da0adyye.aspx)
2. [credentials Element for forms for authentication (ASP.NET Settings Schema)](https://msdn.microsoft.com/en-us/library/e01fc50a%28v=vs.100%29.aspx)
3. [FormsAuthentication.Authenticate Method (String, String)](https://msdn.microsoft.com/en-us/library/system.web.security.formsauthentication.authenticate%28v=vs.110%29.aspx)
4. [MVC 4 Forms Authentication not working with [Authorize]](http://stackoverflow.com/questions/16665660/mvc-4-forms-authentication-not-working-with-authorize)
