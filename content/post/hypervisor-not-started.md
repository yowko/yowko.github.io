---
title: "無法啟動 Hyper-V 虛擬機器 - Hypervisor 未執行"
date: 2017-08-16T23:23:00+08:00
lastmod: 2018-09-24T23:23:37+08:00
draft: false
tags: ["Debug","Hyper-V","Windows Server 2016","Windows Service"]
slug: "hypervisor-not-started"
aliases:
    - /2017/08/hypervisor-not-started.html
---
# 無法啟動 Hyper-V 虛擬機器 - Hypervisor 未執行
之前為了測試 Windows Container 功能，在筆電上安裝 Windows Server 2016，加上需要 Linux 環境，也啟用了 Hyper-V 功能安裝了 CentOS 7，測試靠一段落後有陣子沒摸它們，最近竟無法順利開啟 CentOS 7 VM

印象中這是第二次遇到這個問題，依慣例就是要紀錄一下解法囉

## Hypervisor 未執行

1.  錯誤訊息

    ```
    嘗試啟動選取的虛擬機器時發生錯誤
    
    'CentOS 7' 無法啟動。
    
    因為 Hypervisoe 未執行，所以無法啟動虛擬機器 'CentOS 7'。
    ```

2.  錯誤截圖

    ![1notactive](https://user-images.githubusercontent.com/3851540/29370759-ec69d1b6-82d8-11e7-8831-631521495d53.png)

3.  問題原因

    > `HV 主機服務` 服務未啟動

    ![2noservice](https://user-images.githubusercontent.com/3851540/29370750-ec3b87de-82d8-11e7-8f94-8c3e38c99915.png)

## 無法啟動 `HV 主機服務`

1.  錯誤訊息

    ```
    Windows 無法啟動 本機電腦 的 HV 主機服務 服務。

    錯誤 31：連結到系統的某個裝置失去作用。
    ```

2.  錯誤截圖

    ![3activefail](https://user-images.githubusercontent.com/3851540/29370751-ec3e6256-82d8-11e7-94ef-040bf3114067.png)

## 可能解決方案

1.  檢查 CPU 是否支援 Virtualization
2.  檢查 BIOS 的 Intel Virtualization 是否開卲
3.  檢查 Hyper-V 虛擬機器管理 服務是否啟動
4.  以系統管理員身份開啟 command prompt 並執行 `bcdedit /set hypervisorlaunchtype auto` 後重新開機
5.  檢查事件檢視器還有沒有別的錯誤訊息


## 其他解決方案

因為將上述可能解決方案試過一輪，錯誤訊息依舊完全沒有改善，最後才試到真正的解法 - 自行加入 regedit value

1.  開啟 `regedit`

2.  開啟 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\hcmon\`

    > 如果 `hcmon` 不存在就自行建立

    ![4newregedit](https://user-images.githubusercontent.com/3851540/29370752-ec3f68e0-82d8-11e7-97df-3cf0f802b5d4.png)

3.  加入新機碼 `Parameters`

    ![5parameters](https://user-images.githubusercontent.com/3851540/29370753-ec3fc308-82d8-11e7-944c-3512bf28b16a.png)

4.  在 `Parameters` 中加入名為 `DisableDriverCheck` 的 `DWORD` 並設定為 `1`

    ![6DisableDriverCheck](https://user-images.githubusercontent.com/3851540/29370755-ec4508cc-82d8-11e7-81e1-975f7a41de2a.png)

5.  重新開機

## 正常運作

1.  HV 主機服務 Service 正確啟動

    ![7HVserviceOK](https://user-images.githubusercontent.com/3851540/29370754-ec42c760-82d8-11e7-875f-14a6e923f7ec.png)

2.  Hyper-V VM 正常啟動

    ![8vmOK](https://user-images.githubusercontent.com/3851540/29370757-ec634ada-82d8-11e7-9c7c-7debac0049ea.png)

## 心得

雖然問題解決了，但發生原因還是不明，這時只能怪人品 XD

這其實就是 Windows 最令人頭痛的地方，發生錯誤都不太好處理，每個人發生錯誤的原因也是各有千秋，讓解決問題變得不容易

經過這次再次確認我真的不適合擔任系統工程師，一來解決問題的效率很差，二來只解決問題的表面還是不懂真正的原因，覺得很虛

# 參考資訊

1.  [Hyper-V問題](https://social.technet.microsoft.com/Forums/zh-TW/7b2f667e-0103-4cd7-bf5f-46b6651418df/hyperv?forum=windowsserver2008zhcht)
2.  [Error 31: A device attached to the system is not functioning, Windows 7 host, v. 7.1.4](https://communities.vmware.com/thread/450807)
