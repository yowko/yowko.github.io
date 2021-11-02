---
title: "在Local Git Remote SVN(本機使用 Git，版控 Server 使用 SVN)"
date: 2016-12-12T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Git","SVN"]
slug: "local-git-remote-svn"
aliases:
    - /2016/12/local-git-remote-svn.html
---
## Local Git Remote SVN(本機使用 Git，版控 Server 使用 SVN)

大家一定或多或少感受到 Git 愈來愈多人採用，~~為了不輸別人(咦?)~~應該是為了體會 Git 的過人之處，最好的方式就是開始使用它，但公司的版控豈是說換就換的。這邊簡單紀錄一下個人使用經驗：該如何在版控 server 仍使用 SVN 的前提下，開發人員在開發環境改用 Git 管理。

## 建議流程

1. 從 SVN 取得 source code

    > 將 source code 從 SVN 取出，並改由 Git 管理

2. 建立 feature branch

    > 每次修改前，都建立分支

3. 修改 --> commit

    > 每次修改後，就留下版本紀錄

4. 確定完成 --> merge 到本機 git-svn repository

    > 確認版本後，就將 feature branch merge 回一開始從 SVN 取回的 master branch 中

5. 儲存到 SVN

    > dcommit(push) 到 svn

6. 刪除 feature branch

    > 將 feature branch 刪除，每次修改都建立 branch

## 1. 從 svn 取得 source code (擇一即可)

- A. 使用指令

    ```cmd
    git svn clone {svn_repository_url} {資料夾名稱}
    ```

  - 沒有指定 `資料夾名稱` 時，預設使用`專案名稱`當作資料夾名稱

    ```cmd
    git svn clone https://svnserver/svn/TestGitSvn
    ```

  - 會建立 `專案名稱` 資料夾，並將 source code 下載至 `專案名稱` 資料夾中

    ![gitsvn1](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/5834fc369c173c6969d43d41/2c4cb54f51118760acc6e122241a0c7f/_output_gitsvn1.png)

  - 指定 `資料夾名稱`

    ```cmd
    svn clone https://svnserver/svn/TestGitSvn/ test
    ```

    - 會建立 `test` 資料夾，並將 source code 下載至 `test`
            ![gitsvn2](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/677x442/f591972cc3381299cedf03f0f732f3f6/_output_gitsvn2.png)

- B. 使用 TortoiseGit
    1. clone 專案
        - 資料夾空白處，按右鍵 --> `Git Clone`
            ![gitclone1](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/532x283/5e3ee2500648b0c21b6bd7ddb386ef16/_output_gitclone1.png)

    2. from svn
        ![gitclone2](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/555x392/4f46450bc7a182d86a4583e6a21ad079/_output_gitclone2.png)

        - 2-1. URL

            > SVN url

        - 2-2. Directory

            > 目標目錄

        - 2-3. From SVN Repository

            > 表示從 SVN 下載，相關下載屬性也在這個區塊設定

- C. 大型專案
    檔案數較多或是 Commit 數量較多的專案

    1. 使用指令
        - 使用 `-r HEAD` , 只取得最新版

            ```cmd
            git svn clone -r HEAD {svn_repository_url} {資料夾名稱}
            ```

        - 如果中間發生斷線，可以試試`git svn fetch`*

    2. 使用 TortoiseGit
        - 指定 From
            - 無法像指令指定 `HEAD`，就挑最後一版

                ![tgit_from](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/555x392/a22e1eddb6c70f199bb06eda6590cafd/_output_tgit_from.png)

                ![tgit_from_done](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/757x438/1b347e613d9b988a4a385a8cd7a7498d/_output_tgit_from_done.png)

        - git SVN Fetch

            ![git_svn_fetch](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/719x823/dfcea68bfa255d60114e16760f9a664d/_output_gitsvnfetch.png)

## 2. 新增 feature branch

建立 feature branch，僅在本機作業，不會影響 SVN

- A. 使用指令

    ```cmd
    git checkout -b {branchbname}
    ```

  - `-b` 會先建立 `{branchname}` 同時並切換過去，效果等同於下列兩句
        1. `git branch {branchbname}`
        2. `git checkout {branchbname}`

- B. 使用 TortoiseGit

    1. 在欲建立 branch 的資料夾上按右鍵 --> `TortoiseGit`--> `Create Branch`

        ![createbranch](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/651x481/c5fc467adb4f631339e12e83b876fdb5/_output_createbranch.png)

    2. branch 相關屬性

        ![createbranch1](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/460x383/0dd4ef1f6afbc8c9fef725c6e990ac16/_output_createbranch1.png)

        - 2-1. Name

            > 填入 branch name

        - 2-2. Base On

            > 新分支的來源，預設以目前工作分支，也可以選擇其他分支

        - 2-3. Switch to new branch

            >建立分支後，直接將工作目錄，切換過去

## 3. Git Commit

>- 在本機將變更留下紀錄，其他人不會看到變更，SVN 也沒有這個變更
>- 養成留下版本變更的習慣，也不會讓團隊其他人拿到修改中的版本

- A. 使用指令

    ```cmd
    git commit -a -m 'something'
    ```

  - `-a` 是將新增的檔案也納入 commit 範圍，等同於  `git add .`
  - `-m 'something'`, 是在 commit 時直接給註解

- B. 使用 TortoiseGit
    1. 欲 commit 位置的資料夾 按右鍵 --> `Git Commit -> "branchname"...`

        ![gitcommit](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/460x293/f76e02d8695b7e98c1a44543a947007e/_output_gitcommit.png)

    2. Commit

        ![gitcommit2](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/583x556/5d9c8dff671302b56c3cfee03a298627/_output_gitcommit2.png)

  - 2-1. Message

    > commit 的註解

  - 2-2. trick `Not Versioned Files`

    > 挑選要加入版控的新增檔案

## 4. merge 到 git svn

>- 修改已經完成，將 commit merge 到一開始下載的 repository 中，準備儲存到 svn
>- `merge` 指的是將特定 commit 合併至目前工作目錄分支，所以要留意目前工作目錄的位置

- A. 使用指令
    1. 切換到一開始建立 git repository 的 branch (預設 master)

        ```cmd
        git checkout master
        ```

    2. 合併分支(merge)

        ```cmd
        git merge branchbname
        ```

- B. 使用 TortoiseGit
    1. 切換到一開始建立 git repository 的 branch (預設 master)
        - 1-1. git show log
            - 在資料夾按右鍵，選 `Git Show Log`

                ![gitlog](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/475x292/fbb47e9842f5654f10798cd0d5e349ee/_output_gitlog.png)

        - 1-2. 在 `master` 上，按右鍵 --> `Switch/Checkout to "master"`

            ![checkoutmaster](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/1021x747/1c55c0603dd931d704ac1b4292b7e3c8/_output_checkoutmaster.png)

            ![checkouted](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/1021x747/24b59846fb8702bf875f7e11dba3c258/_output_checkouted.png)

    2. 合併分支(merge)
        - 2-1. 在欲合併的分支上按右鍵
        - 2-2. 點選`Merge to "master"`...

            ![mergeto](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/1021x747/6f912aca9d25b45d0f9e11778147c4db/_output_mergeto.png)

        - 2-3. option
            - 可以直接使用預設即可

                ![mergeoption](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/508x461/ba2dedb82878e9966864518995dedb08/_output_mergeoption.png)

                ![mergedone](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/1021x747/ecbed5c0555acd43b3f5b40a466ce92b/_output_mergedone.png)

## 5. push 至 SVN

> 將本機 Git 的變更，存進 SVN，使用 `dcommit` 指令

- A. 使用指令
    1. 更新版本

        ```cmd
        git svn rebase
        ```

    2. dcommit 至 SVN

        ```cmd
        git svn dcommit
        ```

- B. 使用 TortoiseGit

    ![GitSVN](https://trello-attachments.s3.amazonaws.com/580186252406cb39e4a22eed/5834fc369c173c6969d43d41/8ea295193874f76277aa9129e7c5fb35/_output_GITSVN.png)

    1. 資料夾右鍵
    2. 點選 `TortoiseGit`
    3. 點選 `SVN DCommit...`
        - 預設會執行`Git SVN Rebase` 進行更新

            ![committype](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/379x184/b2e695981354c57dbc1cda9b5b0bb55b/_output_committype.png)

            ![commited](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/757x438/8cc13c0be8e24bbc6b48fed5a93594cc/_output_commited.png)

## Note: 合併多個 git commit，一次儲存到 SVN

> 合併多次版本變更，讓 SVN 紀錄相對好閱讀

- A. 使用指令
    1. Rebase

        ```cmd
        git rebase -i HEAD~3
        ```

        - `HEAD~3` 指最近三個 commit

    2. 合併 (squash)
        - 要留下的用 `p` or `pick`
        - 要合併的用 `s` or  `squash`
        - 改完，直接按`:wq` 存檔離開

- B. 使用 TortoiseGit
    1. `Git SVN Rebase`
        - 1-1. 資料夾 --> 右鍵 --> TortoiseGit --> Git Rebase

            ![git_rebase](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/404x323/4859f3f8bc0f97a94418397f679e8ceb/_output_git_reabse.png)

        - 1-2. 選擇 `Squash ALL` --> `Start Rebase`

            ![Squash](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/682x553/4cea7fde523d47ab3862f847e9af70a5/_output_Squash.png)

        - 1-3. Commit --> DONE

            ![REBASECOMMIT](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/682x553/3a521971126b495dc779867ab37e01c4/_output_REBASECOMMIT.png)

            ![rebasedone](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/682x553/6aa516a8d1b6f053fb01e9f3cae02912/_output_rebasedone.png)

    2. `Combine to one commit`
        - 2-1. git show log
            - 在資料夾按右鍵，選 `Git Show Log`

                ![gitlog](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/475x292/fbb47e9842f5654f10798cd0d5e349ee/_output_gitlog.png)

        - 2-2. 選擇欲合併的 commit --> 按右鍵 -->`Combine to one commit`

            ![combine2one](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/1021x747/fc905d78143b1b0ea38d13ce0a1d213a/_output_combine2one.png)

        - 2-3. 修改訊息 --> `commit`

            ![combine2one_commit](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/583x556/2d0f36b129b22a30b5b4437304c3e2d5/_output_combine2one_commit.png)

## 6. 刪除 featur branch

> featur branch 不需推送到 SVN，feature 完成後就可以刪除

- A. 使用指令

    ```cmd
    git branch -d {branchname}
    ```

- B. 使用 TortoiseGit
    1. merge 後會提示刪除

        ![removebranch](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/757x438/91b44b63eb26b387aabb75e592a53421/_output_removebranch.png)

    2. branch 管理
        - 2-1. 資料夾右鍵 --> `TortoiseGit` --> `Switch/Checkout..`

            ![switchcheckout](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/782x426/230d7acb2d2334714b79e66ab0dbb9e7/_output_switchcheckout.png)

        - 2-2. 選擇其他 branch

            ![otherbranch](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/460x339/732d311e1a45acaaf2e99987c4fb02db/_output_otherbranch.png)

        - 2-3. DELETE

            ![delete](https://trello-attachments.s3.amazonaws.com/5834fc369c173c6969d43d41/714x452/f28926c4e0641acbc4d1586b11d55a11/_output_delete.png)

## 參考資料

1. [git-svn](https://git-scm.com/docs/git-svn)
2. [使用 git-svn 工具管理 SVN 專案](http://blog.lyhdev.com/2014/01/git-svn-svn.html)
3. [我的 git-svn 用法](http://plasma.z6i.org/archives/007533.html)
4. [git 與 git-svn 簡單教學](http://blog.chhsu.org/2010/05/git-git-svn.html)
