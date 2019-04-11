---
title: "Windows 10 滑鼠滾動方向翻轉"
date: 2019-02-03T23:45:00+08:00
lastmod: 2019-02-03T23:44:30+08:00
draft: false
tags: ["Windows 10"]
slug: "windows10-reverse-scroll"
---
# Windows 10 滑鼠滾動方向翻轉

自從用慣了行動裝置後總覺得在電腦上的操作有些卡，但行動裝置的使用時間並不長問題不大，直到後來工作用電腦換為 mac 後突然覺得個人用的 Windows 環境怎麼跟其他人都不一樣：行動裝置與 mac 是站在頁面的立場來思考：往上滑動是讓頁面往上捲動、頁面下方內容往上;  Windows 平台則是站在捲軸的立場來思考：往上滑動就是將捲軸往頁面上方。

實際使用沒有好壞差別，只是習慣問題，不過 Windows 的預設行為跟行動裝置、mac 相反，使用上需要點磨合的時間，在時間急迫下又被操作系統的習慣卡住  不免心裡一堆 ooxx....  於是乾脆就把 Windows 的滑動行為改成跟 mac 一致免得常要適應


## 基本環境
1. Windows 10 Version 1803 (OS Build 17134.523)

## 取得 Device instance path
1. 開啟 Device Manager

    ![2devicemanager](https://user-images.githubusercontent.com/3851540/52179357-88bc2780-2814-11e9-985b-45a6f4592d56.png)

2. 在目標裝置上按右鍵 --> Properties

    ![3properties](https://user-images.githubusercontent.com/3851540/52179358-88bc2780-2814-11e9-8ae4-42d5b514e6b6.png)

3. Property --> Device instance path --> Copy

    ![4devicepath](https://user-images.githubusercontent.com/3851540/52179359-88bc2780-2814-11e9-80c6-bca33901bc1a.png)


## 設定方式
1. 方式一：直接修改 registry
    - 範例 registry 位置
    
        ```
        Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\{device instance path}\Device Parameters
        ```
    - 實際 registry 位置

        ```
        Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\HID\VID_05AC&PID_0262&MI_01&Col01\7&35bd79e6&0&0000\Device Parameters
        ```
    
    ![1registry](https://user-images.githubusercontent.com/3851540/52179356-88239100-2814-11e9-8b17-dee591c98eeb.png)

2. 方式二：透過 PowerShell

    > 需要使用 administrator 開啟 powershell

    ```ps1
    # 提示輸入 device instance path (input device instance path)
    $deviceInstancePath=Read-Host -Prompt 'Input your device instnace path'
    # 指定 registry 位置 (set registry path)
    $RegKey="HKLM:\SYSTEM\CurrentControlSet\Enum\"+$deviceInstancePath+"\Device Parameters"

    # 顯示目前設定值(show current setting)
    Get-ItemProperty -path $regKey -Name FlipFlopWheel
    # 設定捲軸反轉(set reverse scrll)
    Set-ItemProperty -path $regKey -Name FlipFlopWheel -Value 1
    # 確認設定結果 (confirm new setting)
    Get-ItemProperty -path $regKey -Name FlipFlopWheel
    ```
3. 重新啟動

    > 我設定完後重新登入並無法使新設定值生效，需重新開啟

## 心得
我怎麼印象以前要找這個設定的參考資料很容易呀，這次卻花了不少時間，果然還是應該早點做些紀錄  這樣才能可以有效節省自己時間呀

[GitHub](https://github.com/yowko/Reverse-Windows-Scroll) 可以下載 ps1 ，cmd 透過 `Powershell.exe -File windows-reverse-scroll.ps1` 即可執行

# 參考資料
1. [Reverse the scroll of mouse](https://answers.microsoft.com/en-us/windows/forum/windows_10-other_settings/reverse-the-scroll-of-mouse/334669c3-8a45-4600-830a-8df628d7415e)