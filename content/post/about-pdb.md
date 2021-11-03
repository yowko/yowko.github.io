---
title: "你可能會想知道的關於 .PDB 檔的一些事"
date: 2017-01-01T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Debug","Visual Studio"]
slug: "about-pdb"
aliases:
    - /2017/01/Whta-you-need-to-know-about-pdb-file.html
---
## 你可能會想知道的關於 .PDB 檔的一些事

## .PDB 檔是什麼？

**P**rogram **D**ata**B**ase file 的縮寫

1. 編譯過程中會產生
2. 儲存所有模組 (module) 中符號跟位址的清單、檔案名稱、符號定義的行號
3. 通常在編譯期間從原始碼建立 PDB 文件。它儲存模組中所有符號的列表及位址以及檔案名稱和定義符號的行號。

## .PDB 檔中有什麼資料？

1. 區域變數名稱
2. 程式碼檔案名稱
3. 程式碼行號
4. 程式碼索引

    >會將命令嵌入 pdb 中，可以參考這篇[Source Indexing and Symbol Servers: A Guide to Easier Debugging](http://www.codeproject.com/Articles/115125/Source-Indexing-and-Symbol-Servers-A-Guide-to-Easi)

## 如何確認內容

1. 使用 `Dia2Dump`

    > 可以參考[阿尼文章-.pdb 檔案的內容](http://anferneehardaway.pixnet.net/blog/post/6273453)

    以 Visual Studio 2015 為例
    - 1-1. 開啟 `%ProgramFiles(x86)%\Microsoft Visual Studio 14.0\DIA SDK\Samples\DIA2Dump\DIA2Dump.sln`
    - 1-2. 編譯 (Build)
    - 1-3. 使用 `Dia2Dump.exe`
        - 1-3-1. 可以進到`%ProgramFiles(x86)%\Microsoft Visual Studio 14.0\DIA SDK\Samples\DIA2Dump\x64\Debug\`
        - 1-3-2. 直接指定 exe "C:\Program Files (x86)\Microsoft Visual Studio 14.0\DIA SDK\Samples\DIA2Dump\x64\Debug\Dia2Dump.exe"
    - 1-4. `Dia2Dump.exe -?` 可以查看命令參數

        ![dia2dump_question](https://cloud.githubusercontent.com/assets/3851540/21678288/1eeba552-d379-11e6-90ff-6f3a4465e442.png)

    - 1-5. 參數

        ```cmd
        usage: Dia2Dump.exe [ options ] <filename>
        ```

        |參數|說明
        ---|---
        `-?`|print this help
        `-all`| print all the debug info
        `-m`| print all the mods
        `-p`|print all the publics
        `-g`|print all the globals
        `-t`|print all the types
        `-f`|print all the files
        `-s`| print symbols
        `-l [RVA [bytes]]`| print line number info at RVA address in the bytes range
        `-c`| print section contribution info
        `-dbg`| dump debug streams
        `-injsrc [file]`| dump injected source
        `-sf`| dump all source files
        `-oem`| dump all OEM specific types
        `-fpo [RVA]`| dump frame pointer omission information for a func addr
        `-fpo [symbolname]`|dump frame pointer omission information for a func symbol
        `-compiland [name]`|dump symbols for this compiland
        `-lines <funcname>`|dump line numbers for this function
        `-lines <RVA>`|dump line numbers for this address
        `-type <symbolname>`| dump this type in detail
        `-label <RVA>`|dump label at RVA
        `-sym <symbolname> [childname]`|dump child information of this symbol
        `-sym <RVA> [childname]`|dump child information of symbol at this addr
        `-lsrc  <file> [line]`|dump line numbers for this source file
        `-ps <RVA> [-n <number>]`|dump symbols after this address, default 16
        `-psr <RVA> [-n <number>]`|dump symbols before this address, default 16
        `-annotations <RVA>`| dump annotation symbol for this RVA
        `-maptosrc <RVA>`|dump src RVA for this image RVA
        `-mapfromsrc <RVA>`|dump image RVA for src RVA
    - 1-6. 取得所有內容

        ```cmd
        "%ProgramFiles(x86)%\Microsoft Visual Studio 14.0\DIA SDK\Samples\DIA2Dum
        p\x64\Debug\Dia2Dump.exe" -all C:\tt.pdb
        ```

        ![PDB_ALL](https://cloud.githubusercontent.com/assets/3851540/21678292/1f0a892c-d379-11e6-86fc-37ed9890400a.png)

        - 有 source code file 位置跟行號資訊

            ![pdb_file_lines](https://cloud.githubusercontent.com/assets/3851540/21678294/1f0ef408-d379-11e6-998f-49e2b2a244e6.png)

## PDB 如檔如何被載入

程式碼編譯成 dll 或是 exe 的二進位檔(binary)時，會將 PDB 路徑存入 binary 中 ，並產生一組 `GUID` 與 `遞增 Age` 同時封裝進 binary 及 PDB 中，
透過比對 `GUID` 與 `Age` 就可以識別 binary 與 PDB 版本是否相同(如果程式碼未異動，重新編譯的過程仍會重新產生 `GUID` 及  `遞增 Age` 並封裝，因此可能造成 PDB 無法使用)

1. 執行階段

    >pdb 需與 執行檔在同一個資料夾中，才會直接出現錯誤的行號

2. debug the program 載入順序
    - 2-1. 二進位檔中 (exe or dll) 中所封裝的 PDB 路徑
    - 2-2. 二進位檔的所在路徑中(只能在同資料夾中)

        ![error](https://cloud.githubusercontent.com/assets/3851540/21678290/1f047546-d379-11e6-8b65-dc203a96844c.png)

        ![debug1](https://cloud.githubusercontent.com/assets/3851540/21678285/1ee5265a-d379-11e6-92ca-b4ad0029ecae.png)

        ![debug](https://cloud.githubusercontent.com/assets/3851540/21678284/1ee3be82-d379-11e6-94a5-5a84bbb9273d.png)

    - ***修改 .pdb 檔名就會對應不到***

        ![notfound](https://cloud.githubusercontent.com/assets/3851540/21678291/1f07db5a-d379-11e6-8316-a7a0b5fbc416.png)

3. Visual Studio Debugger 載入順序
    - 3-1. 二進位檔中 (exe or dll) 中所封裝的 PDB 路徑
    - 3-2. 二進位檔的所在路徑中(只能在同資料夾中)
    - 3-3. 組件(assembly)載入的路徑
    - 3-4. 符號儲存快取
    - 3-5. 符號儲存的路徑

    > **二進位檔中 (exe or dll) 中所封裝的 PDB 路徑**
    >
    >![pdbpathinbinary](https://cloud.githubusercontent.com/assets/3851540/21678297/1f267d80-d379-11e6-8861-3e3164f236df.png)

## Visual Studio 設定符號位置

1. Tools --> Options..

    ![toolsoption](https://cloud.githubusercontent.com/assets/3851540/21678283/1ee1ce92-d379-11e6-883f-a9f247d12994.png)

2. Debugging --> Symbols

    ![debugging](https://cloud.githubusercontent.com/assets/3851540/21678287/1ee98b28-d379-11e6-89d6-95e22777e715.png)

3. Symbol file(.pdb) locations

    ![location](https://cloud.githubusercontent.com/assets/3851540/21678293/1f0e23ac-d379-11e6-9a26-2fa0d5eb2949.png)

## 原始檔已被修改

- 會造成 debug 位置不在正確的位置上

    ![pdbmismatch](https://cloud.githubusercontent.com/assets/3851540/21678295/1f10541a-d379-11e6-97e7-12a988285a91.png)

    ![debuglocationerror](https://cloud.githubusercontent.com/assets/3851540/21678286/1ee86d74-d379-11e6-83cb-88758f1f1ec9.png)

## 手動產出 .pdb

```cmd
"%ProgramFiles(x86)%\MSBuild\14.0\bin\csc.exe" /out:D:\My.exe /debug /pdb:C:\tt.pdb "C:\Program.cs"
```

- 可以透 `csc` 指令來指定 binary 跟 pdb 的位置

## Build Setting 的影響

1. `none`

    > 不會產生 `pdb`

2. `full`

    > 有`pdb`，且會產生可偵錯的程式碼(會產生 DebuggableAttribute 來通知 JIT 編譯器有可用的偵錯資訊)

3. `pdb-only`

    > 僅產生 `pdb` ，但無法偵錯

## 參考資料

1. [MSDN](https://msdn.microsoft.com/zh-tw/library/ms241903%28v=vs.100%29.aspx)
2. [Know Program Database file (PDB)](http://www.codeproject.com/Articles/349076/Know-Program-Database-file-PDB)
3. [Command-line Building With csc.exe](https://msdn.microsoft.com/zh-tw/library/78f4aasd.aspx)
4. [Source Indexing and Symbol Servers: A Guide to Easier Debugging](http://www.codeproject.com/Articles/115125/Source-Indexing-and-Symbol-Servers-A-Guide-to-Easi)
5. [How to Inspect the Content of a Program Database (PDB) File](http://www.codeproject.com/Articles/37456/How-To-Inspect-the-Content-of-a-Program-Database-P)
6. [PDB Files: What Every Developer Must Know](http://devcenter.wintellect.com/jrobbins/pdb-files-what-every-developer-must-know)
7. [MATCHING DEBUG INFORMATION](http://www.debuginfo.com/articles/debuginfomatch.html)
8. [Advanced .NET Debugging - PDBs and Symbol Stores](http://www.lionhack.com/2014/01/14/advanced-dotnet-debugging-pdbs-and-symbols/)
