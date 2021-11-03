---
title: "Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event) - PART 2"
date: 2017-03-12T02:42:34+08:00
lastmod: 2021-10-13T00:42:34+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-pre-build-event-post-build-event-2"
aliases:
    - /2017/03/visual-studio-build-event.html
---
## Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event) - PART 2

之前文章 [Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event)](/visual-studio-pre-build-event-post-build-event) 介紹到該如何使用 visual studio 的兩種 build event 跟內建的巨集。後來同事問幾個相關問題，馬上就漏氣，查了資料立馬來紀錄一下

## 撰寫原則

1. 執行內容以 `()` 包著
    - 判斷式與執行內容皆在同一行
    - 正確範例
        - `IF 1 == 3  (echo "pass") ELSE (echo "fail")`
        - 印出正確資訊

            ![2correct](https://cloud.githubusercontent.com/assets/3851540/23689501/0fdb9312-03f6-11e7-9187-246ad4df5d44.png)
    - 錯誤範例
        - `IF 1 == 3  echo "pass" ELSE echo "fail"`
        - 無法印出正確資訊
            >![1wonge](https://cloud.githubusercontent.com/assets/3851540/23689500/0fdb0186-03f6-11e7-941a-d808fcb122c0.png)

2. 多行運算式需留意 `()` 位置
    - 判斷式與執行內容橫跨多行
    - `(` 在前一行運算式結尾，`)`在當行運算式開頭 --> 比照 javascript 慣例
    - 正確範例

        ```cmd
        IF 1 == 3  (
        echo "pass"
        ) ELSE (
        echo "fail"
        )
        ```

        ![3mutiplepass](https://cloud.githubusercontent.com/assets/3851540/23689503/0fe1310a-03f6-11e7-927d-032ca5b09800.png)
    - 錯誤範例
        - "(" 在運算式的開頭

            ```cmd
            IF 1 == 3
            (echo "pass"
            ) ELSE 
            (echo "fail"
            )
            ```

            ![4mutiplewrong](https://cloud.githubusercontent.com/assets/3851540/23689502/0fdcf59a-03f6-11e7-9daa-86db031c9dfa.png)

3. 邏輯判斷式與保留字間需留意間距
    - 判斷式與 `IF` 與 `(` 須有空格
        - 正確範例

            ```cmd
            IF 1 == 3 (
            ```

            ![5CORRECT](https://cloud.githubusercontent.com/assets/3851540/23689504/0fe22c90-03f6-11e7-9344-6af1f56938bf.png)
        - 錯誤範例

            ```cmd
            IF 1 == 3(
            ```

            ![6wrong](https://cloud.githubusercontent.com/assets/3851540/23689505/0fe36718-03f6-11e7-8fc1-1dcf5ed30497.png)
    - 判斷式本身不需有空格,下列兩者結果相同
        - 範例 1

            ```cmd
            IF 1 == 3 (
            ```

        - 範例 2

            ```cmd
            IF 1==3 (
            ```

        ![5CORRECT](https://cloud.githubusercontent.com/assets/3851540/23689504/0fe22c90-03f6-11e7-9344-6af1f56938bf.png)

4. 使用正確的邏輯比較運算字
    - 符號僅有 "==" 可以用，詳細內容請看下方
    - 正確範例

        ```cmd
        IF 1 NEQ 3 (
        echo "pass"
        ) ELSE (
        echo "fail"
        )
        ```

        ![7pass](https://cloud.githubusercontent.com/assets/3851540/23689506/0fff23b8-03f6-11e7-9dc2-761774fbdb78.png)

    - 錯誤範例 1

        ```cmd
        IF 1 <> 3 (
        echo "pass"
        ) ELSE (
        echo "fail"
        )
        ```

        ![8fail](https://cloud.githubusercontent.com/assets/3851540/23689507/1000fddc-03f6-11e7-8d17-0d34013b9e10.png)

## 邏輯比較運算子

operator|meaning|comment
:---|:---|:---
EQU | Equal,`==`| 可以用 `==`
NEQ | Not equal,`<>`,`!=`|不可替換
LSS | Less than, `<`|不可替換
LEQ | Less than or Equal,`<=`|不可替換
GTR | Greater than ,`>`|不可替換
GEQ | Greater than or equal, `>=`|不可替換

## 心得

經過這次的再釐清，觀念比較清楚，希望下次不會再被問倒XD，

## 參考資訊

1. [How to run Visual Studio post-build events for debug build only](http://stackoverflow.com/questions/150053/how-to-run-visual-studio-post-build-events-for-debug-build-only)
2. [IF](https://ss64.com/nt/if.html)
