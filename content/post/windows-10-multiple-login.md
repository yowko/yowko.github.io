---
title: "讓 Windows 10 允許多個使用者同時登入"
date: 2018-10-15T00:10:10+08:00
lastmod: 2018-10-15T00:10:10+08:00
draft: false
tags: ["Windows 10"]
slug: "windows-10-multiple-login"
aliases:
    - /2018/10/windows-10-multiple-login
---
# 讓 Windows 10 允許多個使用者同時登入
這個需求是發生在已做為測試機的筆電上，平常如果需要測試程式一般都會透過遠端桌面登入操作，但有時需要測試網路層相關功能無法透過遠端桌面而直接登入實體機操作，不過執行時間一久忘記正在執行其他程式又經由遠端桌面進去踢掉原來的使用者 XD

雖然發生頻率不高，但一發生都損失慘重，乾脆紀錄一下該如何修改吧

## 方法一：修改 `termsrv.dll`

1. 位置
    
    > `C:\Windows\System32\termsrv.dll`
2. 備份：

    >避免改壞救不回來

3. 停止 `Remote Desktop Services` 

    > Net stop TermService

    ![1termservice](https://user-images.githubusercontent.com/3851540/46962895-e5190100-d0d6-11e8-8df7-5fe4efc39b0d.png)
4. 找到 `00010270` 位置 (可以使用 `HxD editor`)

    > 將數值修改為 `B8 00 01 00 00 89 81 38 06 00 00 90`，因為每個 Windows 10 的預設數值不定，透過數值搜尋很容易因為版本不同找不到

    ![2offset](https://user-images.githubusercontent.com/3851540/46962896-e5190100-d0d6-11e8-9c4d-a1946967d8e0.png)
5. 重新啟動 `Remote Desktop Services`

    > Net start TermService

* ~~這個方式我一直遇到權限問題無法順利修改~~

    > 經發哥提點，發現該檔案預設擁有者不是 OS 的使用者以致無法修改，只要將擁有者改為 OS 上的使用者即可

    ![owner](https://user-images.githubusercontent.com/3851540/50097795-99f5a880-0255-11e9-8c8c-961cd462a7b9.png)

## 方法二：使用 `RDP Wrapper`

1. 下載並解壓縮 [RDP Wrapper](https://github.com/stascorp/rdpwrap/releases)

2. 執行 `install.bat`

    ![3installbat](https://user-images.githubusercontent.com/3851540/46962898-e5b19780-d0d6-11e8-8f3a-d8d94ff1e761.png)

    ![3installbatdeon](https://user-images.githubusercontent.com/3851540/46963331-f4e51500-d0d7-11e8-8f79-a225a0ac9c27.png)
3. 確認是否已完成 - 執行 `RDPConf.exe`

    ![4rdpconf](https://user-images.githubusercontent.com/3851540/46962899-e5b19780-d0d6-11e8-9403-fac94566562a.png)

    ![5wrapok](https://user-images.githubusercontent.com/3851540/46962900-e5b19780-d0d6-11e8-84f0-32a6611140fe.png)

## 心得
原本期望可以使用一個使用者帳號同時登入，但沒找到 Windows 10 上的相關設定，退而求其次讓不同帳號可以同時登入也是能解決問題

實際效果

![6result](https://user-images.githubusercontent.com/3851540/46962901-e64a2e00-d0d6-11e8-8355-8d855f2fd181.png)

# 參考資訊
1. [Multiple RDP (Remote Desktop) sessions in Windows 10](https://www.mysysadmintips.com/windows/clients/545-multiple-rdp-remote-desktop-sessions-in-windows-10)
2. [Remote Desktop Connections for Multiple Users on Windows 10 and Windows Server 2012](https://www.serverwatch.com/server-tutorials/remote-desktop-connections-for-multiple-users-on-windows-10-and-windows-server-2012.html)