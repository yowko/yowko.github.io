---
title: "將 Mitmproxy log 存至檔案"
date: 2020-06-21T18:30:00+08:00
lastmod: 2020-06-21T18:30:31+08:00
draft: false
tags: ["Linux","Netowrk"]
slug: "mitmproxy-log-to-file"
---

## 將 Mitmproxy log 存至檔案

之前筆記 [安裝 Mitmproxy](https://blog.yowko.com/mitmproxy) 提到雖然 Mitmproxy 提供了互動式且資訊豐富的存取紀錄，但這對於日常維運上不方便，一般沒有問題時不會有人去看 log，但覺得行為不如預期時就需要調閱 log 出來比對，所以趁這機會順便紀錄一下將 Mitmproxy 紀錄到檔案的方式

## 基本環境說明

1. Azure VM 標準 B2s (2 vcpu，4 GiB 記憶體)
2. CentOS 7.7
3. Mitmproxy v5.1.1

## 設定方式

之前筆記 [安裝 Mitmproxy](https://blog.yowko.com/mitmproxy) 在啟動 Mitmproxy 時使用

```bash
mitmproxy
```

改用 `mitmdump`

- 語法

    ```bash
    mitmdump  --listen-port {port} -w {file}
    ```

- 範例

    ```bash
    mitmdump -w result.log
    ```

## 使用方式及實際效果

- 使用方式

    ```bash
    curl -x localhost:8080 -k -L https://blog.yowko.com
    ```

- 實際效果

    ```bash
    cat result.log
    ```

    ![1log](https://user-images.githubusercontent.com/3851540/85227775-f7058800-b411-11ea-8778-f3bfb3d84dc5.jpg)

    > mitmdump console 也會同步列出對應的簡易資訊

    ![2consolelog](https://user-images.githubusercontent.com/3851540/85227776-f8cf4b80-b411-11ea-8f9d-1233466e0444.jpg)

## 心得

大致上應該還算滿足目標，但整體說來我覺得輸出至檔案中的 log 有太多資訊了，連完整 response 內容跟憑證資訊都包含在內，很容易讓 log 長太大

## 參考資訊

1. [How to extract all content of http/https traffic flows to a txt file from mitmproxy's console view](https://github.com/mitmproxy/mitmproxy/issues/3020#issuecomment-377229582)
2. [Tools](https://docs.mitmproxy.org/stable/overview-tools/)
3. [安裝 Mitmproxy](https://blog.yowko.com/mitmproxy)
