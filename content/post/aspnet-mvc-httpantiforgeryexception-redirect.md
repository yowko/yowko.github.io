---
title: "在 ASP.NET MVC 出現 HttpAntiForgeryException 錯誤時 Redirect 至指定 Controller、Action"
date: 2018-05-01T16:43:00+08:00
lastmod: 2018-10-06T22:28:47+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "aspnet-mvc-httpantiforgeryexception-redirect"
aliases:
    - /2018/05/aspnet-mvc-httpantiforgeryexception-redirect.html
---
# ASP.NET MVC 的 System.Web.Mvc.HttpAntiForgeryException 錯誤
user 反應在操作系統時出現 System.Web.Mvc.HttpAntiForgeryException錯誤，而 System.Web.Mvc.HttpAntiForgeryException 的錯誤在 ASP.NET MVC 上並不算罕見，是個為了避免 XSRF 的保護機制：在網頁表單上加上個隱藏欄位用來儲存 token ，實際 submit 時 server 會檢查這個 token 是否有效，常發生在網頁開啟閒置一段時間後再次 submit 資料的情境下，通常只要 refresh 頁面讓網頁重新取得 token 即可解決

本次專案的 owner 希望可以給 user 更好的使用者體驗，所以希望在 token 過期失效時重新導向登入頁面並提示需重新登入(經 user 這麼說，我也覺得讓錯誤直接裸奔在外實在不好，但我以前怎麼不會這麼想，難道我愈來愈客戶導向了嗎XD)


## 錯誤訊息
遇到的錯誤有兩個：

1. 錯誤一：The required anti-forgery cookie "__RequestVerificationToken" is not present.
    - 訊息內容 
        ```
        Server Error in '/' Application.
        The required anti-forgery cookie "__RequestVerificationToken" is not present.
        Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 

        Exception Details: System.Web.Mvc.HttpAntiForgeryException: The required anti-forgery cookie "__RequestVerificationToken" is not present.

        Source Error: 

        An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.

        Stack Trace: 


        [HttpAntiForgeryException (0x80004005): The required anti-forgery cookie "__RequestVerificationToken" is not present.]
        System.Web.Helpers.AntiXsrf.TokenValidator.ValidateTokens(HttpContextBase httpContext, IIdentity identity, AntiForgeryToken sessionToken, AntiForgeryToken fieldToken) +461
        System.Web.Helpers.AntiXsrf.AntiForgeryWorker.Validate(HttpContextBase httpContext) +71
        System.Web.Helpers.AntiForgery.Validate() +92
        System.Web.Mvc.ValidateAntiForgeryTokenAttribute.OnAuthorization(AuthorizationContext filterContext) +18
        System.Web.Mvc.ControllerActionInvoker.InvokeAuthorizationFilters(ControllerContext controllerContext, IList`1 filters, ActionDescriptor actionDescriptor) +97
        System.Web.Mvc.Async.<>c__DisplayClass21.<BeginInvokeAction>b__19(AsyncCallback asyncCallback, Object asyncState) +743
        System.Web.Mvc.Async.WrappedAsyncResult`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +14
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Async.AsyncControllerActionInvoker.BeginInvokeAction(ControllerContext controllerContext, String actionName, AsyncCallback callback, Object state) +343
        System.Web.Mvc.Controller.<BeginExecuteCore>b__1c(AsyncCallback asyncCallback, Object asyncState, ExecuteCoreState innerState) +25
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +30
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Controller.BeginExecuteCore(AsyncCallback callback, Object state) +465
        System.Web.Mvc.Controller.<BeginExecute>b__14(AsyncCallback asyncCallback, Object callbackState, Controller controller) +18
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +20
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Controller.BeginExecute(RequestContext requestContext, AsyncCallback callback, Object state) +374
        System.Web.Mvc.Controller.System.Web.Mvc.Async.IAsyncController.BeginExecute(RequestContext requestContext, AsyncCallback callback, Object state) +16
        System.Web.Mvc.MvcHandler.<BeginProcessRequest>b__4(AsyncCallback asyncCallback, Object asyncState, ProcessRequestState innerState) +52
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +30
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.MvcHandler.BeginProcessRequest(HttpContextBase httpContext, AsyncCallback callback, Object state) +384
        System.Web.Mvc.MvcHandler.BeginProcessRequest(HttpContext httpContext, AsyncCallback callback, Object state) +48
        System.Web.Mvc.MvcHandler.System.Web.IHttpAsyncHandler.BeginProcessRequest(HttpContext context, AsyncCallback cb, Object extraData) +16
        System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +103
        System.Web.HttpApplication.ExecuteStepImpl(IExecutionStep step) +48
        System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +159

        Version Information: Microsoft .NET Framework Version:4.0.30319; ASP.NET Version:4.7.2623.0
        ``` 
    - 錯誤截圖
        >![1error_notpresent](https://user-images.githubusercontent.com/3851540/39466336-f47e9236-4d5a-11e8-9e02-23e42318c862.png)
2. 錯誤二：The anti-forgery cookie token and form field token do not match.
    - 錯誤訊息
        ```
        Server Error in '/' Application.
        The anti-forgery cookie token and form field token do not match.
        Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 

        Exception Details: System.Web.Mvc.HttpAntiForgeryException: The anti-forgery cookie token and form field token do not match.

        Source Error: 

        An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.

        Stack Trace: 


        [HttpAntiForgeryException (0x80004005): The anti-forgery cookie token and form field token do not match.]
        System.Web.Helpers.AntiXsrf.TokenValidator.ValidateTokens(HttpContextBase httpContext, IIdentity identity, AntiForgeryToken sessionToken, AntiForgeryToken fieldToken) +554
        System.Web.Helpers.AntiXsrf.AntiForgeryWorker.Validate(HttpContextBase httpContext) +71
        System.Web.Helpers.AntiForgery.Validate() +92
        System.Web.Mvc.ValidateAntiForgeryTokenAttribute.OnAuthorization(AuthorizationContext filterContext) +18
        System.Web.Mvc.ControllerActionInvoker.InvokeAuthorizationFilters(ControllerContext controllerContext, IList`1 filters, ActionDescriptor actionDescriptor) +97
        System.Web.Mvc.Async.<>c__DisplayClass21.<BeginInvokeAction>b__19(AsyncCallback asyncCallback, Object asyncState) +743
        System.Web.Mvc.Async.WrappedAsyncResult`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +14
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Async.AsyncControllerActionInvoker.BeginInvokeAction(ControllerContext controllerContext, String actionName, AsyncCallback callback, Object state) +343
        System.Web.Mvc.Controller.<BeginExecuteCore>b__1c(AsyncCallback asyncCallback, Object asyncState, ExecuteCoreState innerState) +25
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +30
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Controller.BeginExecuteCore(AsyncCallback callback, Object state) +465
        System.Web.Mvc.Controller.<BeginExecute>b__14(AsyncCallback asyncCallback, Object callbackState, Controller controller) +18
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +20
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.Controller.BeginExecute(RequestContext requestContext, AsyncCallback callback, Object state) +374
        System.Web.Mvc.Controller.System.Web.Mvc.Async.IAsyncController.BeginExecute(RequestContext requestContext, AsyncCallback callback, Object state) +16
        System.Web.Mvc.MvcHandler.<BeginProcessRequest>b__4(AsyncCallback asyncCallback, Object asyncState, ProcessRequestState innerState) +52
        System.Web.Mvc.Async.WrappedAsyncVoid`1.CallBeginDelegate(AsyncCallback callback, Object callbackState) +30
        System.Web.Mvc.Async.WrappedAsyncResultBase`1.Begin(AsyncCallback callback, Object state, Int32 timeout) +128
        System.Web.Mvc.MvcHandler.BeginProcessRequest(HttpContextBase httpContext, AsyncCallback callback, Object state) +384
        System.Web.Mvc.MvcHandler.BeginProcessRequest(HttpContext httpContext, AsyncCallback callback, Object state) +48
        System.Web.Mvc.MvcHandler.System.Web.IHttpAsyncHandler.BeginProcessRequest(HttpContext context, AsyncCallback cb, Object extraData) +16
        System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +103
        System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +155

        Version Information: Microsoft .NET Framework Version:4.0.30319; ASP.NET Version:4.6.1586.0
        ```
    - 錯誤截圖
        >![2error_notmatch](https://user-images.githubusercontent.com/3851540/39466337-f4c0e924-4d5a-11e8-9f66-7ea4b9e9faf0.png)

## 解決方式
1. 自訂 `AntiForgeryErrorHandlerAttribute` 繼承自 `HandleErrorAttribute`
    
    ```cs
    public class AntiForgeryErrorHandlerAttribute : HandleErrorAttribute
    {
        //用來指定 redirect 的目標 controller
        public string Controller { get; set; }
        //用來儲存想要顯示的訊息
        public string ErrorMessage { get; set; }
        //覆寫預設發生 exception 時的動作
        public override void OnException(ExceptionContext filterContext)
        {
            //如果發生的 exception 是 HttpAntiForgeryException 就轉導至設定的 controller、action (action 在 base HandleErrorAttribute已宣告)
            if (filterContext.Exception is HttpAntiForgeryException)
            {
                //這個屬性要設定為 true 才能接手處理 exception 也才可以 redirect
                filterContext.ExceptionHandled = true;
                //將 errormsg 使用 TempData 暫存 (ViewData 與 ViewBag 因為生命週期的關係都無法正確傳遞)
                filterContext.Controller.TempData.Add("Timeout", ErrorMessage);
                //指定 redirect 的 controller 偶 action
                filterContext.Result = new RedirectToRouteResult(
                    new RouteValueDictionary
                    {
                        { "action", View },
                        { "controller", Controller},
                    });
            }
            else
                base.OnException(filterContext);// exception 不是 HttpAntiForgeryException 就照 mvc 預設流程
        }
    }
    ``` 
2. 在需要處理 AntiForgeryError 的 action 上套用自訂的 `AntiForgeryErrorHandlerAttribute`
    ```cs
    [AntiForgeryErrorHandler(ExceptionType = typeof(HttpAntiForgeryException), View = "Login", Controller = "Account", ErrorMessage = "Session Timeout")]
    ``` 
    ```cs
    // POST: /Account/Login
    [HttpPost]
    [AllowAnonymous]
    [ValidateAntiForgeryToken]
    [AntiForgeryErrorHandler(ExceptionType = typeof(HttpAntiForgeryException), View = "Login", Controller = "Account", ErrorMessage = "Session Timeout")]
    public async Task<ActionResult> Login(LoginViewModel model, string returnUrl)
    {
        if (!ModelState.IsValid)
        {
            return View(model);
        }
        // This doesn't count login failures towards account lockout
        // To enable password failures to trigger account lockout, change to shouldLockout: true
        var result = await SignInManager.PasswordSignInAsync(model.Email, model.Password, model.RememberMe, shouldLockout: false);
        switch (result)
        {
            case SignInStatus.Success:
                return RedirectToLocal(returnUrl);
            case SignInStatus.LockedOut:
                return View("Lockout");
            case SignInStatus.RequiresVerification:
                return RedirectToAction("SendCode", new { ReturnUrl = returnUrl, RememberMe = model.RememberMe });
            case SignInStatus.Failure:
            default:
                ModelState.AddModelError("", "Invalid login attempt.");
                return View(model);
        }
    }
    ```
3. redirect 目標的 action 處理暫存資料
    ```cs
    [AllowAnonymous]
    public ActionResult Login(string returnUrl)
    {
        ViewBag.ReturnUrl = returnUrl;
        //將顯示用訊息從 TempData 取出
        var errmsg = TempData["Timeout"] as string;
        if (!string.IsNullOrWhiteSpace(errmsg))
        {
            ViewBag.Timeout = errmsg;//將顯示訊息放入 ViewBag 供 view 使用
        }
        return View();
    }
    ``` 
4. 前端頁面處理提示訊息
    ```js
    @section Scripts {
        @Scripts.Render("~/bundles/jqueryval")
        <script>
            $(function () {
                var errmsg = '@ViewBag.Timeout'
                console.log(errmsg);
                if (errmsg!=="") {
                    alert(errmsg);
                }

            })
        </script>
    }
    ```
5. 實際效果
    
    >![3result](https://user-images.githubusercontent.com/3851540/39466338-f4ed2d4a-4d5a-11e8-8140-a7687c52eae0.png)

## 心得
原本覺得只是 redirect 應該很容易，但動手實做才發現眉眉角角還不少，像是 ExceptionHandler 預設是導向 shared 下的 view、master 的設定根本沒用到，還要將 ExceptionHandled 屬性設為 true，另外還有 ViewData 及 ViewBag 都是無效的，雖然說的這些都是我沒有詳閱公開說明書導致在未充份瞭解整個運作機製造成的，幸虧透過實作還是多少接觸了其他中的原理，這也就是我為什麼喜歡自己動作做的原因：搞清楚運作機制的踏實跟完成目標後所帶來的成就感都相當令人難以割捨呀

# 參考資訊
1. [Best Practices for Error Handling in ASP.NET MVC](https://stackify.com/aspnet-mvc-error-handling/)
2. [ASP.NET MVC - ValidateAntiForgeryToken expiring](https://stackoverflow.com/questions/12678938/asp-net-mvc-validateantiforgerytoken-expiring)
3. [Error Handling - Controller's OnException and Application_Error](https://codereview.stackexchange.com/questions/60759/error-handling-controllers-onexception-and-application-error)
4. [s it possible to use RedirectToAction() inside a custom AuthorizeAttribute class?](https://stackoverflow.com/questions/2472578/is-it-possible-to-use-redirecttoaction-inside-a-custom-authorizeattribute-clas)
5. [A way of properly handling HttpAntiForgeryException in MVC 4 application](https://stackoverflow.com/questions/12967917/a-way-of-properly-handling-httpantiforgeryexception-in-mvc-4-application)
