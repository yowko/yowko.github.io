---
title: "如何在 Windows Server 2016 上安裝 Hyper-V"
date: 2017-05-02T01:00:00+08:00
lastmod: 2018-09-18T01:00:08+08:00
draft: false
tags: ["Windows Server 2016"]
slug: "windows-server-2016-hyper-v"
aliases:
    - /2017/05/windows-server-2016-hyper-v.html
---
# 如何在 Windows Server 2016 上安裝 Hyper-V
Windows Server 2016 經過與 dcoker 公司共同重新設計架構，已經支援使用原生的 Windows container ，但市面上多數的軟體仍然以 Linux container 為大宗，現階段要在 Windows Server 2016 上使用 linux container 官方還沒有正式建議的做法，但還是可以透過 Hyper-V 自行安裝 linux 來測試，所以要先準備 Hyper-V 環境

## 確認 Hyper-V 硬體需求

1.  使用第二層位址轉譯 (SLAT) 的 64 位元處理器。
2.  CPU 對 VM 監視模式延伸模組的支援 (Intel CPU 上的 VT-c)。
3.  至少 4 GB 記憶體
4.  BIOS 需啟用
    *   虛擬化技術
    *   防止硬體強制的資料執行


*   可以透過在 PowerShell 或是 command prompt 中執行 `Systeminfo.exe` 來確認是否符合

    ![1sysinfo](https://cloud.githubusercontent.com/assets/3851540/25576207/cf895a96-2e8f-11e7-8565-bf53384542dc.png)

## 安裝 Hyper-V

*   使用伺服器管理員
    1.  伺服器管理員 --> 管理 --> 新增角色及功能

        ![2addrole](https://cloud.githubusercontent.com/assets/3851540/25576209/cf8a38a8-2e8f-11e7-975a-d7d7f5cd96db.png)

    2.  在您開始前

        ![3beforestart](https://cloud.githubusercontent.com/assets/3851540/25576205/cf88fc7c-2e8f-11e7-99f7-77fc3ca438c4.png)

    3.  安裝類型 --> 角色型或功能型安裝

        ![4installtype](https://cloud.githubusercontent.com/assets/3851540/25576208/cf89fb18-2e8f-11e7-964d-b39b65f85cae.png)

    4.  伺服器選取項目

        ![5serverselect](https://cloud.githubusercontent.com/assets/3851540/25576210/cfbd30aa-2e8f-11e7-9ed5-8ba181749c5f.png)

    5.  伺服器角色 --> Hyper-V

        ![6selecthyperv](https://cloud.githubusercontent.com/assets/3851540/25576211/cfc577e2-2e8f-11e7-869b-36d7ec5277ec.png)

    6.  Hyper-V

        ![7hyperv](https://cloud.githubusercontent.com/assets/3851540/25576198/cf5db1b6-2e8f-11e7-8e99-f65c4bdd9426.png)

        *   虛擬交換器

            ![8virtualswitch](https://cloud.githubusercontent.com/assets/3851540/25576200/cf5ef648-2e8f-11e7-9031-9b6214c4578f.png)

        *   移轉

            ![9transform](https://cloud.githubusercontent.com/assets/3851540/25576202/cf5fdff4-2e8f-11e7-9fbe-e716e0fa88b3.png)

        *   預設存放區

            ![10defaultstore](https://cloud.githubusercontent.com/assets/3851540/25576199/cf5e6ebc-2e8f-11e7-86c8-1bdf49d5f40f.png)

    7.  確認

        ![11confirm](https://cloud.githubusercontent.com/assets/3851540/25576203/cf647a78-2e8f-11e7-9129-86c303c38f99.png)

    8.  結果

        ![12install](https://cloud.githubusercontent.com/assets/3851540/25576201/cf5f61c8-2e8f-11e7-8047-7f70a2194023.png)

        *   需要重新開機

            ![13reboot](https://cloud.githubusercontent.com/assets/3851540/25576204/cf88991c-2e8f-11e7-93d7-66ac006a2e7c.png)

*   使用 PowerShell
    1.  以管理員身份開啟 `Windows PowerShell`

    2.  安裝 Hyper-V

        ```ps1
        Install-WindowsFeature -Name Hyper-V -ComputerName <computer_name> -IncludeManagementTools -Restart
        ```

        *   如果是安裝在本機，可以忽略 `-ComputerName <computer_name>`

    3.  確認安裝狀況

        > `Get-WindowsFeature -Name Hyper-V -ComputerName <computer_name>`

        *   如果是安裝在本機，可以忽略 `-ComputerName <computer_name>`

        ![14installed](https://cloud.githubusercontent.com/assets/3851540/25576206/cf89227e-2e8f-11e7-8cf0-16f9c8b9795d.png)

# 參考資訊
1.  [System requirements for Hyper-V on Windows Server 2016](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/System-requirements-for-Hyper-V-on-Windows)
2.  [Install the Hyper-V role on Windows Server 2016](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server)
