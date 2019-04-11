---
title: "Test Framework 套件不好用嗎？！ 自己做一個囉"
date: 2017-05-29T18:08:00+08:00
lastmod: 2018-09-22T17:08:25+08:00
draft: false
tags: ["NUnit","Unit Test","Visual Studio"]
slug: "customize-test-framework-project-template"
aliases:
    - /2017/05/customize-test-framework-project-template.html
---
# Test Framework 套件不好用嗎？！ 自己做一個囉
前幾天在測試不同 Test Framework 特性時發現 NuGet 上 Visual Studio 2017 沒有 xUnit.net 2.0 的專案範本套件 ([在 Visual Studio 2017 中安裝其他 Test Framework - NUnit](//blog.yowko.com/2017/05/visual-studio-2017-test-framework-nunit.html))，所以就自己做了一個，使用上自己覺得很方便，詳情可以參考 [在 Visual Studio 2017 中安裝其他 Test Framework - xUnit.net 2.0](//blog.yowko.com/2017/05/visual-studio-2017-test-framework-xunit.html)。

完成了 xUnit 後，突然想起安裝 NUnit 時需要自行安裝

1.  Test Generator NUnit extension

    > 用來建立 unit test 的專案範本

2.  NUnit 3 Test Adapter

    > 用來執行 NUnit 測試

自行安裝不是很困難，只是使用上有些不便，那就來做個自己覺得好用的吧！順便紀錄一下過程

目標：

1.  只安裝需要的版本(不要同時裝 NUnit2 跟 NUnit3)
2.  不需自行手動安裝 test adapter


接著會以建立 NUnit3 的測試套件為例(有特別註記的步驟，可能有改善或是省略的空間)

## 建立空方案(Empty Solution)

> 主要是為了套件命名用，等等會加入兩個專案
> 
> 1.  Unit Test 的專案範本
> 2.  Visual Studio 封裝專案

1.  Visual Studio 主選單 File --> New --> Project...

    ![1newproject](https://cloud.githubusercontent.com/assets/3851540/26543095/1c4bd236-448f-11e7-979b-aedfd5c3f020.png)

2.  搜尋 `empty` --> Blank Solution --> 給個名字

    ![2blanksolution](https://cloud.githubusercontent.com/assets/3851540/26543096/1c522532-448f-11e7-8c65-4b8f3d949250.png)

3.  建立完成

    ![3created](https://cloud.githubusercontent.com/assets/3851540/26543097/1c52d0a4-448f-11e7-8b11-f2b1238efbe5.png)

## 建立 Unit Test 專案範本

1.  方案上按 右鍵 --> Add --> New Project...

    ![4newproject](https://cloud.githubusercontent.com/assets/3851540/26543098/1c554046-448f-11e7-9a09-52576820f960.png)

2.  建立空專案 (Empty Project)

    *   搜尋 `empty` --> 選 `Empty Project(.Net Framework)` --> 給個名字

        ![5emptyproject](https://cloud.githubusercontent.com/assets/3851540/26543099/1c565292-448f-11e7-8591-a7d582cf7dc5.png)

3.  刪除專案中的 `App.config`

    > 這個專案是用來定義如何建立測試專案的範本，用不到

    *   `App.config` 上按右鍵 --> Delete

        ![6delconfig](https://cloud.githubusercontent.com/assets/3851540/26543100/1c59a668-448f-11e7-9b65-9de5510b5901.png)

4.  加入專案參考

    ![7addref](https://cloud.githubusercontent.com/assets/3851540/26543101/1c70f5ca-448f-11e7-9c5a-ec6fb151ce42.png)

    ![8addref2](https://cloud.githubusercontent.com/assets/3851540/26543102/1c79a832-448f-11e7-8590-45924cc97078.png)

    > 參考的部份，版本我沒有特別實驗，僅附上個人使用版本供參考


    |Name|Version|Path|
    |--- |--- |--- |
    |envdte.dll|8.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\envdte.dll|
    |envdte80.dll|8.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\envdte80.dll|
    |Microsoft.VisualStudio.Shell.Interop.dll|7.1.40304.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\Microsoft.VisualStudio.Shell.Interop.dll|
    |Microsoft.VisualStudio.Shell.Interop.8.0.dll|8.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\Microsoft.VisualStudio.Shell.Interop.8.0.dll|
    |Microsoft.VisualStudio.TestPlatform.TestGeneration.dll|15.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestGeneration\Microsoft.VisualStudio.TestPlatform.TestGeneration.dll|
    |Microsoft.VisualStudio.TestPlatform.TestGeneration.dll|15.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestGeneration\Microsoft.VisualStudio.TestPlatform.TestGeneration.dll|
    |Microsoft.VisualStudio.TestPlatform.TestGeneration.Package.dll|15.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestGeneration\Microsoft.VisualStudio.TestPlatform.TestGeneration.Package.dll|
    |System.ComponentModel.Composition.dll|4.0.0.0|C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework.NETFramework\v4.6.2\System.ComponentModel.Composition.dll|
    |VSLangProj.dll|7.0.3300.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\VSLangProj.dll|
    |VSLangProj80.dll|8.0.0.0|c:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\PublicAssemblies\VSLangProj80.dll|


5.  修改專案 property

    *   將 Assembly name 與 Default namespace 修改為方案名稱

        > 這個步驟我是參考 [nunit3-vs-adapter](https://github.com/nunit/nunit3-vs-adapter) 的寫法，沒有特別嘗試沒寫行不行

    *   將 Output type 改為 `Class Library`

        ![10changeproperty](https://cloud.githubusercontent.com/assets/3851540/26543104/1c7c2daa-448f-11e7-900f-1cc45ea42ade.png)

        ![11changeproperty2](https://cloud.githubusercontent.com/assets/3851540/26543103/1c7bd85a-448f-11e7-9ea5-499c78c1c27a.png)

6.  加上 NuGet 套件 `Microsoft.VSSDK.BuildTools`

    ![9addvssdk](https://cloud.githubusercontent.com/assets/3851540/26543105/1c811054-448f-11e7-967d-d33aa91baa7b.png)

7.  加入 `NUnit3UnitTestProjectManager.cs`

    > 用來定義 test project 產生時所需的 name space --> 會在 class 上加上 using framework namespace

    *   加上 using

        ```cs
        using System;
        using EnvDTE;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Model;
        ```

    *   繼承 `UnitTestProjectManagerBase`

        ```cs
        /// <summary>
        /// A unit test project for NUnit3 unit tests.
        /// </summary>
        public class NUnit3UnitTestProjectManager : UnitTestProjectManagerBase
        {
            /// <summary>
            /// Initializes a new instance of the <see cref="NUnit3UnitTestProjectManager"/> class.
            /// </summary>
            /// <param name="serviceProvider">The service provider to use to get the interfaces required.</param>
            /// <param name="naming">The naming object used to decide how projects, classes and methods are named and created.</param>
            public NUnit3UnitTestProjectManager(IServiceProvider serviceProvider, INaming naming): base(serviceProvider, naming)
            {
            }
            /// <summary>
            /// Returns the full namespace that contains the test framework code elements for a given source project.
            /// </summary>
            /// <param name="sourceProject">The source project.</param>
            /// <returns>The full namespace that contains the test framework code elements.</returns>
            public override string FrameworkNamespace(Project sourceProject) => "NUnit.Framework";
        }
        ```

8.  加入 `NUnit3UnitTestClassManager.cs`

    > 用來定義 test class 生出的預設長相

    *   using

        ```cs
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Data;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Model;
        ```

    *   繼承 `UnitTestClassManagerBase`

        ```cs
        /// <summary>
        /// A unit test class for NUnit unit tests.
        /// </summary>
        public class NUnit3UnitTestClassManager : UnitTestClassManagerBase
        {
            /// <summary>
            /// Initializes a new instance of the <see cref="NUnit3UnitTestClassManager"/> class.
            /// </summary>
            /// <param name="configurationSettings">The configuration settings object to be used to determine how the test method is generated.</param>
            /// <param name="naming">The object to be used to give names to test projects.</param>
            public NUnit3UnitTestClassManager(IConfigurationSettings configurationSettings, INaming naming): base(configurationSettings, naming)
            {
            }
            /// <summary>
            /// The attribute name for marking a class as a test class.
            /// </summary>
            public override string TestClassAttribute => "TestFixture";
            /// <summary>
            /// The attribute name for marking a method as a test.
            /// </summary>
            public override string TestMethodAttribute => "Test";
            /// <summary>
            /// The code to force a test failure.
            /// </summary>
            public override string AssertionFailure => "Assert.Fail()";
        }
        ```

9.  加入 `NUnit3SolutionManager.cs`

    > 指定所需相依套件

    *   using

        ```cs
        using System;
        using EnvDTE;
        using EnvDTE80;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Data;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Logging;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Model;
        using VSLangProj80;
        ```

    *   繼承 `SolutionManagerBase`

        ```cs
        public class NUnit3SolutionManager : SolutionManagerBase
        {
            /// <summary>
            /// Initializes a new instance of the <see cref="NUnit3SolutionManager"/> class.
            /// </summary>
            /// <param name="serviceProvider">The service provider to use to get the interfaces required.</param>
            /// <param name="naming">The naming object used to decide how projects, classes and methods are named and created.</param>
            /// <param name="directory">The directory object to use for directory operations.</param>
            public NUnit3SolutionManager(IServiceProvider serviceProvider, INaming naming, IDirectory directory): base(serviceProvider, naming, directory)
            {
            }
            /// <summary>
            /// Performs any preparatory tasks that have to be done after a new unit test project has been created.
            /// </summary>
            /// <param name="unitTestProject">The <see cref="Project"/> of the unit test project that has just been created.</param>
            /// <param name="sourceMethod">The <see cref="CodeFunction2"/> of the source method that is to be unit tested.</param>
            protected override void OnUnitTestProjectCreated(Project unitTestProject, CodeFunction2 sourceMethod)
            {
                if (unitTestProject == null)
                {
                    throw new ArgumentNullException(nameof(unitTestProject));
                }
                TraceLogger.LogInfo("NUnitSolutionManager.OnUnitTestProjectCreated: Adding reference to NUnit assemblies through nuget.");
                
                base.OnUnitTestProjectCreated(unitTestProject, sourceMethod);
                this.EnsureNuGetReference(unitTestProject, "NUnit", null);
                this.EnsureNuGetReference(unitTestProject, "NUnit3TestAdapter", null);
                var vsp = unitTestProject.Object as VSProject2;
                var reference = vsp?.References.Find(GlobalConstants.MSTestAssemblyName);
                if (reference != null)
                {
                    TraceLogger.LogInfo("NUnitSolutionManager.OnUnitTestProjectCreated: Removing reference to {0}", reference.Name);
                    reference.Remove();
                }
            }
        }
        ```

10.  加入 `NUnit3FrameworkProvider.cs`

    > 提供 create unit test 選單的資訊

    *   using

        ```cs
        using System;
        using System.ComponentModel.Composition;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Data;
        using Microsoft.VisualStudio.TestPlatform.TestGeneration.Model;
        ```

    *   繼承 `FrameworkProviderBase`

        ```cs
        /// <summary>
        /// The provider for the NUnit 3 unit test framework.
        /// </summary>
        [Export(typeof(IFrameworkProvider))]
        public class NUnit3FrameworkProvider : FrameworkProviderBase
        {
            /// <summary>
            /// Initializes a new instance of the <see cref="NUnit3FrameworkProvider"/> class.
            /// </summary>
            /// <param name="serviceProvider">The service provider to use to get the interfaces required.</param>
            /// <param name="configurationSettings">The configuration settings object to be used to determine how the test method is generated.</param>
            /// <param name="naming">The naming object used to decide how projects, classes and methods are named and created.</param>
            /// <param name="directory">The directory object to use for directory operations.</param>
            [ImportingConstructor]
            public NUnit3FrameworkProvider(IServiceProvider serviceProvider, IConfigurationSettings configurationSettings, INaming naming, IDirectory directory): base(new NUnit3SolutionManager(serviceProvider, naming, directory), new NUnit3UnitTestProjectManager(serviceProvider, naming), new NUnit3UnitTestClassManager(configurationSettings, naming))
            {
            }
            /// <summary>
            /// Gets the name of the provider.
            /// </summary>
            public override string Name => "NUnit3";
            /// <summary>
            /// Gets the name of the assembly.
            /// </summary>
            public override string AssemblyName => "nunit.framework";
        }
        ```

## 以方案名稱加入 VSIX Project

1.  方案上按 右鍵 --> Add --> New Project...

    ![4newproject](https://cloud.githubusercontent.com/assets/3851540/26543098/1c554046-448f-11e7-9a09-52576820f960.png)

2.  Extensibility --> VSIX Project --> 填入方案名稱

    ![12addvsix](https://cloud.githubusercontent.com/assets/3851540/26543106/1c81f352-448f-11e7-8ab1-444cbbed8243.png)

    *   看不到 `Extensibility` 時請重新執行 Visual Studio 2017 安裝程式，並加入 Visual Studio extension development 開發套件

        ![13noextension](https://cloud.githubusercontent.com/assets/3851540/26543107/1c9a3200-448f-11e7-8b65-8c563d707d6b.png)

        ![14noextension2](https://cloud.githubusercontent.com/assets/3851540/26543089/1c24dd48-448f-11e7-8e5f-7afb04de7203.png)

        ![15noextension3](https://cloud.githubusercontent.com/assets/3851540/26543092/1c29382a-448f-11e7-8f3e-a59f0e3088a9.png)

3.  移除 `index.html`、`stylesheet.css`

    > 預設專案範本中的這兩個檔案用不到

4.  編輯 source.extension.vsixmanifest

    *   Metadata

        > 填寫套件相關資訊，如果沒有要上架 marketplace，可以輕鬆填(個人經驗分享：檔案請用選的不要用 key 的，我卡這問題好久XD)

        ![17metadata](https://cloud.githubusercontent.com/assets/3851540/26543090/1c266820-448f-11e7-954c-8a2c2c8ef488.png)

    * Install Targets

        > 指定安裝對象

        ![18installtarget](https://cloud.githubusercontent.com/assets/3851540/26543091/1c27ab72-448f-11e7-835c-fbf2b393b9f4.png)

    *   Assets

        > 指定專案資訊，其中的 Type 無法用選擇的，需要直接複製貼上 `Microsoft.VisualStudio.TestGenerationExtension`

        ![19assets](https://cloud.githubusercontent.com/assets/3851540/26543093/1c2c2fa8-448f-11e7-8562-25bd31071596.png)

5.  如果沒有上架需求，直接 build 專案後，可以從 bin/debug or release 資料夾中找到 `.vsix` 檔即可用來安裝

6.  如果有上架需要請參考 [如何發行 Visual Studio 專案範本(project Template)](//blog.yowko.com/2016/12/publish-visual-studio-project-template.html)


## 測試成品

![16preview](https://cloud.githubusercontent.com/assets/3851540/26543094/1c2c3a98-448f-11e7-8f57-207f19464bde.png)

## 其他資訊

~~完整程式碼請參考 [NUnit3.TestGenerator](https://github.com/yowko/NUnit3.TestGenerator)~~

# 參考資訊

1.  [在 Visual Studio 2017 中安裝其他 Test Framework - NUnit](//blog.yowko.com/2017/05/visual-studio-2017-test-framework-nunit.html)
2.  [在 Visual Studio 2017 中安裝其他 Test Framework - xUnit.net 2.0](//blog.yowko.com/2017/05/visual-studio-2017-test-framework-xunit.html)
3.  [nunit3-vs-adapter](https://github.com/nunit/nunit3-vs-adapter)
4.  [如何發行 Visual Studio 專案範本(project Template)](//blog.yowko.com/2016/12/publish-visual-studio-project-template.html)
5. ~~[NUnit3.TestGenerator](https://github.com/yowko/NUnit3.TestGenerator)~~
