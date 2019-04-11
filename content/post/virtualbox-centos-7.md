---
title: "VirtualBox 如何安裝 Centos 7 並設定網路"
date: 2017-05-05T01:00:00+08:00
lastmod: 2018-09-18T16:01:20+08:00
draft: false
tags: ["Linux","Tools","VirtualBox"]
slug: "virtualbox-centos-7"
aliases:
    - /2017/05/virtualbox-centos-7.html
---
# VirtualBox 如何安裝 Centos 7 並設定網路
公司電腦是 Windows 7，無法使用 Hyper-V 所以使用 VirtualBox 來建立 linux 環境，經過上次 ~~嚴密~~ 比較後依然還是選擇 CentOS 7，筆記一下安裝及設定過程

## 安裝 CentOS 7

1.  VirtualBox 管理員 --> 新增

    ![1create](https://cloud.githubusercontent.com/assets/3851540/25650245/27f31314-300e-11e7-926d-5736066a9406.png)

2.  名稱和作業系統

    ![2name](https://cloud.githubusercontent.com/assets/3851540/25650249/27fa6ed4-300e-11e7-85be-ce14ea62c5cc.png)

    *   名稱
    *   類型
    *   版本
        
        > VirtualBox 沒有提供 CentOS 的選項，就選 Red Hat (64-bit)

3.  記憶體大小

    > 可以保留預設大小或是依實際資源調整

    ![3memory](https://cloud.githubusercontent.com/assets/3851540/25650246/27f9530a-300e-11e7-8362-93061496e018.png)

4.  硬碟

    > 建立或是使用現有虛擬硬碟檔案

    ![4hdtype](https://cloud.githubusercontent.com/assets/3851540/25650248/27fa7122-300e-11e7-9523-a1b24aa60565.png)

    * 硬碟類型

        > 如果不需與其他虛擬化工具共用，可以使用 VDI 就好

        ![5hdtype](https://cloud.githubusercontent.com/assets/3851540/25650247/27fa500c-300e-11e7-9c26-8edd6d1c5970.png)

        *   VDI (VirtualBox 磁碟映像)
        *   VHD (虛擬硬碟)
        *   VMDK (虛擬機器磁碟)

    *   實體硬碟中存放裝置

        ![6hdsizetype](https://cloud.githubusercontent.com/assets/3851540/25650250/27fbbc76-300e-11e7-87df-2f4aac879cde.png)

        *   動態配置

            > 會依實際使用重新配置，上限是`固定大小`的最大值，我用的這版是 2TB

        *   固定大小

            > 一開始就決定硬碟空間，使用上比較快

    *  檔案位置和大小

        > 硬碟檔案位與硬碟大小

        ![7hdsize](https://cloud.githubusercontent.com/assets/3851540/25650251/2814c112-300e-11e7-9fd3-7821e07c335e.png)

5. 調整 cpu

    > 建議使用 CPU 數量放寬至 4 ，不然安裝會很慢

    *   開啟設定


        *   選取 VM --> 設定值

            ![8setting](https://cloud.githubusercontent.com/assets/3851540/25650256/282291b6-300e-11e7-8275-ea914bfc2742.png)

    *   系統 --> 處理器 --> 4

        ![9cpu](https://cloud.githubusercontent.com/assets/3851540/25650252/281cc740-300e-11e7-8345-469f04f13835.png)

6. 指定安裝 iso

    *   存放裝置 --> 控制器 --> 光碟圖示 --> 選擇適合 iso

        ![10insertiso](https://cloud.githubusercontent.com/assets/3851540/25650253/281d8b3a-300e-11e7-84be-a1e673434a9b.png)

7.  安裝 CentOS

    *   啟動 VM

        ![11start](https://cloud.githubusercontent.com/assets/3851540/25650254/281e2392-300e-11e7-89db-9c75be0f8ba7.png)

        > 需要留意一下開機順序，光碟機需優先於硬碟

        ![12bootpriority](https://cloud.githubusercontent.com/assets/3851540/25650255/281e853a-300e-11e7-9c9f-d7d8015ef4a7.png)

    * 開始安裝

        > Install CentOS Linux 7

        ![13installcentos](https://cloud.githubusercontent.com/assets/3851540/25650257/283832be-300e-11e7-842c-9bffac1ad030.png)

    * 環境檢查

        ![14envcheck](https://cloud.githubusercontent.com/assets/3851540/25650258/284049c2-300e-11e7-86f1-bfb3dcd39424.png)

    * 安裝語言

        ![15lang](https://cloud.githubusercontent.com/assets/3851540/25650260/28413ce2-300e-11e7-82d6-04485981abff.png)

    * 安裝摘要

        ![16summary](https://cloud.githubusercontent.com/assets/3851540/25650259/2840a908-300e-11e7-946e-c35258e51f04.png)

        *   軟體選擇

            > 最小型安裝只會安裝基本作業環境，如果需要其他功能需要另外安裝，e.g. ifconfig

        *  安裝目的地

            > 選擇剛建立的硬碟，使用預設值由系統自行配置即可

        ![17autoconfig](https://cloud.githubusercontent.com/assets/3851540/25650261/2843c9a8-300e-11e7-931b-fb64e60e5fee.png)

    *  用戶設定

        ![18rootpass](https://cloud.githubusercontent.com/assets/3851540/25650262/284bb2e4-300e-11e7-9053-de73792ebf5e.png)

        *   ROOT 密碼

            > 一定需要設定 ROOT 密碼，密碼強度不足需按兩次完成

            ![19PASS](https://cloud.githubusercontent.com/assets/3851540/25650263/285b343a-300e-11e7-8dac-ac61e3103332.png)

        *   用戶建立

            > 密碼強度不足需按兩次完成

    * 重新開機

        ![20reboot](https://cloud.githubusercontent.com/assets/3851540/25650264/28635bc4-300e-11e7-8ae9-7b96960785c8.png)

## 設定網路連線

>需在 VM 關機下才能進行設定

1.  開啟設定


    *   選取 VM --> 設定值
        
        ![8setting](https://cloud.githubusercontent.com/assets/3851540/25650256/282291b6-300e-11e7-8275-ea914bfc2742.png)

2.  檢查網路介面卡 1(預設是開啟狀態，不需調整)


    *   網路 --> 介面卡 1 --> 啟用網路卡 --> 附加到 NAT

        ![21network1](https://cloud.githubusercontent.com/assets/3851540/25650239/27d07ebc-300e-11e7-8208-3d402eb07bde.png)

3.  開啟網路介面卡 2


    *   網路 --> 介面卡 2 --> 啟用網路卡 --> 附加到 "僅限主機"介面卡 --> 混合模式：允許 VM

        ![22network2](https://cloud.githubusercontent.com/assets/3851540/25650241/27d3e3e0-300e-11e7-8bd0-7df5cea0eb5b.png)

4.  使用 nmtui (NetManager-TextUI) 設定

    > 使用方向鍵在各個選項間移動

    *   Edit a connection
        
        ![33editconn](https://cloud.githubusercontent.com/assets/3851540/25650240/27d317a8-300e-11e7-9c40-cb4fb46586b3.png)

    *   確保每個網路卡都有開 `Automatically connect`

        ![34editall](https://cloud.githubusercontent.com/assets/3851540/25650242/27d422ec-300e-11e7-9888-9fcfa8846680.png)

        ![35noauto](https://cloud.githubusercontent.com/assets/3851540/25650244/27d64fea-300e-11e7-9e03-be1f4595aa71.png)

        ![36auto](https://cloud.githubusercontent.com/assets/3851540/25650243/27d56120-300e-11e7-94aa-df773da94e2a.png)

# 參考資訊

1.  [CentOS 7 最小安裝後在VirtualBox的網路設定筆記](http://jimc1682000.blogspot.tw/2015/09/centos-7-virtualbox.html)
