---
title: "Git Submodule partial clone"
date: 2020-07-19T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["Git"]
slug: "git-submodule-partial-clone"
---

## Git Submodule partial clone

之前筆記 [Git partial clone](/git-partial-clone) 嘗試了期待已久的 partial clone 功能，雖然跟預期有落差，但對於想要的目標有了不同想法

之前以為 partial clone 可以在既有 repo 的內容中，疉加另個 repo 的部份內容，但嘗試下來：取得 repo 的部份內容這個沒有問題，但卻不是疉加，而是取代覆蓋，但這也讓我想到是不是可以使用 submodule 加上 partitial clone 來達成目的，打鐵趁熱立馬來紀錄一下

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

2. 設定 submodule

    > 這個動作是我覺得整個概念最大缺陷，因為 `git submodule add` 預設會直接下載整個 repo

    - 語法

        ```bash
        git submodule add {git uri} {path}
        ```

    - 範例

        ```bash
        git submodule add git@gitlab.com:yowko/partial-clone.git modules
        ```

3. 針對特定資料夾啟用以路徑為條件的 partial clone (sparse checkout)

    - 語法

        ```bash
        git -C {submodule path} config core.sparseCheckout true
        ```

    - 範例

        ```bash
        git -C modules config core.sparseCheckout true
        ```

4. 設定 partial clone 的路徑條件 (sparse checkout)

    - 語法

        ```bash
        git archive --format=tar --remote={git uri} {branch}:{folder path} -- {filename} | tar -O -xf - >>.git/modules/{submodule folder name}/info/sparse-checkout
        ```

    - 範例

        ```bash
        git archive --format=tar --remote=git@gitlab.com:yowko/partial-clone.git master:moduleA -- .gitfilterspec | tar -O -xf - >>.git/modules/modules/info/sparse-checkout
        ```

5. 使用 partial clone filter 來更新 submodule

    - 語法

        ```bash
        git submodule update --force --checkout {submodule folder}
        ```

    - 範例

        ```bash
        git submodule update --force --checkout modules
        ```

6. 套用 submodule 更新

    - 語法

        ```bash
        git submodule update --remote {submodule folder}
        ```

    - 範例

        ```bash
        git submodule update --remote modules
        ```

    ![2submoduleupdate](https://user-images.githubusercontent.com/3851540/87876711-9ebfa700-ca0c-11ea-9798-1f25980f17f8.jpg)

## 心得

![1submoduleinit](https://user-images.githubusercontent.com/3851540/87876709-9cf5e380-ca0c-11ea-8e07-68eb3c9cebff.jpg)

上圖呈現了我覺得這個做最大的缺陷：一開始 submodule 會下載所有內容，待 partial clone 的設定完成後執行 submodule update 才會是預期狀態，但如果可以找到讓 submodule 一開始不要執行 init 的設定，做法就更漂亮了

另外實驗過程中，我發現了另個 "特性"，partial clone 的相關設定跟 git hook 有一樣的問題，都是每個環境需要獨立設定的，沒辦法跟著 repo 走，所以在使用上較為繁瑣，希望日後有機會改善 (或者只是我用錯而已XD)

## 參考資訊

1. [Git partial clone](/git-partial-clone)
2. [How to sparsely checkout only one single file from a git repository?](https://stackoverflow.com/a/2467629)
3. [How to do submodule sparse-checkout with Git?](https://stackoverflow.com/questions/45688121/how-to-do-submodule-sparse-checkout-with-git)
4. [Using submodules in Git - Tutorial](https://www.vogella.com/tutorials/GitSubmodules/article.html)
