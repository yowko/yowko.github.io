---
title: "[JMeter] 跨 Thread 取得變數內容 - 設定 Load Test 的前置動作"
date: 2019-09-11T01:30:00+08:00
lastmod: 2020-12-11T01:30:31+08:00
draft: false
tags: ["JMeter"]
slug: "jmeter-set-get-variable-cross-thread"
---

## [JMeter] 跨 Thread 取得變數內容 - 設定 Load Test 的前置動作

之前筆記 [[JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數](/jmeter-decrypt-to-parameter/) 紀錄到使用 JSR223 PostProcessor 來針對加密的 response body 做解密後再設定為 JMeter 的參數供後續其他的 Http Request 當做 header，雖然已經可以符合核心目的，但用心檢查會發現存在個大缺失：Login 與目標 Api 在相同 Thread Group 下，這就表示我們修改的 Thread 數會同時套用在 Login 與目標 Api，這樣一來就變成同時進行了 Login 與目標 Api 的壓測，如果這是實際我們想要的測試情境也就沒問題，不過若只是想單獨取得目標 Api 的效能數據就需要做些調整了

![1issue1](https://user-images.githubusercontent.com/3851540/64711506-48c8cb80-d4ec-11e9-8337-98a68d770855.png)

![2issue2](https://user-images.githubusercontent.com/3851540/64711507-48c8cb80-d4ec-11e9-88b7-58eb2a6f13a0.png)

剛好趁這個機會順手紀錄一下設定方式，大致流程與設定方式會延續 [[JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數](/jmeter-decrypt-to-parameter/)

## 基本環境說明

1. macOS Mojave 10.14.6
2. JMeter 5.1.1
3. 基本設定請參考 [[JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數](/jmeter-decrypt-to-parameter/)

## JMeter 設定

1. 加入 setUp Thread Group

    > 讓目標 API 在實際測試前先執行 setUp Thread Group 的內容，進行暖機

    ![3setupthread](https://user-images.githubusercontent.com/3851540/64711508-48c8cb80-d4ec-11e9-9421-f7fcdad45ec3.png)

2. 將 Login 的相關設定移至 setUp Thread Group 中

    > 可以將之前設定的 Logic Controller 整個移過來即可

    ![4movelogin](https://user-images.githubusercontent.com/3851540/64711509-48c8cb80-d4ec-11e9-9a8f-cb1984eb628c.png)

3. 修改 JSR223 PostProcessor 中 JMeter 參數設定方式

    > property 才能跨 thread 存取

    - 將原有的 variable 設定

        ```java
        vars.put("token", json.getString("Token"));
        ```

    - 修改為 property 設定

        ```java
        props.put("token", json.getString("Token"));
        ```

4. 修改 HTTP Header Manager - Create User 取用

    - 原有說定

        ```java
        ${token}
        ```

    - 新設定

        ```java
        ${__property(token)}
        ```

5. 建議將 View Results Tree 移至 Test Plan 下

    > 可以同時顯示多個 thread 的執行結果

    ![5result](https://user-images.githubusercontent.com/3851540/64711511-49616200-d4ec-11e9-8ccc-97594d3be5cd.png)

## 心得

![1issue1](https://user-images.githubusercontent.com/3851540/64711506-48c8cb80-d4ec-11e9-8337-98a68d770855.png)

- 修改前：多個 thread 執行，Login 與 Create User 都會執行多次

    ![2issue2](https://user-images.githubusercontent.com/3851540/64711507-48c8cb80-d4ec-11e9-88b7-58eb2a6f13a0.png)

- 修改後：多個 thread 執行，僅 Create User 執行多次，Login 只有一次

    ![5result](https://user-images.githubusercontent.com/3851540/64711511-49616200-d4ec-11e9-8ccc-97594d3be5cd.png)

設定本身很簡單，但官網上的文件我暫時還沒上手該怎麼看懂XD

## 參考資訊

1. [[JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數](/jmeter-decrypt-to-parameter/)
