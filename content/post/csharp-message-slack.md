---
title: "使用 C# (.NET Core) 傳訊息至 Slack"
date: 2019-02-06T23:45:00+08:00
lastmod: 2021-11-03T23:44:30+08:00
draft: false
tags: ["csharp","dotnet core"]
slug: "csharp-message-slack"
---
## 使用 C# (.NET Core) 傳訊息至 Slack

公司有個臨時性需求：某個重要功能開啟或是關閉時，立即通知營運團隊及各級主管知道，讓大家在討論 production issue 有共同的討論基準。

經過一番討論後決定將功能開關通知透過 slack 來實作，雖然用 slack 整點好段時間卻沒真的自己寫過，一時間沒能快速找到程式碼可以抄XD  所以就能紀錄一篇以後就省事了  哈哈

## 建立 Slack workspace

1. [Create a new workspace](https://slack.com/get-started#create)
2. 確認 email

    ![1checkemail](https://user-images.githubusercontent.com/3851540/52357861-e600e480-2a71-11e9-91bc-99f3c17611dc.png)

3. 輸入 workspace 名稱

    ![2workspacename](https://user-images.githubusercontent.com/3851540/52357862-e600e480-2a71-11e9-8c56-6a0b74c94c9a.png)
4. 輸入 channel 名稱

    ![3channelname](https://user-images.githubusercontent.com/3851540/52357863-e600e480-2a71-11e9-96af-5ca9c106c555.png)
5. 邀請其他人或略過

    ![4inviteorskip](https://user-images.githubusercontent.com/3851540/52357864-e6997b00-2a71-11e9-9630-dd0a6fff991e.png)

6. 完成基本設定

    ![5complete](https://user-images.githubusercontent.com/3851540/52357867-e6997b00-2a71-11e9-8365-7f4828dcbcda.png)

    ![7initscreenshot](https://user-images.githubusercontent.com/3851540/52357868-e6997b00-2a71-11e9-99ff-5bd6ee5f385a.png)

## 建立 Token

1. 使用 user token

    - 建立個人 token : [Legacy tokens](https://api.slack.com/custom-integrations/legacy-tokens)

        ![12usertoken](https://user-images.githubusercontent.com/3851540/52357876-e8633e80-2a71-11e9-8135-cd5c317014dd.png)

        > 成功建立後會連帶建立 `Slack API Tester` app 用來發送訊息

2. 使用 bot token

    - 透過 [Build something amazing.](https://api.slack.com/apps) 建立 Apps

        ![13buildapp](https://user-images.githubusercontent.com/3851540/52357877-e8633e80-2a71-11e9-88a0-c7fad5ccd34a.png)

    - 填入名稱及選擇目標 workspace

        ![14createslackapp](https://user-images.githubusercontent.com/3851540/52357879-e8633e80-2a71-11e9-86ed-9fd84f8d456f.png)

    - 加入 bot 功能

        ![15addfeature](https://user-images.githubusercontent.com/3851540/52357881-e8fbd500-2a71-11e9-968b-6237d25d19b3.png)

        ![16addfeature2](https://user-images.githubusercontent.com/3851540/52357883-e8fbd500-2a71-11e9-9a16-ff12941892e6.png)

        ![17addfeature3](https://user-images.githubusercontent.com/3851540/52357884-e8fbd500-2a71-11e9-991f-b7dd8f6c5246.png)

    - 將 app 安裝至 workspace 中

        ![18installapp](https://user-images.githubusercontent.com/3851540/52357885-e9946b80-2a71-11e9-8c06-671980ef12b9.png)

        ![19installapp2](https://user-images.githubusercontent.com/3851540/52357886-e9946b80-2a71-11e9-9908-e8044d7da402.png)

    - 取得 Bot User OAuth Access Token

        ![20accesstoken](https://user-images.githubusercontent.com/3851540/52357887-ea2d0200-2a71-11e9-85b6-8b2442672d87.png)

- 關於 token 類型請參考 [Types of tokens](https://api.slack.com/docs/token-types)

## 取得 channel id

![23channelid](https://user-images.githubusercontent.com/3851540/52357892-eb5e2f00-2a71-11e9-80bf-4711d65a47c3.png)

## Slack 測試傳送訊息

> 可以透過 [chat.postMessage](https://api.slack.com/methods/chat.postMessage/test) 來測試發送訊息

1. User Token

    ![21usertoken](https://user-images.githubusercontent.com/3851540/52357889-ea2d0200-2a71-11e9-8310-bb6113c5d008.png)

2. bot token

    ![22bottoken](https://user-images.githubusercontent.com/3851540/52357890-eac59880-2a71-11e9-985e-41d4a3edf1a0.png)

## 傳送訊息至 Slack

> Slack 的 `chat.postMessage` 是 REST API,以下透過 .NET Core console 搭配 HttpClientFactory 來傳送訊息

1. 基本設定

    > 如果想更深入了解相關設定可以參考 [在 .NET Core console 上使用 Dependency Injection - DI](/dotnet-core-console-di/) 與 [在 .NET Core 與 .NET Framework 上使用 HttpClientFactory](/httpclientfactory-dotnet-core-dotnet-framework/)

    ```cs
    var builder =
        //新建 HostBuilder
        new HostBuilder()
        //設定所需 service
        .ConfigureServices((hostContext, services) =>
        {
            //設定每次 request 都會建立 具名 HttpClient
           services.AddHttpClient("yowkoslacksender", c =>
            {
                c.BaseAddress = new Uri("https://slack.com/api/");
            });
        });

  
    
    var clientFactory =
        //使用上面建立的 host builder 來初始化 host
        builder.Build()
        //從 service provider 中取得建立的 IHttpClientFactory 服務
        .Services.GetRequiredService<IHttpClientFactory>();

    //由 HttpClientFactory 來建立 HttpClient
    var client = clientFactory.CreateClient("yowkoslacksender");
    
    var token = "xoxb-xxxx";//access token
    var channel = "xxxx";//channelId
    ```

2. 簡易內容 (僅文字)

    ```cs
    var text = System.Web.HttpUtility.UrlEncode("yowko 測試");//實際內容
    //透過 Get 傳送至 slack
    client.GetAsync($"chat.postMessage?token={token}&channel={channel}&text={text}");
    ```

    ![24simpletext](https://user-images.githubusercontent.com/3851540/52357893-ebf6c580-2a71-11e9-8ded-a2ef63da492c.png)

3. 格式化內容 (包含圖片、超連結、或是按鈕....)

    ```cs
    //準備格式化內谷實際顯示值
    var attachments = new List<Attachment>() {
                new Attachment(){
                    color="#36a64f",//實際顯示時的引用線顏色
                    pretext= "原本 text 位置內容",
                    author_name= "第一行(作者)",
                    author_link= "",
                    author_icon= "http://flickr.com/icons/bobby.jpg",
                    title= "標題",
                    title_link= "",
                    text= "實際內文",
                    fields= new List<Field>(){
                        new Field(){
                            title= "Priority",
                            value= "High"}
                    }
                    ,
                    footer= "置底內容",
                    footer_icon= "https://platform.slack-edge.com/img/default_application_icon.png",

                }
            };
    //格式化內容轉為 string 再進行 urlencode 以利 get 傳送
    var attachment = System.Web.HttpUtility.UrlEncode(JsonConvert.SerializeObject(attachments));
    //透過 Get 傳送至 slack
    client.GetAsync($"chat.postMessage?token={token}&channel={channel}&attachments={attachment}");
    ```

    ![25richcontent](https://user-images.githubusercontent.com/3851540/52357894-ebf6c580-2a71-11e9-9779-48a26aca066c.png)

    > 格式化內容相關設定可以參考 [Basic message formatting](https://api.slack.com/docs/message-formatting)

## 心得

Slack 文件滿雜亂的，感覺起來應該歷經了許多演化，就連 API 的使用也有類似的影子，就以傳送訊息為例，可以找到好幾種做法，每種做法間似乎又沒有什麼脈絡可以依循，很容易搞混

另外最令我困擾的是 REST API 的說明，明明文件說要用 POST，但怎麼用就是失敗，查了資料才發現很多人說應該是 GET 才對XD  將程式改為 GET 問題就解決了，不知道官方的立場是什麼  感覺不是基本的錯誤而是有其他層面的考量  

## 參考資訊

1. [Create a new workspace](https://slack.com/get-started#create)
2. [Legacy tokens](https://api.slack.com/custom-integrations/legacy-tokens)
3. [Build something amazing.](https://api.slack.com/apps)
4. [Types of tokens](https://api.slack.com/docs/token-types)
5. [chat.postMessage](https://api.slack.com/methods/chat.postMessage/test)
6. [在 .NET Core console 上使用 Dependency Injection - DI](/dotnet-core-console-di/)
7. [在 .NET Core 與 .NET Framework 上使用 HttpClientFactory](/httpclientfactory-dotnet-core-dotnet-framework/)
8. [Basic message formatting](https://api.slack.com/docs/message-formatting)
