---
title: "使用 Git LFS 儲存大型檔案"
date: 2017-08-25T21:00:00+08:00
lastmod: 2021-11-02T21:00:07+08:00
draft: false
tags: ["Git"]
slug: "git-lfs"
aliases:
    - /2017/08/git-lfs.html
---
## 使用 Git LFS 儲存大型檔案

分散式版控 Git 雖然已經改善了許多集中式版控的缺點，但針對內容 hash 的作法對於大型 binary 檔案，效能還是不夠令人滿意，針對這個問題 GitHub 與 GitLab 分別提出 Git LFS 與 Git Annex 做為處理大型檔案的解決方案

而市場上普遍使用 GitHub 所提出的 Git LFS(Large File Storage)，就來看看 Git LFS 與 Git Annex 的差異與該如何使用 Git Lfs 吧

## Git LFS 與 Git Annex 的不同之處

|-|Git LFS|Git Annex|
|--- |--- |--- |
|傳輸協定|SSH and Https|SSH|
|指令難易度|簡單|複雜|
|儲存方式|Repository 外的檔案伺服器  
透過 text pointers 指向|Repository 中的子目錄|
|平台支援度|各平台皆支援|Windows x64 不是全功能支援|
|雲端服務支援度|github, bitbucket,gitlab|gitlab|

## 安裝 `Git LFS`

1. 下載 `Git-LFS-Windows` 並安裝 [下載位置](https://github.com/git-lfs/git-lfs/releases/download/v2.2.1/git-lfs-windows-2.2.1.exe)

    ![1install1](https://user-images.githubusercontent.com/3851540/29709376-ae9202d6-89be-11e7-83d7-440a80b0b8a6.png)

    ![2install2](https://user-images.githubusercontent.com/3851540/29709377-ae936d7e-89be-11e7-8532-adaa54fea145.png)

2. 初始化 Git LFS

    ```cmd
    git lfs install
    ```

    ![3ini](https://user-images.githubusercontent.com/3851540/29709379-ae940e78-89be-11e7-8b59-ad7d39a1b85e.png)

## 如何使用 Git LFS

1. 將大檔加入 Git LFS 追蹤

    > 可以針對 `單檔`、`特定附檔名` 或是 `資料夾` 加入

    ```cmd
    git lfs track "*.exe
    ```

    ![4lfstrack](https://user-images.githubusercontent.com/3851540/29709378-ae93ccf6-89be-11e7-98be-762268c3c7fe.png)

2. 會修改 `.gitattributes` 記得加入版控

    ```cmd
    git add .gitattributes
    ```

    ![5gitattribute](https://user-images.githubusercontent.com/3851540/29709381-aeb4ccd0-89be-11e7-8427-87b08e24c8a2.png)

3. Push 到 Git server

    ```cmd
    git add .
    git commit -m "Add large file"
    git push origin master
    ```

## 如何 Clone 及 Pull

> 可以直接使用 `git clone` 及 `git pull`，但可以使用 `git lfs clone` 及 `git lfs pull` 來加速

1. Clone

    * `git clone`

        ![6gotclone](https://user-images.githubusercontent.com/3851540/29709382-aeb7375e-89be-11e7-8665-5a69bfd7b900.png)

    * `git lfs clone`

        ![7gitlfsclone](https://user-images.githubusercontent.com/3851540/29709383-aeb87812-89be-11e7-8b02-58253b55fbce.png)

2. Pull

    > 使用情境和 `clone` 不同，一般情境只需使用 `git pull`，如果遇到無法順利取得完整檔案時才需要明確使用 `git lfs pull`

    * `git pull`

        ![8gitpull](https://user-images.githubusercontent.com/3851540/29709384-aebdac9c-89be-11e7-8baa-9b3bfe214502.png)

    * `git lfs pull`

        > 下載缺漏 lfs 檔案

## 其他指令

1. 設定只取得 repository 指標不取得實際檔案，只在需要時才透過 `git lfs pull` 取得檔案

    * 全域設定

        ```cmd
        git -c filter.lfs.smudge= -c filter.lfs.required=false pull && git lfs pull
        ```

    * 單一 repository

        ```cmd
        git -c filter.lfs.smudge= -c filter.lfs.required=false clone https://github.com/user/repo.git
        ```

2. 取得幾天內的版本
    * 設定近期天數

        ```cmd
        git config lfs.fetchrecentcommitsdays 7
        ```

    * 永遠使用最近

        ```cmd
        git config lfs.fetchrecentalways true
        ```

    * 取得指令

        ```cmd
        git lfs fetch --recent
        ```

3. 從 local 刪除遠端已被移除的檔案

    ```cmd
    git lfs prune
    ```

4. 納入或排除特定資料夾或是檔案

    * 納入特定資料夾

        ```cmd
        git config lfs.fetchinclude 'folder/**'
        ```

    * 排除特定標案

        ```cmd
        git config lfs.fetchexclude 'folder/a.mp4'
        ```

## 心得

Git LFS 的內容很少，指令也很容易，但一陣子沒用，指令又生疏不少，所以紀錄一下

剛好趁這個機會釐清了 git lfs 的指令涵義

## 參考資訊

1. [超大影音檔版本控管更簡單了!GitHub釋出LFS擴充機制能幫忙](http://www.ithome.com.tw/news/95281)
2. [Differences between Git Annex and Git LFS](https://docs.gitlab.com/ee/workflow/lfs/migrate_from_git_annex_to_git_lfs.html)
3. [How do Git LFS and git-annex differ?](https://stackoverflow.com/questions/39337586/how-do-git-lfs-and-git-annex-differ)
4. [Git Large File Storage](https://git-lfs.github.com/)
5. [git-lfs](https://www.atlassian.com/git/tutorials/git-lfs)
