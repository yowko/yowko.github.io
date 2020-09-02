---
title: "如何得知電腦上 .Net FrameWork 版本？ - 使用 PowerShell"
date: 2017-09-22T00:01:00+08:00
lastmod: 2020-09-01T22:08:39+08:00
draft: false
tags: ["PowerShell"]
slug: "get-dot-net-framework-version"
aliases:
    - /2017/09/get-dot-net-framework-version.html
---
# 如何得知電腦上 .Net FrameWork 版本？ - 使用 PowerShell
今天再次遇到想要確認 server 上 .Net FrameWork 版本的情境，套用著相同的 SOP： google 關鍵字，找到 microsoft docs 的說明網頁，再花了幾十秒看完 microsoft docs 上的說明，接著再用幾十秒開啟 regedit 來找到正確的值，然後再拿著值回去查表，最後才終於得到 server 上所使用的 .Net Framework 版本

每次需要確認 .Net Framework 版本時，都得重複上述的動作，每次都會覺得好沒效率，但自動化又省不了多少時間，整體效益不高，所以一直沒做什麼，剛好今天 production server 有問題，火燒屁股時那幾十秒的時間真是難熬，所以下定決心來解決這個狀況

市面上有不少小工具跟軟體可以達到一樣的目的，但隨便下載軟體不免有資安疑慮，唯有自己動手能不用提心吊膽

## .Net Framework 版本

[如何：判斷安裝的 .NET Framework 版本](https://docs.microsoft.com/zh-tw/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed?WT.mc_id=DOP-MVP-5002594)，微軟網站上有完整的教學，教大家如何看 .Net Framework 版本 節錄如下

|Release DWORD 的值|版本|
|--- |--- |
|`378389`|.NET Framework 4.5|
|`378675`|隨 Windows 8.1 或 Windows Server 2012 R2 安裝的 .NET Framework 4.5.1|
|`378758`|Windows 8、Windows 7 SP1 或 Windows Vista SP2 上安裝的 .NET Framework 4.5.1|
|`379893`|.NET Framework 4.5.2|
|Windows 10 系統：`393295`  <br/>所有其他作業系統版本：`393297`|.NET Framework 4.6|
|Windows 10 11 月更新系統：`394254`  <br/>所有其他作業系統版本：`394271`|.NET Framework 4.6.1|
|Windows 10 年度更新版：`394802`  <br/>所有其他作業系統版本：`394806`|.NET Framework 4.6.2|
|Windows 10 Creators Update：`460798`|.NET Framework 4.7|



## 使用 PowerShell 來快速取得 .Net Framework 版本

透過取得 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full` 的 `Release` 值來判斷 .Net Framework 版本

1.  將上述的 regedit 值轉為 dictionary (準備查表用)

    ```ps1
    $version = @{378389 = ".NET Framework 4.5"; 
        378675 = ".NET Framework 4.5.1"; 
        378758 = ".NET Framework 4.5.1";
        379893 = ".NET Framework 4.5.2";
        393295 = ".NET Framework 4.6";
        393297 = ".NET Framework 4.6";
        394254 = ".NET Framework 4.6.1";
        394271 = ".NET Framework 4.6.1";
        394802 = ".NET Framework 4.6.2";
        394806 = ".NET Framework 4.6.2";
        460798 = ".NET Framework 4.7";
    }
    ```

2.  取得 regedit 值

    ```ps1
    $localvalue=(Get-ItemProperty  "hklm:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\FUll").release
    ```

    > `Get-ItemProperty` 的別名用法為 `gp`，所以也可以用下列的寫法 `(gp "hklm:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\FUll").release`

3.  查表

    ```pa1
    $version.Item($localvalue)
    ```

4.  完整程式碼

    ```ps1
    $version = @{378389 = ".NET Framework 4.5"; 
        378675 = ".NET Framework 4.5.1"; 
        378758 = ".NET Framework 4.5.1";
        379893 = ".NET Framework 4.5.2";
        393295 = ".NET Framework 4.6";
        393297 = ".NET Framework 4.6";
        394254 = ".NET Framework 4.6.1";
        394271 = ".NET Framework 4.6.1";
        394802 = ".NET Framework 4.6.2";
        394806 = ".NET Framework 4.6.2";
        460798 = ".NET Framework 4.7";
    }
    $localvalue=(Get-ItemProperty  "hklm:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\FUll").release
    $version.Item($localvalue)
    ```
5.  實際使用

    > 將上述程式碼存為 `ps1` 檔

    *   使用 cmd

        > 透過 `powershell.exe` 執行 `Powershell.exe -File GetDotNetFramework.ps1`

    *   使用 powershell

        > 透過 `.\` 執行 `.\GetDotNetFramework.ps1`

## 心得

小小的一段程式碼，也沒什麼邏輯及功能，卻可以省下每次額外花的那些時間，寫程式就是這樣迷人，讓人願意廢寢忘食呀

# 參考資訊

1.  [如何：判斷安裝的 .NET Framework 版本](https://docs.microsoft.com/zh-tw/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed?WT.mc_id=DOP-MVP-5002594)
2.  [How to run a PowerShell script?](https://stackoverflow.com/questions/2035193/how-to-run-a-powershell-script)
3.  [Windows PowerShell Tip of the Week](https://technet.microsoft.com/en-us/library/ee692803.aspx)
4.  [More succinct way of getting a registry value as a string than (Get-ItemProperty $key $valueName).*VALUENAME*?](https://stackoverflow.com/questions/16318211/more-succinct-way-of-getting-a-registry-value-as-a-string-than-get-itemproperty)
