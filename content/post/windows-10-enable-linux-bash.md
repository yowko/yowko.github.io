---
title: "在 Windows 10 上啟用 Linux Bash Shell"
date: 2018-02-20T15:31:00+08:00
lastmod: 2021-10-08T15:31:33+08:00
draft: false
tags: ["Linux","Windows 10"]
slug: "windows-10-enable-linux-bash"
aliases:
    - /2018/02/windows-10-enable-linux-bash.html
---
## 在 Windows 10 上啟用 Linux Bash Shell

主要使用的筆電在某次更新失敗後就再也無法完成更新，一直停留在 Windows 10 舊版本，雖然在備用機上的 Windows 10 仍正常運作不至於錯過了幾個有趣的功能，但近期的 CPU 漏洞更新在每次開機都會重新執行，讓我實際無法在忽視這件事，所以趁著農曆新年的空閒時間重新安裝了 Windows 10，結果發現 Windows 10 上啟用 Bash 的流程有些不同，紀錄一下

<!--## 開啟 Windows 10 開發者模式
1. 按下 Win 圖示 (或是 鍵盤 Win key) -> Setting
    >![1winsetting](https://user-images.githubusercontent.com/3851540/36411526-0c663052-1651-11e8-97d7-7c274f2d89b0.png)
2. Update & Security
    >![2updatesecurity](https://user-images.githubusercontent.com/3851540/36411527-0c8fd83a-1651-11e8-8e66-9b4455a2725f.png) 
3. For Developers -> Developer mode
    >![3devmode](https://user-images.githubusercontent.com/3851540/36411529-0cba083a-1651-11e8-9c25-3ef43aa2a4e8.png)
    - 確認訊息
        >![4confim](https://user-images.githubusercontent.com/3851540/36411530-0ce3b900-1651-11e8-8d01-b0deb6786f09.png)
    - 安裝套件
        >![4installpkg](https://user-images.githubusercontent.com/3851540/36411532-0d3c41b0-1651-11e8-89c4-cfb9e6545b70.png) 
    - 完成設定
        >![6installed](https://user-images.githubusercontent.com/3851540/36411533-0d66304c-1651-11e8-8d34-5b4d5c665e61.png) -->

## 安裝 `Windows Subsystem for Linux`

* 方法一：使用 GUI

    1. 按下 Win 圖示 (或是 鍵盤 Win key) --> 搜尋 `Turn Windows features on or off`

        ![7search](https://user-images.githubusercontent.com/3851540/36411534-0d97808e-1651-11e8-908b-f8be3ccfe83a.png)

    2. 勾選 `Windows Subsystem for Linux`

        ![8subsystem](https://user-images.githubusercontent.com/3851540/36411518-0af9037a-1651-11e8-9a54-3fa38926e078.png)

    3. 重新開機

        ![8-1restart](https://user-images.githubusercontent.com/3851540/36411517-0acf3202-1651-11e8-86b7-e1d182305c1b.png)

* 方法二：使用 PowerShell

    1. 使用管理者權限開啟 PowerShell
    2. 執行 `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`
    3. 重新開機

    ![9ps](https://user-images.githubusercontent.com/3851540/36411519-0b210db6-1651-11e8-9c49-c6ecbef0e14e.png)

## 安裝 Linux 子系統

擇一即可，目前支援的有

* [Ubuntu](https://www.microsoft.com/store/p/ubuntu/9nblggh4msv6)
* [OpenSUSE](https://www.microsoft.com/store/apps/9njvjts82tjx)
* [SLES](https://www.microsoft.com/store/apps/9p32mwbh6cns)

1. 開啟 Microsoft Store 搜尋需要的 OS (或是直接點擊上述各系統的直接連結)
2. 按下 `Get`

    ![10getlinux](https://user-images.githubusercontent.com/3851540/36411520-0b4afebe-1651-11e8-8f12-deb91f0b28dd.png)

3. 下載完成後 --> Launch

    ![11launch](https://user-images.githubusercontent.com/3851540/36411521-0b740048-1651-11e8-8b32-0ea017cad651.png)

4. 安裝系統

    ![12install](https://user-images.githubusercontent.com/3851540/36411522-0b9c74c4-1651-11e8-8962-a86dffb1f529.png)

5. 建立使用者帳號(不需要與 Windows 使用者帳號相同) 與密碼

    ![13usernamepassword](https://user-images.githubusercontent.com/3851540/36411523-0be80e70-1651-11e8-8724-86f6319198c7.png)

6. 完成設定

    ![14done](https://user-images.githubusercontent.com/3851540/36411524-0c10ae8e-1651-11e8-85f5-ea7373cf654f.png)

## 未安裝 Linux 子系統直接開啟 `bash` 的錯誤

* 錯誤訊息

    ```txt
    Windows Subsystem for Linux has no installed distributions.
    ```

* 錯誤截圖

    ![15error](https://user-images.githubusercontent.com/3851540/36411525-0c3b626e-1651-11e8-8cf4-bcdca7b2c895.png)

## 心得

之前安裝 bash 時還需要先啟用開發者模式，Windows subsystem for linux 也有 beta 字樣，Windows 10 新版本則都不需要了，但因為搭配的 linux 版本有 ubuntu 之外的其他選擇而需自行安裝，步驟上相距不遠，但由此可見 Windows 10 在這些功能上仍持續發展中，也許再過一陣子又有全新不同的安裝方式了

## 參考資訊

1. [Install the Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10?WT.mc_id=DOP-MVP-5002594)
