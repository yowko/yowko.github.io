---
title: "Git reset 的三種模式( soft mixed hard )比較"
date: 2016-12-25T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Git"]
slug: "git-reset-type"
aliases:
    - /2016/12/difference-between-soft-mixed-hard-of-git-reset.html
    - /2016/12/git-reset-type/
---
## Git reset 的三種模式( soft mixed hard )比較

git reset 的三種主要模式(--soft, --mixed,--hard)，一直困擾著我不太確定知不知道其中差異，似懂非懂，最近剛好為公司同事進行 `Git` 教育訓練，藉機來把觀念釐清

## 情境

1. 建立 git repository
2. 新增 a.txt
3. 修改 a.txt 並 commit
4. 修改 a.txt 並新增 b.txt 後 commit

    ![1_SOURCE](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/952x642/fbb323daa95aa4d9da206edc0cb54143/_output_1_SOURCE.png)

- 原始檔案狀態

    ![filelifecycle](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/1023x654/ad3080ce58b146f8f0b5e343771a8a17/_output_gitlifecycle.png)

## 測試

![2_reset](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/958x635/577d02482f68c2f9b1c25ceb7435e6e6/_output_2_reset.png)

## 1. git reset --soft

    ![3_SOFT](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/675x476/f7ec2961e6fa3f652cb477e8d18c736f/_output_3_SOFT.png)

1. head 指向上一版

    ![4.softhead](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/952x832/b0744c92775133450dcf233aa04d8c51/_output_4.softhead.png)

2. index 未改變(`a.txt` 標記成`Modified`,`b.txt` 標記成`Added`)

    ![5_softindex](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/988x651/605a98cbf10f584a98973ca575576ce4/_output_5_softindex.png)

3. working tree 未改變(內容是新版)

4. 僅移除`commit`

    ![gitresetsoft](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/1023x654/c967b1f91050593912065d3d24535c55/_output_gitresetsoft.png)

## 2. git reset --mixed

> - 不指定時的預設行為
> - 保留檔案變更

1. head 指向上一版

    ![6.mixhead](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/952x832/1be06b9fa35b8138c356064d50461e14/_output_6.mixhead.png)

2. index 移除`staged`標記(表示不在 commit 的範圍)

    ![7_mixindex](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/988x651/b7867d26b415a3fdd5d04ea7d6c717fc/_output_7_mixindex.png)

    - 2-1. 以`a.txt`來看

        >保留`修改`，但沒有被納入 `staged`

    - 2-2. 以`b.txt`來看

        >保留`新增`，但是`untracked`

3. working tree 未改變(內容是新版)

4. 移除`commit`及`staged`

    ![gitresetmixed](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/1023x654/65554fd1cd534abda5bc57f6f8fe108e/_output_gitresetmixed.png)

## 3. git reset --hard

![8.HARDhead](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/675x476/96895d4a0bd4273025fd0e3645aab22c/_output_8.HARDhead.png)

1. head 指向上一版

    ![9.HARDhead](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/952x832/2aca1c9c357aa31b0cf185ba6a7f8b71/_output_9.HARDhead.png)

2. index 移除`staged`標記(表示不在 commit 的範圍，且因修改的內容被移除，所以也不會被標記`Modified`)

    ![10_hardindex](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/988x651/b46e17ab1c93049c32666342129b330a/_output_10_hardindex.png)

3. working tree 移除修改的內容

4. 完全回到上一版

    ![gitresethard](https://trello-attachments.s3.amazonaws.com/583c75c90173173906e0b4ce/1023x654/85b2502de294d4689dd8ffd8a3c604dc/_output_gitresethard.png)

## 結論

|名詞|解釋|
|---|---|
|head|所在位置|
|index|變更狀態紀錄|
|working tree|工作目錄|

|mode    |head    |index    |working tree|說明|
|---    |---    |:---    |---    |---|
|soft    |<span style='color:red'>changed</span>|<span style='color:green'>unchanged</span>|<span style='color:green'>unchanged</span>    |僅移除`commit`變成新版未 commit，內容仍是新版的|
|mixed    |<span style='color:red'>changed</span>|<span style='color:red'>changed</span>|<span style='color:green'>unchanged</span>|index 移除`staged`標記，變成`Modified`or `Untracked`，內容是新版的|
|hard    |<span style='color:red'>changed</span>|<span style='color:red'>changed</span>|<span style='color:red'>changed</span>|回到上一版版本，其間變更完全移除(接近 svn revert),內容及狀態皆是上一版|

操作 Repository：[GitHub](https://github.com/yowko/demo_git_reset)

整理後清楚多了，就怕結論是錯的XD，如果哪邊寫錯，要請大家多指教。

## 參考資料

1. [Git 學習筆記 (1)：安裝、選項設定、在本地使用 Git 工具](http://blog.miniasp.com/post/2013/08/18/Learning-Git-Part-1-Installation-Options-Tool-Usage-on-Local.aspx)
2. [30 天精通 Git 版本控管 (07)：解析 Git 資料結構 - 索引結構](http://ithelp.ithome.com.tw/articles/10134531)
3. [30 天精通 Git 版本控管 (05)：瞭解儲存庫、工作目錄、物件與索引之間的關係](http://ithelp.ithome.com.tw/articles/10133653)
