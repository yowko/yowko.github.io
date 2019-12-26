---
title: "讓 iOS 裝置可以存取自訂 domain"
date: 2019-12-25T21:30:00+08:00
lastmod: 2019-12-26T21:30:31+08:00
draft: false
tags: ["iOS","Tools","macOS"]
slug: "ios-private-domain"
---

## 讓 iOS 裝置可以存取自訂 domain

近幾年行動裝置的瀏覽量佔比有愈來愈高的趨勢，身為 web 工程師也得朝著 mobile first 的方向做調整優化，所以在不同行動裝置上做測試也在所難免，為了避免 key 錯 ip、方便記憶，多數的專案也都採用 private domain 的做法，因此不可能為了 iOS 裝置測試方便就調整不同服務間的 connection string，想了幾種方式最後還是搬出老方法 - 透過某台開發環境的電腦架設 proxy server 來處理

iOS 沒有 hosts file 可以設定，同事說可以透過 jb 來設定，我沒有實際用過，留給大家自行嘗試看看囉

## 基本環境說明

1. macOS Catalina 10.15.1
2. iOS 13.3
3. docker 19.03.5
4. Mitmproxy 5
5. 在內網中使用自訂 domain :`yowko.test`

    > 開發環境透過修改 `etc/hosts`，將 `yowko.test` 指向特定內網 ip 獲取服務,iOS 不認得自訂 domain

    ![19cannotaccess](https://user-images.githubusercontent.com/3851540/71453405-0c348d80-27c6-11ea-87e9-66616d444088.png)

## 設定方式

1. 使用 `Mitmproxy` 在電腦上建立 proxy server

    > 這邊可以使用其他工具 (過去在 Windows 平台，我多是使用 fiddler 詳細資訊可以參考之前筆記 [使用fiddler 內建proxy 來截錄手機或是程式封包](https://blog.yowko.com/fiddler-proxy-collect-traffic/)，因為 mac 使用 fiddler 需要透過 mono 以方便性來看不是首選)

    - 透過 homebrew 安裝

        - 安裝指令

            ```bash
            brew install mitmproxy
            ```

        - 啟動 `Mitmproxy`

            - 僅啟動 proxy server

                ```bash
                mitmproxy
                ```

                > 畫面會顯示出所有 proxy 經手的資訊

                ![1mitmproxy](https://user-images.githubusercontent.com/3851540/71447427-5cc9cd80-2769-11ea-90a8-cd918b9a2d9b.png)

            - 啟動 proxy server 及 web ui

                > 會一併啟動 proxy server

                ```bash
                mitmweb
                ```

                > 有 web ui 顯示出所有 proxy 經手的資訊

                ![2mitmweb](https://user-images.githubusercontent.com/3851540/71447428-5d626400-2769-11ea-9d0f-e38e122a84c5.png)

                ![3mitmweb2](https://user-images.githubusercontent.com/3851540/71447429-5d626400-2769-11ea-90fa-837b5798e905.png)

    - 使用 docker

        > 其他使用方式，請參考 dockerhub - [mitmproxy/mitmproxy](https://hub.docker.com/r/mitmproxy/mitmproxy/)，預設 mitmproxy 會使用 stdout 輸出資訊，如果使用 `-d` daemon 模式啟動， container 啟動完成就會直接 shutdown 了

        ```bash
        docker run --rm -it -p 8080:8080 mitmproxy/mitmproxy
        ```

2. 將 iOS 裝連上與電腦共用的 wifi，並設定 proxy server 指向裝設 mitmproxy 的電腦 ip

    ![5wifi](https://user-images.githubusercontent.com/3851540/71447523-3ce6d980-276a-11ea-9504-92dc11ebdf05.PNG))

    ![6proxyserver](https://user-images.githubusercontent.com/3851540/71447431-5dfafa80-2769-11ea-9d8a-81ed64e77aa5.png)

    ![7proxyip](https://user-images.githubusercontent.com/3851540/71447432-5dfafa80-2769-11ea-80f5-b48db9010dbe.png)

3. 已可正常存取自訂 domain

    ![8yowkotest](https://user-images.githubusercontent.com/3851540/71447433-5dfafa80-2769-11ea-80ec-7cdbf7d0a34e.png)

4. 無法連線 https 網頁或是 apple 服務 (optional)

    > 雖然可以正常存取自訂 domain，但部份服務卻出現異常，主要是因為 mitmproxy 無法正確轉導 https 內容，此時可以透過信依 mitmproxy 憑證解決問題，或是裝置只在自訂 domain 測試使用，需要其他服務時停用 proxy

    ![9httpsfail](https://user-images.githubusercontent.com/3851540/71447434-5dfafa80-2769-11ea-909b-a41e1b5f7dca.png)

    - 在 iOS 上使用 `Safari` 連線至 `http://mitm.it` 下載並安裝 Mitmproxy 用憑證

        > 一定要透過 `Safari` 開啟網頁，否則無法進行安裝

        ![10mitmit](https://user-images.githubusercontent.com/3851540/71447435-5e939100-2769-11ea-9285-9e1a65649e3d.png)

        ![11download](https://user-images.githubusercontent.com/3851540/71447436-5e939100-2769-11ea-819f-19724efbecf7.png)

        ![12downloaded](https://user-images.githubusercontent.com/3851540/71447437-5e939100-2769-11ea-82a2-0f100aa78109.png)

    - 設定 --> 一般 --> 描述檔

        ![13description](https://user-images.githubusercontent.com/3851540/71447438-5f2c2780-2769-11ea-98d5-46e981bbdee1.png)

        ![14mitmdesc](https://user-images.githubusercontent.com/3851540/71447439-5f2c2780-2769-11ea-9729-999548f4d3ca.png)

        ![15install](https://user-images.githubusercontent.com/3851540/71447441-5f2c2780-2769-11ea-9c2a-2c24df11a80b.png)

        ![15-1warning](https://user-images.githubusercontent.com/3851540/71447440-5f2c2780-2769-11ea-8c3b-d12927dc9ede.png)

        ![16installed](https://user-images.githubusercontent.com/3851540/71447442-5fc4be00-2769-11ea-8344-4b15c001e62e.png)

    - 設定 --> 一般 --> 關於本機 --> 憑證信任設定

        > 啟用根憑證完整信任

        ![17roottrust](https://user-images.githubusercontent.com/3851540/71447443-5fc4be00-2769-11ea-9c0e-c84da3561dd9.png)

        ![18alerttrust](https://user-images.githubusercontent.com/3851540/71447444-5fc4be00-2769-11ea-86c9-fd33dddaa78f.png)

## 心得

這招是以前跟寫 app 的同事合作時常用來 debug 的方法，想不到過這麼多年了還是相當受用，非常感謝當年同事的指導

說實話我不清楚這個做法是不是最好，甚至是不是正確的，不過以結果來看的確可以解決當下的問題，提供大家參考，也請大家指教其他做法

## 參考資訊

1. [mitmproxy](https://mitmproxy.org/)
2. [mitmproxy/mitmproxy](https://hub.docker.com/r/mitmproxy/mitmproxy/)