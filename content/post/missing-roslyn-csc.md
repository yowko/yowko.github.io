---
title: "找不到 roslyn\\csc.exe ?！"
date: 2017-07-24T23:45:00+08:00
lastmod: 2018-09-24T23:44:30+08:00
draft: false
tags: ["Debug"]
slug: "missing-roslyn-csc"
aliases:
    - /2017/07/missing-roslyn-csc.html
    - /2017/07/missing-roslyn
---
# 找不到 roslyn\csc.exe ?！
今天在整合 Jenkins 跟新的 ASP.NET Web Api 專案時，一如往常流暢地完成設定，過程中該卡的一樣也沒漏掉：application pool 要從 .net 2 改為 .net 4、要啟用 32 bit 相容....，但今天卻多了個不常見的錯誤：找不到 roslyn\csc.exe 檔案，印象中也發生過，今天又遇到了就該紀錄一下

## 錯誤訊息

*   訊息內容

    ```
    Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'. 
    
    Server Error in '/' Application.
    
    Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'. 
    Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. 
    
    Exception Details: System.IO.DirectoryNotFoundException: Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'.
    
    Source Error: 
            
    An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.  
                Stack Trace: 
                [DirectoryNotFoundException: Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'.]
    System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath) +366
    System.IO.FileStream.Init(String path, FileMode mode, FileAccess access, Int32 rights, Boolean useRights, FileShare share, Int32 bufferSize, FileOptions options, SECURITY_ATTRIBUTES secAttrs, String msgPath, Boolean bFromProxy, Boolean useLongPath, Boolean checkHost) +736
    System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share) +65
    Microsoft.CodeDom.Providers.DotNetCompilerPlatform.Compiler.get_CompilerName() +91
    Microsoft.CodeDom.Providers.DotNetCompilerPlatform.Compiler.FromFileBatch(CompilerParameters options, String[] fileNames) +656
    Microsoft.CodeDom.Providers.DotNetCompilerPlatform.Compiler.CompileAssemblyFromFileBatch(CompilerParameters options, String[] fileNames) +186
    System.CodeDom.Compiler.CodeDomProvider.CompileAssemblyFromFile(CompilerParameters options, String[] fileNames) +24
    System.Web.Compilation.AssemblyBuilder.Compile() +950
    System.Web.Compilation.BuildProvidersCompiler.PerformBuild() +10099549
    System.Web.Compilation.ApplicationBuildProvider.GetGlobalAsaxBuildResult(Boolean isPrecompiledApp) +9994932
    System.Web.Compilation.BuildManager.CompileGlobalAsax() +44
    System.Web.Compilation.BuildManager.EnsureTopLevelFilesCompiled() +260
                [HttpException (0x80004005): Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'.]
    System.Web.Compilation.BuildManager.ReportTopLevelCompilationException() +62
    System.Web.Compilation.BuildManager.EnsureTopLevelFilesCompiled() +435
    System.Web.Compilation.BuildManager.CallAppInitializeMethod() +33
    System.Web.Hosting.HostingEnvironment.Initialize(ApplicationManager appManager, IApplicationHost appHost, IConfigMapPathFactory configMapPathFactory, HostingEnvironmentParameters hostingParameters, PolicyLevel policyLevel, Exception appDomainCreationException) +545
                [HttpException (0x80004005): Could not find a part of the path 'C:\WebAPI\bin\roslyn\csc.exe'.]
    System.Web.HttpRuntime.FirstRequestInit(HttpContext context) +9963380
    System.Web.HttpRuntime.EnsureFirstRequestInit(HttpContext context) +101
    System.Web.HttpRuntime.ProcessRequestNotificationPrivate(IIS7WorkerRequest wr, HttpContext context) +254
    ```

*   錯誤訊息截圖

    ![1error](https://user-images.githubusercontent.com/3851540/28531349-2e9f0302-70c9-11e7-86e0-105fdecfd3d0.png)

## 解決方法

1.  設定 `AfterBuild` 事件
    *   修改 `.csproj`

        ```xml
        <Target Name="CopyRoslynFiles" AfterTargets="AfterBuild" Condition="!$(Disable_CopyWebApplication) And '$(OutDir)' != '$(OutputPath)'">
            <ItemGroup>
            <RoslynFiles Include="$(CscToolPath)\*" />
            </ItemGroup>
            <MakeDir Directories="$(WebProjectOutputDir)\bin\roslyn" />
            <Copy SourceFiles="@(RoslynFiles)" DestinationFolder="$(WebProjectOutputDir)\bin\roslyn" SkipUnchangedFiles="true" Retries="$(CopyRetryCount)" RetryDelayMilliseconds="$(CopyRetryDelayMilliseconds)" />
        </Target>
        ```

    *   將 `csc.exe` 複製至正確位置

2.  移除 NuGet 套件

    > 如果確認沒有使用到 Roslyn 相關功能，移除 Roslyn 相關套件是比較快的做法

    *   `Microsoft.Net.Compilers`
    *   `Microsoft.CodeDom.Providers.DotNetCompilerPlatform`
    *   移除 web.config 相關定義

        ```xml
        <system.codedom>
            <compilers>
                <compiler language="c#;cs;csharp" extension=".cs" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.CSharpCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:6 /nowarn:1659;1699;1701"></compiler>
                <compiler language="vb;vbs;visualbasic;vbscript" extension=".vb" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.VBCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:14 /nowarn:41008 /define:_MYTYPE=\&quot;Web\&quot; /optionInfer+"></compiler>
            </compilers>
        </system.codedom>
        ```

## 心得

記得上次遇到這個問題時，嘗試找了一些資料，但還是不知道該怎麼科學化地判斷專案到底需不需要 roslyn，或是缺少 roslyn csc.exe 會失去哪些功能，只能盲目測試，最後找到看似解決問題的方法，但實際上不知道原因，這次也一樣 XD

仍然歸納不出問題發生的條件，似乎只有某些條件下才會出現這個錯誤，只能期待下次手氣好一點可以找到問題發生原因

# 參考資訊

1.  [Could not find a part of the path … bin\roslyn\csc.exe](https://stackoverflow.com/questions/32780315/could-not-find-a-part-of-the-path-bin-roslyn-csc-exe)
2.  [Publish website without roslyn](https://stackoverflow.com/questions/32282880/publish-website-without-roslyn)
3.  [Enabling the .NET Compiler Platform (「Roslyn」) in ASP.NET applications](https://blogs.msdn.microsoft.com/webdev/2014/05/12/enabling-the-net-compiler-platform-roslyn-in-asp-net-applications/)
