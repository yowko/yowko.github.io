---
title: "Jenkins 2 整合 postman collection runner"
date: 2017-03-08T01:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Tools","Jenkins"]
slug: "jenkins-postman"
aliases:
    - /2017/03/jenkins2-integrate-with-postman.html
---
## Jenkins 2 整合 postman collection runner

之前的文章 [利用 POSTMAN 中的 collection 來快速測試 application 功能是否正常](/postman-collection-runner) 介紹到如何使用 postman 內建的 collection runner 來為 api 做基本的測試，今天則要延用其中的概念，在每次上線部署後都做一次基本的 api 測試來保障基本的錯誤不會重複發生。當然這層`基本`保護絕對無法取代單元測試，只是希望在程式正式服務前能具備基本的品質，而更多測試保護也可以讓我們不用擔心程式修改時而造成無謂的錯誤

## 安裝 `newman`

可以使用 command line 環境來執行 postman collection runner 的套件

* node.js 套件 (node.js 版本需大於 v4)

  * 安裝方式，擇一即可
      1. 使用 npm 安裝

          >`npm install newman --global;`
      2. 使用 yarn 安裝

          >`yarn global add newman`

* 確認安裝成功

    >`newman -v`
  * 可以正確得到版本資訊

* 如果執行 newman 指令就會關閉 command prompt
  * 我就是這樣XD
  * 我的解法
        1. 先檢查 command prompt 下使用的 newman 位置

            ![1where](https://cloud.githubusercontent.com/assets/3851540/23580685/27f4dc22-0141-11e7-8e1e-9ea609158639.png)
        2. 確認使用的 newman.cmd 內容

            ![2newmad](https://cloud.githubusercontent.com/assets/3851540/23580687/28178042-0141-11e7-92cf-f44675bc5be4.png)
        3. 我不懂這麼寫 `exit $?` 的用意，但給了另個 newman.cmd 的位置，就試試看唄 --> 正常

            ![3newmanok](https://cloud.githubusercontent.com/assets/3851540/23580686/281761e8-0141-11e7-88e3-d80e3b4cd2a1.png)
        4. 在錯誤的資料夾中發現有個 `newman.cmd.cmd`
            * 將原本的 `newman.cmd` rename 備用
            * 將 `newman.cmd.cmd` rename 為 `newman.cmd`
            * 再執行一次 `newman -v` --> 已可正確執行

                ![4ok](https://cloud.githubusercontent.com/assets/3851540/23580688/28178916-0141-11e7-9c9d-98a852365b08.png)

## 準備 collection 及 environment

* 匯出 collection
    1. `Collections` tab --> `...` --> `Export`

        ![5export](https://cloud.githubusercontent.com/assets/3851540/23580689/28199d00-0141-11e7-96f3-d6e891834b12.png)
    2. 選擇版本
        * v1 與 v2 都可以使用
        * v2 比較簡捷，人類比較好閱讀

            ![6version](https://cloud.githubusercontent.com/assets/3851540/23580690/2819ec74-0141-11e7-813c-34d74a8a537f.png)
* 匯出 envinment
    1. `齒輪` 圖示 --> `Shared Environments`

        ![7environment](https://cloud.githubusercontent.com/assets/3851540/23580691/28438b4c-0141-11e7-87ea-9743c90ddcfb.png)
    2. 點選所需環境組合的下載按鈕

        ![8download](https://cloud.githubusercontent.com/assets/3851540/23580693/2843b7b6-0141-11e7-9567-f2bc85e0aa7e.png)

## jenkins 設定

1. 新增 freestyle job

    ![9newjob](https://cloud.githubusercontent.com/assets/3851540/23580692/28437bd4-0141-11e7-9b00-a15989b81c32.png)
2. 設定 job 內容
    * 新增 build step --> Execute Windows batch commnad

        ![10buildstep](https://cloud.githubusercontent.com/assets/3851540/23580682/27f14b02-0141-11e7-9807-d04d70b97e75.png)
    * 輸入 newman 指令
        >`newman run {collection path} -e {environment path}`

        ![11newmancommand](https://cloud.githubusercontent.com/assets/3851540/23580683/27f1a656-0141-11e7-85e2-db67358f9fef.png)
3. 加入 pipeline 中
    * pipeline 詳細設定請參考  [Jenkins 2 如何建立 Pipeline job](/jenkins-2-pipeline-job)
4. 實際效果
    * 測試中

        ![12building](https://cloud.githubusercontent.com/assets/3851540/23580684/27f437ea-0141-11e7-86e9-1c4c3eea4f0c.png)
    * 測試結果有成功也有失敗

        ![13result](https://cloud.githubusercontent.com/assets/3851540/23580681/27f13db0-0141-11e7-8c3d-836d8189ebbd.png)

## 參考資料

1. [How to write powerful automated API tests with Postman, Newman and Jenkins](http://blog.getpostman.com/2015/09/03/how-to-write-powerful-automated-api-tests-with-postman-newman-and-jenkins/)
2. [newman](https://www.npmjs.com/package/newman)
3. [利用 POSTMAN 中的 collection 來快速測試 application 功能是否正常](/postman-collection-runner)
