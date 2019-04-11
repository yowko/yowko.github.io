---
title: "在 macOS 上 Build Redis Desktop Manager(RDM)"
date: 2019-01-27T21:30:00+08:00
lastmod: 2019-01-27T21:30:31+08:00
draft: false
tags: ["Redis","macOS"]
slug: "build-redis-desktop-manager-on-mac"
---
# 在 macOS 上 Build Redis Desktop Manager(RDM)
Redis 是套 in-memory 的 key-value databse，也被現在許多稍具規模系統拿來當做 cache 層以減輕 application cluster 直接大量存取 database 壓力，相信許多開發人員都認識它

Redis 在安裝時就內建了 redis-cli 可以用來執行 redis 相關指令，只是一旦 server 多了起來光連線管理就令人頭痛，而 Redis Desktop Manager(RDM) 就是套簡單易用的 GUI Redis 管理工具，加上支援跨平台，使用體驗在各個平台上相當一致，更是方便

只是 [Redis Desktop Manager 官網](https://redisdesktop.com/download) 上已不提供直接下載連結，而是改採付費訂閱的模式，所幸 Redis Desktop Manager 是 open source 起家還可以自行透過 build source code 來取得，而我在 mac 上 build Redis Desktop Manager 時可以說是阻礙重重，於是紀錄一下避免日後又 build 不起來XD 

## 基本環境說明
1. macOS Mojave 10.14.2
2. XCode 10.1
3. Qt 5.12.0
4. Qt Creator 4.8.1

## 安裝基本工具


1. 安裝 XCode

    >透過 APP Store 搜尋並安裝 XCode

    ![1searchappstore](https://user-images.githubusercontent.com/3851540/51802959-56e21880-228a-11e9-8d03-fbdae93c6b7b.png)

2. 安裝 homebrew

    ```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

    ![2installhomebrew](https://user-images.githubusercontent.com/3851540/51802960-56e21880-228a-11e9-9adc-dc59040fb83c.png)

3. 透過 homebrew 安裝 git

    ```bash
    brew install git
    ```

4. 透過 homebrew 安裝 qt

    ```
    brew install qt
    ```

5. 透過 homebrew 安裝 qt-creator

    ```bash
    brew cask install qt-creator
    ```

## 編譯 Redis Desktop Manager

1. 下載 source code 

    > 下載 Redis Desktop Manager 及相關 submodule 至 `rdm` 資料夾並切換工作目錄至 `rdm`

    ```bash
    git clone --recursive https://github.com/uglide/RedisDesktopManager.git -b 0.9 rdm && cd ./rdm
    ````

2. 透過 Info.plist.sample 建立 `Info.plist`

    ```bash
    cd ./src && cp ./resources/Info.plist.sample ./resources/Info.plist
    ```

5. build
    
    
    ```
    ./configure
    ```
   
   - 錯誤 一：找不到 Xcode

        - 錯誤訊息
      
            ```
            xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
            ```
        - 錯誤截圖
            
            ![3configerror1](https://user-images.githubusercontent.com/3851540/51802961-56e21880-228a-11e9-9bb5-84078ad91686.png)

        - 解決方式
            
            > Xcode 安裝位置與 rdm 使用的位置有差異，將路徑修改至 `/Applications/Xcode.app/Contents/Developer` (實際 Xcode 安裝位置)


            ```bash
            sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
            ```


   - 錯誤 二：不再支援 macOS 10.6 之前版本
    
        - 錯誤訊息
            ```
            error 3rdparty/gbreakpad/src/client/mac/sender/Breakpad.xib:global: error: Compiling for earlier than macOS 10.6 is no longer supported.
            ```
        - 錯誤截圖

            ![4configerror2](https://user-images.githubusercontent.com/3851540/51802962-577aaf00-228a-11e9-88c4-a4c8750ddd30.png)

        - 解決方式
            - 方法一：將 Xcode 降版至 10，不要用最新版
            - 方法二：不降版，使用 Xcode 開啟並修改 `rdm/3rdparty/gbreakpad/src/client/mac/sender/Breakpad.xib`

                ![5configeditxib](https://user-images.githubusercontent.com/3851540/51802963-577aaf00-228a-11e9-9a51-acd28c300f5c.png)

    - 錯誤 三：Error: openssl not installed

        - 錯誤訊息
            
            ```
            Error: openssl 1.0.2q already installed
            Warning: openssl 1.0.2q is already installed and up-to-date
            To reinstall 1.0.2q, run `brew reinstall openssl`
            ``` 
        - 錯誤截圖
            
            ![6configerror3](https://user-images.githubusercontent.com/3851540/51802964-577aaf00-228a-11e9-8cf8-9c4976adb5e3.png)
        - 解決方式
            
            >沒有解決但仍能順利 build rdm

6. 透過 Qt Creator 開啟 rdm.pro

    > 如果遇到以下問題可以參考說明的解決方式

    - No valid kits found.
        
        > 解決方式可以參考 [在 macOS 上的 Qt Creator 中出現 No valid kits found](https://blog.yowko.com/no-valid-kits-found-on-mac) 

    - failed to parse default search paths from compiler output

        > 解決方式可以參考 [在 macOS 上的 Qt Creator 中出現 failed to parse default search paths from compiler output](https://blog.yowko.com/failed-to-parse-default-search-paths-from-compiler-output)

7. 修改 rdm.pro

    > 未修改只能產生 exec 檔，修改完產生的 app 檔才能拉進 application 中正常執行

    - 將下列兩者註解 
        - `debug: CONFIG-=app_bundle`
        -  crashreporter

            ```
            release {
                CRASHREPORTER_APP.files = $$DESTDIR/crashreporter
                CRASHREPORTER_APP.path = Contents/MacOS
                QMAKE_BUNDLE_DATA += CRASHREPORTER_APP
            }
           ```   
        
        > 將會產生 `rdm.app` 在 ⁨`/rdm⁩/bin⁩/osx⁩/debug/⁩` 中
    
    - 將所有 debug 內容與 crashreporter 內容都註解掉
        
        >將會產生 `rdm.app` 在 ⁨`/rdm⁩/bin⁩/osx⁩/release/⁩` 中

    ![7editedm](https://user-images.githubusercontent.com/3851540/51802965-58134580-228a-11e9-907e-f0b1e5f094bb.png)

## 心得
原本只是抱著工程師順手 build 個工具來自用是理所當然的浪漫，但想不到比預期多花了許多時間，當然其中有一部份是不熟悉 macos 的行為，另一大部份則是不了解 Qt 開發特性的關係，只是經過這次改天如果還需要自己 build 什麼工具我還真的會認真考慮一下XD


# 參考資訊
1. [Redis Desktop Manager - build from source](http://docs.redisdesktop.com/en/latest/install/#build-from-source)
2. [從源碼編譯macOS版本的RedisDesktopManager](https://dalao.page/2018/11/12/build-rdm-for-mac-from-source/)
3. [./confgiure failed on mac os 10.14 with "earlier than macOS 10.6 is no longer supported."](https://github.com/uglide/RedisDesktopManager/issues/4284#issuecomment-437241877)
4. [在 macOS 上的 Qt Creator 中出現 No valid kits found](https://blog.yowko.com/no-valid-kits-found-on-mac) 
5. [在 macOS 上的 Qt Creator 中出現 failed to parse default search paths from compiler output](https://blog.yowko.com/failed-to-parse-default-search-paths-from-compiler-output)