---
title: "Windows 環境下為 Git Repository 加上 Git Hook"
date: 2017-05-07T01:30:00+08:00
lastmod: 2018-09-18T14:50:56+08:00
draft: false
tags: ["Git","Windows"]
slug: "git-hook-windows"
aliases:
    - /2017/05/git-hook-windows.html
---
# Windows 環境下為 Git Repository 加上 Git Hook
同事因為專案相依的關係，在不同 branch 間可能會誤用不同屬性或是方式，所以想在 repository 切換 branch 時加入提示，提醒工程師記得切換相依專案的 branch，這樣的需求挺適合透過 Git Hook 來處理：在 Git Repository 出現指定行為時執行指定的 script；網路上的範例多為 linux 指令，筆記一下 Windows 環境的使用方式

## About Git Hook

在 repository 建立時，.git 資料夾中會自動建立名為 `hooks` 的子資料夾，預設會提供幾個常見 hook 情境的設定範例，這幾個範例是用來在 linux bash 執行用的，windows 環境中大多無法直接使用

啟用 hook 時，需要加上 `hook 類型`為檔名且`無附檔名`的檔案，內容可以是任何 script 類型的語法(e.g. python、ruby、vb script...)

*   自動建立 hooks 資料夾

    ![1hookfolder](https://cloud.githubusercontent.com/assets/3851540/25749070/ccaba326-31df-11e7-8ee3-8681f1108156.png)

*   hook 範例

    ![2hooksample](https://cloud.githubusercontent.com/assets/3851540/25749075/ccd41b58-31df-11e7-8f31-d0880c41929a.png)

*   支援的 hook 類型

    > 詳細資料可以參考 [Git Hooks](http://githooks.com/)

    *   applypatch-msg
    *   pre-applypatch
    *   post-applypatch
    *   pre-commit
    *   prepare-commit-msg
    *   commit-msg
    *   post-commit
    *   pre-rebase
    *   post-checkout
    *   post-merge
    *   pre-receive
    *   update
    *   post-receive
    *   post-update
    *   pre-auto-gc
    *   post-rewrite
    *   pre-push



## 如何設定 Git Hook

接著一步步來看該如何滿足需求：checkout branch 時提示

1.  repository 加上 post-checkout hook


    *   複製一個 `*. sample` --> 改名為 post-checkout 並移除附檔名

        ![3addhook](https://cloud.githubusercontent.com/assets/3851540/25749076/ccd6d12c-31df-11e7-9e36-9cf4191f8d7c.png)

    *   因為檔案格式的關係，請記得 <span style="color:red">**不要自行新增檔案**</span>，否則將無法正確執行

        *   正確執行版本

            > ![8correctformat](https://cloud.githubusercontent.com/assets/3851540/25749071/ccae74fc-31df-11e7-9809-bc8d1af49eb8.png)

        *   無法執行版本

            ![7wrongformat](https://cloud.githubusercontent.com/assets/3851540/25749069/ccaaf1ba-31df-11e7-9170-9df64cc69173.png)

2.  加上 hook 內容


    *   在檔案第一行加入 `#!/bin/bash` 標記使用 script 語法
    *   加入實際執行動作

        *   使用 mshta

            ```
            #!/bin/bash
            mshta "javaScript:var sh=new ActiveXObject( 'WScript.Shell' ); sh.Popup( 'Message!', 10, 'Title!', 64 );close()"
            ```

        *   使用 msg

            *   將實際執行動作寫成 .bat

                > `msg * "Enter Your Message"`

            *   hook 內容就 call 該 .bat 檔

                ```
                #!/bin/bash
                "C:\alert.bat"
                ```

3.  加上執行權限(<span style="color:red">**非必要**</span>)

    > 如果前面步驟完成後，仍無法成功執行，可以嘗試以下設定作法

    *  錯誤訊息

        - 訊息內容
            
            ```
            cannot spawn .git/hooks/post-checkout: No such file or directory
            ```
        - 錯誤截圖
            
            ![5errormsg](https://cloud.githubusercontent.com/assets/3851540/25749074/ccd3f3e4-31df-11e7-9a19-6b875c67df0e.png)

    * 在 repository 所在位置開啟 git bash

        ![6gitbash](https://cloud.githubusercontent.com/assets/3851540/25749072/ccaf7a0a-31df-11e7-9ed1-dcb5e33fc7f4.png)

    * 執行 `chmod +x .git/hooks/post-checkout` 加上執行權限

## 實際效果

1.  使用 mshta


    *   可自訂 title 與大小
    *   不是永遠在視窗最上層

        ![9mshta](https://cloud.githubusercontent.com/assets/3851540/25749068/ccaa30cc-31df-11e7-9106-c2c60f2b9859.png)

2.  使用 msg


    *   永遠在視窗最上層
    *   實際執行語法要另外存放

        ![10msg](https://cloud.githubusercontent.com/assets/3851540/25749067/cca8121a-31df-11e7-8bb2-071ce5712ec9.png)

# 參考資訊

1.  [8.3 Customizing Git - Git Hooks](https://git-scm.com/book/zh-tw/v2/Customizing-Git-Git-Hooks)
2.  [githooks - Hooks used by Git](https://git-scm.com/docs/githooks)
3.  [Git Hooks](http://githooks.com/)
