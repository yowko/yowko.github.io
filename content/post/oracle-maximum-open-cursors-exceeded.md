---
title: "C# 連線 Oracle 出現 ORA-01000: maximum open cursors exceeded"
date: 2018-06-07T23:52:00+08:00
lastmod: 2021-10-29T14:08:59+08:00
draft: false
tags: ["套件","csharp","Debug","Oracle"]
slug: "oracle-maximum-open-cursors-exceeded"
aliases:
    - /2018/06/oracle-maximum-open-cursors-exceeded.html
---
## C# 連線 Oracle 出現 ORA-01000: maximum open cursors exceeded

同事負責的系統在 production 環境出現異常問題：原本系統已經運作了一段時間，某天突然出現 `ORA-01000: maximum open cursors exceeded` 造成相關功能無法運作，經過 IIS reset 後又可以正常使用，發生頻率不定，只是出現異常的間隔有日趨縮小的現象，造成問題的程式雖然已經找到也完成了修正，但隱含在背後的真正原因還是令我相當好奇，所以多花了不少時間來進行驗證及測試，過程中學到了許多東西一定要好好紀錄才行

## 情境說明

1. 使用 C# 搭配 `Oracle.DataAccess.Client` 存取 Oracle StoredProcedure
2. Oracle StoredProcedure 回傳 sys_refcursor

    > 模擬實際使用的 StoredProcedure 結構及用法

    ```sql
    create or replace procedure test_cursor (p_cursor out sys_refcursor)
    is
    begin
        open p_cursor for
            select * from USERS;
    end;
    ```

3. C# 程式碼

    ```cs
    //建立 oracle connection 物件
    using (OracleConnection oracleConnection = new OracleConnection("DATA SOURCE=localhost:1521/xe;PASSWORD=password;PERSIST SECURITY INFO=True;USER ID=TEST"))
    //建立 oracle command 物件
    using (OracleCommand oracleCommand = oracleConnection.CreateCommand())
    {
        //開啟連線
        oracleConnection.Open();
        //指定 sp name
        oracleCommand.CommandText = "TEST_CURSOR";
        //宣告使用 sp
        oracleCommand.CommandType = CommandType.StoredProcedure;

        //建立 cursor 物件
        OracleParameter p1 = oracleCommand.Parameters.Add("P_CURSOR", OracleDbType.RefCursor);
        //指定從 db 讀取輸出內容
        p1.Direction = ParameterDirection.Output;

        // 執行命令，並取得回傳內容，一般用來接單一資料內容 (e.g. 整筆數)
        oracleCommand.ExecuteNonQuery();

        //將 sp 使用 DataReader 讀取資料
        using (OracleDataReader reader = oracleCommand.ExecuteReader())
        {
            //逐筆取得 DataReader 內容
            while (reader.Read())
            {
                Debug.WriteLine($"{Thread.CurrentThread.ManagedThreadId}_{reader[0]}|{reader[1]}|{reader[2]}");
            }
            // DataReader 物件務必手動關閉
            reader.Close();

        }
    }
    ```

## 錯誤訊息

1. 訊息內容

    ```log
    Server Error in '/' Application.
    ORA-01000: maximum open cursors exceeded
    ORA-06512: at "TEST.TEST_CURSOR", line 4
    ORA-06512: at line 1
    Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 

    Exception Details: Oracle.DataAccess.Client.OracleException: ORA-01000: maximum open cursors exceeded
    ORA-06512: at "TEST.TEST_CURSOR", line 4
    ORA-06512: at line 1

    Source Error: 


    Line 44: 
    Line 45:                     oracleCommand.ExecuteNonQuery();
    Line 46:                     using (OracleDataReader reader = oracleCommand.ExecuteReader())
    Line 47:                     {
    Line 48:                         while (reader.Read())

    Source File: C:\Users\yowko\source\repos\TestEFConcurrency\TestConcurrencyOracle\Controllers\HomeController.cs    Line: 46 

    Stack Trace: 
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/41110836-f50eb784-6aac-11e8-8364-04d619c1c2df.png)

## Oracle 相關基本知識

1. 預設單一 session 的 cursor 上限是 `300` 個
2. 查詢資料的過程中雖然不一定會使用 cursor 語法，但 Oracle 內部都會透過 cursor 來處理
3. 透過以下指令可以取得 open 中的 cursor 數量

    ```sql
    select s.username, a.value CURSOR_COUNT, s.sid, s.serial#, s.machine
    from v$sesstat a, v$statname b, v$session s
    where a.statistic# = b.statistic#  
    and s.sid=a.sid
    and b.name = 'opened cursors current'
    and s.username = 'TEST'
    --and s.machine like '%Yowko%'
    order by a.value desc;
    ```

4. 列出 cursor 相關資訊

    ```sql
    select sid,sql_text,cursor_type  from v$open_cursor where saddr in (
    select saddr
    from gv$session
    where 1=1
    --and username like 'TEST%'
    --and machine like '%yowko%'
    and sid=71 and serial#=23765
    ) 
    and user_name = 'TEST' 
    --and cursor_type = 'OPEN'
    order by sid ,sql_text;
    ```

## 問題發生原因

1. 回傳資料集合不該使用 `oracleCommand.ExecuteNonQuery();`
2. 造成重複建立 cursor 物件且未被關閉及回收直到到上限

    ![2dump](https://user-images.githubusercontent.com/3851540/41110838-f5486420-6aac-11e8-88f3-d7f77d9f84af.png)

## 解決方式 (擇一即可)

1. 改用 `Oracle.ManagedDataAccess.Client`

    > 相同寫法在 `Oracle.ManagedDataAccess.Client` 不會造成 cursor 持續累積

2. 將 cursor 物件 dispose

    ```sql
    //建立 oracle connection 物件
    using (OracleConnection oracleConnection = new OracleConnection("DATA SOURCE=localhost:1521/xe;PASSWORD=password;PERSIST SECURITY INFO=True;USER ID=TEST"))
    //建立 oracle command 物件
    using (OracleCommand oracleCommand = oracleConnection.CreateCommand())
    {
        //開啟連線
        oracleConnection.Open();
        //指定 sp name
        oracleCommand.CommandText = "TEST_CURSOR";
        //宣告使用 sp
        oracleCommand.CommandType = CommandType.StoredProcedure;

        //建立 cursor 物件
        OracleParameter p1 = oracleCommand.Parameters.Add("P_CURSOR", OracleDbType.RefCursor);
        //指定從 db 讀取輸出內容
        p1.Direction = ParameterDirection.Output;

        // 執行命令，並取得回傳內容，一般用來接單一資料內容 (e.g. 整筆數)
        oracleCommand.ExecuteNonQuery();

        //使用 using 確保 cursor 物件使用結束後會被 dispose
        using (var refCursor =(Oracle.DataAccess.Types.OracleRefCursor)oracleCommand.Parameters["P_CURSOR"].Value)
        {
            // 使用 cursor 物件建立 DataReader
            using (var reader = refCursor.GetDataReader())
            {
                //逐筆取得 DataReader 內容
                while (reader.Read())
                {
                    Debug.WriteLine(
                        $"{Thread.CurrentThread.ManagedThreadId}_{reader[0]}|{reader[1]}|{reader[2]}|{reader[3]}");
                }
                // DataReader 物件務必手動關閉
                reader.Close();

            }
        }
    }
    ```

## 心得

感謝 DBA 大大強力支援，不僅提供許多 Oracle 相關知識，還幫忙寫模擬用 StoredProcedure，甚至過程中一度推測是 `Oracle.DataAccess.Client` 用到的 unmanaged dll 偷偷塞了什麼指令造成問題，還請 DBA 大大協助側錄程式執行當下 db 實際收到的 script 來進行確認，也因此最後得以排除異常指令的疑慮

以結果來推敲，可能是當時候寫該段功能的工程師東抄西抄，看到有人用 `oracleCommand.ExecuteNonQuery();` 取資料但發現多筆資料不適用時而改用 `oracleCommand.ExecuteReader()` 後卻忘記刪除 `oracleCommand.ExecuteNonQuery()` 恰巧測試開發階段也沒有出現錯誤，連帶造成後面維護人員不敢主動去刪除，直到出現問題後才重新認真檢驗程式碼

這次為了釐清造成問題的真實原因，測試 `Oracle.DataAccess.Client` v2 及 v4 兩個版本後，發現都有 cursor 未及時關閉的狀況，至於 `Oracle.ManagedDataAccess.Client` 則已確定沒有相同問題，所以發自內心地再次推薦請改用 `Oracle.ManagedDataAccess.Client` 問題真的會比較少，當然寫錯 code 是人禍，不過選擇有防衛性設計的 library 還是可以省不少麻煩呀

## 參考資訊

1. [Oracle ODP.NET Cursor Leak?](https://stackoverflow.com/questions/9759697/reading-a-file-used-by-another-process)
2. [Calling Oracle stored procedures from Microsoft.NET](https://www.c-sharpcorner.com/article/calling-oracle-stored-procedures-from-microsoft-net/)
3. [Debugging managed code memory leak with memory dump using windbg](https://blogs.msdn.microsoft.com/paullou/2011/06/28/debugging-managed-code-memory-leak-with-memory-dump-using-windbg/)
