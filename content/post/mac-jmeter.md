---
title: "在 Mac 上安裝 JMeter"
date: 2019-08-28T21:30:00+08:00
lastmod: 2019-08-28T21:30:31+08:00
draft: false
tags: ["JMeter"]
slug: "mac-jmeter"
---

## 在 Mac 上安裝 JMeter

之前只在 Windows 安裝過 JMeter，沒有 Mac 上的 JMeter 使用經驗，趁著最近專案需要順手紀錄一下遇到的問題

- 關於 JMeter

    > Apache JMeter 是一個 Apache 專案，目的是用來作 load test 工具，可以提供於分析和測量各種服務的性能，主要目標是 Web application。 JMeter 也可以用來進行 JDBC數據庫連接，FTP，LDAP，WebService，JMS，HTTP，一般 TCP 連線和 OSnative processes 的單元測試工具。

## 基本環境說明

1. macOS Mojave 10.14.6
2. Java SE 11.0.1 / openJDK 12.0.2
3. Java SE 8 (1.8.0_192)
4. JMeter 5.1.1

## 基本安裝流程

> 兩個方法各有好壞，擇一即可

1. 手動安裝
   - 下載並解壓 [Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi)
   - 執行  `apache-jmeter-5.1.1/bin/jmeter.sh`

2. 使用 homebrew

    ```bash
    brew install jmeter
    ```

    > 會自動加入環境變數，不用指定執行路徑

* 順利啟動

    ![jmeter](https://user-images.githubusercontent.com/3851540/63869019-126a5700-c9ea-11e9-9b08-3d4b366f73f0.png)

## 遇到的問題

我個人比較懶，理所當然挑的是使用 homebrew 來安裝，不過使用時卻遇到無法從 View Results Tree 中看到 http request 的 Response data (畫面會 hang 住)，但手動安裝的環境中並沒有遇到相同問題

1. 錯誤訊息

    ```txt
    Uncaught Exception java.lang.ClassCastException: class javax.swing.text.AbstractDocument$DefaultDocumentEventUndoableWrapper cannot be cast to class javax.swing.text.AbstractDocument$DefaultDocumentEvent (javax.swing.text.AbstractDocument$DefaultDocumentEventUndoableWrapper and javax.swing.text.AbstractDocument$DefaultDocumentEvent are in module java.desktop of loader 'bootstrap'). See log file for details.
    Uncaught Exception java.lang.ClassCastException: class javax.swing.text.DefaultStyledDocument cannot be cast to class jsyntaxpane.SyntaxDocument (javax.swing.text.DefaultStyledDocument is in module java.desktop of loader 'bootstrap'; jsyntaxpane.SyntaxDocument is in unnamed module of loader org.apache.jmeter.DynamicClassLoader @2eee9593). See log file for details.
    ```

2. 錯誤截圖

    ![2hang](https://user-images.githubusercontent.com/3851540/63869020-1302ed80-c9ea-11e9-933c-a61ff128aced.png)

    ![3error](https://user-images.githubusercontent.com/3851540/63869022-1302ed80-c9ea-11e9-9e0b-1f7c9b2f1914.png)

3. 解決方式

    > 將 JDK or JRE 降版至 1.8 (有 [網友](https://stackoverflow.com/a/47572280/3600583) 說 JMeter 不支援 JAVA 9，但我覺得應該只是誤會，我直接下載就可以用，個人不負責猜測只是 homebrew 包的版本不支援而已)

    - 列出可用 java 版本

        ```bash
        /usr/libexec/java_home -V
        ```

        ![4javalist](https://user-images.githubusercontent.com/3851540/63870090-dcc66d80-c9eb-11e9-8a95-71aa292725ad.png)

    - 設定 java 版本

        ```bash
        export JAVA_HOME=$(/usr/libexec/java_home -v 1.8.0_192)
        ```

## 心得

這次在 Mac 上使用 JMeter 可謂是踩雷不斷呀，目前先紀錄無法看到 GUI 無法正確呈現資訊的狀況，接著會持續紀錄這次使用 Mac 版 JMeter 遇到的點點滴滴，供日後使用參考

## 參考資訊

1. [Apache JMeter](https://www.apache.org/)
2. [Homebrew Formulae - jmeter](https://formulae.brew.sh/formula/jmeter)
3. [How to resolve updating GUI error in JMeter](https://stackoverflow.com/questions/47571351/how-to-resolve-updating-gui-error-in-jmeter)
