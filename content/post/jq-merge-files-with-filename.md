---
title: "使用 jq 將多個檔案內容組成一份 key/value json"
date: 2021-12-17T00:30:00+08:00
lastmod: 2021-12-17T00:30:31+08:00
draft: false
tags: ["Tools","jq","Linux"]
slug: "jq-merge-files-with-filename"
---

## 使用 jq 將多個檔案內容組成一份 key/value json

之前筆記 [使用 jq 達成覆寫相同 json key 的效果](/jq-merge-json) 紀錄到使用 jq 來處理合併兩個檔案內容時：相同的 key 覆寫的做法，今天要來紀錄更一步的挑戰：將不同 module 的 config 整理成一份以 module name 為 key，個別 config 內容為 value 的 json

- config 結構

    - moduleA

        - appsettings.json

            ```json
            {
                "redisconnection": "127.0.0.1:6379, password=pass.123"
            }
            ```

    - moduleB

        - appsettings.json

            ```json
            {
                "mongodbconnection": "127.0.0.1:27017"
            }
            ```

- 期望結果

    ```json
    [
        {
            "key": "moduleA",
            "value": {
                         "redisconnection": "127.0.0.1:6379, password=pass.123"
                     }
        },
        {
            "key": "moduleb",
            "value": {
                         "mongodbconnection": "127.0.0.1:27017"
                     }
        }
    ]
    ```

## 基本環境說明

1. macOS Monterey 12.0.1
2. jq-1.6

## 使用方式

- 語法

    ```bash
    jq -n 'reduce inputs as $s (.; .[input_filename]= $s)' ./**/{json file name} | jq --arg reg_exp ".*/(?<module>.*)/{json file name}" 'to_entries|map({key:(.key|match($reg_exp).captures[0].string),value:(.value)})'
    ```

- 範例

    ```bash
    jq -n 'reduce inputs as $s (.; .[input_filename]= $s)' ./**/appsettings.json | jq --arg reg_exp ".*/(?<module>.*)/appsettings.json" 'to_entries|map({key:(.key|match($reg_exp).captures[0].string),value:(.value)})'
    ```

- 語法說明：

    - `jq -n 'reduce inputs as $s (.; .[input_filename]= $s)' ./**/appsettings.json`

        > - `-n` 用來建立 json 結構
        > - 當前目錄的子資料夾中的所有 appsettings.json 做為 `inputs`
        > - 使用 `reduce` 函數遍巡 `inputs` 取得內容為 `$s`
        > - `.` 將 `輸入的內容` 直接當做 `輸出的內容`，並將從 `inputs` 取得的內容 (`$s`) 設定給 `[input_filename]` 為 key 做為 value，`reduce` 函數會將每個設定後的結果疉加起來

        - 實際效果

            ```json
            {
              "./moduleA/appsettings.json": {
                "redisconnection": "127.0.0.1:6379, password=pass.123"
              },
              "./moduleB/appsettings.json": {
                "mongodbconnection": "127.0.0.1:27017"
              }
            }
            ```

    - `| jq --arg reg_exp ".*/(?<module>.*)/{json file name}" 'to_entries|map({key:(.key|match($reg_exp).captures[0].string),value:(.value)})'`

        > - `|` 將前面處理的結果做為 input 餵給下個動作
        > - `--arg reg_exp ".*/(?<module>.*)/{json file name}"` 設定 jq 變數 `reg_exp` 內容為 `".*/(?<module>.*)/{json file name}"`
        > - `to_entries` 將 object 轉型為 key/value 的陣列
        > - `map` 用來遍巡陣列內容逐一調整
        > - `{key:(.key|match($reg_exp).captures[0].string),value:(.value)}` 重新調整 json object，key 使用原本的 key 來加工：透過前面的 regular expression 取得 module name，value 則不調整

        - `to_entries` 效果

            - before

                ```json
                {
                  "./moduleA/appsettings.json": {
                    "redisconnection": "127.0.0.1:6379, password=pass.123"
                  },
                  "./moduleB/appsettings.json": {
                    "mongodbconnection": "127.0.0.1:27017"
                  }
                }
                ```

            - after

                ```json
                [
                  {
                    "key": "./moduleA/appsettings.json",
                    "value": {
                      "redisconnection": "127.0.0.1:6379, password=pass.123"
                    }
                  },
                  {
                    "key": "./moduleB/appsettings.json",
                    "value": {
                      "mongodbconnection": "127.0.0.1:27017"
                    }
                  }
                ]
                ```

        - 完整效果

            - before

                ```json
                {
                  "./moduleA/appsettings.json": {
                    "redisconnection": "127.0.0.1:6379, password=pass.123"
                  },
                  "./moduleB/appsettings.json": {
                    "mongodbconnection": "127.0.0.1:27017"
                  }
                }
                ```

            - after

                ```json
                [
                  {
                    "key": "moduleA",
                    "value": {
                      "redisconnection": "127.0.0.1:6379, password=pass.123"
                    }
                  },
                  {
                    "key": "moduleB",
                    "value": {
                      "mongodbconnection": "127.0.0.1:27017"
                    }
                  }
                ]
                ```

## 心得

這次是因為想要將 config 打進 consul k/v 中，所以才會衍生將多個 folder 下的 json file 整併的需求，剛好趁著這次機會再熟悉熟悉 jq 的語法，紀錄一下用法跟自己對語法的解讀避免日後又要花好久時間來重新理解

## 參考資訊

1. [使用 jq 達成覆寫相同 json key 的效果](/jq-merge-json)
2. [jq](https://stedolan.github.io/jq/manual/)
3. [jq merge two json files and create a key as the filename for each in merged file](https://stackoverflow.com/questions/59768530/jq-merge-two-json-files-and-create-a-key-as-the-filename-for-each-in-merged-file)
4. [Passing regular expression from variable in jq](https://stackoverflow.com/questions/38396689/passing-regular-expression-from-variable-in-jq)
