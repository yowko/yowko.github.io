---
title: "如何使用 Powershell 與 robocopy 複製檔案至同一目錄下多個子目錄中"
date: 2017-03-09T01:42:34+08:00
lastmod: 2021-10-28T00:42:34+08:00
draft: false
tags: ["PowerShell"]
slug: "powershell-recursive-robocopy"
aliases:
    - /2017/03/powershell-recursive-robocopy.html
---
## 如何使用 Powershell 與 robocopy 複製檔案至同一目錄下多個子目錄中

同事問到如何將檔案複製至同一個根目錄下的多個子目錄中，原本的做法將有這個檔案的路徑列出來，再用 robocopy 複製檔案過去，但遇到增加新的子目錄就需要將路徑加到部署清單中

## 取出有特定檔案的路徑

>`$filesList =Get-ChildItem -Path {D:\} -Filter {a.txt} –Recurse`

- 列出 `{d:\}` 所有 `{a.txt}` 的路徑

## 將檔案覆蓋至所有特定檔案的路徑中

```ps1
#針對全部有 a.txt 的路徑
foreach($item in $filesList) 
{
    #把根目錄下的 a.txt copy 至有 a.txt 的 folder
    robocopy {$targetFolder} $item.Directory.FullName {a.txt}
}
```

- 取得 $item 所在的 folder path

    >$item.Directory.FullName
- 將 `{$targetFolder}` 下的 `{a.txt}` 複製至 $item.Directory.FullName 中

## 個人無知的做法(僅供參考)

- 先將檔案複製至目標 server 的根目錄
- 其他子目錄的檔案就由根目錄 copy
  - 問題 ：這樣是為了節省 network io，但直覺上還是會佔用本地端資源，也許還更耗資源(從遠端讀取資源進 memory 再 copy 至遠端) --> 這有待驗證  純粹是聯想的問題
- 程式碼如下

    ```ps1
    $sourceFolder='D:\roboSource'
    $targetFolder='\\172.16.1.2\D$\TestRobocopy'
    
    #列出 targetFolder 下全部 a.txt 的路徑
    $filesList =Get-ChildItem -Path $targetFolder -Filter a.txt –Recurse
    
    #根目錄的 a.txt
    $baseFile=$targetFolder+'\a.txt';
    
    #檢查根目錄的 a.txt 是否存在
    if((Test-Path $baseFile) -eq $false)
    {
        #不存在就 copy a.txt 至根目錄
        robocopy $sourceFolder $targetFolder a.txt
    }
    
    #針對所有 a.txt 的路徑
    foreach($item in $filesList) 
    {
        #把根目錄下的 a.txt copy 至有 a.txt 的 folder
        robocopy $targetFolder $item.Directory.FullName a.txt
    }
    ```

## 參考資料

1. [Robocopy commands to copy a file to over 50 remote machines](http://stackoverflow.com/questions/26805835/robocopy-commands-to-copy-a-file-to-over-50-remote-machines)
2. [How to get the Parent's parent directory in Powershell?](http://stackoverflow.com/questions/9725521/how-to-get-the-parents-parent-directory-in-powershell)
3. [Recursive file search using PowerShell](http://stackoverflow.com/questions/8677628/recursive-file-search-using-powershell)
