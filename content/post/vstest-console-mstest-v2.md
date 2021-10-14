---
title: "使用命令列指令 (VSTest.Console.exe) 執行 MSTest V2 測試"
date: 2018-04-09T01:00:00+08:00
lastmod: 2021-10-13T01:00:15+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "vstest-console-mstest-v2"
aliases:
    - /2018/04/vstest-console-mstest-v2.html
---
## 使用命令列指令 (VSTest.Console.exe) 執行 MSTest V2 測試

之前筆記 [使用 MSTest.exe 指令來進行測試](/2017/06/mstest-exe.html) 曾經介紹到使用 MSTest.exe 在 cmmand line 環境中執行測試，筆記結尾有提到未支援 MSTest V2 的測試功能，在原本使用 MSTest V2 專案不多的情況下影響不大，但近期專案為了使用 Live Unit Testing 逐漸改用 Visual Studio 2017 搭配 MSTest V2，除了開發階段方便外，為了讓 CI 充份發揮功能，得另外調整 unit test 的語法，就來看看該如何使用命令列指令執行 MSTest V2 測試

## 關於 `VSTest.Console.exe`

VSTest.Console.exe 是在 Visual Studio 2012 首見的命令列指令，用來取代 Visual Studio 2012 及更新版本的 Visual Studio 中的  MSTest.exe，針對執行效能已做過優化，可以用來進行單元測試及程式碼 UI 測試

VSTest.Console.exe 允許指定不同選項參數，而參數不分大小寫及順序

透過 `VSTest.Console/?` 可以看到指令使用方式及相關參數說明

```txt
Usage: vstest.console.exe [Arguments] [Options] [[--] <RunSettings arguments>...]]

Description: Runs tests from the specified files.

Arguments:

[TestFileNames]
      Run tests from the specified files. Separate multiple test file names
      by spaces.
      Examples: mytestproject.dll
                mytestproject.dll myothertestproject.exe

Options:

--Tests|/Tests:<Test Names>
      Run tests with names that match the provided values. To provide multiple
      values, separate them by commas.
      Examples: /Tests:TestMethod1
                /Tests:TestMethod1,testMethod2

--TestCaseFilter|/TestCaseFilter:<Expression>
      Run tests that match the given expression.
      <Expression> is of the format <property>Operator<value>[|&<Expression>]
         where Operator is one of =, != or ~  (Operator ~ has 'contains'
         semantics and is applicable for string properties like DisplayName).
         Parenthesis () can be used to group sub-expressions.
      Examples: /TestCaseFilter:"Priority=1"
                /TestCaseFilter:"(FullyQualifiedName~Nightly
                                  |Name=MyTestMethod)"

--Framework|/Framework:<Framework Version>
      Target .Net Framework version to be used for test execution.
      Valid values are ".NETFramework,Version=v4.5.1", ".NETCoreApp,Version=v1.0" etc.
      Other supported values are Framework35, Framework40, Framework45 and FrameworkCore10.

--Platform|/Platform:<Platform type>
      Target platform architecture to be used for test execution.
      Valid values are x86, x64 and ARM.

--Settings|/Settings:<Settings File>
      Settings to use when running tests.

RunSettings arguments:
      Arguments to pass runsettings configurations through commandline. Arguments may be specified as name-value pair of the form [name]=[value] after "-- ". Note the space after --.
      Use a space to separate multiple [name]=[value].
      More info on RunSettings arguments support: https://aka.ms/vstest-runsettings-arguments

-lt|--ListTests|/lt|/ListTests:<File Name>
      Lists discovered tests from the given test container.

--Parallel|/Parallel
      Specifies that the tests be executed in parallel. By default up
      to all available cores on the machine may be used.
      The number of cores to use may be configured using a settings file.

--TestAdapterPath|/TestAdapterPath
      This makes vstest.console.exe process use custom test adapters
      from a given path (if any) in the test run.
      Example  /TestAdapterPath:<pathToCustomAdapters>

--Blame|/Blame
      Runs the test in blame mode. This option is helpful in isolating the problematic test causing test host crash. It creates an output file in the current directory as "Sequence.xml", that captures the order of execution of test before the crash.

--Diag|/Diag:<Path to log file>
      Enable verbose logs for test platform.
      Logs are written to the provided file.

--logger|/logger:<Logger Uri/FriendlyName>
      Specify a logger for test results.  For example, to log results into a
      Visual Studio Test Results File (TRX) use /logger:trx [;LogFileName=<Defaults to unique file name>]
      Creates file in TestResults directory with given LogFileName.

      Change the verbosity level for console logger. Allowed values for verbosity: quiet, minimal and normal.
      Example: /logger:console;verbosity=<Defaults to "minimal">

      To publish test results to Team Foundation Server, use TfsPublisher as shown below
      Example: /logger:TfsPublisher;
                Collection=<team project collection url>;
                BuildName=<build name>;
                TeamProject=<team project name>
                [;Platform=<Defaults to "Any CPU">]
                [;Flavor=<Defaults to "Debug">]
                [;RunTitle=<title>]

--ResultsDirectory|/ResultsDirectory
      Test results directory will be created in specified path if not exists.
      Example  /ResultsDirectory:<pathToResultsDirectory>

--ParentProcessId|/ParentProcessId:<ParentProcessId>
      Process Id of the Parent Process responsible for launching current process.

--Port|/Port:<Port>
      The Port for socket connection and receiving the event messages.

-?|--Help|/?|/Help
      Display this usage message.

--Collect|/Collect:<DataCollector FriendlyName>
      Enables data collector for the test run. More info here : https://aka.ms/vstest-collect

--InIsolation|/InIsolation
      Runs the tests in an isolated process. This makes vstest.console.exe
      process less likely to be stopped on an error in the tests, but tests
      may run slower.

@<file>
      Read response file for more options.

  To run tests:
    >vstest.console.exe tests.dll
  To run tests with additional settings such as  data collectors:
    >vstest.console.exe  tests.dll /Settings:Local.RunSettings
```

## `VSTest.Console.exe` 指令

- 指令用法

    ```cmd
    vstest.console.exe [Arguments] [Options] [[--] <RunSettings arguments>...]]
    ```

- 參數(Arguments)
    > 執行指定檔案的測試，可以使用 `空白符號` 當做分隔符號來指定多個檔案

  - 範例

      ```cmd
      vstest.console.exe WebAPITest_MSTestV1.dll  WebAPITest_MSTestV2.dll
      ```

- 選項(Options)

    選項|說明|用法
    :---|:---|:---
    `--Tests:<Test Names>`<br/>`/Tests:<Test Names>`|指定測試方法<br/>使用 `,` 為分隔符號指定多個|`vstest.console.exe WebAPITest_MSTestV2.dll /Tests:TestMethod1`<hr>`vstest.console.exe WebAPITest_MSTestV2.dll --Tests:V2_GetTest,V2_GetByIdTest`
    `--TestCaseFilter:`<br/>`<Expression>`<hr>`/TestCaseFilter:`<br/>`<Expression>`| 執行符合運算式的測試<br/>`<Expression>` 格式是 <br/>`<property>Operator<value>`<br/>`[|&<Expression>]` <br/>其中 Operator 可以用 `=`, `!=` or `~` <br/> `~` 表示 '包含' 可以用在字串屬性上<br/>括號 `()` 可以用來將 `子運算式` 分群.|`vstest.console.exe WebAPITest_MSTestV2.dll /TestCaseFilter:"Priority=1"`<hr>`vstest.console.exe WebAPITest_MSTestV2.dll  --TestCaseFilter:"(Name!=V2_GetByIdTest)｜(Name~id)"`
    `--Framework:Version>`<br/>`/Framework:<Version>`|指定測試的 .Net Framework 版本.<br/>有效的格式為 <br/>`.NETFramework,Version=v4.5.1`,<br/> `.NETCoreApp,Version=v1.0` <br/>也支援 `Framework35`, `Framework40`<br/>, `Framework45`, `FrameworkCore10`.|`vstest.console.exe WebAPITest_MSTestV2.dll /Framework:Framework45`<hr>`vstest.console.exe WebAPITest_MSTestV2.dll --Framework:.NETFramework,Version=v4.5`
    `--Platform:<type>` <br/>`/Platform:<type>`|指定測試平台架構.<br/>有效值為 `x86`、`x64` 和 `ARM`|`vstest.console.exe WebAPITest_MSTestV2.dll /Platform:x86`<hr>`vstest.console.exe WebAPITest_MSTestV2.dll --Platform:x64`
    `--Settings:<File>`<br/>`/Settings:<File>`|指定測試額外設定檔|`vstest.console.exe WebAPITest_MSTestV2.dll /Settings:x86`<hr>`vstest.console.exe WebAPITest_MSTestV2.dll --Settings:x64`

- RunSettings arguments
    
    > 相關設定目前沒用過，先列上當做參考資訊，所有參數都是使用 [name]=[value] 格式來指定，並且接在 vstest.console.exe 命令列最後面，使用 `--` 來串接，以 ` `(空白字元) 當做連接符號,有關 RunSettings 參數的更多可以參考: [vstest-docs/docs/RunSettingsArguments.md](https://github.com/Microsoft/vstest-docs/blob/master/docs/RunSettingsArguments.md)

    選項|說明
    :---|:---
    -lt｜--ListTests｜/lt｜/ListTests:<File Name>|列出指定 container 中的測試
    --Parallel｜/Parallel|使用多核心平行測試<br/>預設值為所有可的 cpu 核心數<br/>cpu 數可以透過設置檔進行設定
    --TestAdapterPath｜/TestAdapterPath|指定 test adapters 的路徑
    --Blame｜/Blame|使用 `blame` 模式進行測試<br/>可以用來隔離導致測試 host crash 的問題測試<br/>會建立 `Sequence.xml` 以紀錄 crash 前的 test 執行順序
    --Diag｜/Diag:<Path to log file>|啟用測試平台的詳細紀錄<br/>log 至指令檔案中
    --logger｜/logger:<Logger Uri/FriendlyName>|將測試結果 log 至指定位置
    --ResultsDirectory｜/ResultsDirectory|在指定路徑中建立測試結果目錄
    --ParentProcessId｜/ParentProcessId:<ParentProcessId>|負責啟動當前 process 的 parent process 的 process Id
    --Port｜/Port:<Port>|通訊端連接並接收事件消息的 port
    -?｜--Help｜/?｜/Help|顯示 help 訊息
    --Collect｜/Collect:<DataCollector FriendlyName>|啟用測試回合的資料收集器<br/>更多資訊在 [HTTPs://aka.ms/vstest-collect](HTTPs://aka.ms/vstest-collect)
    --InIsolation｜/InIsolation|在獨立的 process 中運行測試

## 實際使用

- 前提說明

    > 使用 MSTest v1 與 MSTest v2 分別建立完全相同的測試 

    - MSTest v1

        ```cs
        using Microsoft.VisualStudio.TestTools.UnitTesting;
        using FluentAssertions;
        
        namespace WebAPITest.Controllers.MSTestV1
        {
            [TestClass()]
            public class ValuesControllerTests
            {
                [TestMethod()]
                public void V1_GetTest()
                {
                    ValuesController target = new ValuesController();
                    var expected = new string[] { "value1", "value2" };
        
                    //action
                    var actual = target.Get();
        
                    //assert
                    actual.Should().BeEquivalentTo(expected);
                }
                [TestMethod()]
                public void V1_GetByIdTest()
                {
                    //arrange
                    ValuesController target = new ValuesController();
                    var expected = "value";
                    int id = 0;
        
                    //action
                    var actual = target.Get(id);
        
                    //assert
                    actual.Should().BeEquivalentTo(expected);
                }
            }
        }
        ```

    - MSTest v2

        ```cs
        using Microsoft.VisualStudio.TestTools.UnitTesting;
        using FluentAssertions;
        
        namespace WebAPITest.Controllers.MSTestV2
        {
            [TestClass()]
            public class ValuesControllerTests
            {
                [Priority(1)]
                [TestMethod()]
                public void V2_GetTest()
                {
                    //arrange
                    ValuesController target = new ValuesController();
                    var expected = new string[] { "value1", "value2" };
        
                    //action
                    var actual = target.Get();
        
                    //assert
                    actual.Should().BeEquivalentTo(expected);
                }
                [TestMethod()]
                public void V2_GetByIdTest()
                {
                    //arrange
                    ValuesController target = new ValuesController();
                    var expected = "value";
                    int id = 0;
        
                    //action
                    var actual = target.Get(id);
        
                    //assert
                    actual.Should().BeEquivalentTo(expected);
                }
            }
        }
        ```

- 結果
    - mstest.exe
        - 測試 MSTest v1 : **<span style="color:green">正常</span>**

            >![1mstestv1](https://user-images.githubusercontent.com/3851540/38466808-57a1ce70-3b61-11e8-876b-48e1a21cb2c5.png)

        - 測試 MSTest v2 : **<span style="color:red">無法測試</span>**

            >![2mstestv2](https://user-images.githubusercontent.com/3851540/38466809-57ce130e-3b61-11e8-9dd4-acda7bed6cc8.png)
    - VSTest.console.exe
        - 測試 MSTest v1 : **<span style="color:green">正常</span>**

            >![3vstestv1](https://user-images.githubusercontent.com/3851540/38466810-57fdec82-3b61-11e8-8c26-5ab58b424294.png)

        - 測試 MSTest v2 : **<span style="color:green">正常</span>**

            >![4vstestv2](https://user-images.githubusercontent.com/3851540/38466811-5825cb8a-3b61-11e8-9d30-57feeb0c8562.png)

## 心得

VSTest.console.exe 在基本使用上並不複雜，但相關說明文件就不是很好找，多個來源都有不同的參數列表，對於釐清詳細功能比較不利

>![5replace](https://user-images.githubusercontent.com/3851540/38466807-575d3ed6-3b61-11e8-81b0-acb237654377.png)

以文件說明及結果來看，最好全面改用 VSTest.console.exe 來執行測試：

1. mstest.exe 完全不支援 MSTest v2
2. VSTest.console.exe 在執行 MSTest v1 與 MSTest v2 並沒有出現問題，可直接使用

## 參考資訊

1. [Using VSTest.console from the command line](https://msdn.microsoft.com/en-us/library/jj155800.aspx)
2. [vstest-docs/docs/RunSettingsArguments.md](https://github.com/Microsoft/vstest-docs/blob/master/docs/RunSettingsArguments.md)
3. [vstest-docs/docs/analyze.md](https://github.com/Microsoft/vstest-docs/blob/master/docs/analyze.md)
4. [Configure unit tests by using a .runsettings file](https://msdn.microsoft.com/en-us/library/jj635153.aspx)
5. [使用 MSTest.exe 指令來進行測試](/2017/06/mstest-exe.html)
