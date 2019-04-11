---
title: "如何使用 NUnit3 console 在沒有 Visual Studio 的環境下執行 NUnit 測試"
date: 2017-04-16T18:11:00+08:00
lastmod: 2018-09-17T00:42:22+08:00
draft: false
tags: ["NUnit","Tools","Unit Test"]
slug: "nunit3-console"
aliases:
    - /2017/04/nunit3-console.html
---
# 如何使用 NUnit3 console 在沒有 Visual Studio 的環境下執行 NUnit 測試
一般情況下我們都是透過 Visual Studio 來開發測試程式，但並非所有環境都有 Visula Studio 像是 CI Server 一般都不會安裝 Visual Studio，這時候我們就可以透過 NUnit console 來執行測試，就讓我們來看看可以怎麼使用吧

## 基礎環境

1.  測試框架：NUint 3

    > Visula Studio 中的相關使用可以參考 [Visual Studio 2015 如何產生 NUnit 或 xUnit 的測試專案](//blog.yowko.com/2017/02/visual-studio-2015-use-nunit-xunit.html)

2.  測試範例：使用 ASP.NET WebApi 預設專案範本(包含測試)

    ```cs
    [Test()]
    public void ValuesController_Get_Return_value1_value2()
    {
        // Arrange
        ValuesController controller = new ValuesController();
                    // Act
        IEnumerable<string> result = controller.Get();
                    // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(2, result.Count());
        Assert.AreEqual("value1", result.ElementAt(0));
        Assert.AreEqual("value2", result.ElementAt(1));
    }
    ```
3.  測試工具：NUnit console

    > 請至 [NUnit download](https://www.nunit.org/index.php?p=download) 下載

## NUnit console 指令及參數

* 指令

    > `NUNIT3-CONSOLE [inputfiles] [options]`

    *   inputfiles
        
        >  一個以上的 dll 或是 測試專案
    *   options

        >   任意數量的選項

*   參數

    選項|	說明
    :---|:---
    @FILE|	指定內容是 command-line 參數的檔案. 每一行都是一個 command-line 參數
    --test=NAMES|	指定要執行的測試名稱，以 `,` 分隔，可以使用 `--where` 來代替
    --testlist=FILE|	指定內容是要執行測試的檔案,一行是一個測試.
    --where=EXPRESSION|	使用運算式來過濾測試對象，指定 test names, classes, methods, catgories or properties 的比較運算子是 `==`, `!=`, `=~` and `!~`. 關於運算式詳細說明可以參考 Test Selection Language.
    --params｜p=PARAMETER|	指定測試所需參數．多個參數可以 1.使用 `;` 分隔 2.重複多次 --params e.g. --params=env=prd
    --config=NAME|	指定專案 configuration (e.g.: Debug).
    --process=PROCESS|	執行緒隔離方式：`Single`, `Separate`, `Multiple`. 預設單一 dll 使用 `Separate` ;多個 dll 則是 `Multiple`. 預設情況執行緒都會平行執行.
    --inprocess|	與 `--process=Single` 相同
    --agents=NUMBER|	指定可以並行執行的測試 agent.預設啟動所以測試 agent 一起執行. 這個設定是給平行執行用的.
    --domain=DOMAIN|	dll 的隔離方式：`None`, `Single`, `Multiple`.預設單一 dll 使用 `Single`;多個 dll 則是 `Multiple`
    --framework=FRAMEWORK|	指定測試執行 type/version .Examples: mono, net-4.5, v4.0, 2.0, mono-4.0
    --x86|	在 64 位元系統上使用 32 位元執行緒來進行測試
    --dispose-runners|	在每個 runner 執行完成後 dispose
    --timeout=MILLISECONDS|	設定每個 test case 的執行逾時時間(毫秒)
    --seed=SEED|	設定重複測試次數
    --workers=NUMBER|	指定 worker threads 數量. 用來控制平行測試預設是 cpu 數
    --stoponerror|	如果測試失敗或是出現錯就立即停止
    --skipnontestassemblies|	忽略所有非測試的 dll，不會吐錯誤
    --debug|	測試前先啟動 debugger，用來測試需 attach 在其他 process 的情境
    --debug-agent|	只有使用 debug build 產生的 nunit console 可以使用，可以用來 debug nunit console
    --pause|	讓 NUnit 開啟一個訊息視窗，可以 attach debugger 主要用在 --debug 無法正常運作時
    --wait|	不立即關閉 console
    --work=PATH|	output 檔案的資料夾
    --output, --out=PATH|	測試中輸出文字的內容儲存檔的案路徑
    --err=PATH|	測試紀錄錯誤的檔案路徑
    --result=SPEC|	可以用來設定測試結果檔名，也可以設定測試結果的種類：`nunit3`、`nunit2`。e.g. --result=b.xml;format=nunit2
    --explore[=SPEC]|	僅顯示或儲存測試資訊息，不進行測試。可指定輸出 SPEC： `nunit3`,`cases`。`未指定` 會直接將資訊顯示在 console 上，指定 SPEC 會拿來當做儲存的 file name e.g. --explore=case.xml;format=cases
    --noresult|	不儲存任何測試結果
    --labels=VALUE|	是否將 tese case 名稱輸出：`Off`, `On`, `All`，我測試起來結果都一樣
    --trace=LEVEL|	設定內部 trace level：`Off`, `Error`, `Warning`, `Info`, `Verbose (Debug)`
    --encoding=CODEPAGE|	指定 console CODEPAGE。除非輸出包含特殊字符，否則通常不需要此選項。
    --shadowcopy|	通知 .NET 將載入的 dll 複製到shadowcopy目錄。
    --teamcity|	使用 TeamCity 服務訊息
    --loaduserprofile|	將 user 設定載入至獨立的測試 process 中。
    --list-extensions|	列出所有擴展點和每個擴展點上安裝的擴展名。 Lists all extension points and the extensions installed on each of them.這個不知道在幹嘛XD
    --set-principal-policy=POLICY|	設定測試 domain 的主要策略：`UnauthenticatedPrincipal`，`NoPrincipal`，`WindowsPrincipal`
    --noheader, --noh|	不顯示 NUnit console 本身相關資訊
    --nocolor, --noc|	console 無顏色輸出
    --help, -h|	顯示參數說明


## 心得

因為 NUnit 2 與 NUnit 3 有不少的改變，所以在 NUnit console 的參數上也有不少異動，一般我們是不會只使用 NUnit console 來進行測試，這只是為了與 CI 整合的前置作業

# 參考資訊

1.  [NUnit download](https://www.nunit.org/index.php?p=download)
2.  [Console Command Line](https://github.com/nunit/docs/wiki/Console-Command-Line)
3.  [What is the difference between nunit3 xml format and nunit2 xml format?](http://stackoverflow.com/questions/39333023/what-is-the-difference-between-nunit3-xml-format-and-nunit2-xml-format)
