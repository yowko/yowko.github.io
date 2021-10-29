---
title: "PowerShell 基本語法筆記 2"
date: 2017-03-14T02:42:34+08:00
lastmod: 2021-10-28T00:42:34+08:00
draft: false
tags: ["PowerShell"]
slug: "powershell-operator-pipeline-xml"
aliases:
    - /2017/03/powershell-operator-pipeline-xml.html
---
## PowerShell 基本語法筆記 2

自從開始使用 PowerShell 來與 Jenkins 2 整合進行 CI 與 CD 的流程之後，有愈來愈多需求出現，原本的文章 [powershell 基本語法筆記](/powershell-basic-syntex) 的內容無法負荷，因此補充一下其他語法

## if 判斷式

- 比較運算子需加上 `-` 來表示

    比較運算子|說明
    :---|:---
    -eq|相等
    -ne|不相等
    -ge|大於等於
    -gt|大於
    -lt|小於
    -le|小於等於
    -like|萬用字元比對，有特定字元
    -notlike|萬用字元比對，排除特定字元
    -match|Regular expression 比對，選擇符合的
    -notmatch|Regular expression 比對，排除符合的

  - 預設比較不分大小寫
  - 可以在比較子前加上 `c` 強制區分大小寫
  - 範例
    `if( 1 -eq 2 )`

- 邏輯運算子需加上 `-` 來表示

    邏輯運算子|說明
    :---|:---
    -and| and
    -or | or
    -not<br/>!| not
    -xor|xor

  - 範例
    `if( (1 -eq 1) -or (2 -eq 2))`

## 資料夾檔案處理

- 取得資料夾下物件

    >`Get-ChildItem -Path $folder`

- 取得資料夾所有物件(包含子資料夾)

    >`Get-ChildItem -Path $folder –Recurse`

- 取得資料夾特定類型檔案(*.txt)

    >`Get-ChildItem -Path $folder -Filter *.txt –Recurse` 

## pipeline

> - 可以持續串連
> - 製造順序性作業

- 過濾資料(e.g.過濾檔名為 'A' 開頭的 json 檔)

    >`Get-ChildItem -Path $folder -Filter *.json –Recurse| Where-Object {$_.Name.StartsWith("A")`

- 輸出資料(e.g.輸出物件內容)

    >`Get-ChildItem -Path $folder | Out-Host`

- 處理資料(e.g.取得檔名)

    > `Get-ChildItem -Path $folder -Filter *.json –Recurse| ForEach-Object -Process {$_.BaseName}`

## parse xml

- 範例 xml

    ```xml
    <YowkoProfile>
        <Name>yowko</Name>
        <Yowko.Tel>0912345678</Yowko.Tel>
        <Accounts>
            <Site>
                <Url>http://tw.yahoo.com/</Url>
                <UserName>yowkotsai</UserName>
                <Password>pa$$w0rd</Password>
            </Site>
            <Site>
                <Url>http://www.google.com/</Url>
                <UserName>yowkotsai</UserName>
                <Password>pa$$w0rd</Password>
            </Site>
        </Accounts>
    </YowkoProfile>
    ```

- 將檔案轉為 xml object

    > `[xml]$xmldata = get-content "C:\yowkoprofile.xml"`

- 依階層取值
  - Name

    ```ps1
    $xmldata.YowkoProfile.Name
    ```

- element 中有 `.`，使用 `''`來取值
  - Yowko.Tel

    ```ps1
    $xmldata.YowkoProfile.'Yowko.Tel'
    ```

- 使用 index 來取值
  - Site

    ```ps1
    $xmldata.YowkoProfile.Accounts.Site[0]
    ```

  - Url

    ```ps1
    $xmldata.YowkoProfile.Accounts.Site[0].Url
    ```

## 參考資訊

1. [Comparison Operators](https://ss64.com/ps/syntax-compare.html)
2. [POWERSHELL: PARSING XML PART 1](http://www.thomasmaurer.ch/2010/06/powershell-parsing-xml-part-1/)
3. [Understanding the Windows PowerShell Pipeline](https://msdn.microsoft.com/en-us/powershell/scripting/getting-started/fundamental/understanding-the-windows-powershell-pipeline)
