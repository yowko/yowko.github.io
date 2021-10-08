---
title: "在 Windows 環境將特定網址指向不同 IP"
date: 2018-12-09T23:43:00+08:00
lastmod: 2021-10-08T23:44:30+08:00
draft: false
tags: ["Windows","IIS"]
slug: "windows-host-file"
---
## 在 Windows 環境將特定網址指向不同 IP

這幾天正在測試 HttpClient 幾個過去被誤用的現象與解決方式，其中一個可能遇到的問題是使用 static HttpClient instance 時如果遇到 DNS 異動時無法即時反應，[Singleton HttpClient? Beware of this serious behaviour and how to fix it](http://byterot.blogspot.com/2016/07/singleton-httpclient-dns.html) 作者在文中提到在做 blue-green 部署或是 Azure 上的 slot swap 時特別容易遇到這個問題：對外 url 沒有異動，實際 ip 卻已改變，此時 static HttpClient instance 就會持續使用舊 ip 連線

原本想要實際建立 Azure instance 來做實驗，但後來想到可以透過低成本又相對簡單的 local hosts file 來模擬，順手紀錄一下

## 關於 hosts file

Windows 使用 hosts 檔案儲存主機名稱與數字網際網路通訊協定 (IP) 位址對應，以用來識別並尋找網路上的主機 IP

>- 第一欄：IP
>- 第二欄: 主機名稱
>- 欄位間可以用 `空白字元` or `tab` 分隔
>- 註解符號為 `#`

1. hosts 中有資料

    ![hostsflow](https://user-images.githubusercontent.com/3851540/49699241-c41de980-fc09-11e8-909f-691528d33222.png)

2. hosts 中沒有資料，向 DNS 查詢

    ![dnsflow](https://user-images.githubusercontent.com/3851540/49699240-c41de980-fc09-11e8-8b29-55d5fba82aab.png)

## 將 blog.yowko.com 指向本機 IIS

1. 修改 hosts 檔案
    - 位置：`%windir%\System32\drivers\etc\hosts` (C:\Windows\System32\drivers\etc\hosts)

    - 將 ip 與 domain 加入 hosts 中

        ```txt
        127.0.0.1    blog.yowko.com #for local test
        ```

    - 無法直接存檔

        > hosts 一般使用者只有 `read&execute` 權限，修改後需先儲存至其他位置再提供管理者權限覆蓋至 `%windir%\System32\drivers\etc\`

        ![3rights](https://user-images.githubusercontent.com/3851540/49699242-c41de980-fc09-11e8-80aa-70d32aacd61d.png)

        ![4errprsaving](https://user-images.githubusercontent.com/3851540/49699243-c41de980-fc09-11e8-9474-2470bd575f95.png)

        ![5replace](https://user-images.githubusercontent.com/3851540/49699244-c4b68000-fc09-11e8-9ff3-681516a587c7.png)

        ![6adminpermission](https://user-images.githubusercontent.com/3851540/49699245-c4b68000-fc09-11e8-87ee-da3bbaf409dc.png)

    - 可以使用 administrator 權限開啟編輯器再修改 hosts 即可直接儲存

        ![7runasadmin](https://user-images.githubusercontent.com/3851540/49699247-c4b68000-fc09-11e8-8f44-9ad94172e399.png)

        ![8adminopen](https://user-images.githubusercontent.com/3851540/49699248-c54f1680-fc09-11e8-918b-c12f8643148b.png)
2. 加入 IIS website binding

    > 未指定 binding 的站台就會直接指向該 ip 的 80 port 站台

    ![9default](https://user-images.githubusercontent.com/3851540/49699249-c54f1680-fc09-11e8-8318-b0f81dae3a37.png)

    - 開啟 IIS Manager --> 目標站台 --> Bindings

        ![10bindings](https://user-images.githubusercontent.com/3851540/49699250-c54f1680-fc09-11e8-9b71-b821b237eecc.png)

    - Add

        ![11add](https://user-images.githubusercontent.com/3851540/49699251-c54f1680-fc09-11e8-8b56-c30b557d9943.png)
    - 指定 Host name --> OK

        ![12addhost](https://user-images.githubusercontent.com/3851540/49699252-c5e7ad00-fc09-11e8-9cf4-f42c6dfc98cc.png)

3. 實際效果

    - 修改前
        - ping

            ![13pingbefore](https://user-images.githubusercontent.com/3851540/49699236-c2ecbc80-fc09-11e8-99d3-e886a90b18ee.png)

        - HTTP request

            ![14httpbeofre](https://user-images.githubusercontent.com/3851540/49699237-c3855300-fc09-11e8-99f0-00d9b1e1f3c0.png)
    - 修改後
        - ping

            ![15pingafter](https://user-images.githubusercontent.com/3851540/49699238-c3855300-fc09-11e8-856f-ab3146ec149d.png)

        - HTTP request

            ![16httpafter](https://user-images.githubusercontent.com/3851540/49699239-c3855300-fc09-11e8-998b-22bb248f865f.png)

## 心得

紀錄至此，突然發現離原始的想法好遠，每次看到自己掌握度沒那麼高的內容，都會鑽進去了解一番，也因為如此常常多花了許多時間，卻反而累積了更多東西要學XD

相信 hosts file 對於許多網站開發人員都很熟悉，我過去也有相關使用經驗，一般情況應該都能正確設定，但透過紀錄再次理解查找 server ip 的流程還是滿值得的

## 參考資訊

1. [Singleton HttpClient? Beware of this serious behaviour and how to fix it](http://byterot.blogspot.com/2016/07/singleton-httpclient-dns.html)
2. [手動設定網址與 IP 對應的 hosts 檔教學，適用 Windows、Mac OS X 與 Linux 系統](https://blog.gtwang.org/windows/windows-linux-hosts-file-configuration/)
