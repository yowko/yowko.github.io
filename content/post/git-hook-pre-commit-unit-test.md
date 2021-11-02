---
title: ".NET 專案在 Git commit 前自動執行 Unit Test"
date: 2017-09-19T23:45:00+08:00
lastmod: 2021-11-02T23:45:17+08:00
draft: false
tags: ["Git","Unit Test"]
slug: "git-hook-pre-commit-unit-test"
aliases:
    - /2017/09/git-hook-pre-commit-unit-test.html
---
## .NET 專案在 Git commit 前自動執行 Unit Test

身為一個優秀的工程師對於提升專案品質都有一定的堅持，但對於 Unit Test 所帶來的效益，相信大家應該都會給予正面評價。而專案測試的 code coverage 則是由所有專案團隊成員共同保持的，除了團隊的內部規範之外，似乎沒有其他強制手段來要求新增功能需要有對應的測試，但至少可以做到讓既有的測試保障依然存在

這個時候我們就可以透過 Git hook，加入 `pre-commit` 的 event 讓程式碼在 commit 前至少通過既有測試，維持基本的程式碼品質，如果無法通過既有的測試就不允許 commit，如此一來就有基本強制手段來避免程式改壞，就來看看可以怎麼做吧

## 加上 Git Hook

之前文章 [Windows 環境下為 Git Repository 加上 Git Hook](/2017/05/git-hook-windows.html)，對於 Git Hook 有簡單介紹可以參考

1. 在專案的 `.git\hooks` 資料夾將 `pre-commit.sample` rename 為 `pre-commit` (去掉附檔名)

    ![1precommit](https://user-images.githubusercontent.com/3851540/30600991-66f0f7d2-9d93-11e7-8321-35bc6e9f86ed.png)

2. 將 `pre-commit` 調整為執行測試語法

    ```bash
    #!/bin/sh
    OUTPUT=$("C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" "..\ShoppingCart\ShoppingCart\bin\Debug\ShoppingCart.dll")
    if [ $? -ne 0 ]; then
        echo "Unit Test Failed"
        echo "${OUTPUT}"
        exit 1
    fi
    ```

    * `#!/bin/sh`

        > 指定 script 語法，`#!/bin/bash` 也行

    * `OUTPUT=$(command)`

        > 將測試指令寫在 `$()` 中，並將結果指令給 `OUTPUT`，執行測試的程式位置(`vstest.console.exe`) 每台電腦不一定相同

    * `if [ $? -ne 0 ]; then`

        > 如果測試正確執行 `$?` 會是 `0`，`$? -ne 0` 表示測試結果是 `failed`

    * `echo "Unit Test Failed"`

        > 在畫面上輸出 `"Unit Test Failed"`

    * `echo "${OUTPUT}"`

        > 將測試結果輸出

    * `exit 1`

        > 結束程式，表示無法進行 commit

    * `fi`

        > if 的結尾

## 與其他團隊成員共用

上述的設定只適用於 local repository，因為在 `.git` folder 下的所有內容都不會加入版控，如果想將 git hook 與其他團隊成員共用，就需要加上下列設定

1. 將 `hooks` 資料夾從 `.git` 中移至專案根目錄

    > ![4movetoroot](https://user-images.githubusercontent.com/3851540/30600994-66f85536-9d93-11e7-86f6-6be872347ced.png)

2. 將 git hooks 的路徑從 `.git\hooks` 指向專案根目錄的 `hooks`

    ```cmd
    git config core.hooksPath hooks
    ```

3. 可能需要調整 git hook 內容

    > 確認測試目標的 dll 路徑

## 實際效果

1. 未通過測試，無法 commit

    ![2cannotcommit](https://user-images.githubusercontent.com/3851540/30600993-66f8034c-9d93-11e7-8c02-42d096acc967.png)

2. 通過測試，順利 commit

    ![3commit](https://user-images.githubusercontent.com/3851540/30600992-66f212b6-9d93-11e7-91a0-465e5f3d90b8.png)

## 心得

透過 Git hook 加上 pre-commit 執行單元測試，可以讓新 commit 的內容不會造成已經存在的 unit test fail，但說實話這樣的做法只能算是治標，最好的方式還是應該從源頭開始讓所有團隊成員養成寫測試的習慣，讓所有人可以體會到測試的好處，自然而然大家就會自動自發的寫測試了

## 參考資訊

1. [Windows 環境下為 Git Repository 加上 Git Hook](/git-hook-windows)
2. [使用git hook在commit 前進行unittest](http://yodalee.blogspot.tw/2016/12/git-hook-unittest.html)
3. [Two Ways to Share Git Hooks with Your Team](https://www.viget.com/articles/two-ways-to-share-git-hooks-with-your-team)
