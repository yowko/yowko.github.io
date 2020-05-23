---
title: "清除 Git Commit 紀錄"
date: 2020-05-23T21:30:00+08:00
lastmod: 2020-05-23T21:30:31+08:00
draft: false
tags: ["Git"]
slug: "clean-git-commit-history"
---

## 清除 Git Commit 紀錄

為了加快 Git clone 的速度，在之前筆記 [加快大型 GIT Repository 下載速度(指定 depth)](https://blog.yowko.com/git-clone-depth/) 曾經紀錄到使用 `--depth` 參數，用來限縮 clone 的資料量，只是預設情境下大家不會刻意加上這個參數，都是被 clone 時間嚇個幾次才會加，有時不免還是會忘記XD

最近專案告了個段落，所以決定將開發時期的 commit history 都清除，來個重新開始，順手紀錄一下以供參考

之前有類似需求，我都會直接刪除整個 `.git` 資料夾，不過近期的專案都有好幾個 submodule，為了避免重新 add submodule 的麻煩以及可能的錯誤，所以才改用別的方式

## 基本環境說明

1. macOS Catalina 10.15.4
2. git version 2.23.0
3. 以 gitlab 為例，原 commit 紀錄如下

    ![1original](https://user-images.githubusercontent.com/3851540/82733251-33779280-9d45-11ea-905b-3ef42a107dce.jpg)

## 操作流程

1. gitlab ui 操作解除 repo master protected (為了可以 force push)

    > gitlab 與 github 皆有類似保護 master 不被 force push 的機制

    ![2unprotected](https://user-images.githubusercontent.com/3851540/82733253-35d9ec80-9d45-11ea-9bb7-621d2e463da0.jpg)

2. 在 master  上打 tag 留下舊資料 (optional)

    > 這個動作是為了需要查找過去 commit 而做的 (觀察期後沒有使用需求，即可刪除該 tag)，如果百分之百確認沒這個需求，這個步驟可以省略

    ```bash
    git checkout master && git tag final
    ```

3. 執行 squash

    > 建立一個全新且獨立的 branch `tmp`

    ```bash
    git checkout --orphan tmp && git commit -m "squash"
    ```

4. 取代 master

    > 刪除舊 `master` 並用 `tmp` 替換 master

    ```bash
    git branch -D master && git branch -m master
    ```

5. 刪除 remote 其他 branch

    > 刪除 remote 上除了 `master` 外的所有 branch，避免殘留

    ```bash
    REMOTE="origin" && MASTER="master" && git branch -r |  grep "^  ${REMOTE}/" | sed "s|^  ${REMOTE}/|:|" | grep -v "^:HEAD" | grep -v "^:${MASTER}$" | xargs git push ${REMOTE}
    ```

6. force push

    > 用整理後新版的 master 取代 remote 的 master

    ```bash
    git push -f origin master && git push --tags
    ```

7. 清除 local 舊資料 (optional)

    > 清除 local 不需要的舊資料

    ```bash
    git gc --aggressive --prune=all
    ```

8. 完成示意

    ![3done](https://user-images.githubusercontent.com/3851540/82733254-36728300-9d45-11ea-8fce-caf9a2be7ecb.jpg)

    > 如果前面有打上 tag，就可以在 branch 選擇該 tag，再勾選 `Begin with the selected commit` 來顯示過去紀錄

    ![4tag](https://user-images.githubusercontent.com/3851540/82733255-370b1980-9d45-11ea-873c-25322bff893c.jpg)

## 心得

如果撇除需要保留 submodule 的緣故，我想我更喜歡直接刪除 `.git` 資料夾，感覺更直接XD

不過依我後知後覺的個性，要不是有這樣的需求  我也不會去嘗試不同做法，剛好多學了個處理方式囉

## 參考資訊

1. [stephenhardy/git-clearHistory](https://gist.github.com/stephenhardy/5470814)
2. [加快大型 GIT Repository 下載速度(指定 depth)](https://blog.yowko.com/git-clone-depth/)
