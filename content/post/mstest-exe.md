---
title: "使用 MSTest.exe 指令來進行測試"
date: 2017-06-22T23:12:00+08:00
lastmod: 2018-09-23T16:11:41+08:00
draft: false
tags: ["MSTest","Unit Test","Visual Studio"]
slug: "mstest-exe"
aliases:
    - /2017/06/mstest-exe.html
---
# 使用 MSTest.exe 指令來進行測試
MSTest.exe 是用於執行 MSTest 測試的命令列命令，功能與 Nunit-console 相同，都是用來讓我們在沒有 Visual Studio 的情境下可以執行測試，像是 CI Server，就讓我們來看看 MSTest.exe 的相關可用參數吧

## 一般參數


|參數|說明|使用方式|
|--- |--- |--- |
|/testcontainer:{file name}|載入包含測試的檔案<br/>可以是測試專案 dll 或是 .orderedtest 檔,<br/>不能與 `/testmetadata` 與混用|/testcontainer:tests.dll|
|/testmetadata:{file name}|載入內含測試 metadata 的檔案<br/>(.vsmdi 檔，由  Test List Editor 建立<br/>但 vs2013 之後已經移除該項功能)|/testmetadata:test.vsmdi|
|/testlist:{test list path}|指定要執行的測試清單 (於 metadata 檔案中指定)<br/>可以指定多次|/testlist:balancetests|
|/category:{test category filter}|指定並篩選要執行的測試分類，需搭配 `/testcontainer` 使用<br/>每次執行只能指定一個 `/category:`<br/>但可以用 `&`、`|`、`!`、`&!`，有篩選條件時才需要加上引號|/category:"Priority1&ShoppingCart"|
|/test:{test name}|指定要執行的測試名稱<br/>可以指定多次，可搭配 `/testcontainer` 或 `/testmetadata`<br/>使用文字比對，部份符合即可|/test:Calculate|
|/noisolation|不另外開啟 process 而是在 MSTest.exe 處理序內執行測試<br/>這個動作會提升測試回合的速度<br/>但是在測試程式碼擲回未處理的例外狀況 (Exception)<br/>可能會導致 MSTest.exe 處理序毀損|/noisolation|
|/testsettings: {file name}|使用指定的測試設定檔|/testsettings:Local.Testsettings|
|/runconfig:{file name}|使用指定的執行 config 檔<br/>這是為了舊版 Visual Studio 的相容性而存在的<br/>新版 Visual Studio 已由 testsettings 取代|/runconfig:localtestrun.Testrunconfig|
|/resultsfile:{file name}|將測試回合結果儲存到指定的檔案中|/resultsfile:testResults.trx|
|/detail:{property id}|指定顯示測試案例的屬性值|/detail:name|
|/help|顯示 MSTest.exe 使用方式訊息(也可以用 `/?` or `/h`)|`/help`、`/?`、`/h`|
|/nologo|不顯示程式啟始資訊及著作權訊息|/nologo|
|/usestderr|使用標準錯誤來輸出錯誤資訊|/usestderr|



## 輸出結果參數

> 必需是 Visual Studio Ultimate 或 Visual Studio Premium 以上版本，且需搭配 Team Foundation Server


|參數|說明|使用方式|
|--- |--- |--- |
|/publish:{server name}|將結果發行至指定之伺服器 Team 專案集合的資料庫|/publish:[http://OurTFSMachine:8080/tfs/OurProjectCollection](http://OurTFSMachine:8080/tfs/OurProjectCollection)|
|/publishresultsfile:{file name}|指定要發行的結果檔檔案名稱。<br/>若未指定結果檔名，則使用目前回合所產生的檔案|/publishresultsfile:test.trx|
|/publishbuild:{build id}|使用這個 build ID 發行測試結果|/publishbuild:build0001|
|/teamproject:{team project name}|指定這個 build 所屬之 Team 專案的名稱|/teamproject:UnitTestCalc|
|/platform:{platform}|指定要發行其測試結果之 build 的平台|/platform:AnyCPU or /platform:x86|
|/flavor:{flavor}|指定要發行其測試結果之 build 的類別|/flavor:debug or /flavor:retail|


## 檔案位置

1.  Visual Studio 2015

    > `%ProgramFiles(x86)%\Microsoft Visual Studio 14.0\Common7\IDE\MSTest.exe`

    *   之前版本只差在版號而已

2.  Visual Studio 2017 - Enterprise

    > `%ProgramFiles(x86)%\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\MSTest.exe`

    *   要留意 VS 的版本

        ![4mstest](https://user-images.githubusercontent.com/3851540/27441097-1f4b8fe4-579f-11e7-9628-55e965a41c3c.png)

## 執行時出現錯誤訊息

1.  訊息內容

    >Index was outside the bounds of the array.
    
2. 錯誤截圖
    
    ![5mismatch](https://user-images.githubusercontent.com/3851540/27441098-1f77fc82-579f-11e7-83cb-d271a32f7144.png)

##  問題發生原因

> 測試對象所使用的 mstest 版本與現在執行的 mstest 版本不一致

*   需要留意的是 Visual Studio 2017 新加入的 MSTestV2 無法使用 Visual Studio 2017 所帶的 mstest.exe 進行測試

## 心得

相關文件最新版本已經是 Visual Studio 2013，沒有找到 Visual Studio 2015 或是 Visual Studio 2017 的版本，Visual Studio 2015 還 OK，使用上沒什麼不同的，但對於 Visual Studio 2017 就比較有問題

1.  檔案位置變了

    > 還算好找

2.  MSTestV2 測試

    > 目前測試功能尚未支援，有使用到的朋友要特別留意

# 參考資訊

1.  [MSTest.exe command-line options](https://msdn.microsoft.com/en-us/library/ms182489.aspx)
2.  [Command-Line options for publishing test results](https://msdn.microsoft.com/en-us/library/ms243151.aspx)
3.  [Why am I getting 「Index was outside the bounds of the array」 when running mstest.exe on the commandline?](https://stackoverflow.com/questions/7285268/why-am-i-getting-index-was-outside-the-bounds-of-the-array-when-running-mstest)
