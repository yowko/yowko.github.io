---
title: "Facebook Messenger Bot 需要申請什麼呢？"
date: 2016-12-05T00:42:34+08:00
lastmod: 2018-09-05T00:42:34+08:00
draft: false
tags: ["Bot","facebook"]
slug: "facebook-messenger-bot"
aliases:
    - /2016/12/facebook-messenger-bot.html
---
# Facebook Messenger Bot 需要申請什麼呢？
要建立一個 facebook messenger 的 bot，在 `Microsoft Bot Framework` 的幫助下，只需幾個簡單設定，很快就能完成，不過申請的流程有點雜亂，特別紀錄一下備忘

# 1.建立一個 Facebook Page(粉絲專頁)
直接開啟 [facebook](https://www.facebook.com)

- 1-1. 建立粉絲專頁

    ![createPage](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/488x531/c4fef064784b358b1b17dec4c933979d/_output_createPage.png)

- 1-2. 選擇 page 類型

    ![pagetype](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/959x701/b575bc05acadbd23a3152dcbf6f3a48e/_output_pagetype.png)

- 1-3. 輸入 page 名稱

    ![typename](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/939x688/7ac48ff5aff6bc05529da46353f1df13/_output_typename.png)

- 1-4. 成功建立後，取得`Facebook 專頁編號`(這就是`Page id`，botframework 設定會用到)

    ![PAGEID](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x521/e886ce23920c0663985e07c68e8a8eea/_output_PAGEID.png)

# 2. 建立一個 `Facebook App`
facebook 開發人員網站[facebook for developers](https://developers.facebook.com/)

- 2-1. 建立應用程式

    ![createApp](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x560/a252ede431112ed79a5110d48effa550/_output_createApp.png)

- 2-2. 建立新的應用程式編號
    - 2-2-1. 填入`顯示名稱`
    - 2-2-2. 選擇`類別`
    - 2-2-3. 建立應用程式編號
        
        ![newappno](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1076x543/5ecfd6f8eae47ff6acc235b2a28420b7/_output_newappno.png)

- 2-3. 啟用`Messenger`
  
    ![enablemessenger](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x543/93bbcb2f6f428f370dadc2658cbddba8/_output_enablemessenger.png)

- 2-4. 設定 `Messenger`
    - 2-4-1. 權杖產生--選擇粉絲專頁

        ![gentoken](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x322/a3dd385a100392172fc32eade98ff94d/_output_gentoken.png)
    - 2-4-2. 授權
        
        ![AUTH](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/913x896/870e06195e76a808fa5d8429ee1c371f/_output_AUTH.png)
    - 2-4-2. 取得 token (這就是`Page Access Token`，botframework 設定會用到)

        ![tokengot](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x325/3da5e4af38c153a4a3e9664045d97b9f/_output_tokengot.png)

- 2-5. 設定 `Webhooks`

    ![WEBHOOK](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x242/1ffe2e67616bcd1bd5ca6399083abeb0/_output_webhook.png)

    - 詳細資料可以看[這邊](https://developers.facebook.com/docs/graph-api/webhooks)
    -  callback url 需要是 `https`，允許 `get` 與 `post`，並回應 `200`
    - 以 `Microsoft Bot Framework`為例，設定如下
        ![BOTFRAMEWORK_CALLBACK](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/879x739/cf89fa714cc906f9e44b30c12e5d124b/_output_BOTFRAMEWORK_CALLBACK.png)

    - 2-5-1. 回呼網址
    - 2-5-2. 驗證權杖
    - 2-5-3. 訂閱欄位(`message_deliveries`,`messages`, `messaging_optins`,`messaging_postbacks`)
        ![WEBhookok](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x734/b9bcb0c23ce73404d57a35460c1d42d1/_output_WEBhookok.png)

- 2-6. `Webhooks` 訂閱粉絲團
    
    ![subscrib1](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1196x441/9bee97fc95ab60c6e3fccb0c9755ed8d/_output_subscrib1.png)

- 2-7. 取得`應用程式編號(Facebook App Id)`,`應用程式密鑰(Facebook App Secret)`
    
    ![appidandsecret](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x387/7b49cef0d8316d9fd6f746c94d845db5/_output_appidandsecret.png)


# 3. 需要用到的資料
- 3-1. Facebook Page Id
    
    ![PAGEID](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x521/e886ce23920c0663985e07c68e8a8eea/_output_PAGEID.png)

- 3-2. Facebook App Id
    
    ![appidandsecret](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x387/7b49cef0d8316d9fd6f746c94d845db5/_output_appidandsecret.png)

- 3-3. Facebook App Secret
    
    ![appidandsecret](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x387/7b49cef0d8316d9fd6f746c94d845db5/_output_appidandsecret.png)

- 3-4. Page Access Token
    
    ![tokengot](https://trello-attachments.s3.amazonaws.com/583b1dd86f1d8a8cf50daa8d/1200x325/3da5e4af38c153a4a3e9664045d97b9f/_output_tokengot.png)




# 參考資料
1. [Messenger 平台](https://developers.facebook.com/docs/messenger-platform)
2. [Microsoft Bot Framework](https://dev.botframework.com/)