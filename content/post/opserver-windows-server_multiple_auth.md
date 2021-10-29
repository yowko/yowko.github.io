---
title: "如何客製化 Opserver - 使用不同帳號密碼來監控多台 Windows Server"
date: 2017-03-30T01:00:00+08:00
lastmod: 2021-10-29T01:00:44+08:00
draft: false
tags: ["csharp","Monitoring","Tools"]
slug: "opserver-windows-server_multiple_auth"
# aliases:
#     - /2017/03/opserver-windows-server_30.html
#     - /2017/03/oopserver-windows-server_multiple_auth
---
## 如何客製化 Opserver - 使用不同帳號密碼來監控多台 Windows Server

之前文章 [如何使用 Opserver 來監控 Windows Server](/opserver-windows-server) 介紹該怎麼設定 Opserver 來監控 Windows Server，但我們也發現只能使用同一組帳號密碼

<span style="color:red">注意事項</span>：目前我只用到 WMI，所以修改是針對 WMI 來調整，<span style="color:red">bosun 與 orion 暫不適用</span>(其實是我還不會用 `bosun` 與 `orion`)

## 文章大綱

1. 下載 Opserver
2. 修改原始碼
3. 修改 Windows Server 監控設定

## 下載 Opserver

1. clone
    * 從 [GitHub](https://github.com/opserver/Opserver) 下載(clone)

2. 解壓縮至硬碟

## 修改原始碼

1. 修改 `Opserver.Core\Data\Dashboard\Providers\DashboardDataProvider.cs`
    * 修改 `public abstract class DashboardDataProvider<TSettings> : DashboardDataProvider where TSettings : class, IProviderSettings`
        * 將 Settings 改成可以吃多筆

            ```cs
            public IEnumerable<TSettings> Settings { get; protected set; }
            ```

        * `DashboardDataProvider` 建構式參數改成可吃多筆

            ```cs
            protected DashboardDataProvider(TSettings settings) : base(settings)
            ```

    * 修改 `public abstract class DashboardDataProvider : PollNode, IIssuesProvider`
        * `DashboardDataProvider` 建構式參數改成可吃多筆，指定名稱

            ```cs
            protected DashboardDataProvider(IProviderSettings settings) : base(settings.Name + "Dashboard")
            {
                Name = settings.Name;
            }
            ```

    * 詳細修改內容請參考 [GitHub](https://github.com/yowko/Opserver/commit/bdb50eab44445faaa0e3075ad27973bfc264f3a2#diff-dc8af8a3de713096dd2d7cd3e0e0210c)

2. 排除以下檔案(exclude from project - 沒用到相關功能避免造成編譯不過)
    * Opserver.Core\Data\Dashboard\Providers\BosunDataProvider.cs
    * Opserver.Core\Data\Dashboard\Providers\OrionDataProvider.cs
    * Opserver.Core\Data\Dashboard\Providers\BosunDataProvider.Metrics.cs
    * Opserver.Core\Data\Dashboard\Providers\BosunDataProvider.Nodes.cs

3. 修改 `Opserver.Core\Data\Dashboard\DashboardModule.cs`
    * `DashboardModule` 建構式迴圈取值多一層

        ```cs
        foreach (var p in providers.All)
        {
            if (p!=null)
            {
                foreach (var item in p)
                {
                    item?.Normalize();
                }   
            }
        }
        ```

    * 將使用到 `bosun` 與 `orion` 部份移除

        ```cs
        //if (providers.Bosun != null)
        //    Providers.Add(new BosunDataProvider(providers.Bosun));
        //if (providers.Orion != null)
        //    Providers.Add(new OrionDataProvider(providers.Orion));
        ```

    * 詳細修改內容請參考 [GitHub](https://github.com/yowko/Opserver/commit/bdb50eab44445faaa0e3075ad27973bfc264f3a2#diff-16a161cce293d5725eb6b9d6f2ef7d63)

4. 修改 `Opserver.Core\Data\Dashboard\Providers\WmiDataProvider.cs`
    * 修改地方太多，大意就是把原本吃單一設定改為吃多組設定
    * 詳細修改內容請參考 [GitHub](https://github.com/yowko/Opserver/commit/bdb50eab44445faaa0e3075ad27973bfc264f3a2#diff-49a6104a58e6c64d263daaee2b3b41ec)

5. 修改 `Opserver.Core\Monitoring\Wmi.cs`
    * 將原本取一次帳號密碼的動作改為每次連線都取當下設定
    * 詳細修改內容請參考 [GitHub](https://github.com/yowko/Opserver/commit/bdb50eab44445faaa0e3075ad27973bfc264f3a2#diff-da68dd116d9f34cb99f5004b2f504dbe)

6. 修改 `Opserver.Core\Settings\DashboardSettings.Providers.cs`
    * 將設定改為多組資料
    * 詳細修改內容請參考 [GitHub](https://github.com/yowko/Opserver/commit/bdb50eab44445faaa0e3075ad27973bfc264f3a2#diff-551787f02b5a757b4df54449e9dcd280)

## 修改 Windows Server 監控設定

* providers --> wmi 改使用 array

    ```json
    "providers": {
        "wmi": [
        {
            "nodes": [ "192.168.1.30" ],
            "staticDataTimeoutSeconds": 300,
            "dynamicDataTimeoutSeconds": 5,
            "historyHours": 2,
            "Username": "ad1\\yowko1",
            "Password": "password1"
        },
        {
            "nodes": [ "10.11.2.55" ],
            "staticDataTimeoutSeconds": 300,
            "dynamicDataTimeoutSeconds": 5,
            "historyHours": 2,
            "Username": "ad2\\yowko2",
            "Password": "password2"
        },
        {
            "nodes": [ "localhost" ],
            "staticDataTimeoutSeconds": 300,
            "dynamicDataTimeoutSeconds": 5,
            "historyHours": 2
        }
        ]
    },
    ```

* 監控結果

    ![1RESULT](https://cloud.githubusercontent.com/assets/3851540/22058136/d8f37fc8-dda2-11e6-996b-dd3a9319c193.png)

## 參考資料

1. [Opserver GitHub](https://github.com/opserver/Opserver)
2. [Yowko's Repository(GitHub)](https://github.com/yowko/Opserver)
