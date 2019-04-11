---
title: "使用 Git Attributes 的合併策略(Merge Strategy) 避免特定檔案被變更"
date: 2017-05-14T12:31:00+08:00
lastmod: 2018-09-22T12:31:40+08:00
draft: false
tags: ["Git"]
slug: "git-attributes-merge-strategy"
aliases:
    - /2017/05/git-attributes-merge-strategy.html
---
# 使用 Git Attributes 的合併策略(Merge Strategy) 避免特定檔案被變更
公司專案在經由 CI - Continuous integration (Jenkins) 成功 build 後會將 build 結果全部 commit 回 SVN，以確保最後的 production code 是有被版控的，如果有退版需求時也可以快速完成。

最近公司正在如火如荼地將 SCM 從 SVN 改為 GIT，在之前的文章裡也有提到，過程中難免遇到一些問題需要克服，其中一個狀況是公司政策 - production code 需要版控這件事不會因為 SVN 或是 Git 而有不同，以前在 SVN 的做法是會先將 repository 的內容全部刪除，再將最新結果全部 commit，確保最新結果完整被版控不會因為 conflict 造成版控有遺漏，加上現在程式更版都是自動化 CD - Continouous Deployment 作業，如果出現 conflict 會造成部署錯誤程式，影響很大，因此需要調整 git 的合併策略，就來看看可以怎麼使用 git attribute 達到目的吧

## 關於 Git Attributes

透過在專案目錄下的 `.gitattributes` 來設定該專案專屬的特性及行為，其中包含：

1.  Binary Files (二進位檔案的處理方式)

    *   指定特定附檔名的檔案為 binary，讓 git 使用 binary 的方式來比較檔案

        > `*.jpg binary`

    *   將特定的 binary 轉為文字來比對

        > `*.doc diff=word`

2.  Keyword Expansion(關鍵字擴充)

    > 可以自訂關鍵字置換的規則，用來避免特定關鍵字被 commit or checkout

    *   smudge


    *   checkout 時觸發

        ![smudge](https://git-scm.com/figures/18333fig0702-tn.png)

    *   clean


        *   staged(commit 前將變更納入修改清單中的步驟)時觸發

            ![clean](https://git-scm.com/figures/18333fig0703-tn.png)

3.  Exporting Your Repository(匯出 git 儲存庫)

    > 將 repository 匯出壓縮檔時可以忽略特定目錄或是執行文字取代

    *   export-ignore(特定資料夾，正常版控但匯出時忽略)

        > `testdata/ export-ignore` -->匯出時略過 `testdata`

    *   export-subst

        ```
        echo 'Last commit date: $Format:%cd$' > LAST_COMMIT.txt
        echo "LAST_COMMIT export-subst" >> .gitattributes
        git add LAST_COMMIT.txt .gitattributes
        git commit -am 'adding LAST_COMMIT.txt file for archives'
        ```

        > 會將 `$Format:%cd$` 取代為最後 commit 的時間

4.  Merge Strategies(合併策略)

    > 可以用來針對特定檔案設定不同的合併策略

    - web.config merge=ours 
        
        >出現衝突時永遠使用我的版本


## 合併策略(Merge Strategies)

1.  resolve

    > 使用 3-way merge 演算法來解決 merge 兩個 head (當前 branch 及另一個 branch)的情境，較快速且安全，但對於交叉合併效能較差

2.  recursive

    > pull 及 merge 時的預設做法

    - 使用 3-way merge 演算法來解決 merge 兩個 head 的情境，可有效減少合併衝突

        *   ours

            > 衝突時使用 ours 版本，binary 就都使用 ours

        *   theirs

            > 衝突時使用 theirs 版本，binary 就都使用 theirs

        *   patience

            > 較耗時，會忽略像是符號造成的衝突

        *   diff-algorithm=[patience|minimal|histogram|myers]

            > 指定 diff 的演算法，避免符號造成的衝突，依不同算法耗時也會不同

        *   ignore-space-change|ignore-all-space|ignore-space-at-eol

            > 指定對於空白字元的處理方式

        *   renormalize

            > 忽略行末比對

        *   no-renormalize

            > 開啟行末比對

        *   no-renames

            > 停用 rename 的檢查

        *   find-renames[=&lt;n&gt;]

            > 開啟 rename 檢查，可指定相似性門檻值，預設是開啟的


        *   rename-threshold=&lt;n&gt;

            > 排除 rename 相似性檢查的同義詞

        *   subtree[=&lt;path&gt;]

            > 指定排除路徑來進行子目錄比對


3.  octopus

    > 適用於多個 head 合併，但不能有衝突，是 pull or merge 二個以上 branch 的預設策略

4.  ours

    > 會使用 ours 做為最終版本

5.  subtree

    > 將 branch 指定為子目錄的方式來合併

## 設定合併策略(Merge Strategies)

> 依同事的描述：永遠使用最新版本，那可以使用下列設定

1.  開啟合併策略功能
    *   檢查是否開啟
        
        ```
        git config --global -l
        ```

        ![1check](https://cloud.githubusercontent.com/assets/3851540/26031283/d76d298a-389f-11e7-9507-75f671fb2a12.png)

    *   如果沒有`merge.ours.driver=true`則需開啟功能

        ```
        git config --global merge.ours.driver true
        ```

        ![2addconfig](https://cloud.githubusercontent.com/assets/3851540/26031284/d76d3c54-389f-11e7-8883-062d46fdb649.png)

2.  加入 `.gitattributes` 檔案並將合併策略加入 `.gitattributes` 內容中

    ```
    echo "1.txt merge=ours" >.gitattributes
    ```

3.  設定前後比較

    *   設定前

        *   出現衝突

            ![3conflict](https://cloud.githubusercontent.com/assets/3851540/26031282/d76c479a-389f-11e7-9848-2fbc534849b2.png)

        *   需要手動解決

            ![4munual](https://cloud.githubusercontent.com/assets/3851540/26031285/d7712544-389f-11e7-880f-93742d3d32d3.png)

    *   設定後

        *   永遠使用本地版本

            ![5automerge](https://cloud.githubusercontent.com/assets/3851540/26031280/d722ff4a-389f-11e7-89cc-6b5aa4c6bccd.png)

        *   檔案以 head branch 為主，不受其他 branch 影響

            ![6ours](https://cloud.githubusercontent.com/assets/3851540/26031281/d7549686-389f-11e7-8654-846db0083571.png)


## 注意事項
1.  設定 `ours` 會永遠使用 head branch 來當做最終版本，無論是否有衝突，其他 branch 的變更都會被捨棄
2.  測試結果只有 `ours` 可以直接使用，其他合併策略都需要配合自訂 script


# 參考資訊
1.  [8.2 Customizing Git - Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)
2.  [【Git】attribute, 合併後保留原本的設定檔](http://maxhu.logdown.com/posts/1586262)
3.  [merge strategy in .gitattributes not working](http://stackoverflow.com/questions/27920526/merge-strategy-in-gitattributes-not-working)
