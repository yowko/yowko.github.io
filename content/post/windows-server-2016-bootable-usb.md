---
title: "建立可開機的 Windows Server 2016 安裝 USB"
date: 2016-12-06T00:42:34+08:00
lastmod: 2021-10-04T00:42:34+08:00
draft: false
tags: ["Windows Server 2016"]
slug: "windows-server-2016-bootable-usb"
aliases:
    - /2016/12/windows-server-2016-usb.html
---
## 建立可開機的 Windows Server 2016 安裝 USB

Windows Server 2016 有好多新功能想要試試(ex:docker、Nano Server、Web Proxy )，於是打算透過 USB 利用舊筆電來安裝測試，就讓我們來看看該如何製作可以用來安裝 Windows Server 2016 的開機 USB 吧.

## 設定步驟

1. 把 Windows Server 2016 的 ISO 檔以虛擬光碟掛載

    在`ISO`檔上按右鍵選`掛載`

    ![mount](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/1200x721/b4a0a82800add7278de7d0bdc08de9a9/mount_%E7%BB%93%E6%9E%9C.png)

2. 確認 USB 已連接至電腦

3. 以管理員身份開啟 commnad line(cmd.exe)

    - 3-1. 按 `Ctrl+X,A`

        ![CTRL_X_A](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/375x915/ab8b8ecc6ffabdf3302ab560ee5722c0/ctrl_x_a_%E7%BB%93%E6%9E%9C.png)

    - 3-2. 按 `Win` 鍵，輸入 `cmd.exe`, 並按右鍵`以管理員執行`

        ![CMD.EXE](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/521x864/281eb23ac031e66386a3dd226196c21b/runasadmin_%E7%BB%93%E6%9E%9C.png)

4. 設定 USB

    - 4-1. 執行 `Diskpart`

        ![diskpart](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/84b3d2e1b23a432e727cfb5d58767eae/diskpart_%E7%BB%93%E6%9E%9C.png)

    - 4-2. 列出現有磁碟 `list disk`

        ![list disk](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/b68f405cb8761116c331114b832c3689/listdisk_output.png)

    - 4-3. 選擇 USB `select disk #`
        - `#` 是 USB 磁碟的所在數字
            ![select disk](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/1686c4f7e2c44da637b54b0e765c51df/Select_output.png)

    - 4-4. 確認有選到 USB `list disk`
        - 磁碟前方會出現 `*`

            ![selected](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/6d8944c9f97b59fb213217dd0a70534b/Selected2_output.png)

    - 4-5. 清除 USB `clean`<span style="color:red;">會直接清除 USB 所有資料 </span>

        ![clean](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/5e046e5ee5fe399aaa28802901bcca1f/clean_output.png)

    - 4-6. 建立主要分割 `create partition primary`

        ![Create](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/2b4138efed657c2c6de3faadbedff312/create_partition_primary_output.png)

    - 4-7. 還擇主要分割 `select partition 1`

        ![partition](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/422b2af4cf1ff947e0cdd2be868ad9a5/select_partition_1_output.png)

    - 4-8. 將所選的分割標記為啟動 `active`

        ![active](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/c16a94bde025738c7ce97395bd307509/active_output.png)

    - 4-9. format USB `format fs=ntfs quick label=」Server2016」`
        - 如果無法曾經將 USB format 成 `fat32`, 建議將 `quick` 參數移除

            ![format](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/440ec86c28f2192d804ea6f6a8cf77b3/format_output.png)

    - 4-10. 完成，確認沒有錯誤發現即可離開 `exit`

        ![exit](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/85280fe7231e7d1ccb5ba322e165734e/exit_output.png)

5. 設定 USB 可供開機資訊

    - 5-1. 切換到掛載 ISO 的磁區

        ![change](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/c82d8a76c29e2fd4319f503562708740/changetoboot_%E7%BB%93%E6%9E%9C.png)

    - 5-2. 進到 `boot`

        ![cdboot](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/2c1253045d43bcdb37ff745f5bbb6621/cdboot_output.png)

    - 5-3. 設定開機資訊 `bootsect /nt60 f:`(可以用 `bootsect /help` 查看相關參數)
        -`f` 為 USB 磁碟代碼

        ![bootOK](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/5d21cde782e4315d59f17e095c36cc64/bootsecOK_%E7%BB%93%E6%9E%9C.png)

6. 從虛擬光碟複製資料到 usb

    - robocopy G: F: /E
        - `G:`  來源的虛擬光碟的磁碟代碼
        - `F:`  USB 磁碟代碼
        - `/E`  全部複製

        ![robocopy](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/9fc1d2435a7fdbb57ac5c4d7b965e818/rococopy_%E7%BB%93%E6%9E%9C.png)

     - <span style='color:red'>注意目錄位置，如果仍停留在`5. 設定 USB 可供開機資訊` 中，請切換到根目錄再進行</red>

        ![copydone](https://trello-attachments.s3.amazonaws.com/581f4a15aec088e5d581d875/988x651/7020f70b22b027f909dd480bca80a81a/copydone_%E7%BB%93%E6%9E%9C.png)

## 參考資料

1. [Create Bootable Windows Server 2016 USB Installation Drive Step-by-step](https://channel9.msdn.com/Blogs/ITProGuru/Create-Bootable-Windows-Server-2016-USB-Installation-Drive-Step-by-step)
2. [Create Bootable Windows Server 2016 USB Thumb Drive for Installing OS](http://itproguru.com/expert/2016/05/create-bootable-windows-server-2016-usb-thumb-drive-for-installing-os/)
3. [DiskPart 命令列選項](https://technet.microsoft.com/zh-tw/library/cc766465.aspx)
