---
title: "如何在 ASP.NET MVC 的 BaseController 或是 Controller 建構式中存取 ASP.NET Identity User 資料"
date: 2017-01-28T00:42:34+08:00
lastmod: 2018-09-10T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC","ASP.NET Identity"]
slug: "aspnet-mvc-access-identity-user-data-in-basecontroller"
aliases:
    - /2017/01/aspnet-mvc-access-identity-user-data-in-basecontroller.html
---
# 如何在 ASP.NET MVC 的 BaseController 或是 Controller 建構式中存取 ASP.NET Identity User 資料
從 ASP.NET MVC 5 開始預設使用 ASP.NET Identity ，全新的會員身份系統（membership system），雖然已經實際在幾個專案使用，但因為對 ASP.NET Identity 沒有很清楚的理解跟認識，每次使用時都會遇到類似，甚至相同的問題。
最近的專案又遇到了相同狀況(慚愧~~)，趁著記憶猶新趕快紀錄一下，避免每次遇到都要查好久

## 文章大綱
1. 錯誤訊息
2. 新增 BaseController
3. 建立 ASP.NET Identity 資料服務 
4. 其他 Controller 實際使用
5. Web API 下的作法


## 1. 錯誤訊息
- 訊息內容

    ``` 
    Object reference not set to an instance of an object.
    ```
- 錯誤截圖
    
    ![1error](https://cloud.githubusercontent.com/assets/3851540/22275196/e5a68d1c-e2e5-11e6-92b7-9230db985907.png)

## 2. 新增 BaseController
1. 加上 `[Authorize]` attibute
    
    ```cs
    [Authorize]
    public class BaseController : Controller
    ```
2. 新增 User property

    ```cs 
    IUser _UserData;
    public IUser UserData
    {
        get { return _UserData; }
        set { _UserData = value; }
    }
    ```

3. 建立 BaseController 建構式
    - 取得執行緒目前原則
    - 檢查目前原則是否有效登入
    - 依執行原則中的名稱反查 User 資料
        
        ```cs
        public BaseController()
        {
            var userFromAuthCookie = System.Threading.Thread.CurrentPrincipal;

            if (userFromAuthCookie != null && userFromAuthCookie.Identity.IsAuthenticated) // && !String.IsNullOrEmpty(userFromAuthCookie.Identity.Name))
            {
                string userNameFromAuthCookie = userFromAuthCookie.Identity.Name;
                _UserData = UserRepository.GetUser(userNameFromAuthCookie);
            }
        }
        ```
4. 完整程式碼
    
    ```cs
    [Authorize]
    public class BaseController : Controller
    {
        IUser _UserData;

        public IUser UserData
        {
            get { return _UserData; }
        }

        public BaseController()
        {
            var userFromAuthCookie = System.Threading.Thread.CurrentPrincipal;

            if (userFromAuthCookie != null && userFromAuthCookie.Identity.IsAuthenticated)
            {
                string userNameFromAuthCookie = userFromAuthCookie.Identity.Name;
                _UserData = UserRepository.GetUser(userNameFromAuthCookie);
            }
        }


    }
    ```

## 3. 建立 ASP.NET Identity 資料服務 
    
```cs
public static class UserRepository
{
    public  static IUser GetUser(string username)
    {
        var UserManager = HttpContext.Current.GetOwinContext().GetUserManager<ApplicationUserManager>();
        return UserManager.FindByName(username);
    }
}
```

## 4. 其他 Controller 實際使用
1. 繼承 BaseController
    
    ```cs
    public class HomeController : BaseController
    ```

2. 自訂 property
    
    ```cs
    private string userID;
    ```

3. 建構式存取 BaseController property
    
    ```cs
    public HomeController()
    {
        userID = UserData.Id;
    }
    ```

## 5. Web API 下的作法
相關做法請參考 [Identity 2.0 User Identity is null in MVC 5 controller initializer](http://stackoverflow.com/questions/25908209/identity-2-0-user-identity-is-null-in-mvc-5-controller-initializer)

1. 加入自訂 Authorize attibute
    
    ```cs
    public class MyAuthorizeAttribute : AuthorizeAttribute 
    {
        public override async Task OnAuthorizationAsync(HttpActionContext actionContext, CancellationToken cancellationToken)
        {
            if (actionContext.ActionDescriptor.GetCustomAttributes<AllowAnonymousAttribute>().Any()) return;
    
            base.OnAuthorization(actionContext);
    
            Guid userId;
    
            if (actionContext.RequestContext.Principal.Identity.IsAuthenticated
                && Guid.TryParse(actionContext.RequestContext.Principal.Identity.GetUserId(), out userId))
            {
                actionContext.Request.Properties.Add("userId", actionContext.RequestContext.Principal.Identity.GetUserId());
            }
        }
    }
    ```

2. 將自訂的 Authorize attibute 加入 Startup.cs 中
    
    ```cs
    public void Configuration(IAppBuilder app)
    {
        ConfigureOAuth(app);
    
        HttpConfiguration config = new HttpConfiguration();
        WebApiConfig.Register(config);
        config.Filters.Add(new MyAuthorizeAttribute());
    }
    ```

3. 使用 ActionContext 取得 userId
    
    ```cs
    Guid userId = (Guid) ActionContext.Request.Properties["userId"];
    ```

# 參考資料
1. [Defining a User with User.Identity.Name in controller constructor](http://stackoverflow.com/questions/1313190/defining-a-user-with-user-identity-name-in-controller-constructor)
2. [ASP MVC5 - Identity. How to get current ApplicationUser](http://stackoverflow.com/questions/20925822/asp-mvc5-identity-how-to-get-current-applicationuser)
3. [Identity 2.0 User Identity is null in MVC 5 controller initializer](http://stackoverflow.com/questions/25908209/identity-2-0-user-identity-is-null-in-mvc-5-controller-initializer)