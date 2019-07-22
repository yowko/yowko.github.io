---
title: "從 Mac 移除 .NET Core Runtime 與 SDK"
date: 2019-07-22T21:30:00+08:00
lastmod: 2019-07-22T21:30:31+08:00
draft: false
tags: ["dotnet core","Mac"]
slug: "remove-dotnet-from-mac"
---

## 從 Mac 移除 .NET Core Runtime 與 SDK

可以不時更新 .NET Core SDK 是幸福，也是痛苦;

幸福的是可以使用新的語言特性或是新功能，而痛苦的是不僅又有新的 feature 要學習適應還要想辦法移除舊的 SDK。

過去 .NET Framework 時代的做法是直接取代舊版本，到了 .NET Core 時代則允許同時存在不同版本的 .NET Core SDK，讓開發人員自行選擇使用版本，當然是立意良善，不過像是 .NET Core 1 SDK，我相信絕對不會再用到，如果沒有手動移除它就還是一直留在電腦中，雖然不造成影響，但總是心裡有些疙瘩：既佔空間又怕誤用

過去都是透過 [如何移除 .NET Core 執行階段和 SDK](https://docs.microsoft.com/zh-tw/dotnet/core/versions/remove-runtime-sdk-versions?tabs=macos) 的步驟來執行移除，但我老是覺得步驟有點冗餘，另外讓我更不能接受的事有時候會刪不乾淨 (猜測是部份檔案仍在使用中而無法刪除)

今天趁著再次更新 .NET Core SDK 之餘，順手筆記一下

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core 2.2.107 --> 2.2.301

## 移除 .NET Core SDK

> 以下語法是參考官網中的 shell [dotnet/cli/scripts/obtain/uninstall/dotnet-uninstall-pkgs.sh](https://github.com/dotnet/cli/blob/master/scripts/obtain/uninstall/dotnet-uninstall-pkgs.sh) 簡化而來的 (我不想要更新 .NET Core SDK 還必需要先找到移除用 shell 下載並執行 XD，不過如果不嫌麻煩，還是建議使用官方的 shell，畢竟小弟不才可能會把 shell 簡化出 bug 呀)

1. 移除 .NET Core 相關 pkgs

    ```bash
    pkgutil --pkgs | grep com.microsoft.dotnet  | xargs -n 1 sudo pkgutil --forget
    ```

    ![1forget](https://user-images.githubusercontent.com/3851540/61641657-1069f580-acd2-11e9-8f27-c4dca7498446.png)

2. 移除 dotnet 相關檔案及工具

    ```bash
    sudo rm -rf /usr/local/share/dotnet && sudo rm -f /etc/paths.d/dotnet && sudo rm -f /etc/paths.d/dotnet-cli-tools
    ```

## 重新安裝 .NET Core SDK

> 以下兩個方法 (或是其他方式皆可) 擇一執行即可，主要目的就是安裝新版的 .NET Core SDK

1. [官網](https://dotnet.microsoft.com/download/dotnet-core) 下載 .NET Core SDK installer
2. 使用 homebrew

    ```bash
    brew cask install dotnet-sdk
    ```

## 心得

沒錯，我其實沒有真的更新 .NET Core SDK，我就是直接刪除所有 .NET Core SDK 再重新安裝最新版本的 .NET Core SDK，也許不一定比較省時間，但個人還是覺得清爽不少，花這個時間划算  哈哈

## 參考資訊

1. [如何移除 .NET Core 執行階段和 SDK](https://docs.microsoft.com/zh-tw/dotnet/core/versions/remove-runtime-sdk-versions?tabs=macos)
2. [Steps to uninstall a DotNet CLI version on Mac OS X](https://gist.github.com/sandrovicente/5590f5ac993d9d7fef038fd2858efcc3)
3. [dotnet/cli/scripts/obtain/uninstall/dotnet-uninstall-pkgs.sh](https://github.com/dotnet/cli/blob/master/scripts/obtain/uninstall/dotnet-uninstall-pkgs.sh)
