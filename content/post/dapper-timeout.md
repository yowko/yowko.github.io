---
title: "C# 使用 Dapper 連線 DB 時指定逾時時間 (timeout)：0x80004005"
date: 2018-04-22T23:34:00+08:00
lastmod: 2018-10-06T23:34:43+08:00
draft: false
tags: ["套件","C#","Dapper","SQL Server"]
slug: "dapper-timeout"
aliases:
    - /2018/04/dapper-timeout.html
---
# C# 使用 Dapper 連線 DB 時指定逾時時間 (timeout)：0x80004005

最近專案在 production 環境執行時常常遇到 [Win32Exception (0x80004005): The wait operation timed out]，造成程式未完整執行，但再執行一次後又正常了，想當然爾這個狀況在開發階段未曾發現過(再次證實在我的電腦上都是好的XD)，經過追查 log 後發現是 db 資料量略大，加上查詢語法在目標 db 上沒有建立對應的 index，所以執行查詢時都會耗費較多時間，但第二次之後的查詢則因為第一次查詢的犧牲已 trigger db 協助建立相關 cache，讓後續查詢得已順利完成

追查問題的過程中，發現沒能在第一時間快速精確鎖定相關設定解決問題，所以馬上紀錄一下加強印象

## 預設環境設定

1. 使用 sql 語法模擬高耗時 db 操作
    
    ```sql
    select getdate();
    waitfor delay '00:00:30';
    select getdate();
    ```
2. 使用 dapper 連線
    ```cs
     var sql = @"select getdate();
                 waitfor delay '00:00:30';
                 select getdate();";
     var connection = ConfigurationManager.ConnectionStrings["TestDB"].ConnectionString;
     using (var cn = new SqlConnection(connection))
     {
         var result = cn.Query<DateTime>(sql);
     }
    ```

## 錯誤訊息

1. 訊息內容
    ```text
    Server Error in '/' Application.
    The wait operation timed out
    Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 
    
    Exception Details: System.ComponentModel.Win32Exception: The wait operation timed out
    
    Source Error: 
    
    
    Line 22:             {
    Line 23:                 
    Line 24:                 var result = cn.Query<DateTime>(sql);
    Line 25:             }
    Line 26: 
    
    Source File: C:\Users\yowko\source\repos\TestSQLTimeout\TestSQLTimeout\Controllers\HomeController.cs    Line: 24 
    
    Stack Trace: 
    
    
    [Win32Exception (0x80004005): The wait operation timed out]
    
    [SqlException (0x80131904): Execution Timeout Expired.  The timeout period elapsed prior to completion of the operation or the server is not responding.]
       System.Data.SqlClient.SqlConnection.OnError(SqlException exception, Boolean breakConnection, Action`1 wrapCloseInAction) +2444190
       System.Data.SqlClient.SqlInternalConnection.OnError(SqlException exception, Boolean breakConnection, Action`1 wrapCloseInAction) +5775712
       System.Data.SqlClient.TdsParser.ThrowExceptionAndWarning(TdsParserStateObject stateObj, Boolean callerHasConnectionLock, Boolean asyncClose) +285
       System.Data.SqlClient.TdsParser.TryRun(RunBehavior runBehavior, SqlCommand cmdHandler, SqlDataReader dataStream, BulkCopySimpleResultSet bulkCopyHandler, TdsParserStateObject stateObj, Boolean& dataReady) +4169
       System.Data.SqlClient.SqlDataReader.TryConsumeMetaData() +58
       System.Data.SqlClient.SqlDataReader.get_MetaData() +89
       System.Data.SqlClient.SqlCommand.FinishExecuteReader(SqlDataReader ds, RunBehavior runBehavior, String resetOptionsString, Boolean isInternal, Boolean forDescribeParameterEncryption) +409
       System.Data.SqlClient.SqlCommand.RunExecuteReaderTds(CommandBehavior cmdBehavior, RunBehavior runBehavior, Boolean returnStream, Boolean async, Int32 timeout, Task& task, Boolean asyncWrite, Boolean inRetry, SqlDataReader ds, Boolean describeParameterEncryptionRequest) +2127
       System.Data.SqlClient.SqlCommand.RunExecuteReader(CommandBehavior cmdBehavior, RunBehavior runBehavior, Boolean returnStream, String method, TaskCompletionSource`1 completion, Int32 timeout, Task& task, Boolean& usedCache, Boolean asyncWrite, Boolean inRetry) +911
       System.Data.SqlClient.SqlCommand.RunExecuteReader(CommandBehavior cmdBehavior, RunBehavior runBehavior, Boolean returnStream, String method) +64
       System.Data.SqlClient.SqlCommand.ExecuteReader(CommandBehavior behavior, String method) +240
       System.Data.SqlClient.SqlCommand.ExecuteDbDataReader(CommandBehavior behavior) +41
       System.Data.Common.DbCommand.System.Data.IDbCommand.ExecuteReader(CommandBehavior behavior) +12
       Dapper.SqlMapper.ExecuteReaderWithFlagsFallback(IDbCommand cmd, Boolean wasClosed, CommandBehavior behavior) +60
       Dapper.<QueryImpl>d__136`1.MoveNext() +299
       System.Collections.Generic.List`1..ctor(IEnumerable`1 collection) +189
       System.Linq.Enumerable.ToList(IEnumerable`1 source) +31
       Dapper.SqlMapper.Query(IDbConnection cnn, String sql, Object param, IDbTransaction transaction, Boolean buffered, Nullable`1 commandTimeout, Nullable`1 commandType) +307
       TestSQLTimeout.Controllers.HomeController.Index() in C:\Users\yowko\source\repos\TestSQLTimeout\TestSQLTimeout\Controllers\HomeController.cs:24
       lambda_method(Closure , ControllerBase , Object[] ) +62
       System.Web.Mvc.ActionMethodDispatcher.Execute(ControllerBase controller, Object[] parameters) +14
       System.Web.Mvc.ReflectedActionDescriptor.Execute(ControllerContext controllerContext, IDictionary`2 parameters) +157
       System.Web.Mvc.ControllerActionInvoker.InvokeActionMethod(ControllerContext controllerContext, ActionDescriptor actionDescriptor, IDictionary`2 parameters) +27
       System.Web.Mvc.Async.AsyncControllerActionInvoker.<BeginInvokeSynchronousActionMethod>b__39(IAsyncResult asyncResult, ActionInvocation innerInvokeState) +22
       System.Web.Mvc.Async.WrappedAsyncResult`2.CallEndDelegate(IAsyncResult asyncResult) +29
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.Async.AsyncControllerActionInvoker.EndInvokeActionMethod(IAsyncResult asyncResult) +32
       System.Web.Mvc.Async.AsyncInvocationWithFilters.<InvokeActionMethodFilterAsynchronouslyRecursive>b__3d() +50
       System.Web.Mvc.Async.<>c__DisplayClass46.<InvokeActionMethodFilterAsynchronouslyRecursive>b__3f() +228
       System.Web.Mvc.Async.<>c__DisplayClass33.<BeginInvokeActionMethodWithFilters>b__32(IAsyncResult asyncResult) +10
       System.Web.Mvc.Async.WrappedAsyncResult`1.CallEndDelegate(IAsyncResult asyncResult) +10
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.Async.AsyncControllerActionInvoker.EndInvokeActionMethodWithFilters(IAsyncResult asyncResult) +34
       System.Web.Mvc.Async.<>c__DisplayClass2b.<BeginInvokeAction>b__1c() +26
       System.Web.Mvc.Async.<>c__DisplayClass21.<BeginInvokeAction>b__1e(IAsyncResult asyncResult) +100
       System.Web.Mvc.Async.WrappedAsyncResult`1.CallEndDelegate(IAsyncResult asyncResult) +10
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.Async.AsyncControllerActionInvoker.EndInvokeAction(IAsyncResult asyncResult) +27
       System.Web.Mvc.Controller.<BeginExecuteCore>b__1d(IAsyncResult asyncResult, ExecuteCoreState innerState) +13
       System.Web.Mvc.Async.WrappedAsyncVoid`1.CallEndDelegate(IAsyncResult asyncResult) +29
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.Controller.EndExecuteCore(IAsyncResult asyncResult) +36
       System.Web.Mvc.Controller.<BeginExecute>b__15(IAsyncResult asyncResult, Controller controller) +12
       System.Web.Mvc.Async.WrappedAsyncVoid`1.CallEndDelegate(IAsyncResult asyncResult) +22
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.Controller.EndExecute(IAsyncResult asyncResult) +26
       System.Web.Mvc.Controller.System.Web.Mvc.Async.IAsyncController.EndExecute(IAsyncResult asyncResult) +10
       System.Web.Mvc.MvcHandler.<BeginProcessRequest>b__5(IAsyncResult asyncResult, ProcessRequestState innerState) +21
       System.Web.Mvc.Async.WrappedAsyncVoid`1.CallEndDelegate(IAsyncResult asyncResult) +29
       System.Web.Mvc.Async.WrappedAsyncResultBase`1.End() +49
       System.Web.Mvc.MvcHandler.EndProcessRequest(IAsyncResult asyncResult) +28
       System.Web.Mvc.MvcHandler.System.Web.IHttpAsyncHandler.EndProcessRequest(IAsyncResult result) +9
       System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +9748665
       System.Web.HttpApplication.ExecuteStepImpl(IExecutionStep step) +48
       System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +159
    
    Version Information: Microsoft .NET Framework Version:4.0.30319; ASP.NET Version:4.7.2633.0
    ```
2. 錯誤截圖
    
    >![1error](https://user-images.githubusercontent.com/3851540/39096566-27cacb7e-4684-11e8-941b-62a55c6b9635.png)

## ConnectionTimeout vs CommandTimeout 

- ConnectionTimeout
    - 嘗試建立連線的時間
    - 預設值為 `15 秒`，`0` 表示永遠不 timeout
    - 可以針對 Connection 設定 (ConnectTimeout ) 或是在 connection string 指定 (Connection Timeout )

- CommandTimeout
    - 完成執行 sql 語法的時間
    - 預設值為 `30 秒`，`0` 表示永遠不 timeout
    - 對於非同步 command 無效：System.Data.SqlClient.SqlCommand.BeginExecuteReader
    - 對於 context connection 無效，connection string 有 `context connection=true`

## dapper 設定

>在執行實際 sql 的語法上指定所需要的 CommandTimeout 秒數,

```cs
var result = cn.Query<DateTime>(sql,commandTimeout:60);
```

## 心得

經過這次處理 timeout 後，發現一個問題：不知道是記憶模糊還是以往對於 timeout 的設定都是錯誤的，我一直認為 timeout 的相關設定都可以在 connection string 中指定，但查完資料 connection string 僅能設定 ConnectionTimeout 才是呀XD

所以趁著這次機會筆記一下，加深印象，下次再有疑問時就有個對照基準了

# 參考資訊

1. [SqlConnection.ConnectionTimeout](https://msdn.microsoft.com/zh-tw/library/system.data.sqlclient.sqlconnection.connectiontimeout%28v=vs.110%29.aspx)
2. [SqlCommand.CommandTimeout](https://msdn.microsoft.com/zh-tw/library/system.data.sqlclient.sqlcommand.commandtimeout%28v=vs.110%29.aspx)
3. [Context Connections vs. Regular Connections](https://docs.microsoft.com/en-us/sql/relational-databases/clr-integration/data-access/context-connections-vs-regular-connections?view=sql-server-2017&WT.mc_id=DOP-MVP-5002594)
4. [C# 中Sql 執行超時的問題](http://fecbob.pixnet.net/blog/post/38121651-c%23-中sql-執行超時的問題)
5. [Adjusting CommandTimeout in Dapper.NET?](https://stackoverflow.com/questions/8794858/adjusting-commandtimeout-in-dapper-net)