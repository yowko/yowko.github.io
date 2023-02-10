---
title: "使用 C# 訂閱 GKE 更新通知"
date: 2023-01-30T00:30:00+08:00
lastmod: 2023-01-30T00:30:31+08:00
draft: false
tags: ["gcp","csharp","gke"]
slug: "csharp-subscribe-gke-update"
---

## 使用 C# 訂閱 GKE 更新通知

目前團隊的產品在 production 有不少各式各樣的監控：有針對網站的 health check、有針對 log 異常情境的、有針對 kubernetes 上 application 運行狀態的.....所以每次 GKE 有排定的更新都會觸發大量的監控警報，雖然可能不一定會影響產品運作，但收到大量通知還是不免心驚膽顫，除此之外也需要人工介入檢查產品狀態，也就興起了接收 GKE 更新通知的想法，希望讓團隊至少有個心理準備，避免收到大量警示時會手忙腳亂

Google Cloud 上的相關文件很豐富，但相對比較零散，趁著這個機會，簡單紀錄整理一下，方便日後查閱

## 基本環境說明

1. macOS Ventura 13.1
2. .NET SDK 6.0.400
3. NuGet packages

    - Google.Cloud.PubSub.V1 3.2.0

4. 建立 GKE instance

    > 在 project `clean-skill-374402` 中建立一個名為 `gke-test-pubsub` 版本為 `1.21.14-gke.4300` 位於 `asia-east1-a` node num 為 `1` 的 private cluster

    ```bash
    gcloud beta container --project "clean-skill-374402" clusters create "gke-test-pubsub" --zone "asia-east1-a" --no-enable-basic-auth --cluster-version "1.21.14-gke.4300" --release-channel "None" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "1" --enable-private-nodes --enable-private-endpoint --master-ipv4-cidr "172.16.0.0/28" --enable-master-global-access --enable-ip-alias --network "projects/clean-skill-374402/global/networks/default" --subnetwork "projects/clean-skill-374402/regions/asia-east1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --no-enable-autoupgrade --no-enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --no-shielded-integrity-monitoring --no-shielded-secure-boot --node-locations "asia-east1-a"
    ```

## GCP Pub/Sub 設定

1. 建立 Pub/Sub 的 topic

    > 建立名為 `gkeupdate` 的 topic

    ```bash
    gcloud pubsub topics create gkeupdate
    ```

2. 建立 topic 的 subscription

    > 建立名為 `gkeupdate` 的 subscription

    ```bash
    gcloud pubsub subscriptions create gkeupdate --topic=gkeupdate 
    ```

## GKE 設定

- 啟用 GKE 通知

    > 啟用位於 `asia-east1-a` 的 `gke-test-pubsub` cluster 更新通知：將 `UpgradeEvent` 通知寫至 `clean-skill-374402` 這個 project 中的 `gkeupdate` topic

    ```bash
    gcloud container clusters update gke-test-pubsub --region=asia-east1-a --notification-config=pubsub=ENABLED,pubsub-topic=projects/clean-skill-374402/topics/gkeupdate,filter=UpgradeEvent
    ```

## C# 訂閱

1. 新增 NuGet

    ```bash
    dotnet add package Google.Cloud.PubSub.V1
    ```

2. 新增 TimedHostedService.cs

    ```cs
    using Google.Cloud.PubSub.V1;

    namespace GCPPubSub;
    
    public class TimedHostedService : BackgroundService
    {
        private readonly ILogger<TimedHostedService> _logger;
        
        public TimedHostedService(ILogger<TimedHostedService> logger)
        {
            _logger = logger;
        }
    
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            _logger.LogInformation("GcpPubSubHostedService is startting.");
            await PullMessages(stoppingToken);
        }

    
        private Task PullMessages(CancellationToken stoppingToken)
        {
            var projectId = "clean-skill-374402";
            var subscriptionId = "gkeupdate";
            var acknowledge = true;
    
            var subscriptionName = SubscriptionName.FromProjectSubscription(projectId, subscriptionId);
            var subscriber = SubscriberClient.Create(subscriptionName);
    
            return subscriber.StartAsync((PubsubMessage message, CancellationToken cancel) =>
            {
                var result = System.Text.Encoding.UTF8.GetString(message.Data.ToArray());
    
                // 這邊可以呼叫其他服務
                Console.WriteLine(
                    $"[{DateTimeOffset.Now}]{result}@{message.PublishTime.ToDateTimeOffset()} from {message.Attributes["cluster_name"]}");
                
                // 下面是用來處理呼叫其他服務後的結果，再決定是否要 ack 刪除通知
                return Task.FromResult(acknowledge ? SubscriberClient.Reply.Ack :SubscriberClient.Reply.Nack);
            });
        }
        
        public override async Task StopAsync(CancellationToken stoppingToken)
        {
            _logger.LogInformation("Consume Scoped Service Hosted Service is stopping.");

            await base.StopAsync(stoppingToken);
        }
    }
    ```

3. Program.cs

    > 註冊 TimedHostedService

    ```cs
    builder.Services.AddHostedService<TimedHostedService>();
    ```

## 手動測試與實際效果

1. 更新 node pool

    > 更新 `gke-test-pubsub` cluster 中的 `default-pool` node pool 至 `1.21.14-gke.5300` (原為 `1.21.14-gke.4300`)

    ```bash
    gcloud container clusters upgrade gke-test-pubsub --region=asia-east1-a --node-pool=default-pool --cluster-version 1.21.14-gke.5300
    ```

    ![1nodepool](https://user-images.githubusercontent.com/3851540/215448136-7ade2c4c-eed2-48eb-9f73-b4418f909781.png)

2. 更新 control plane

    > 更新 `gke-test-pubsub` cluster 中的 control plane 至 `1.22.15-gke.1000` (原為 `1.21.14-gke.4300`)

    ```bash
    gcloud container clusters upgrade gke-test-pubsub --master --region=asia-east1-a --cluster-version 1.22.15-gke.1000
    ```

    ![2controlplane](https://user-images.githubusercontent.com/3851540/215448146-aaa1d01f-d8d4-44bd-9231-bb784bc2bff1.png)

## 心得

一開始有這個想法在嘗試時，就遇到筆記開頭提到的文件太多、太零散的問題，整理完後自己看起來是清楚滿多的

之前的 poc 加上正式環境，相同設定流程我做了 3-4 次，今天紀錄時還是滿卡的，還好有下定決心紀錄，不然之後一定忘記怎麼做

完整程式碼可以參考：[yowko/GCPPubSub](https://github.com/yowko/GCPPubSub)

## 參考資訊

1. [Create and manage topics](https://cloud.google.com/pubsub/docs/create-topic)
2. [Create and use subscriptions](https://cloud.google.com/pubsub/docs/create-subscription)
3. [Receive cluster notifications](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-notifications)
4. [Pull subscriptions](https://cloud.google.com/pubsub/docs/pull)
5. [Manually upgrading a cluster or node pool](https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster)
6. [GoogleCloudPlatform/dotnet-docs-samples](https://github.com/GoogleCloudPlatform/dotnet-docs-samples/blob/HEAD/pubsub/api/Pubsub.Samples/PullMessagesAsync.cs)
7. [yowko/GCPPubSub](https://github.com/yowko/GCPPubSub)
