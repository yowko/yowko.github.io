---
title: "修改 Git Server Repository Branch 名稱"
date: 2017-07-25T23:10:00+08:00
lastmod: 2021-11-02T23:10:45+08:00
draft: false
tags: ["Git"]
slug: "git-rename-remote-branch"
aliases:
    - /2017/07/git-rename-remote-branch.html
---
## 修改 Git Server Repository Branch 名稱

公司的版控正在從 SVN 轉為 Git，過程中當然問題不斷，其中 Repository 的命名方式與 Branch 的用法都是大家討論的重點，但這是比較偏流程跟政策面的決定，一般工程師只需要努力試著上手 Git 觀念與實際操作即可 --- 本來這麼想，結果未如所願呀

因為命名規則異動，就直接影響 Git Repository，原本 push 的 code 都需要調整，今天就來紀錄一下如何修改 Git Repository Branch 名稱

## 方法一：Rename local branch 再更新 server

1. 將 local branch 改名

    > `git branch -m {old_branch_name} {new_branch_name}`

2. 刪除 server 的舊 branch

    > `git push origin :{old_branch_name}`

3. 將新 branch puch 至 server

    > `git push origin {new_branch_name}`

    ![1method1](https://user-images.githubusercontent.com/3851540/28578942-c59d29ea-718d-11e7-8956-8f4d82f91460.png)

## 方法二：Rename local branch 再更新 server

1. 將 local branch 改名

    > `git branch -m {old_branch_name} {new_branch_name}`

2. 將新 branch push 至 server 並從 server 上刪除 branch

    > `git push origin {new_branch_name} :{old_branch_name}`

    ![2method2](https://user-images.githubusercontent.com/3851540/28578943-c5a01704-718d-11e7-89ad-8c3171e750c2.png)

## 方法三：使用遠端 branch 建新 branch 再更新 server

1. 以 server 上的舊 branch 建立新 branch

    > `git branch {new_branch_name} origin/{old_branch_name}`

2. 將新 branch push 至 server

    > `git push origin {new_branch_name}`

3. 刪除 server 上的舊 branch

    > `git push origin :{old_branch_name}`

    ![3method3](https://user-images.githubusercontent.com/3851540/28578945-c5ae5346-718d-11e7-8394-e5a91fee4742.png)

* 注意事項：

  * 以遠端建立的 new branch 預設 track origin old branch，所以刪除遠端 branch 後會出現 `the upstream is gone.`，需自行修復

        ![4fix](https://user-images.githubusercontent.com/3851540/28578944-c5abfdd0-718d-11e7-8980-180428babc34.png)

        > `git branch --unset-upstream`

  * 需自行刪除 local branch

        > `git branch -d {old_branch_name}`

## 心得

其實仔細觀察語法，會發現語法都很接近，尤其是方法一與方法二，原則上都一樣，只是方法二語法更加簡潔而已，可以依大家對語法的熟悉度自行選用

比較特別的是方法三，會需要額外刪除 old branch 跟 upstream branch 遺失的問題，雖然都是些小狀況，但如果沒有絕對必要還是建議使用方法一或方法二

## 參考資料
