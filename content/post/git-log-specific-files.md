---
title: "使用 GIT 找出特定期間內特定檔案的修改資料"
date: 2018-02-26T23:17:00+08:00
lastmod: 2021-11-02T23:17:39+08:00
draft: false
tags: ["Git"]
slug: "git-log-specific-files"
aliases:
    - /2018/02/git-log-specific-files.html
---
## 使用 GIT 找出特定期間內特定檔案的修改資料

過去幾次 production 環境的更版都遇到 config 異常的狀況，輕則只影響特定功能，嚴重的讓整個 application 都 crash，根本原因是 config 的調整是由 developer 開發但實際上線內容都是 change manager 使用工具批次套用的結果，造成 developer 沒辦法即時知道實際上線的內容，所以想透過 git 的 log 撈出針對 config 有修改紀錄的 developer 在上線前夕可以預先知道該提醒哪些 developer 進行確認

站在客觀的角色，我不認為這是個好做法，只是我理想中相對好的做法就需要改變既有的流程，但流程可就不是說改就改的，需要時間好好討論大家的需求與顧慮，所以才想暫時透過 `git log` 列出 config 類檔案的修改人員清單

## GI 語法

原本有著雄心壯志想要將 `git log` 的所有參數都測試過，順便紀錄一下用法，不過 git log 的參數實在很多，最後決定只簡單地紀錄一下我這次有用到的部份，想了解有沒有更適合的參數可以參考 [官網](https://git-scm.com/docs/git-log)

* git log 語法

    ```cmd
    git log [<options>] [<revision range>] [[\--] <path>…]
    ```

* 實際使用語法

    ```cmd
    git log --no-merges --pretty=format:\"%an_%ae|%s|%ad\" --name-only --since='2018/2/20' -- *.props *.md > logs.txt
    ```

* 語法說明
  * `--no-merges`

    > 只列出只有一個 parent 的 commit，效果等同 `--max-parents=1`

  * `--pretty=format:\"%an_%ae|%s|%ad\`

    > 指定 log 輸出格式

    |選項|輸出說明|
    |--- |--- |
    |%H|該commit SHA-1 hash|
    |%h|該commit簡短的 SHA-1 hash|
    |%T|「樹（tree）」物件的 SHA-1 hash|
    |%t|「樹」物件簡短的 SHA-1 hash|
    |%P|parent commit 的 SHA-1 hash|
    |%p|parent commit 簡短的 SHA-1 hash|
    |%an|作者名字|
    |%ae|作者電子郵件|
    |%ad|作者日期（依據 --date 選項值而有不同的格式）|
    |%ar|作者日期，相對時間格式。|
    |%cn|commit 者名字|
    |%ce|commit 者電子郵件|
    |%cd|commit 者日期|
    |%cr|commit 者日期，相對時間格式。|
    |%s|標題|

  * `--name-only`

    > 只顯示檔案列表，不顯示詳細修改內容

  * `--since`

    > 指定特定時間日期(2018/2/21)或是相對日期(2.weeks)

  * `--`

    > 用來區隔指定特定路徑或是檔名與其他選項，需是最後一個參數(這邊指定過濾是 `.md` 與 `.props` 檔)，實測下不加 `--` 也可正常運行

  * `>`

    > 用來將結果輸出至檔案

## 測試案例

使用 [ASP.NET Core MVC](https://github.com/aspnet/Mvc) 當做範例

* 執行語法

    > ![1executecmd](https://user-images.githubusercontent.com/3851540/36676953-f0f6f256-1b47-11e8-93e9-8e52ea577172.png)

* 實際結果

    > ![2result](https://user-images.githubusercontent.com/3851540/36676955-f127a52c-1b47-11e8-8ee7-ba6ecad3810e.png)

## 心得

繞了一大圈也只能提醒 developer 做檢查，既無法解決根本問題也很沒效率，只是流程的調整牽涉極廣，如果冒然調整可以想見需要經過持續調整的陣痛期才能真正符合需求，我的看法是應急方案跟長期計劃應該並行，避免空窗期讓問題持續發生，也應該有計劃化地做通盤考量及改善

回到原始問題：developer 修改卻由 change manager 上線，過程中人為介入的成份太高，容易出現人為錯誤，我個人覺得透過 VSTS (TFS) 的 Release Manager 功能是最適合的，讓 developer 加入 config 設定，由主管 approve，最後一樣可以由 change manager 執行部署，主要工作都由系統進行並加上人工 review 降低錯誤機會

## 參考資訊

1. [官網](https://git-scm.com/docs/git-log)
2. [ASP.NET Core MVC](https://github.com/aspnet/Mvc)
