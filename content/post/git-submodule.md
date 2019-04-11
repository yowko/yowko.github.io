---
title: "Git 專案引用其他 Repository 的作法(Git Submodule)"
date: 2017-05-13T10:55:00+08:00
lastmod: 2018-09-19T20:00:08+08:00
draft: false
tags: ["Git"]
slug: "git-submodule"
aliases:
    - /2017/05/git-submodule.html
---
# Git 專案引用其他 Repository 的作法(Git Submodule)
最近公司正在大刀闊斧地將專案從 SVN 搬到 Git 上，流程的改變、不同的版控觀念以及操作方式不免造成些衝擊，但這些只要經過幾個專案實戰演練，熟悉相關 Git 用法後很快就可以克服的，直到同事提出個問題就是屬於比較不好解決的類型，發生原因是兩種版控架構設計面不同造成的：SVN 可以將多個專案放在同一個 folder 下來進行版控，而這樣的做法在 GIT 雖然可行但卻不是那樣的優雅;對於專案間如果存在相性時 SVN 的方式似乎方便些，但 GIT 身為強大的後起之秀還是有相關的功能可以達成相同目的，就來看看可以如何設定

## 情境說明

我們在開發 .NET 專案時常常會引用 NuGet 套件，但資安管控嚴謹的環境可能不允許有對外連線，面對無法直接使用 NuGet server 的開發團隊可能會選擇自行架設 NuGet server 供內部使用，避免相關內部資訊外流，有效提高安全性，一旦如果沒有內部 NuGet server 時，使用團隊共用元件的常見方式就是手動為專案加入 dll 參考，同事提出的問題就是這樣：公司為了簡化開發複雜度並加快開發速度，所以將共用元件都抽離單獨包裝成 dll，讓各個專案及團隊可以重複使用，而這組共用的 dll 們該如何被 Git 版控?

*   svn 本來的做法 - 資料夾結構如下
    *   projects
        *   CommonLib
        *   Application1
        *   Application2
        *   Application3

> Application 只要都固定引用外層 `CommonLib` 的內容即可，各 application 間可以單獨存在、clone，也可以單獨 update 跟 commit


> 雖然 Git 也可以直接延用這樣的版控方式，但面臨的問題就是各專案間無法單獨存在一次就是要 clone 整個 repository 的內容，pull、commit 就有機會跟其他 application 的開發人員互相影響

## 使用 Git Submodule

> Git 不愧是受歡迎又強大的工具，經過長時間的檢驗與調整，各式各樣的使用情境都已經有對應的成熟作法，根據上面的情境說明我們就可以透過 GIT 的 Submodule 來進行

*   將其他 Repository 以 Submodule 加入(以下做法擇一即可)

    *   使用 command

        *   語法：

            >git submodule add {git repository url} {folder name}

        *   範例：

            ```
            git submodule add http://localhost:10001/TestLib.git TestLib
            ```

    *   使用 TortoiseGit

        *   專案空白處按右鍵 --> TortoiseGit --> Submodule Add...

            ![1addsubmodule](https://cloud.githubusercontent.com/assets/3851540/26021375/621d8b98-37be-11e7-92ad-c3fd84483089.png)

        * 填入 submodule 相關資訊 (e.g. repository url、存放路徑、使用 branch...)

            ![2submoduleinfo](https://cloud.githubusercontent.com/assets/3851540/26021377/62317ca2-37be-11e7-9ac1-6000de0f2e3d.png)

            ![3submoduleadded](https://cloud.githubusercontent.com/assets/3851540/26021378/6231eae8-37be-11e7-80e7-efa526b1328d.png)

*   使用 Git Submodule 後 Repository 的變化

    1.  增加.gitmodules 檔案
        
        ![4gitmodule](https://cloud.githubusercontent.com/assets/3851540/26021379/62340896-37be-11e7-8b48-2af5e18fb8b0.png)

    2.  增加引用 repository 的資料夾

        ![5modulerepo](https://cloud.githubusercontent.com/assets/3851540/26021376/62318102-37be-11e7-8a80-0ee80c87ca01.png)

    3.  `.git` 資料夾的變化


        *   config 多了 submodule 的 url 設定

            ```
            [submodule "TestLib"]
            url = http://localhost:10001/TestLib.git
            ```

            ![6gitconfig](https://cloud.githubusercontent.com/assets/3851540/26021373/61d85adc-37be-11e7-8f66-f3024d76047e.png)

        *   多了 modules 的資料夾存放引用的 repository

            ![7modules](https://cloud.githubusercontent.com/assets/3851540/26021374/620778da-37be-11e7-81d3-0b9f0143de6b.png)

## 實際使用

1.  submodule 版控獨立於主要 repository 外
    
    > 如果不在 submodule 的資料夾下，git 不會追蹤 submodule 的變更

2.  submodule 單獨擁有完整 git 功能

    > 可以 commit、push、create or checkout branch

3.  全新 clone 有 submodule 專案時，submodule 會沒有內容，需要額外步驟(以下做法擇一即可)：

    > 在主要的專案 repository 這層執行動作

    *   使用 command

        - `git submodule init`

            ![8submodinit](https://cloud.githubusercontent.com/assets/3851540/26021891/6563b0ec-37c9-11e7-807b-41e2883c17bb.png)

        - `git submodule update`

            ![9submodupdate](https://cloud.githubusercontent.com/assets/3851540/26021889/654bacb8-37c9-11e7-9a17-12875cda2c21.png)

    *   使用 TortoiseGit
        *   專案空白處按右鍵 --> TortoiseGit --> Submodule Update...

            ![10subupdate](https://cloud.githubusercontent.com/assets/3851540/26021888/654a1420-37c9-11e7-9ee5-51b0782c1b82.png)

        *   預設就會指定 submodule folder 跟執行 submodule init 的動作

            ![11submoduleint](https://cloud.githubusercontent.com/assets/3851540/26021890/655dacc4-37c9-11e7-8f28-03eada93d2fe.png)


# 參考資訊
1.  [6.6 Git 工具 - 子模組 (Submodules)](https://git-scm.com/book/zh-tw/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E7%B5%84-Submodules)
