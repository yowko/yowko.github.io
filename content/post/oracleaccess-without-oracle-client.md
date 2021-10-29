---
title: "不需安裝 Oracle client 使用 C# 搭配 Oracle.DataAccess 連線 Oracle"
date: 2018-05-31T01:45:00+08:00
lastmod: 2021-10-29T01:45:32+08:00
draft: false
tags: ["csharp","Oracle"]
slug: "oracleaccess-without-oracle-client"
aliases:
    - /2018/05/oracleaccess-without-oracle-client.html
---
## 不需安裝 Oracle client 使用 C# 搭配 Oracle.DataAccess 連線 Oracle

之前公司電腦因為註冊檔毀損，讓電腦上的 Oracle client 一直無法正常運作，就算是重灌多次 Oracle client 還是一樣無法正確運作就連移除功能也壞了，所以在 local 開發時我都會暫時將 `Oracle.DataAccess` 改為 `Oracle.ManagedDataAccess`，在多數情況下都可以正常運作，真的非得用到 `Oracle.DataAccess` 就透過 VM 來開發，實際上遇到問題的次數一隻手數得出來，加上近期已經很少用到 `Oracle.DataAccess` 也就沒有特別想解決 Oracle client 無法正常運作的問題，直到最近有個困擾已久的 issue 一直沒有找到真正原因，透過 LINQPad 模擬時一直搞不定，讓我有了不得不解決的動力

憑心而論，如果沒有其他考量，建議升級為 `Oracle.ManagedDataAccess`，原生就不需要 Oracle client，如果你還有舊系統的相依無法搞定，就來看看如何可以不安裝 Oracle client 並使用 `Oracle.DataAccess` 連線吧

## C# 連線 Oracle 範例

```cs
using (OracleConnection oracleConnection = new OracleConnection("DATA SOURCE=localhost:1521/xe;PASSWORD=password;PERSIST SECURITY INFO=True;USER ID=TEST"))
using (OracleCommand oracleCommand = oracleConnection.CreateCommand())
{
    oracleConnection.Open();
    oracleCommand.CommandText = "select * from Users1";
    var reader = oracleCommand.ExecuteReader();
    try
    {
        if (reader.HasRows)
        {
            while (reader.Read())
            {
                $"{Thread.CurrentThread.ManagedThreadId}_{reader[0]}|{reader[1]}|{reader[2]}".Dump();
            }
        }
        else
        {
            "No rows found.".Dump();
        }
    }
    catch (Exception ex)
    {
        throw;
    }
    finally{
        reader.Close();    
    }
    
}
```

## 錯訊訊息

1. 訊息內容

    ```txt
    InnerException|The provider is not compatible with the version of Oracle client 

    Message|
    The type initializer for 'Oracle.DataAccess.Client.OracleConnection' threw an exception. 

    Source|Oracle.DataAccess 

    StackTrace|
    at Oracle.DataAccess.Client.OracleConnection..ctor(String connectionString)
    at UserQuery.Main()
    at System.Threading.ThreadHelper.ThreadStart_Context(Object state)
    at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
    at System.Threading.ThreadHelper.ThreadStart() 
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/40736952-00f94d48-6472-11e8-90d3-234abfd9f425.png)

## 設定步驟

1. 下載並解壓 Oracle Data Access Components (ODAC)

    > 請至 [32-bit Oracle Data Access Components (ODAC) and NuGet Downloads](http://www.oracle.com/technetwork/database/windows/downloads/utilsoft-087491.html) 對應 oracle 版本的 ODAC
    > 我自己測試下來，版本無法向下相容，e.x. ODAC 12.x 無法正常連線 Oracle 11g

2. 手動加入 `Oracle.DataAccess.dll` 參考
    - 位置

        > `{ODAC 所在路徑}\odp.net4\odp.net\bin\4\Oracle.DataAccess.dll`

    - 實際範例

        >`C:\Users\yowko\Downloads\ODAC112040Xcopy_32bit\odp.net4\odp.net\bin\4\Oracle.DataAccess.dll`

    - 實測後 NuGet 上的版本，都無法使用
        - [NuGet] Oracle.DataAccess.x86

            ![2odax862](https://user-images.githubusercontent.com/3851540/40736953-012dd8f6-6472-11e8-9fe7-42e8e5c50453.png)

        - [NuGet] Oracle.DataAccess.x86.4

            ![3odax864](https://user-images.githubusercontent.com/3851540/40736954-01629d8e-6472-11e8-992c-5bd18006f89b.png)

        - 下載的 ODAC

            ![4odacdll](https://user-images.githubusercontent.com/3851540/40736955-0192bdb6-6472-11e8-8f4b-08ef12151f0c.png)

3. 手動複製三個檔案至執行檔所在資料夾中
    > console 專案與 web 專案就放在 `bin` 資料夾中，LINQPad 就跟 `LINQPad.exe` 放在一起
    - oci.dll
        - 位置

            >`{ODAC 所在路徑}\instantclient_11_2\oci.dll`

        - 實際範例

            > `C:\Users\yowko\Downloads\ODAC112040Xcopy_32bit\instantclient_11_2\oci.dll`

    - oraociei11.dll
        - 位置

            >`{ODAC 所在路徑}\instantclient_11_2\oraocci11.dll`

        - 實際範例

            > `C:\Users\yowko\Downloads\ODAC112040Xcopy_32bit\instantclient_11_2\oraocci11.dll`

    - OraOps11w.dll
        - 位置

            >`{ODAC 所在路徑}\odp.net4\bin\OraOps11w.dll`

        - 實際範例

            >`C:\Users\yowko\Downloads\ODAC112040Xcopy_32bit\odp.net4\bin\OraOps11w.dll`

- 已可正常從 Oracle 取得資料

    ![5result](https://user-images.githubusercontent.com/3851540/40736956-01c3dcb6-6472-11e8-9902-14ccc1378af7.png)

## 心得

之前筆記 [不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle](/2017/11/c-sharp-oracle-tns-without-client.html) 紀錄到如何在不安裝 Oracle client 透過 tns 來連線 Oracle，當時還因為終於避開直接相依 Oracle client 的一個重要功能而洋洋得意，今天再次搞定 `Oracle.DataAccess` 對 Oracle client 的依賴，更是開心呀，讓我再也不用擔心我的 Oracle client 無法正常運作而影響工作進度了

不過再次強調，如果情況允許，請改用 `Oracle.ManagedDataAccess` 真的省事很多，不用搞這些有的沒的，還可以在 64-bit 下執行，可以使用的記憶體增加後也能讓程式效率變好，值得推薦呀

## 參考資訊

1. [the type initializer for Oracle.DataAccess.Client.OracleConnection threw an exception](https://www.daniweb.com/programming/software-development/threads/372248/the-type-initializer-for-oracle-dataaccess-client-oracleconnection-threw-an-exception)
2. [不用安裝 Oracle Client 使用 C# 透過 tnsnamses.ora 連結 Oracle](/2017/11/c-sharp-oracle-tns-without-client.html)
