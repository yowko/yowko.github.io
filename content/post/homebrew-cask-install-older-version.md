---
title: "使用 Homebrew Cask 安裝舊版本軟體"
date: 2019-12-13T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["macOS","Tools"]
slug: "homebrew-cask-install-older-version"
---

## 使用 Homebrew Cask 安裝舊版本軟體

前幾天筆記 [JetBrains Rider 在升級 .NET Core 3.1 後無法編譯紀錄](/rider-dotnetcore3-cannot-build/) 簡單紀錄到在 macOS 上升級 .NET Core 3.1 後，JetBrains Rider 無法 build 的狀況，經同事反應各式情境後，就需要反覆安裝 .NET Core 3.0 與 .NET Core 3.1 來驗證，之前筆記 [從 Mac 移除 .NET Core Runtime 與 SDK](/remove-dotnet-from-mac/) 提到可以透過 `brew cask install dotnet-sdk` 來安裝 .NET Core SDK，但預設都會直接安裝最新版本，今天就來紀錄該如何透過 `brew cask install` 安裝舊版軟體

不確定是不是正確做法，有錯請指教

## 基本環境說明

1. macOS Catalina 10.15.1

## 安裝方式

1. 先上 Homebrew/homebrew-cask 的 [GitHub](https://github.com/Homebrew/homebrew-cask/tree/master/Casks) 找到需要的 Casks

2. 找到需要的版本

    > 下圖以 dotnet-sdk 為例

    ![1history](https://user-images.githubusercontent.com/3851540/70808721-3f0c7800-1dfb-11ea-9481-1e0ccdfe9c69.png)

    ![2history2](https://user-images.githubusercontent.com/3851540/70808722-3f0c7800-1dfb-11ea-9f8c-4ee3eb071c65.png)

3. 取得需要版本的完整內容

    ![3viewfile](https://user-images.githubusercontent.com/3851540/70808723-3fa50e80-1dfb-11ea-9e00-4346b0323d94.png)

    ![4raw](https://user-images.githubusercontent.com/3851540/70808724-3fa50e80-1dfb-11ea-865f-94a32da9317b.png)

    ![5rawurl](https://user-images.githubusercontent.com/3851540/70808726-3fa50e80-1dfb-11ea-9819-70b8995eeafd.png)

4. 下載 cask

    - 語法

        ```bash
        curl -O {目標版本的 cask url}
        ```

    - 實際範例

        > 以 dotnet-sdk 3.0.100 為例

        ```bash
        curl -O https://raw.githubusercontent.com/Homebrew/homebrew-cask/d1a67e482238a01caa446d903ff85671f3569612/Casks/dotnet-sdk.rb
        ```

5. 使用下載的 cask 安裝

    ```bash
    brew cask install ./dotnet-sdk.rb
    ```

## 心得

透過 `brew cask install {指定版本的 rb}` 會出現 `http 416` 錯誤，所以才需要將安裝拆分為 1. 先下載 2. 透過下載的檔案來安裝，兩步驟執行，以下附上錯誤訊息

1. 錯誤訊息

    ```txt
    Updating Homebrew...
    ==> Auto-updated Homebrew!
    Updated 2 taps (homebrew/core and homebrew/cask).
    ==> Updated Formulae
    helmsman

    ==> Downloading https://raw.githubusercontent.com/Homebrew/homebrew-cask/d1a67e482238a01caa446d903ff85671f3569612/Casks/dotnet-sdk.rb.
    ######################################################################## 100.0%
    curl: (22) The requested URL returned error: 416
    Error: Cask 'dotnet-sdk' is unavailable: Failed to download https://raw.githubusercontent.com/Homebrew/homebrew-cask/d1a67e482238a01caa446d903ff85671f3569612/Casks/dotnet-sdk.rb. Did you mean one of these?
    dotnet-sdk ✔                                                                                           dotnet-sdk-preview
    ```

2. 錯誤截圖

    ![6error](https://user-images.githubusercontent.com/3851540/70808727-403da500-1dfb-11ea-88db-71e465e4116d.png)

回到 `brew cask` 本身，就安裝方式透過 rb 的做法，我個人不負責任猜測應該是不想額外儲存版本與 rb 之間的對應吧，反正永遠裝新版的，如果有特殊需求透過指定 rb 來安裝也能滿足，不過我在 [官方 GitHub 的說明文件](https://github.com/Homebrew/homebrew-cask/blob/master/USAGE.md) 中看到可以使用以下方式安裝

1. 簡單象徵性名稱 (eg: `dotnet-sdk`)
2. 可以透過 curl 取得 cask file 的 URI
3. cask (.rb) 檔案

理論上應該可以直接用 `brew cask install {uri}` 進行安裝才對，感覺是 bug

## 參考資訊

1. [Downgrade an application installed with Brew Cask](https://www.jverdeyen.be/mac/downgrade-brew-cask-application/)
2. [GitHub](https://github.com/Homebrew/homebrew-cask/tree/master/Casks)
