---
title: "Fake Assembly 無法自動產生 *.Fakes dll 及出現 build fail"
date: 2017-02-25T01:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Unit Test"]
slug: "add-fake-assembly-error"
aliases:
    - /2017/02/add-fake-assembly-issue.html
---
## Fake Assembly 無法自動產生 *.Fakes dll 及出現 build fail

在 [如何 Mock System.Web.Hosting.HostingEnvironment.MapPath 虛擬路徑](/mock-hostingenvironment-mappath) 提到同事想要 mock `System.Web.Hosting.HostingEnvironment.MapPath` 的值，試了半天決定用 fake dll 的方法來直接解決，但過程不太順利，所以留下紀錄

## 重現問題：無法出現 System.Web.Fakes/System.Web.Http.Fakes dll

1. 加入 Fakes assembly
    - 開啟測試專案的 References 資料夾
    - `System.Web.Http` 上按右鍵 --> Add Fakes Assembly

        ![1addfake](https://cloud.githubusercontent.com/assets/3851540/23245449/8881b316-f9c7-11e6-9bd9-e3f178a7a9ca.png)  
2. 未出現 System.Web.Http.Fakes/System.Web.Fakes dll
    - Fakes 資料夾已產生 `System.Web.Http.fakes`/`System.Web.fakes` 的設定檔
    - References 資料夾則沒有出現對應版本的 `System.Web.Http.{版本}.Fakes`/`System.Web.{版本}.Fakes` dll

        ![2without](https://cloud.githubusercontent.com/assets/3851540/23245450/88a3a020-f9c7-11e6-8bde-7b0c4cf335b7.png)

## 錯誤訊息 1：CS0430

1. 訊息內容

   ```log
   Error    CS0430    The extern alias 'swhod' was not specified in a /reference option [C:\Users\yowko.tsai\documents\visual studio 2015\Projects\DemoUnitTesting\DemoWebApiTesting.Tests\obj\Debug\Fakes\swh\f.csproj]    DemoWebApiTesting.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\DemoUnitTesting\DemoWebApiTesting.Tests\f.cs    19    Active
   
   Error        project compilation failed with exit code 1    DemoWebApiTesting.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\DemoUnitTesting\DemoWebApiTesting.Tests\GENERATEFAKES        
   
   Error        project compilation failed with exit code 1    DemoWebApiTesting.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\DemoUnitTesting\DemoWebApiTesting.Tests\GENERATEFAKES        
   ```

2. 錯誤截圖

   ![6buildfail](https://cloud.githubusercontent.com/assets/3851540/23245455/88cc6f32-f9c7-11e6-973a-8dd2bd5d3240.png)

3. 解決方式
    - System.Web.Http
        1. 開啟 Fakes 資料夾中的 `System.Web.Http.fakes` 設定檔

            ![3fakesconfig](https://cloud.githubusercontent.com/assets/3851540/23245451/88be6bd0-f9c7-11e6-8a42-9e5b1433668a.png)

            ![4fakesconfigcontent](https://cloud.githubusercontent.com/assets/3851540/23245453/88c478b8-f9c7-11e6-811f-6f28b1ebfcab.png)
        2. 加入下列設定

            ```xml
            <StubGeneration>
                <Clear />
                <Add Interfaces="true"/>
            </StubGeneration>
            <ShimGeneration>
                <Clear />
                <Add Namespace="System.Web.Http.ExceptionHandling"/>
            </ShimGeneration>
            ```

        3. 設定前後比較
            - 修改前

                ```xml
                <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
                  <Assembly Name="System.Web.Http" Version="5.2.3.0"/>
                </Fakes>
                ```

            - 修改後

                ```xml
                <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
                  <Assembly Name="System.Web.Http" Version="5.2.3.0"/>
                  <StubGeneration>
                    <Clear />
                    <Add Interfaces="true"/>
                  </StubGeneration>
                  <ShimGeneration>
                    <Clear />
                    <Add Namespace="System.Web.Http.ExceptionHandling"/>
                  </ShimGeneration>
                </Fakes>
                ```

    - System.Web
        1. 開啟 Fakes 資料夾中的 `System.Web.fakes` 設定檔
        2. 加入下列設定

            ```xml
            <StubGeneration>
                <Clear />
                <Add Interfaces="true"/>
            </StubGeneration>
            <ShimGeneration>
                <Clear />
                <Add Namespace="System.Web"/>
            </ShimGeneration>
            ```

        3. 設定前後比較
            - 修改前

                ```xml
                <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
                  <Assembly Name="System.Web" Version="4.0.0.0"/>
                </Fakes>
                ```

            - 修改後

                ```xml
                <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
                  <Assembly Name="System.Web" Version="4.0.0.0"/>
                  <StubGeneration>
                    <Clear />
                    <Add Interfaces="true"/>
                  </StubGeneration>
                  <ShimGeneration>
                    <Clear />
                    <Add Namespace="System.Web"/>
                  </ShimGeneration>
                </Fakes>
                ```

    - 重新執行產生 fake dll
        1. 找個任意其他的 dll 再執行 `Add Fakes Assembly`一次
        2. 可以使用測試目標的 dll  or anyone else 執行後就會出現

            ![5anotherfake](https://cloud.githubusercontent.com/assets/3851540/23245452/88c2ec14-f9c7-11e6-90a6-eb05883e1db6.png)
        3. 刪除不需要的 fake dll 跟 設定檔

            ![7success](https://cloud.githubusercontent.com/assets/3851540/23245454/88c7f3f8-f9c7-11e6-8735-54dab8e9ee4d.png)

## 錯誤訊息：CS0234

1. 訊息內容

    ```log
    Error    CS0234    The type or namespace name 'EventSourceCreatedEventArgs' does not exist in the namespace 'System.Diagnostics.Tracing' (are you missing an assembly reference?) [C:\Users\yowko.tsai\documents\visual studio 2015\Projects\TestFakeError\TestFakeError.Tests\obj\Debug\Fakes\m\f.csproj]    TestFakeError.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\TestFakeError\TestFakeError.Tests\f.cs    17698    Active

    Error        project compilation failed with exit code 1    TestFakeError.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\TestFakeError\TestFakeError.Tests\GENERATEFAKES        
    Error        project compilation failed with exit code 1    TestFakeError.Tests    C:\Users\yowko.tsai\documents\visual studio 2015\Projects\TestFakeError\TestFakeError.Tests\GENERATEFAKES        
    ```

2. 錯誤畫面

    ![8mscorliberror](https://cloud.githubusercontent.com/assets/3851540/23254791/40f0bdee-f9f4-11e6-81be-1094d397bad9.png)
3. 解決方式
    - 開啟 Fakes 資料夾中的 `mscorlib.fakes` 設定檔
    - 加入下列設定

        ```xml
        <StubGeneration>
            <Remove FullName="System.Diagnostics.Tracing"/>
            <Remove FullName="System.Text.Encoding"/>
            <Remove FullName="System.Security.Cryptography" />
        </StubGeneration>
        ```

    - 設定前後比較
        - 修改前

            ```xml
            <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
              <Assembly Name="mscorlib" Version="4.0.0.0"/>
            </Fakes>
            ```

        - 修改後

            ```xml
            <Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
              <Assembly Name="mscorlib" Version="4.0.0.0"/>
              <StubGeneration>
                <Remove FullName="System.Diagnostics.Tracing"/>
                <Remove FullName="System.Text.Encoding"/>
                <Remove FullName="System.Security.Cryptography" />
              </StubGeneration>
            </Fakes>
            ```

## 錯誤訊息：Unexpected error returned by SetDetourProvider

1. 訊息內容

    ```log
    Microsoft.QualityTools.Testing.Fakes.UnitTestIsolation.UnitTestIsolationException
    "Unexpected error returned by SetDetourProvider in profiler library 'C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\CommonExtensions\Microsoft\IntelliTrace\14.0.0\Microsoft.IntelliTrace.Profiler.14.0.0.dll'."
    ```

2. 錯誤畫面

    ![9intellitrace](https://cloud.githubusercontent.com/assets/3851540/23254792/40f1be92-f9f4-11e6-8591-00180ddf3359.png)
3. 解決方式
    - 關閉 IntelliTrace 的 call information
        1. Visual Studio 主選單 Tools tab --> Options...

            ![10vstools](https://cloud.githubusercontent.com/assets/3851540/23254794/40f82a34-f9f4-11e6-90b5-c4824bdaec99.png)
        2. IntelliTrace --> General (下列兩個方式擇一即可)

            ![11disablecallinfo](https://cloud.githubusercontent.com/assets/3851540/23254793/40f4a062-f9f4-11e6-906a-1dd9a1deb738.png)
            - uncheck Enable IntelliTrace
            - IntelliTrace events only

## 參考資料

1. [Fakes issue with System.Web.Http](https://prashantbrall.wordpress.com/2015/04/29/fakes-issue-with-system-web-http/)
2. [Fakes Issue with System.Web](http://jdav.is/2015/11/03/fakes-issue-with-system-web/)
