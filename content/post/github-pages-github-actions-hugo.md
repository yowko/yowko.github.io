---
title: "GitHub Action 自動建置 Hugo 網站並部署至 Github Pages"
date: 2023-07-23T00:30:00+08:00
lastmod: 2023-07-23T00:30:31+08:00
draft: false
tags: ["markdown","github"]
slug: "github-pages-github-actions-hugo"
---

## GitHub Action 自動建置 Hugo 網站並部署至 Github Pages

之前筆記 [使用 markdown 搭配 Hugo 建立網站](/markdown-hugo) 有提到打算透過 markdown 與 hugo 來為學長建立個人網站，之前筆記 [使用 markdown 搭配 Hugo 建立網站](/markdown-hugo) 已經在 local 環境中成功建立網站並套用 hugo theme，接著就是紀錄如何使用 GitHub Action 自動建置 Hugo 網站並部署至 Github Pages，讓我們可以專注在網站內容的 markdown，其他網站 host、建置與 theme 就交由 GitHub 囉

## 基本環境說明

1. macOS Ventura 13.4
2. Hugo v0.115.3
3. git 2.41.0

## 建立步驟

1. 建立 GitHub repository

    > 詳細內容可以參考官方文件：[GitHub Pages](https://pages.github.com/)

    - 需為 `Public`
    - 名稱需為 `{username}.github.io`

        > 範例：`yowkotsai.github.io`

    ![1repository](https://github.com/yowko/picsbed/assets/3851540/6c21af3c-8be2-4c4e-b7de-875b402fb473)

2. 設定 GitHub Pages build

    - Setting --> Pages --> Build and deployment

        ![2pagesbuild](https://github.com/yowko/picsbed/assets/3851540/69410643-e654-4b82-91b2-d5e595eaa738)

    - Search `Hugo` --> Configure

        ![3configurehugo](https://github.com/yowko/picsbed/assets/3851540/bfd39845-c8fd-43e9-9dbe-e540cfb178db)

    - 將 hugo 設定加入 repository 中

        > 可以直接 commit，或是依需求調整

        ![4commitworkflow](https://github.com/yowko/picsbed/assets/3851540/a421762c-7707-4920-9293-61a2775a40f2)

    - GitHub Actions build

        > 圖中有個 fail 是 Pages 預設 job，在我們設定使用 hugo 之後就不會再觸發

        ![5actions](https://github.com/yowko/picsbed/assets/3851540/a6b6ea32-6540-472a-b59c-f236aa2a78ef)

    - 結果

        ![6result](https://github.com/yowko/picsbed/assets/3851540/f33249cc-15ef-482d-bdef-55fd2f68e328)

## 心得

GitHub Pages 預設使用 Jekyll，我稍微看了一下，我覺得要額外安裝 Jekyll 只為了 build，個人感覺不是很理想，但 GitHub 官方選擇它做為預設應該有其過人之處，就留給其他人試試囉

GitHub Pages 與 GitHub Actions 設定完後，需要 local 的 repository rebase remote 將 workflow 套用，不然剛剛那些設定就白做了

## 參考資訊

1. [GitHub Pages](https://pages.github.com/)
2. [Quickstart for GitHub Actions](https://docs.github.com/en/actions/quickstart)
3. [使用 markdown 搭配 Hugo 建立網站](/markdown-hugo)
