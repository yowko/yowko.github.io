---
title: "使用 markdown 搭配 Hugo 建立網站"
date: 2023-07-22T00:30:00+08:00
lastmod: 2023-07-22T00:30:31+08:00
draft: false
tags: ["markdown","go"]
slug: "markdown-hugo"
---

## 使用 markdown 搭配 Hugo 建立網站

最近大學時的學長準備到師大去當老師，左思又右想不知道該送什麼慶賀，後來想起過去學長曾經想要使用 Google Sites 建立個人形象網站，但一直無法成功被 google search engine 建立索引，所以就興起為學長建立個人網站做為賀禮了表個人的祝賀之意

## 基本環境說明

1. macOS Ventura 13.4
2. Hugo v0.115.3
3. git 2.41.0

## 建立步驟

1. 安裝必需軟體

    - Hugo

        > 完整內容可以參考 [Hugo 官方文件](https://gohugo.io/installation/)

        ```bash
        brew install hugo
        ```

    - git

        > 完整內容可以參考 [Git 官方文件](https://git-scm.com/downloads)

        ```bash
        brew install git
        ```

2. 設定 Hugo 網站專案

    - 使用 hugo cli 建立網路專案

        會以專案名稱建立對應資料夾與相關的 hugo 設定檔

        - 語法

            ```bash
            hugo new site {website_project_name}
            ```

        - 範例

            以 `yowkotsai.github.io` 做為專案名稱 (這是為之後部署至 GitHub Pages 做準備)

            ```bash
            hugo new site yowkotsai.github.io
            ```

    - 建立 git repo 來進行版控

        - 語法

            ```bash
            cd {{folder}} && git init
            ```

        - 範例

            ```bash
            cd yowkotsai.github.io && git init
            ```

    - 將目標的 hugo theme 以 git submodule 加入 `themes` 資料夾中

        > 以 [FixIt Theme | Hugo](https://github.com/hugo-fixit/FixIt) 為例

        - 語法

            ```bash
            git submodule add {your_theme_git} themes/{theme_name}
            ```

        - 範例

            ```bash
            git submodule add git@github.com:hugo-fixit/FixIt.git themes/fixit
            ```

    - 設定 theme

        - 語法

            ```bash
            echo "theme = '{theme name}'" >> hugo.toml
            ```

        - 範例

            ```bash
            echo "theme = 'fixit'" >> hugo.toml
            ```

    - 測試

        > 預設使用 `http://localhost:1313/`

        ```bash
        hugo server
        ```

        ![2result](https://github.com/yowko/picsbed/assets/3851540/7a3c7931-46b6-4644-8cba-10bba6f76cf7)

## 心得

我在引用 theme 時發現個奇怪的點，以這個 theme 為例：相關的 config 是在 theme 的 folder 中，我嘗試將 config 移至根目錄並無法成功套用，這樣一來就是每個 website 需要自行 fork 一份 theme 才能做修改，這樣日後的更新或是與原本 repo 的 sync 就變得複雜，感覺不是很便利，我怎麼記得以前除非要自行客製，不然 config 可以放在外層？！

我試了一下，最後我決定這麼做

1. 將 theme 的 config 改為 bak

    > 避免設定互相 overwrite

    ```bash
    mv themes/fixit/config.toml themes/fixit/config.toml.bak
    ```

2. 將 theme config 內容移至 hugo.toml 中

    ```bash
    cat themes/fixit/config.toml.bak >> hugo.toml
    ```

我有找到這個網站： https://docs.gethugothemes.com/guide/ ，它的處理方式是每個 theme 中都有 example site，將其中的 config copy 到外層就可以正常使用，但缺點是這個網站需要安裝的基礎軟體較多(Hugo、go、Nodejs)、多數的 theme 需要額外付費、theme 的文件不一定齊全

## 參考資訊

1. [Hugo Quick start](https://gohugo.io/getting-started/quick-start/)
2. [FixIt Theme | Hugo](https://github.com/hugo-fixit/FixIt)
3. [Hugo 官方文件](https://gohugo.io/installation/)
4. [Git 官方文件](https://git-scm.com/downloads)
