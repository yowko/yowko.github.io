---
title: "Shell 逐行處理檔案內容"
date: 2020-12-21T21:30:00+08:00
lastmod: 2020-12-21T21:30:31+08:00
draft: false
tags: ["Linux","macOS"]
slug: "shell-read-file-by-line"
---

## Shell 逐行處理檔案內容

這是之前處理 data migrate 時遇到的問題：所有 data 類型的資料調整都使用 script 來進行，目的是希望避免人為操作造成的錯誤

以 influxdb 為例，將需要執行的 script 加至檔案中

```test.iql
CREATE DATABASE test;
CREATE RETENTION POLICY testpolicy ON test DURATION 10d REPLICATION 1 DEFAULT;
```

接著使用 shell 讀取檔案內容並透過 curl 執行語法，但執行下來發現檔案最後一行總是沒執行到，經同事 debug 發現 bash script 有 bug，經過一番研究後，筆記一下  覺得很實用  

## 基本環境說明

1. macOS Catalina 10.15.7
2. docker image
   - influxdb 1.8.1
3. 原始語法

    ```bash
    for filename in ./*.iql; do
        echo "$filename"
        while IFS= read line
        do
            echo "$line"
            curl -XPOST http://${DB_USER}:${DB_PASSWORD}@${CONNECTION}/query --data-urlencode "q=$line"
        done < $filename 
    done
    ```

## 語法與說明

- 語法

    ```bash
    for filename in ./*.iql; do
        echo "$filename"
        # 讀取的那行 含有換行符號或是不為空字串
        while IFS= read -r line || [[ -n "$line" ]]; do
            if [[ -z "$line" ]]; then
                continue
            fi
            echo "$line"
            curl -XPOST http://${DB_USER}:${DB_PASSWORD}@${CONNECTION}/query --data-urlencode "q=$line"
        done < $filename
    done
    ```

- 參數說明

    - `-r`

        > 是否可讀

    - `[[ -n "$line" ]]`

        > 檢查是否不為空字串 (如果 不是空字串 就回傳 `true`)

    - `[[ -z "$line" ]]`

        > 檢查是否為空字串 (如果 空字串 就回傳 `true`)

        這是另外被抓出的 bug：如果檔案的多個 script 間有空白行，也會被拿去執行，加上這行就是為了避開空白行執行失敗的問題

## 心得

雖然 Shell Scripts 寫了一陣子，但依舊寫得零零落落，還是搞不清楚 `(())`、`[[]]`、`[]` 的差別，也常卡在忘了加 `空白` 或是要不要加 `""` XD

幸虧同事很強大，經常寫出令人讚為觀止的 script，有強者可以學習真好

## 參考資訊

1. [第十二章、學習 Shell Scripts](http://linux.vbird.org/linux_basic/0340bashshell-scripts.php)
