---
title: "Bash 中的 try/catch"
date: 2021-12-30T00:30:00+08:00
lastmod: 2021-12-30T00:30:31+08:00
draft: false
tags: ["bash"]
slug: "bash-try-catch"
---

## Bash 中的 try/catch

前幾天在做某個測試性專案，過程中調整 mongodb 的 automation build script 時遇到個狀況：module 會在 init 時執行 create role 跟 user，但有些 role 與 user 會被重複建立而出錯，正統的做法當然是先檢查資料是否存在，不存在才執行建立，但是這樣的動作就直接影響到原始專案的內容與流程，跟測試性專案的出發點不符，所以想要簡單的避開 bash 的 error，直接忽略 error 繼續執行

簡單 google 後，發現寫法簡單，但為了加深印象這是簡單紀錄一下用法

## 基本環境說明

1. macOS Monterey 12.0.1
2. bash 3.2.57(1)-release-(x86_64-apple-darwin21)

## 使用方式

- 使用 `||`：`command1 || command2`

    > 當 command1 出錯就會接著執行 command2

    - 範例1：嘗試 cat 不存在的 `none.log`，最後會印出 `always success`

        ```bash
        cat none.log && echo "try success" || echo "always success"
        ```

        ![1fail](https://user-images.githubusercontent.com/3851540/147733262-689b2130-f027-43c8-8610-a78447b0570f.png)

    - 範例1：cat 存在的 `test.log`，會印出 `try success`

        ```bash
        cat test.log && echo "try success" || echo "always success"
        ```

        ![2pass](https://user-images.githubusercontent.com/3851540/147733264-7bef6abb-260b-43bf-88a2-40eff9bfb928.png)

    - command 是多行的情況

        - 語法

            ```bash
            {
                cat test.log &&
                echo "fail"
            } || {
                echo "success"
            }
            ```

- 使用 `set +e`

    > 忽略 command error

    ```bash
    set +e
    cat none.log
    echo "success"
    set -e
    ```

    ![3sete](https://user-images.githubusercontent.com/3851540/147733265-96db213f-93f9-4b85-81f9-c548bb3d0cea.png)

- 使用 trap

    - 語法

        > 遇到 `信號` 就 `執行語法`

        ```bash
        trap {執行語法} {信號}
        ```

    - 範例

        ```bash
        trap "echo success" EXIT
        cat none.log
        exit 64
        ```

    ![4trap](https://user-images.githubusercontent.com/3851540/147733266-a0180528-4234-41b8-a41d-438b2e95c250.png)

## 心得

bash 沒有 try/catch 的 function，但透過上述的幾種方式還是能達到目的

有人提到不該使用 `set +e` 因為這個設定全域的，可能造成 server 上其他正在執行的 bash 錯誤被吞掉

至於哪個方式比較好，我還在探索中，暫時還沒辦法給出結論

## 參考資訊

1. [Is there a TRY CATCH command in Bash](https://stackoverflow.com/a/22010339)
2. [利用 trap 執行所有 Bash 退出前該做的事情](https://cjk.aiao.today/trap-bash-exit/)
3. [Bash 脚本 set 命令教程](http://www.ruanyifeng.com/blog/2017/11/bash-set.html)
