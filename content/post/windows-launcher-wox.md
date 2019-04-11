---
title: "Windows 中強大的 launcher - Wox"
date: 2017-05-21T23:58:00+08:00
lastmod: 2018-09-22T09:59:46+08:00
draft: false
tags: ["Tools"]
slug: "windows-launcher-wox"
aliases:
    - /2017/05/windows-launcher-wox.html
---
# Windows 中強大的 launcher - Wox
參加 twMVC 活動偶然間看見 91 大在 live demo 的過程使用一個沒看過的工具，功能很強大，可以用來搜尋檔案，功能跟 everything 一樣，也可以當做計算機、又能翻譯，簡直比 Spotlight 還強了呀，後來 91 大在 TDD 課程中正式介紹了這套工具 - Wox 果然功能強大，又是 open source 軟體，功能部份可以透過安裝 plug-in 來強化

## 功能介紹

![intro](https://camo.githubusercontent.com/9db33546d3a905a9ad915e0948d3ba3f47f57b64/687474703a2f2f692e696d6775722e636f6d2f4474784e424a692e676966)

## 安裝方式

> wox 檔案搜尋功能依賴於 everything，部份功能是 python 所以 everything 跟 python 都是必需要安裝的

1.  基本需求

    *   .Net Framework 4.5.2 以上
    *   安裝 everything 並確認服務正常執行

        > 直接執行下面的 .exe 就會自動完成

    *   安裝 python 3，並加入環境變數 (%path%)

        > 直接執行下面的 .exe 就會自動完成

2.  安裝說明

    > [Wox](https://github.com/Wox-launcher/Wox/releases)，留意一下版本就好， lastest-release 才有提供安裝建議

3.  執行安裝


    *   everything.exe

        *   [Everything-1.3.4.686.x64-Setup.exe](https://github.com/Wox-launcher/Wox/releases/download/v1.3.183/Everything-1.3.4.686.x64-Setup.exe)
        *   [Everything-1.3.4.686.x86-Setup.exe](https://github.com/Wox-launcher/Wox/releases/download/v1.3.183/Everything-1.3.4.686.x86-Setup.exe)

    *   python.exe

        *   [python-3.5.1.exe](https://github.com/Wox-launcher/Wox/releases/download/v1.3.183/python-3.5.1.exe)

    *   wox.exe

        *   [Wox-1.3.183.exe](https://github.com/Wox-launcher/Wox/releases/download/v1.3.183/Wox-1.3.183.exe)

## 使用方式

1.  `Alt`+ `Space` 可以叫出 Wox

    ![1launcher](https://cloud.githubusercontent.com/assets/3851540/26285110/f1e7e1f6-3e7b-11e7-830f-d24fd1c19138.png)

2.  Wox 設定

    ![2setting](https://cloud.githubusercontent.com/assets/3851540/26285111/f20dfd32-3e7b-11e7-9a36-3ec50b1bed96.png)

    *   可以調整語言 (支援繁體中文)

        ![3lang](https://cloud.githubusercontent.com/assets/3851540/26285115/f22ed250-3e7b-11e7-895f-9409d58df4cb.png)

    *   管理套件及套件關鍵字(以 Web Searches 為例)

        > `g + {search term}` 就會將關鍵字餵給 google 搜尋

        ![4plugin](https://cloud.githubusercontent.com/assets/3851540/26285112/f22bec5c-3e7b-11e7-9a22-824cfd22caa7.png)

    *   修改 theme、 hotkey...就不一一介紹了

3.  安裝 plugin

    *   [Wox Plugins](http://www.getwox.com/plugin)

    *   plugin 只有 121 個並不多

    *   安裝 `Google Translate` 為例

        ```
        wpm install Google Translate
        ```
        
        ![5install](https://cloud.githubusercontent.com/assets/3851540/26285113/f22cd900-3e7b-11e7-9369-320545bdd39e.png)

    *   確認安裝

        ![6confirm1](https://cloud.githubusercontent.com/assets/3851540/26285114/f22e562c-3e7b-11e7-8bff-3cd43a9a6cb8.png)

        ![7confirm2](https://cloud.githubusercontent.com/assets/3851540/26285117/f2340310-3e7b-11e7-9db6-d0d916d2ad78.png)

        ![8confirm3](https://cloud.githubusercontent.com/assets/3851540/26285116/f232b5d2-3e7b-11e7-81fe-a49033e5c3b9.png)

    *   實際使用

        ```
        tr en zh {word}
        ```

        ![9use](https://cloud.githubusercontent.com/assets/3851540/26285118/f253dfa0-3e7b-11e7-9edc-5af633330bac.png)

*  如果只是要翻 中-英 可以安裝 `有道翻译` 比較方便

    ```
    yd {word}
    ```

    ![10yd](https://cloud.githubusercontent.com/assets/3851540/26285119/f2590be2-3e7b-11e7-9c38-970e60971d05.png)

# 參考資訊

1.  [Wox](https://github.com/Wox-launcher/Wox/releases)
