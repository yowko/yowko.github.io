---
title: "使用 grep 搜尋多個值"
date: 2021-12-17T01:30:00+08:00
lastmod: 2021-12-17T01:30:31+08:00
draft: false
tags: ["Tools","Linux"]
slug: "grep-multiple-values"
---

## 使用 grep 搜尋多個值

這是在 production 查資料時遇到的需求：想要從 log file 中同時查出符合 `關鍵字1` 或是 `關鍵字2` 的資料，相信熟悉的朋友一定可以馬上回答出這個需求的正確做法，無奈小弟太弱，加上不是很常用，每次需要時都要再查一次資料，雖然資料不難找，但需要用時都在 production，就是正急的時候，所以決定紀錄一下，臨時要用時可以更快找到

## 基本環境說明

1. macOS Monterey 12.0.1
2. grep (BSD grep, GNU compatible) 2.6.0-FreeBSD
3. 模擬的 log 資料

    - test.log

        ```log
        2021-12-17 09:00:38.192 [172][ERR][Demo.DomainService.Test.Application.Utilities.ExceptionInterceptor]General error
        message:"UserId:TW001|Name:Yowko|DepartmentId:D01"
        traceId:yowkotraceid
        2021-12-17 09:00:38.192 [172][ERR][Demo.DomainService.Test.Application.Utilities.ExceptionInterceptor]General error
        message:"UserId:SG001|Name:Tsai|DepartmentId:B01"
        traceId:tsaitraceid
        2021-12-17 09:00:38.192 [172][ERR][Demo.DomainService.Test.Application.Utilities.ExceptionInterceptor]General error
        message:"UserId:JP001|Name:Yua|DepartmentId:AV01"
        traceId:yuatraceid
        ```

## 使用方式

1. 使用多個 `-e`

    - 語法

        ```bash
        grep -e {關鍵字1}[ -e {關鍵字2}] {檔案名稱}
        ```

    - 範例

        ```bash
        grep -e Yowko -e Yua test.log  
        ```

    - 實際效果

        ![1eoption](https://user-images.githubusercontent.com/3851540/146521147-6b8d3f65-67d1-4371-82ed-bd0fde644377.png)

2. 使用 regular expression

    - 語法

        ```bash
        grep '{關鍵字1}[\|{關鍵字2}]' {檔案名稱}
        ```

    - 範例

        ```bash
        grep 'Yowko\|Yua' test.log  
        ```

    - 實際效果

        ![2rex](https://user-images.githubusercontent.com/3851540/146521165-5b8a1299-aeb4-4833-8a94-65683df7fe1e.png)

3. 使用 extended regular expressions

    - 使用 `-E`

        - 語法

            ```bash
            grep -E '{關鍵字1}[|{關鍵字2}]' {檔案名稱}
            ```

        - 範例

            ```bash
            grep -E 'Yowko|Yua' test.log  
            ```

        - 實際效果

            ![3Eoption](https://user-images.githubusercontent.com/3851540/146521168-44d6ae5e-214b-4e07-bafb-c58c2b467810.png)

    - 使用 `egrep`

        > 搜尋結果沒有高亮顯示

        - 語法

            ```bash
            egrep '{關鍵字1}[|{關鍵字2}]' {檔案名稱}
            ```

        - 範例

            ```bash
            egrep 'Yowko|Yua' test.log  
            ```

        - 實際效果

            ![4egrep](https://user-images.githubusercontent.com/3851540/146521173-8d2989b2-5cb4-46e9-a01c-a4f95c6d5af0.png)

## 心得

身為偷懶的工程師，我不會選用多個 `-e`，用傳統 regular expression 個人覺得有點難看或者該說容易讓真正要搜尋的內容失焦，而 `egrep` 則是因為結果沒高亮  我也不會選，所以原則上  我大多都是使用 extended regular expressions (`-E`)，不過還是看使用當下的情境來選擇

如果使用時遇到關鍵字可能會被誤判為 grep 的 option，可以在 `option` 與 `關鍵字` 間加上 `--` 來區隔，例： `grep -E -- 'Yowko|Yua' test.log`

## 參考資訊

1. [How do I grep for multiple patterns with pattern having a pipe character?](https://unix.stackexchange.com/a/37316)
