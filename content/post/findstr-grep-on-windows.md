---
title: "findstr - Windows 上的 grep (快速搜尋文件)"
date: 2017-01-04T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "findstr-grep-on-windows"
aliases:
    - /2017/01/findstr-windows-grep.html
---
# findstr - Windows 上的 grep (快速搜尋文件)
寫程式難免會遇到暴力搜尋的情境，在史上最強開發 IDE - Visual Studio 的幫助下，一向不是什麼難事，但如果遇到想要找文件時怎麼辦，在 Linux 環境下有 `grep` 指令可以用，在 Windows 上一直都是使用保哥推薦的工具- [grepWin](http://stefanstools.sourceforge.net/grepWin.html), 保哥有簡單的教學-[介紹好用工具：grepWin](http://blog.miniasp.com/post/2008/01/29/Useful-tool-grepWin.aspx), 最近同事則是推薦 Windows 的 `findstr` 語法

## 語法

```
findstr [/b] [/e] [/l] [/r] [/s] [/i] [/x] [/v] [/n] [/m] [/o] [/p] [/offline] [/g:file] [/f:file] [/c:string] [/d:dirlist] [/a:ColorAttribute] [strings] [[Drive:][Path] FileName [...]]
```

## 參數說明
1. `/b` : 搜尋字串在一行的開頭 
    
    >`findstr /b npm \*.\* `

2. `/e` : 搜尋字串在一行的結尾(這個我測試起來，如果字串在檔案的結尾會找不到)
    
    >`findstr /e npm \*.\* `

3. `/l` : 將字串內的符號當作定字串來搜尋
    
    >`findstr /l "\"npm" t1.txt`
    
    ```
    npm
    12354
    "npm 
    ```
    
    >只有 `"npm` 符合條件


4. `/r`   : 使用 `regular expression` 搜尋
    
    >`findstr /r "[^0-9]" t1.txt`

5. `/s`   : 在目前目錄并包含子目錄搜尋
    
    >`findstr /s npm *.* `

6. `/i`   : 忽略大小寫差異
    
    >`findstr /i npm *.* `

7. `/x`   : 把完全符合的那行輸出(不得與 `/m`共用，行需完全符合才會輸出)
    
    >`findstr /x npm *.*`

8. `/v`   : 把不包含的那行內容輸出
    
    >`findstr /v npm *.*`

8. `/n`   : 把符合條件的行數列出
    
    >`findstr /n npm *.*`

8. `/m`   : 如果有找到符合的內容，僅列出檔案名稱
    
    >`findstr /m npm *.*`

8. `/o`   : 印出該行內容前還有幾個字元
    
    >`findstr /o npm *.*`

8. `/p`   : 忽略不可視字元
    
    >`findstr /p npm *.* `

8. `/offline`   : 搜尋帶有`offline` 屬性的檔案。offline 屬性是別台電腦的檔案留存在本機上的複本或是檔案擁有者是別台電腦. offline 屬性可以看 [File attribute of "O" and unusual file icon overlay.](http://answers.microsoft.com/en-us/windows/forum/windows_xp-files/file-attribute-of-o-and-unusual-file-icon-overlay/4e82a9e3-489a-4309-b595-774a7beebb61)

8. `/f: file`   : 使用特定檔案所列的檔案清單來搜尋
    
    >`findstr /f:files.txt npm`

8. `/c: string`   : 使依指定字串內容搜尋
    
    >`findstr /c:"npm install" \*.\*`
    
    - 會將指定的內容視為一個整體，可以搜尋空白

8. `/g: file`   : 搜尋字串由指定的檔案來
    
    >`findstr /g:t1.txt t2.txt`
    
    - 將 t2.txt 中包含 t1.txt 任一行內容的那行內容輸出

8. `/d: dirlist`   : 指定搜尋目錄清單，以`;`分割
    
    >`findstr /D:"C:\Notes\react";"C:\npm" npm  *.* `
    
    - 在 `C:\Notes\react` 及 `C:\npm` 所有檔案中搜尋 `npm`

8. `/a: ColorAttribute`   : 指定輸出顏色(使用16進位顏色).
    
    >`findstr /a:2F npm *.txt`
    
    - 有所有 txt 檔中搜尋 `npm`，檔名使用藍色輸出

8. `strings`   : 要搜尋的內容
    
    >`findstr  npm \*.\*`
    
    - 在所有檔案中搜尋 `npm`    
    
    - 空格分割多個要搜尋的字串

8. `[ Drive : ][ Path ] FileName [...]` : 指定要搜尋的檔案位置
    
    > `findstr npm c:\t1.txt`
    - 在`c:\t1.txt`中搜尋`npm`

8. `/?`   : 列出語法相關說明
    
    >`findstr /?`

## 小技巧
1. 語法參數可以串連
    - `findstr /s /i /n npm \*.\* `
    - `findstr /sin npm \*.\* `
    - 上面兩句是相同的

2. 語法參數跳脫字元可以用 `/` 及 `-`
    - `findstr /b npm \*.\* `
    - `findstr -b npm \*.\* `
    - 上面兩句是相同的

3. findstr 可以搜尋 office 97-2003 的檔案內容
    - 新版 office 檔案(副檔名有 `x`)***無法***搜尋到    
    - grepwin ***無法***搜尋到 office 97-2003 的檔案內容

# 參考資料
1. [grepwin](http://stefanstools.sourceforge.net/grepWin.html)
2. [介紹好用工具：grepWin](http://blog.miniasp.com/post/2008/01/29/Useful-tool-grepWin.aspx)
3. [Findstr](https://technet.microsoft.com/zh-tw/library/bb490907.aspx)
4. [常用批處理命令總結3之Find和FindStr](http://www.cnblogs.com/doit8791/archive/2012/05/21/2511080.html)
5. [Dos命令學習分享（一、Find及Findstr）](http://tommytanghuaan.blogspot.tw/2012/06/dosfindfindstr.html)
6. [File attribute of "O" and unusual file icon overlay.](http://answers.microsoft.com/en-us/windows/forum/windows_xp-files/file-attribute-of-o-and-unusual-file-icon-overlay/4e82a9e3-489a-4309-b595-774a7beebb61)
6. [Example of a literal search](http://users.encs.concordia.ca/~bcdesai/course/commands/nt/findstr.html)