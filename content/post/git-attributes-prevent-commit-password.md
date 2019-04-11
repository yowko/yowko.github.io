---
title: "如何使用 Git Attributes 來避免 web.config(app.config) 連線字串的機敏資訊被 commit"
date: 2017-05-15T01:30:00+08:00
lastmod: 2018-09-22T01:30:12+08:00
draft: false
tags: ["Git","PowerShell"]
slug: "git-attributes-prevent-commit-password"
aliases:
    - /2017/05/git-attributes-prevent-commit-password.html
---
# 如何使用 Git Attributes 來避免 web.config(app.config) 連線字串的機敏資訊被 commit
自從 GitHub 與雲端服務盛行以來，你一定有聽過類似的鄉野傳聞：某個開發人員不慎把雲端服務的帳號及密碼 commit 至 GitHub，結果讓網路上的有心人士盜用最後付出慘痛的代價，自從聽過這個事件後曾經嘗試過幾種作法來避免類似問題：assume unchanged、skip worktree、gitignore 或是透過 git server 的 lock file 功能讓特定檔案無法被 push

這幾天整理 git attribute 資料時，發現 `Keyword Expansion` 特性也可以達到目的，立馬來試試

## 關於 Git Attributes 的 Keyword Expansion

> Keyword Expansion 有兩種行為：

1.  clean
    
    > commit 時觸發，這是我們接下來要用的做法

    ![clean](https://git-scm.com/figures/18333fig0703-tn.png)

2.  smudge

    > checkout 時觸發

    ![smuge](https://git-scm.com/figures/18333fig0702-tn.png)

## 準備清除敏感資訊的 script

> 下面會使用 PowerShell 來示範

1.  新增 removesecret.ps1(PowerShell) 檔

    > 可以放在專案 `.git/info/attributes` 資料夾或是其他共用資料夾

2.  加入下列 script

    ```ps1
    # 接受外部傳入的 config 檔案位置
    Param([string]$filePath)
    # 將 config 內容以 xml 讀入
    [xml]$Config = Get-Content -Path $filePath
    # 取得 connectionstring 並將 data source、user id、password 取代成 "-"
    $Config.configuration.connectionStrings.add| foreach {$_.connectionString=$_.connectionString -replace '(data source|user id|password)=(.*?);','$1= - ;'}
    # 將取代完成的內容存檔
    $Config.Save($filePath)
    ```

## 設定 Git Attributes

> 以下動作直接在 command prompt 下以指令操作

1.  建立 `.gitattributes` 並設定 `web.config` 使用 `cleandata` 的 filter

    ```
    echo "config.php filter=cleandata" > .gitattributes
    ```

2.  將 `.gitattributes` 變更 commit

    ```
    git add .gitattributes
    git commit -m "clean connectionstring data via gitattributes"
    ```

3.  設定觸發時機、script 位置並傳入 config 位置

    ```
    git config filter.cleandata.clean "powershell.exe -File '.git\info\attributes\removesecret.ps1' -filePath 'TestWebhook\Web.config'"
    ```

## 修改效果

1.  在 web.config 新增一組有連線字串、帳號、密碼的 connectionstring

    ```xml
    <add name="TestEntities" connectionString="metadata=res://*/Models.sn.csdl|res://*/Models.sn.ssdl|res://*/Models.sn.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=YOWKOMAC-WIN10\SQLEXPRESS;Initial Catalog=Test;Persist Security Info=True;User ID=TestUser;Password=TestPwd;Pooling=False;MultipleActiveResultSets=False;App=EntityFramework&quot;" providerName="System.Data.EntityClient" />
    ```

2.  加入其他任意設定

    ```xml
    <appSettings>
        <add key="eee" value="jjjj" />
    </appSettings>
    ```
3.  在將變更加入 index (列入準備 commit) 時會觸發清除動作


    * connrection 的機敏資訊已被清楚

        ![1connect](https://cloud.githubusercontent.com/assets/3851540/26033979/e840c63e-38e7-11e7-80c7-dccb13f3a0d2.png)

    *  其他設定未受影響

        ![2others](https://cloud.githubusercontent.com/assets/3851540/26033978/e83f9534-38e7-11e7-9312-912956dfbc26.png)

## 注意事項

1.  記得先將 `removesecret.ps1` 修改內容的部份調到定位，否則每次 commit 前都會再被 `removesecret.ps1` 修改一次喔
2.  如果需要與其他團隊成員共享，需要將 `removesecret.ps1` 移出 `.git` 資料夾外，不然不會納入 git server


## 心得

assume unchanged、skip worktree 需要由個人設定， lock file 又不是每個 git server 都有提供，gitignore 與 git attribute 都是相對方便的做法，但 gitignore 是忽略整份檔案，如果想要針對 config 檔案內其他部份修改就得連 git ignore 一併調整，這時就會有不小心該機敏資訊曝光的風險了，因此 恭喜 git attribute 獲得此次資訊安全比賽第一名XD

# 參考資訊

1.  [在Git中避免commit你的密碼](http://www.udpwork.com/item/7216.html)
2.  [Git hooks, practical uses (yes, even on Windows)](https://tygertec.com/git-hooks-practical-uses-windows/)
3.  [Running PowerShell scripts as git hooks](http://stackoverflow.com/questions/5629261/running-powershell-scripts-as-git-hooks)
4.  [How do I read/write App.config settings with PowerShell?](http://stackoverflow.com/questions/871549/how-do-i-read-write-app-config-settings-with-powershell)
5.  [Replace only first occurence of a character in filenames](http://stackoverflow.com/questions/32427730/replace-only-first-occurence-of-a-character-in-filenames)
