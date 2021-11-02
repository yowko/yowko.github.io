---
title: "如何發行 Visual Studio 專案範本(project Template)"
date: 2016-12-15T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Visual Studio"]
slug: "issue-visual-studio-project-template"
aliases:
    - /2016/12/publish-visual-studio-project-template.html
---

## 如何發行 Visual Studio 專案範本(project Template)

最近在學習 [Microsoft Bot Framework](https://dev.botframework.com/) 相關應用,其中官網上的 Startup 請大家下載一個 zip 放到指定的目錄(vs2015 放在 `Documents\Visual Studio 2015\Templates\ProjectTemplates\Visual C#`)，接著就可以在 Visual Studio 上出現專案範本了。雖然滿容易的，但我隱約覺得也許有更好的方式。

## 準備 Project Template

這邊就使用 [Bot Framework Downloads](https://docs.botframework.com/en-us/downloads/) 當做範例

## 建立 VSIX 專案

1. File --> New --> Project...

    ![newproject](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/950x258/6664f4b7428384e71bdd7daa7f89b65b/_output_newproject.png)

2. Templates --> Visual C# --> Extensibility  --> VSIX

    ![addvsix](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x685/f35f8b79867f47f59d944ee4f4be2bf7/_output_addvsix.png)

3. 將從[Microsoft Bot Framework](https://dev.botframework.com/) 上下載回來的 `Bot Application.zip` 加入專案

    ![addzip](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/369x802/f72dcb046850193bd4bbec44b93837f5/_output_addzip.png)

4. 將 `Bot Application.zip` 的 `Copy to Output Directory` 屬性改為 `Copy always`

    ![copyalways](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/366x756/d9b9d94d7497c6548de077f8c3535b60/_output_copyalways.png)

5. 編輯 `source.extension.vsixmanifest`(Metadata)

    依序填入
     - 5-1. `Product Name`
     - 5-2. `Product ID`
     - 5-3. `Author`
     - 5-4. `Version`
     - 5-5. `Description`

        ![manifest](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x519/fecb085a9423a185b2bd42aeb262a1ca/_output_manifest.png)

6. 新增 `source.extension.vsixmanifest`(Assets)
    - 6-1. Assets --> New

        ![assetsnew](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x637/9ae0f5def60fbe49227b9a04d7fbcb18/_output_assetsnew.png)

    - 6-2. 選擇 `Type`,`Source`,`Path`
        - Type : `Microsoft.VisualStudio.ProjectTemplate`
        - Source : `File on filesystem`
        - Path : 選擇 `Bot Application.zip`

            ![assetsdone](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/691x590/4ab26ac31dc0557bab9bc6efb8fd31bf/_output_assetsdone.png)

7. Build project
 在專案檔案位置的`bin`資料夾會產出 `.vsix`檔 ，即可用來安裝

## 發行至 [Visual Studio Marketplace](https://visualstudiogallery.msdn.microsoft.com/)

1. 開始發行

    ![upload](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x789/c1b3e7227375f8ecd0a07181f19f40af/_output_upload.png)

2. 選擇`擴充功能類型`--> `專案或文章範本`

    ![extendtype](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/667x356/ab99a0f1fadefb8acd5da2703010fa3a/_output_extendtype.png)

3. 上傳檔案

    ![uploadfile](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/481x524/299492d96969a936abb539c90c69bcf2/_output_uploadfile.png)

4. 填寫`基本資料`
    - `描述`要超過 280 字，記得同意`投稿協議`

        ![basicinfo](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1006x771/17776f7567fdf3f77e1b9378e2710ca2/_output_basicinfo.png)

5. 預覽

    ![readttopublish](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1058x833/30d7dc33273cce0426c0dd5535975b4d/_output_readttopublish.png)

6. publish

    ![published](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1018x828/38ed838b879ea1db5fcb1b30cf59017c/_output_published.png)

## 測試

1. 套件管理員

    ![extensionmanager](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/623x816/13ed794d3bb836413bf41698f00e748e/_output_extensionmanager.png)

2. 在 `Visual Studio Gallery` 搜尋 `bot`

    ![searchbot](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x329/9100a24e1c952b33cb113608f33544df/_output_searchbot.png)

3. 新專案 --> 搜尋已安裝專案範本--> Bot Application

    ![bottemplate](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/1200x685/af40ec7cc26670847aedd64dd97b8f9b/_output_bottemplate.png)

4. 成功產生 `bot application` 專案

    ![PROJECTOK](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/361x311/f758ed9e5b1843e0ce6b93a20adeb47f/_output_PROJECTOK.png)

## 優缺點

- 優點：用 VSIX(擴充套件) 來新增，不需額外下載及記憶 template 位置，方便大家使用.
- 缺點：不像自行丟檔，可以有明顯的 template 位置，必需透過搜尋，新增時比較好找.

- 我自行包裝的 VSIX 不會出現下圖 `Bot applications` 的分類(可能是我設定上的問題)

    ![templatefolder](https://trello-attachments.s3.amazonaws.com/58399cb985f76f96f4a0a0f7/701x321/d02eb03fddc29b5fc0704b028f3fe928/_output_templatefolder.png)

## 參考資料

1. [Microsoft Bot Framework](https://dev.botframework.com/)
2. [Bot Framework Downloads](https://docs.botframework.com/en-us/downloads/)
3. [Getting Started with the VSIX Project Template](https://msdn.microsoft.com/en-us/library/dd885241.aspx)
4. [Visual Studio Marketplace](https://visualstudiogallery.msdn.microsoft.com/)
