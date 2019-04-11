---
title: "建立支援 UEFI 開機的 Windows Server 2016 安裝 USB"
date: 2016-12-18T00:42:34+08:00
lastmod: 2018-09-06T00:42:34+08:00
draft: false
tags: ["Windows Server 2016","PowerShell"]
slug: "windows-server-2016-uefi-usb"
aliases:
    - /2016/12/uefi-windows-server-2016-usb.html
---
# 建立支援 UEFI 開機的 Windows Server 2016 安裝 USB
之前的筆記 [建立可開機的 Windows Server 2016 安裝 USB](http://blog.yowko.com/2016/12/windows-server-2016-usb.html)，我們看了如何建立可開機的 USB。

而我自己在安裝 Windows Server 2016 過程中，卻是一開機就愣住：太久沒重灌忘了我的筆電預設使用 UEFI (UEFI 介紹可以看 [即將換掉傳統 BIOS 的 UEFI，你懂了嗎？（三）](https://www.techbang.com/posts/4361-fully-understand-uefi-bios-theory-and-actual-combat-3-liu-xiudian) 跟 [EFI、UEFI、MBR、GPT的區別](https://read01.com/kg2KyP.html#.W5FV_fn3l3g))，雖然透過調整開機設定還是能正常安裝，但想到下次可能仍然會忘記，所以紀錄一下供日後查閱.

# 使用 diskpart
使用的方式與前一篇 [建立可開機的 Windows Server 2016 安裝 USB](https://blog.yowko.com/2016/12/(http://blog.yowko.com/2016/12/windows-server-2016-usb.html)) 大致相同，只是在針對 UEFI 設定會有些不同

MVP-Weithenn 寫的很詳細，就請大家直接參閱專家的文章- [製作用於 UEFI 的 Bootable USB](http://www.weithenn.org/2016/01/uefi-bootable-usb.html)

# 使用 PowerShell
看過百敬老師的 PowerShell 介紹，不由得讚嘆其強大跟便利，雖然讓人躍躍欲試，但機會實在不多，剛好看到有國外高手用 PowerShell 寫的 [How to create UEFI bootable USB media to install Windows Server 2016](https://p0w3rsh3ll.wordpress.com/2016/10/30/how-to-create-uefi-bootable-usb-media-to-install-windows-server-2016/) 覺得有些改善的空間，就來動手修改囉。

1. 基本要求
    - 1-1. USB 空間需大於 5.3 GB
    - 1-2. UEFI 需要 FAT32 分割
    - 1-3. FAT32 先天的限制，所以製作時需要將其中的大檔拆分成適當的大小(瞭解更多)

2. 使用 `PowerShell` 來建立開機USB
國外高手的文章 - How to create UEFI bootable USB media to install Windows Server 2016，<span style="color:red">因為沒有指定 USB ，有可能造成連結到電腦的多個 USB 資料都被清除</span>，我調整的寫法就是針對這個問題

`.ps` 檔案在這 [GITHUB](https://github.com/yowko/CreateUEFIBootableUSB)，下面是程式碼說明

```ps1
# 指定 Windows Server 2016 的 ISO 檔路徑
$iso = 'C:\ct_windows_server_2016_x64_dvd_9327748.iso'
# 指定 USB 的磁碟代碼
$usbdiskletter = 'F'
# 取得 USB 的磁碟資訊
$usb =Get-Partition| where DriveLetter -eq "$usbdiskletter"|Get-Disk
# 清除 USB 資料
$usb| Clear-Disk -RemoveData -Confirm:$true -PassThru
# 將 USB 設定為 `GPT`
if ($usb.PartitionStyle -eq 'RAW') {
    $usb | Initialize-Disk -PartitionStyle GPT
} else {
    $usb | Set-Disk -PartitionStyle GPT
}
# 建立主要分割並格式化成 `FAT32`
$volume = $usb | New-Partition -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem FAT32
# 檢查 USB 是否已連結
if (Test-Path -Path "$($volume.DriveLetter):\") {
    # 掛載 Windows Server 2016 ISO
    $miso = Mount-DiskImage -ImagePath $iso -StorageType ISO -PassThru
    # 取得 Windows Server 2016 虛擬光碟機代碼
    $dl = ($miso | Get-Volume).DriveLetter
}
# 檢查 Windows Server 2016 虛擬光碟是否有 `install.wim`
if (Test-Path -Path "$($dl):\sources\install.wim") {
    # 複製除 install.wim 外的 Windows Server 2016 虛擬光碟 內容到 USB
    & (Get-Command "$($env:systemroot)\system32\robocopy.exe") @(
        "$($dl):\",
        "$($volume.DriveLetter):\"
        ,'/S','/R:0','/Z','/XF','install.wim','/NP'
    )
    # 分割 install.wim
    & (Get-Command "$($env:systemroot)\system32\dism.exe") @(
        '/split-image',
        "/imagefile:$($dl):\sources\install.wim",
        "/SWMFile:$($volume.DriveLetter):\sources\install.swm",
        '/FileSize:4096'
    )
}
# 退出 USB
(New-Object -comObject Shell.Application).NameSpace(17).
ParseName("$($volume.DriveLetter):").InvokeVerb('Eject')

# 卸載 Windows Server 2016 ISO
Dismount-DiskImage -ImagePath $iso
```

# 參考資料
1. [即將換掉傳統 BIOS 的 UEFI，你懂了嗎？（三](http://www.techbang.com/posts/4361-fully-understand-uefi-bios-theory-and-actual-combat-3-liu-xiudian)
2. [EFI、UEFI、MBR、GPT的區別](https://read01.com/kg2KyP.html)
3. [製作用於 UEFI 的 Bootable USB](http://www.weithenn.org/2016/01/uefi-bootable-usb.html)
4. [How to create UEFI bootable USB media to install Windows Server 2016](https://p0w3rsh3ll.wordpress.com/2016/10/30/how-to-create-uefi-bootable-usb-media-to-install-windows-server-2016/)
5. [Get-Volume(PowerShell)](https://technet.microsoft.com/en-us/library/hh848646.aspx)
6. [Get-Disk(PowerShell)](https://technet.microsoft.com/zh-tw/library/hh848657.aspx)