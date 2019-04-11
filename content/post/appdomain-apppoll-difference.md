---
title: "Application Domain 與 Application Pool 的差異"
date: 2018-12-22T23:45:00+08:00
lastmod: 2018-12-22T23:44:30+08:00
draft: false
tags: ["dotnet","IIS"]
slug: "appdomain-apppoll-difference"
---
# Application Domain 與 Application Pool 的差異
最近同事問到修改 IIS 上站台的 log path 會不會引發重啟，雖然針對同事的問題有九成把握，但對於實際運作細節卻有許多疑問，所以想趁著這個機會順便理解 IIS recycle 的流程，一看發現我對 Application Domain 與 Application Pool 的差異沒有十足把握，趁著整理內容來釐清一些觀念

IIS 重啟可能由兩個部份觸發：

1. Application Domain recycles
2. Application Pool recycles

## 關於 Application Domain 與 Application Pool

> 兩個都是用來隔離執行程式、避免相互干擾，差在實作的方式不同

1. Application Domain (AppDomain)
   - 透過 CLR 來隔離使用在同個 AppPool 下不同 web site 間的交互影響
   - 是 .NET 中的術語
   
2. Application Pool (AppPool)
   - 透過 process 來隔離使用不同 AppPool 的每個 web site
   - 是 IIS 中的術語
   - 一個 AppPool 會執行在一個或多個 w3wp.exe worker process 下
        
        > 可以啟用多個 worker prcess
   - 允許多個 web site (AppDomain) 同時在一個 AppPool 中執行，但建議將每個 application (web site) 獨立使用自己的 application pool 較好
   - 可以指定不同 .NET CLR 版本
3. 兩者關係

    ![appdomain_apppool](https://user-images.githubusercontent.com/3851540/50376494-45cd3880-0648-11e9-8fb3-d1cb5a43b6fd.png)


## Application Domain 與 Application Pool 回收
1. Application Domain
   * 造成 Application Domain 回收的原因
      - 修改 `web.config` 或 `Global.asax`
      - 修改 `bin` 資料夾內容
      - 修改 `虛擬目錄` 的實體路徑
      - 刪除 application (web site) 的子資料夾
      - 動態重新編譯 (`aspx`、`ascx`、`asax`) 數量超出設定值 (預設值: `15`)
          
            > 可以經由 `machine.config` 或 `web.config` 的 `<compilation numRecompilesBeforeAppRestart=/>` 來調整設定
    * Application Domain 回收的影響
      - 效能衝擊

            > 所有組件會從記憶體 卸載、重新載入並重新即時編譯
      - 所有 in-process 變數的內容都會消失

            > 包括 `session`, `application`, `cache` 這些預設儲存在記憶體中的內容

2. Application Pool
   * 造成 Application Pool 回收的原因
        - 可設定回收條件
            - 定期時間間隔 (預設 `1740` 分)
            - 固定 request 數量 (預設 `0` : 無限)
            - 指定時間 (預設 `空 array`: 未指定回收時間)
            - worker process 使用虛擬記憶體用量 (預設 `0`: 無限)
            - worker process 使用私有記憶體用量(預設 `0`: 無限)
        - 閒置時間 (預設 `20` 分)
        - 修改設定
   
            > 預設情況只有修改 Application Pool 會觸發 Application Pool 回收，可以透過修改 `Disable Recycling for Configuration Changes` 來避免，但如此一來想套用新設定則需自行手動重啟

   * Application Pool 回收的影響
        - 所有 in-process 變數的內容都會消失

            > 包括 `session`, `application`, `cache` 這些預設儲存在記憶體中的內容

    * 預設 Application Pool 執行回收時會先另外啟一個 process ，確定可對外提供服務才將 task 移過去並 kill 原 process
        ![1apppoolrecycle](https://user-images.githubusercontent.com/3851540/50376505-609fad00-0648-11e9-8cbc-aa394485a9fe.gif)

## 心得
雖然看了一些介紹與說明文件、大致理解了兩者差異，但還是對於設計理念有些不清楚：

1. AppPool 的存在只是用來讓子資料夾可以使用不同 .NET Framework 嗎？
   
    > 不能直接賦于 AppDomain 這個能力就好嗎
2. 為了批次管理多個 website (AppDomain) ?

    > best practice 又說一個 website 對應一個 Application Pool

內容好多，愈看愈覺得自己還有好多東西不懂，真的學不完呀

# 參考資訊
1. [Application vs. AppDomain](https://weblogs.asp.net/owscott/application-vs-appdomain)
2. [IIS Application Domain and Pool Recycling](https://www.treeloop.com/blog/iis-application-domain-and-pool-recycling)
3. [IIS Application Pool](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc735247%28v=ws.10%29)
4. [Application Pools <applicationPools>](https://docs.microsoft.com/en-us/iis/configuration/system.applicationhost/applicationpools/)
5. [Difference between Application Pool and Application Domain](https://social.msdn.microsoft.com/Forums/vstudio/en-US/fd865e35-a2ee-41b8-b112-5913f15c96f2/difference-between-application-pool-and-application-domain?forum=clr)