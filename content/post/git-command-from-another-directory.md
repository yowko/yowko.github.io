---
title: "Git 如何針對其他路徑下的 Repository 執行指令"
date: 2017-05-12T23:30:00+08:00
lastmod: 2018-09-19T20:00:27+08:00
draft: false
tags: ["Git","Jenkins"]
slug: "git-command-from-another-directory"
aliases:
    - /2017/05/git-command-from-another-directory.html
---
# Git 如何針對其他路徑下的 Repository 執行指令
一般來說我們都會直接在專案 repository 的所在目錄下執行 git 指令，今天同事問的這個問題就是因為不是一般的人為操作：同事使用 Jenkins build 出 binary 後打算將 dll copy 到不同 git repository 下進行版控，所以產生針對非當前路徑執行 git 指令的需求，所幸強大的 Git 也支援這樣的使用情境，就來看看該怎麼使用

## 方法一：

*   指定 `.git` folder 及 `work-tree`(工作目錄)
*   語法

    > git --git-dir={專案路徑}\.git\ --work-tree={專案路徑} {git 指令}

*   範例：

    > 在 c 磁碟對 d 磁碟下的 git repository 下指令

    1.  git status：確認是否有變更

        ```
        git --git-dir=D:\Git\0512\.git\ --work-tree=D:\Git\0512\ status
        ```

        ![1m1status](https://cloud.githubusercontent.com/assets/3851540/25996364/89d60c70-3749-11e7-8d37-cc2a7be715b0.png)

    2.  git add ：將變更加入 git index

        ```
        git --git-dir=D:\Git\0512\.git\ --work-tree=D:\Git\0512\ add .
        ```

        ![2m1add](https://cloud.githubusercontent.com/assets/3851540/25996359/88b63d4c-3749-11e7-8752-2f8bc26edf64.png)

    3.  git commit：將變更加入版控

        ```
        git --git-dir=D:\Git\0512\.git\ --work-tree=D:\Git\0512\ commit -m "remote done"
        ```

        ![3m1commit](https://cloud.githubusercontent.com/assets/3851540/25996361/896fd4fa-3749-11e7-8635-ba0995bfc150.png)

## 方法二：

*   使用 `-C` 參數指定 work-tree(工作目錄)
*   語法

    >git -C {專案路徑} {git 指令}

*   範例：
    
    > 在 c 磁碟對 d 磁碟下的 git repository 下指令

    1.  git status：確認是否有變更

        ```
        git -C D:\Git\0512 status
        ```
        
        ![4m2status](https://cloud.githubusercontent.com/assets/3851540/25996366/8b2e8782-3749-11e7-8333-2fb87f8077bd.png)

    2.  git add ：將變更加入 git index

        ```
        git -C D:\Git\0512 add .
        ```
        
        ![5m2add](https://cloud.githubusercontent.com/assets/3851540/25996365/8b2e4466-3749-11e7-8ec9-eaecbda4ac31.png)

    3.  git commit：將變更加入版控

        ```
        git -C D:\Git\0512 commit -m "remote ok"
        ```
        
        ![6m2commit](https://cloud.githubusercontent.com/assets/3851540/25996370/8ec7d1dc-3749-11e7-98ee-631b4711ca85.png)


# 參考資訊
1.  [Run `git commit` from another directory](http://stackoverflow.com/questions/22691575/run-git-commit-from-another-directory)
