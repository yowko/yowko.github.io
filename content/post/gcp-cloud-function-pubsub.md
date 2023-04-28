---
title: "Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub"
date: 2023-04-26T00:30:00+08:00
lastmod: 2023-04-26T00:30:31+08:00
draft: false
tags: ["csharp","gcp"]
slug: "gcp-cloud-function-pubsub"
---

## Google Cloud Functions 發送訊息到 Google Cloud Pub/Sub

因為公司部份產品建置在 SaaS 基礎上，但這些 SaaS 都有自己維護的時間跟計劃，所以為了避免 SaaS 維護造成產品服務異常，所以想要將 SaaS 的相關通知先打進 Google Cloud Pub/Sub，後續再介接進團隊的 IM 系統

在這個目標下，就想透過 Google Cloud Functions 來處理這個需求，不需額外建立 server 也不用 container，今天就先紀錄一下如何透過 Cloud Functions 將訊息送到 Cloud Pub/Sub

## 基本環境說明

1. macOS Ventura 13.2
2. .NET SDK 7.0.203
3. JetBrains Rider 2023.1

    - [Google.Cloud.Functions.Templates](https://www.nuget.org/packages/Google.Cloud.Functions.Templates)

4. NuGet packages
    - Google.Cloud.Functions.Hosting 1.1.0
    - Google.Cloud.PubSub.V1 3.5.0

## 使用方式

1. 安裝 Google Cloud Functions project template

    ```bash
    dotnet new install Google.Cloud.Functions.Templates::2.0.0
    ```

2. 透過 Google Cloud Functions project template 建立專案
3. 安裝 NuGet packages：Google.Cloud.PubSub.V1 3.5.0
4. 新增 `PublishMessages.cs` (用來發送訊息至 Google Cloud Pub/Sub)

    ```cs
    using Google.Cloud.PubSub.V1;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    
    namespace PublishPubSub;
    
    public class PublishMessages
    {
        public async Task<int> PublishMessagesAsync(string projectId, string topicId,IEnumerable<string> messageTexts)
        {
            TopicName topicName = TopicName.FromProjectTopic(projectId, topicId);
            PublisherClient publisher = await PublisherClient.CreateAsync(topicName);
    
            int publishedMessageCount = 0;
            var publishTasks = messageTexts.Select(async text =>
            {
                try
                {
                    string message = await publisher.PublishAsync(text);
                    Console.WriteLine($"Published message {message}");
                    Interlocked.Increment(ref publishedMessageCount);
                }
                catch (Exception exception)
                {
                    Console.WriteLine($"An error ocurred when publishing message {text}: {exception.Message}");
                }
            });
            await Task.WhenAll(publishTasks);
            return publishedMessageCount;
        }
    }
    ```

5. 修改 `Function.cs`

    ```cs
    using System.IO;
    using System.Text.Json;
    using Google.Cloud.Functions.Framework;
    using Microsoft.AspNetCore.Http;
    using Microsoft.Extensions.Logging;
    using System.Threading.Tasks;
    
    namespace PublishPubSub;
    
    public class Function : IHttpFunction
    {
        private readonly ILogger _logger;
        private const string projectId = "";
        private const string topicId = "";
    
        public Function(ILogger<Function> logger) => _logger = logger;
    
        public async Task HandleAsync(HttpContext context)
        {
            HttpRequest request = context.Request;
    
            // Check URL parameters for "message" field
            string message = request.Query["message"];
    
            // If there's a body, parse it as JSON and check for "message" field.
            using TextReader reader = new StreamReader(request.Body);
            string text = await reader.ReadToEndAsync();
    
            try
            {
                if (text.Length > 0)
                {
                    JsonElement json = JsonSerializer.Deserialize<JsonElement>(text);
                    if (json.TryGetProperty("message", out JsonElement messageElement) && messageElement.ValueKind == JsonValueKind.String)
                    {
                        message = messageElement.GetString();
                    }
                }
    
                if (!string.IsNullOrWhiteSpace(message))
                {
                    var pubsubClient = new PublishMessages();
                    await pubsubClient.PublishMessagesAsync(projectId, topicId, new[] { message });
                }
            }
            catch (JsonException parseException)
            {
                _logger.LogError(parseException, "Error parsing JSON request");
            }
    
    
            await context.Response.WriteAsync(message ?? "Empty message");
        }
    }
    ```

6. GCP 設定

    - 建立 Google Cloud Pub/Sub topic

        > 預設會建立 subscription，最終的 message 是在 subscription 可以 pull 到的

        ![1createtopic](https://user-images.githubusercontent.com/3851540/234823577-aa1d8245-7288-45d6-99a0-db2fc3940c71.png)

    - 建立 Google Cloud Function

        - `Configuration`

            > 除了 name 與 Region 可依需求調整，其他部份皆可使用預設值

        - `Code`

            > 將前面幾個步驟的結果貼上，或是可以打包成 zip 上傳

        > 建立完成後，有個 `TESTING` tab 可以直接執行，並看到相關的 response 與 log

        ![_output_3testing](https://user-images.githubusercontent.com/3851540/235037635-9c5e09b9-efb0-4c03-9a7d-6fe82185b8de.png)


7. 實際效果

    ![2result](https://user-images.githubusercontent.com/3851540/234823608-43cac81f-3730-49b5-8273-811c0a6138f0.png)

## 心得

GCP 的介面可以直接編輯 Cloud Function 的 code，只是 intellisense 跟 lcoal 開發有不小的落差，加上沒有 NuGet 的 GUI，我個人手打沒辦法打出正確的內容，所以才在 local 先開發完再 sync 上去

完成程式碼：[yowko/cloud-function-cloud-pubsub](https://github.com/yowko/cloud-function-cloud-pubsub)

如果需要為 Google Cloud Functions 增加基本保護可以參考 [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key)

## 參考資訊

1. [Write HTTP functions](https://cloud.google.com/functions/docs/writing/write-http-functions#http-example-csharp)
2. [Google.Cloud.Functions.Templates](https://www.nuget.org/packages/Google.Cloud.Functions.Templates)
3. [yowko/cloud-function-cloud-pubsub](https://github.com/yowko/cloud-function-cloud-pubsub)
4. [使用 Google Api Gateway 為 Google Cloud Functions 加上 API Key 保護](/gcp-secure-cloud-function-with-api-key)
