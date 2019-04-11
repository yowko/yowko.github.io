---
title: "IIS 10.0 開啟 ASP.NET 應用程式出現 403.14 錯誤"
date: 2018-02-08T02:04:00+08:00
lastmod: 2018-10-03T02:04:04+08:00
draft: false
tags: ["ASP.NET MVC","IIS","Windows 10","Windows Server 2016"]
slug: "iis-10-aspnet-40314-error"
aliases:
    - /2018/02/iis-10-aspnet-40314-error.html
---
# IIS 10.0 開啟 ASP.NET 應用程式出現 403.14 錯誤
最近一兩周嘗試找出同事提出的問題背後所隱含的根本原因，同事遇到的狀況已經解決，但遲遲沒有找出自己可以接受的答案實在不夠痛快，經過幾天反覆驗證終於好像看見真相的一道曙光時，竟然在建立全新環境時做最後確認時遇到 403.14 錯誤

這個 403.14 錯誤對於常常在 IIS 上建立 ASP.NET 站台的朋友想必一定不陌生，而這次解決問題的方式與以往經驗不同，藉此紀錄備查

## 錯誤訊息

1.  訊息內容

    ```
    HTTP 錯誤 403.14 - Forbidden
    網頁伺服器已設為不列出此目錄的內容。
    
    最有可能的原因:
    未設定所要求 URL 的預設文件，且未在伺服器上啟用瀏覽目錄。
    
    解決方法:
    如果您不想要啟用瀏覽目錄，請確認已設定預設文件且檔案存在。
    使用 IIS 管理員啟用瀏覽目錄。
    開啟 IIS 管理員。
    在 [功能] 檢視中，按兩下 [瀏覽目錄]。
    在 [瀏覽目錄] 網頁的 [動作] 窗格中，按一下 [啟用]。
    確認站台或應用程式設定檔案中的 configuration/system.webServer/directoryBrowse@enabled 屬性已設為 true。
    
    詳細錯誤資訊:
    模組    DirectoryListingModule
    通知    ExecuteRequestHandler
    處理常式    StaticFile
    錯誤碼    0x00000000
    要求的 URL    http://localhost:8081/
    實體路徑    C:\tmpIIS\test
    登入方法    匿名
    登入使用者    匿名
    ```

2.  錯誤截圖

    ![1error40314](https://user-images.githubusercontent.com/3851540/35932519-cb42b614-0c72-11e8-9d71-6c4cedf5e6bd.png)

## 解決方式

> IIS 未安裝 ASP.NET 模組造成

1.  Windows 10
    *   Control Panel --> Programs --> Turn Windows features on or off

        ![2win101](https://user-images.githubusercontent.com/3851540/35932520-cb736b92-0c72-11e8-90ba-9e3dbb005a88.png)

    *   Internet Information Services --> World Wide Web Services --> Application Development Features

        ![2win102](https://user-images.githubusercontent.com/3851540/35932522-cba17794-0c72-11e8-8b2f-b49872886c92.png)

    *   勾選 `ASP.NET`

        ![2WIN103](https://user-images.githubusercontent.com/3851540/35932523-cbd1db82-0c72-11e8-9f29-922cf5934b46.png)

    *   會自動勾選其他其他需要功能

        ![2WIN104](https://user-images.githubusercontent.com/3851540/35932524-cbfb606a-0c72-11e8-8f01-8143797c0e80.png)

2.  Windows 2016
    *   伺服器管理員 --> 管理 --> 新增角色及功能

        ![3win20161](https://user-images.githubusercontent.com/3851540/35932525-cc252ad0-0c72-11e8-8dc2-2ad270fa714d.png)

    *   下一步至 `伺服器角色`

        ![3WIN20162](https://user-images.githubusercontent.com/3851540/35932526-cc4ec868-0c72-11e8-97d2-400871178dda.png)

    *   勾選 `ASP.NET` 後會出現確認畫面

        ![3win20163](https://user-images.githubusercontent.com/3851540/35932527-ccc2355a-0c72-11e8-9ceb-81e5db16c0c3.png)

    *   確認後會自動選取其他需要功能

        ![3win20164](https://user-images.githubusercontent.com/3851540/35932528-cd0f3d28-0c72-11e8-8ea6-31a9c5894da2.png)

    *   安裝功能確認清單

        ![3win20165](https://user-images.githubusercontent.com/3851540/35932529-cd3c849a-0c72-11e8-811b-188e11e13c1d.png)

## 心得

前面提過 403.14 不算是稀有的錯誤，想要特別紀錄解決方式的原因有兩個：

1.  跟過往的處理方式不同

    > 印象中如果是先安裝 .Net Framework 再安裝 IIS，需要自行註冊 ASP.NET (使用 `aspnet_regiis.exe` 指令)，但在 IIS 10.0 的環境中不需自行註冊，安裝 ASP.NET 完成後立即套用

2.  Windows Server 2016 的操作還不熟悉

    > 藉紀錄的機會多摸幾次，順便也留個操作流程，日後參考

# 參考資訊

1.  [ASP.NET 4.0 安裝在 IIS6 最常遇到的四個問題](https://blog.miniasp.com/post/2010/06/22/IIS-6-ASPNET-4-Installation-Notes.aspx)
