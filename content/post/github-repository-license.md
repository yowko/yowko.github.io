---
title: "在 GitHub Repository 中加上 License 宣告"
date: 2017-06-06T21:00:00+08:00
lastmod: 2021-11-02T21:00:34+08:00
draft: false
tags: ["Git"]
slug: "github-repository-license"
aliases:
    - /2017/06/github-repository-license.html
---
## 在 GitHub Repository 中加上 License 宣告

之前在紀錄嘗試不同技術的過程默默地也完成了許多不同的成品，大多數內容也都是透過網路搜尋而來，所以就放在 GitHub 希望有機會讓有類似需求的同好也可以快速搜尋到合適的解法，之前看到 COSCUP 可以利用 open source project 來申請貢獻者門票，於是我就很不要臉地申請了，但卻收到 `補件通知`

原來我的 repository 上缺少 license 宣告，但我要怎麼做才可以在 repository 上加上 license 宣告？ 難道隨便加個 license.txt 就算是了嗎？ 還好 GitHub 文件很齊全，我順手紀錄一下過程， GitHub 文件可以參考 [Adding a license to a repository](https://help.github.com/articles/adding-a-license-to-a-repository/)

## 原始 Repository

> 沒有 License tab

![1nolicense](https://cloud.githubusercontent.com/assets/3851540/26813839/a9efdd1a-4ab3-11e7-8498-dc8d72c36712.png)

## 加上 License

1. Create new file

    > 建立新檔案

    ![2createnewfile](https://cloud.githubusercontent.com/assets/3851540/26813840/a9f1085c-4ab3-11e7-8e9a-50a8261abeba.png)

2. 檔案使用 `LICENSE` 或是 `LICENSE.txt`

    ![3license](https://cloud.githubusercontent.com/assets/3851540/26813841/aa16a5a8-4ab3-11e7-9320-04f1359a7783.png)

3. 使用 license template or 直接編輯內容

    ![4licensetemplate](https://cloud.githubusercontent.com/assets/3851540/26813842/aa1ad5b0-4ab3-11e7-9479-fb5194f29bdb.png)

    * 選擇 template 後會填入預設內容

        ![5template](https://cloud.githubusercontent.com/assets/3851540/26813833/a9c37112-4ab3-11e7-98bf-b62dd0f21949.png)

4. commit

    ![6commit](https://cloud.githubusercontent.com/assets/3851540/26813835/a9c6bd2c-4ab3-11e7-99b0-fb844d543110.png)

    * GitHub 建議不要直接 commit 在 master branch，建立一個新 branch 再 create a pull request 然後 merge pull request

        ![7propose](https://cloud.githubusercontent.com/assets/3851540/26813834/a9c435b6-4ab3-11e7-8179-07bb1e67e04f.png)

        ![8createpullrequest](https://cloud.githubusercontent.com/assets/3851540/26813836/a9c81672-4ab3-11e7-80cd-ff19061e4e0d.png)

        ![9merger](https://cloud.githubusercontent.com/assets/3851540/26813837/a9ecdef8-4ab3-11e7-88c1-f3bd78606b82.png)

5. 回到 repoitory 就會發現多了個 license 類型的 tab

    ![10licensed](https://cloud.githubusercontent.com/assets/3851540/26813838/a9eee48c-4ab3-11e7-89bb-e9ba8570b896.png)

## 參考資訊

1. [Adding a license to a repository](https://help.github.com/articles/adding-a-license-to-a-repository/)
