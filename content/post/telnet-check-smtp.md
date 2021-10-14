---
title: "使用 telnet 檢查 SMTP 是否正常提供服務"
date: 2018-02-24T00:38:00+08:00
lastmod: 2021-10-14T00:38:05+08:00
draft: false
tags: ["Tools"]
slug: "telnet-check-smtp"
aliases:
    - /2018/02/telnet-check-smtp.html
---
## 使用 telnet 檢查 SMTP 是否正常提供服務

近期手上的重要專案到了要上線的最終階段，在正式對外服務前，Server 間的相關設定都需要一一確認與驗證，其中關於 mail server (SMTP) 這塊，因為是全新的 server，沒有額外安裝可以用來測試的工具也沒有其他 application 可以用來測試，這時 Windows 內建的基本工具 telnet 就成為最佳幫手了

而我因為好一陣子沒用生疏不少，趁著這次機會再複習一下，順手紀錄一下用法

## 檢查步驟

1. 使用 telnet 連線至 mail server

    * 方法一：

        * 開啟 `cmd` 並執行 telnet

            ![1cmd](https://user-images.githubusercontent.com/3851540/36604375-880dada8-18f8-11e8-82c9-bb4fbb7e8e65.png)

            ![2telnet](https://user-images.githubusercontent.com/3851540/36604377-8839c28a-18f8-11e8-8e60-bc4ce79c2f0f.png)

        * 連線至 mail server

            * 指令

                ```cmd
                o {mail server} 25
                ```

            * 範例

                ```cmd
                o mail.yowko.com 25
                ```

                ![3connect](https://user-images.githubusercontent.com/3851540/36604378-88607a38-18f8-11e8-87dc-8e4d2e1e63e4.png)

            * 已連線

                ![4connected](https://user-images.githubusercontent.com/3851540/36604379-88854a16-18f8-11e8-8baa-222f329058c2.png)

    * 方法二：

        * 開啟 `cmd` 並使用 telnet 連線 mail server

            * 指令

                ```cmd
                telnet {mail server} 25
                ```

            * 範例

                ```cmd
                telnet mail.yowko.com 25
                ```

                ![5telnetsmtp](https://user-images.githubusercontent.com/3851540/36604381-88ab0a4e-18f8-11e8-9ad8-b8c91e3e828b.png)

        * 已連線

            ![4connected](https://user-images.githubusercontent.com/3851540/36604379-88854a16-18f8-11e8-8baa-222f329058c2.png)

2. 檢查 mail server 是否能正確回應

    > 讓 mail server 向發出指令的電腦說 `Hello`

    ![6helointro](https://user-images.githubusercontent.com/3851540/36604382-88d0baaa-18f8-11e8-8038-b0bc0868d630.png)

    * 使用 `EHLO` (推薦)

        > 這是 `HELO` 的擴充指令，因為 `HELO` 原始設計並沒有提供支援 protocal (e.g. TLS)的資料，詳細說明可以參考 [What is the difference between the HELO/EHLO commands?](http://en.redinskala.com/what-is-the-difference-between-the-heloehlo-commands/)

        * 可以只用 `EHLO`

            ```cmd
            EHLO
            ```

            ![7-0ehlo](https://user-images.githubusercontent.com/3851540/36604383-88f6d2a8-18f8-11e8-8fbc-7b678a8ca365.png)

        * 也可以加上 mail server

            ```cmd
            EHLO {mail server}
            ```

            ![7ehlo](https://user-images.githubusercontent.com/3851540/36604366-86741892-18f8-11e8-9da2-0c66c39a5192.png)

    * 使用 `HELO`
        * 可以只用 `HELO`

            ```cmd
            HELO
            ```

            ![8-0helo](https://user-images.githubusercontent.com/3851540/36604367-869eb94e-18f8-11e8-85d0-4aa8b21c72b2.png)

        * 也可以加上 mail server

            ```cmd
            HELO {mail server}
            ```

            ![8helo](https://user-images.githubusercontent.com/3851540/36604368-86c538f8-18f8-11e8-8b13-88700a719e28.png)

3. 指定寄件人

    > 部份 mail server 不支援指定寄件人，像是 gmail

    * 指令

        ```cmd
        MAIL FROM:{sender@domain.com}
        ```

    * 範例

        ```cmd
        MAIL FROM:yowko@yowko.com
        ```

        ![9mailfrom](https://user-images.githubusercontent.com/3851540/36604369-86eca4c4-18f8-11e8-8385-1121633ddf8d.png)

4. 指定收件人

    * 指令

        ```cmd
        RCPT TO:{recipient@domain.com}
        ```

    * 範例

        ```cmd
        RCPT TO:yowko@yowko.com
        ```

        ![10rcptto](https://user-images.githubusercontent.com/3851540/36604370-8712cea6-18f8-11e8-92d2-81a127e23d32.png)

5. 填寫信件內容

    * 輸入 `Data`

        > 會提示可以開始輸入信件內容，輸入完畢以 `<CRLF>.<CRLF>` 結尾( 按 `Enter` 加上 `.` 再加上一次 `Enter` )

        ![11data](https://user-images.githubusercontent.com/3851540/36604371-8739d1a4-18f8-11e8-9786-6e39dd6caf9c.png)

    * 信件標題

        > 輸入信件標題後，需按 `兩次 Enter` 與內文以一行空白隔開(沒有任何回應訊息)

    * 指令

        ```cmd
        Subject:{標題}
        ```

    * 範例

        ```cmd
        Subject:test subject
        ```

        ![12subject](https://user-images.githubusercontent.com/3851540/36604372-87618ece-18f8-11e8-87ea-55ea8b2df8d7.png)

    * 內文

        > 輸入完成後，需要 按 `Enter` 加上 `.` 再加上一次 `Enter` 當做結尾，成功會收到 mail server 列入處理 queue 的 250 訊息

        ![13done](https://user-images.githubusercontent.com/3851540/36604373-87b8aba0-18f8-11e8-82f3-d3d27046c88b.png)

6. 實際效果

    ![14result](https://user-images.githubusercontent.com/3851540/36604374-87e5ae48-18f8-11e8-9e2e-b683f7ca2a20.png)

## 心得

個人經驗：在 telnet 互動模式中如果有打錯字，使用 `Backspace` 並無法真正刪除，據我的操作，只要打錯字重新輸入當次指令一定都失敗

透過 telnet 想要發送精美的 mail 可能有些不切實際，但在用來測試 SMTP 有效性上就非常好用，除了 Windows 內建的 telnet client 之外不需要額外安裝其他軟體，尤其在資安防護較高的機房環境特別好用

## 參考資訊

1. [What is the difference between the HELO/EHLO commands?](http://en.redinskala.com/what-is-the-difference-between-the-heloehlo-commands/)
2. [如何使用 Telnet 來測試 SMTP 通訊](https://technet.microsoft.com/zh-tw/library/aa995718%28v=exchg.65%29.aspx)
3. [如何使用Telnet指令來測試SMTP是否正常運作？](http://blog.xuite.net/tim0718/note/23603837-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8Telnet%E6%8C%87%E4%BB%A4%E4%BE%86%E6%B8%AC%E8%A9%A6SMTP%E6%98%AF%E5%90%A6%E6%AD%A3%E5%B8%B8%E9%81%8B%E4%BD%9C%EF%BC%9F)
