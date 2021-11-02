---
title: "Git partial clone"
date: 2020-07-18T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Git"]
slug: "git-partial-clone"
---

## Git partial clone

Git 2.19.0 發表了 Partial Clone 功能，技術細節可以參考 [Partial Clone Design Notes](https://github.com/git/git/blob/master/Documentation/technical/partial-clone.txt)

這功能從我開始使用 Git 就覺得需要了，因為 Git 跟過去中央式版控不同，不是以路徑為使用基準，所以沒辦法以路徑思維做相關操作，但專案架構的拆分有時就是會需要將共用模組統一位置，需要引用的個別 application 再分別以所需參考使用

雖然 Git 2.19.0 開始提供 Partial Clone 功能，但在團隊中要能派上用場還是需要 Git Server 支援，截自目前 (2020/07/18) 為止，GitLab 提供了以 `檔案大小`，`物件類型` 與 `檔案路徑` 三種 partial clone 的做法，其中 `檔案路徑` 還在實驗階段，而 GitHub 則尚未看到相關實作

所以今天就來紀錄一下該如何在 GitLab 上執行以 path 為主的 partial clone

以 GitLab 文件上的條件：

1. Git 2.22.0 以上
2. GitLab 12.4 開始支援 partial clone (但沒查到今天用的方法需要哪一版)

## 基本環境說明

1. macOS Catalina 10.15.5
2. git version 2.27.0
3. GitLab.com (GitLab 13.1)
4. 範例專案

    - partial-clone

        > 模擬共用專案，提供給其他 application 參考，放在 [GitLab partial-clone repo](https://gitlab.com/yowko/partial-clone) 

        - moduleA
            - a-1.txt
            - a-2.txt
        - moduleB
            - b-1.txt
            - b-2.txt
            - b-3.txt
        - moduleC
            - c-1.txt
    - application
        - README.md

## 設定方式

1. 建立 `.gitfilterspec`

    > 提供給其他 repo 進行 partial clone 的 repo (以我的範例就是 `partial-clone` )中加入 `.gitfilterspec`
    >
    > - 只有在這個檔案中明確列出的路徑才會被下載 (前提是正確設定)
    > - 建議依據需求建立各個路徑的 `.gitfilterspec` (如果 `moduleA` 會單獨被參考使用就單獨建立一個 `.gitfilterspec` 檔案；`moduleB` 與 `moduleC` 會同時被參考使用可以使用同一個 `.gitfilterspec` 檔案)

    - moduleA/.gitfilterspec

        ```txt
        moduleA/
        ```

    - .gitfilterspec

        ```txt
        moduleB/
        moduleC/
        ```

2. 將參考的 repo 加至 git remote 中

    > 在實際進行 partial clone 的 repo (以我的例子就是 `application`) 將提供 partial clone 的 repo (以我的例子是 `partial-clone`) 加至 git remote 中追蹤

    - 語法

        ```bash
        git remote add {name for remote} {git repo url}
        ```

    - 範例

        ```bash
        git remote add pcsource git@gitlab.com:yowko/partial-clone.git
        ```

3. 針對剛加入的 remote 啟用 partial clone

    - 語法

        ```bash
        git config --local extensions.partialClone {name for remote}
        ```

    - 範例

        ```bash
        git config --local extensions.partialClone pcsource
        ```

4. 取得 partial clone 的 filter 設定

    - 語法

        ```bash
        git fetch --filter=sparse:oid={branch name}:{.gitfilterspec 相對路徑} {name for remote}
        ```

    - 範例 1 (下載 `moduleA`)

        ```bash
        git fetch --filter=sparse:oid=master:moduleA/.gitfilterspec pcsource
        ```

    - 範例 1 (下載 `moduleB` 與 `moduleC`)

        ```bash
        git fetch --filter=sparse:oid=master:.gitfilterspec pcsource
        ```

5. 啟用以路徑為條件的 partial clone (sparse checkout)

    ```bash
    git config --local core.sparsecheckout true
    ```

6. 設定 partial clone 的路徑條件 (sparse checkout)

    - 語法

        ```bash
        git show {name for remote}/{branch}:{.gitfilterspec 相對路徑} >> .git/info/sparse-checkout
        ```

    - 範例 1

        ```bash
        git show pcsource/master:moduleA/.gitfilterspec >> .git/info/sparse-checkout
        ```

    - 範例 2

        ```bash
        git show pcsource/master:.gitfilterspec >> .git/info/sparse-checkout
        ```

7. 執行 partial clone

    - 語法

        ```bash
        git checkout pcsource/master
        ```  

    - 範例

        ```bash
        git checkout {name for remote}/{branch}
        ```

## 心得

測試了一整天XD，重新 review 再次發現步驟不多，現在回頭看官方文件，好像也沒有寫得不清楚、流程步驟也都沒錯，但當下真的是有看沒有懂，不知道是我英文太爛還是表達方式有問題，或者只是我沒有找到文件的觀點才一直卡關

雖然測試完了，但也發現功能跟預想的不太一樣：原本想的是 partial clone 可以另個 repo 的部份內容加至當前 repo 中，但測試完發現功能比較像一開始 git 的設計原理：為了避免下載大檔，是不是很難懂，請看下圖

![1result](https://user-images.githubusercontent.com/3851540/87876656-3b357980-ca0c-11ea-9e63-f97c7e03f968.jpg)

原本存在的內容，因為 branch 不同，而在 checkout 後就不見，跟原本預想的加入 partial clone 不同，我想也許可以透過 submodule 與 partial clone 的組合來達成原本的目標，不過有機會再試囉

本來想要改天有時間再試，但壓抑不住心裡的好奇心，立馬嘗試了起來，雖然有些缺陷，但勉強堪用，請參考 [Git Submodule partial clone](/git-submodule-partial-clone)

## 參考資訊

1. Git 2.22.0 發表了 Partial Clone 功能，技術細節可以參考 [Partial Clone Design Notes](https://github.com/git/git/blob/master/Documentation/technical/partial-clone.txt)
2. [Partial Clone](https://docs.gitlab.com/ee/topics/git/partial_clone.html)
