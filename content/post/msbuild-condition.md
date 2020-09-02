---
title: "依編譯參數來決定 config 內容"
date: 2017-08-14T23:23:00+08:00
lastmod: 2020-09-01T23:23:33+08:00
draft: false
tags: ["MSBuild","web.config"]
slug: "msbuild-condition"
aliases:
    - /2017/08/msbuild-condition.html
---
# 依編譯參數來決定 config 內容
一連兩篇文章 [專案間如何共用 config 設定 - 使用 MSBuildExtensionPack](https://blog.yowko.com/2017/08/shared-config.html) 與 [專案間如何共用 config 設定 - 使用 MSBuild Community Tasks](https://blog.yowko.com/2017/08/config-msbuild-community-tasks.html) 介紹使用不同的 library 來將共用 config 設定值獨立於專案之外，再透過 msbuild 編譯動作調整內容，讓 config 設定能跨專案共用

兩篇文章都忽略專案 configuration 編譯，這當然不是不小心的，主要是我在介紹 library 及相關功能時，覺得加入 configuration 編譯會讓介紹內容變得更亂也更難說明得清楚，所以刻意將這個部份獨立說明；另外條件式參數設定的功能是 msbuild 內建的，使用哪個 library 都可以，也不適合跟特定 library 介紹放在一起

為了讓共用 config 更加實用，configuraion 編譯功能是絕對不可或缺的，就來看看該如何設定吧

## 基本環境設定

主要內容是延續前兩篇文章中的 `參數設定檔` 部份，詳細內容可以參考 [專案間如何共用 config 設定 - 使用 MSBuildExtensionPack](https://blog.yowko.com/2017/08/shared-config.html) 與 [專案間如何共用 config 設定 - 使用 MSBuild Community Tasks](https://blog.yowko.com/2017/08/config-msbuild-community-tasks.html)

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <PropertyGroup>
        <YowkoTest>ValueChanged</YowkoTest>
    </PropertyGroup> 
</Project>
```

## 實際使用條件式參數設定

*   測試指令
    *   pattern

        > `MSBuild.exe {.msbuild file} /p:{參數名稱}={參數值}`

    *   範例

        > `"C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\amd64\MSBuild.exe" msbuildextension.msbuild /p:Configuration=DEBUG`

1.  基本條件式參數設定

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup Condition="'$(Configuration)' == 'DEBUG'">
            <YowkoTest>ValueChanged - $(Configuration)</YowkoTest>
        </PropertyGroup> 
        <PropertyGroup Condition="'$(Configuration)' == ''">
            <YowkoTest>ValueChanged-NoConfig</YowkoTest>
        </PropertyGroup>
    </Project>
    ```

    *   編譯參數 `Configuration` 等於 DEBUG 就輸出 ValueChanged - $(Configuration) 即 `ValueChanged - DEBUG`；
    *   未指定 `Configuration` 參數或是 `Configuration` 是空字串時就輸出 `ValueChanged-NoConfig`

2.  多個條件參數設定

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup Condition="'$(Configuration)' == 'DEBUG' Or '$(Configuration)' == 'RELEASE'">
            <YowkoTest>ValueChanged - $(Configuration)</YowkoTest>
        </PropertyGroup> 
        <PropertyGroup Condition="'$(Configuration)' == ''">
            <YowkoTest>ValueChanged-NoConfig</YowkoTest>
        </PropertyGroup>
    </Project>
    ```

    *   效果與上面範例相同，只是多了 `Configuration` 參數等於 `RELEASE` 時行為與 `DEBUG` 相同

3.  if-else 效果的參數設定

    > 上面的兩個範例，都只能處理特定的參數值，雖然可以使用 `!=` 但條件多時列舉還是很麻煩，此時就可以採用下列方式來達成 if-else 的效果

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <Choose>  
            <When Condition=" '$(Configuration)'=='DEBUG' ">  
                <PropertyGroup>  
                    <YowkoTest>ValueChanged - $(Configuration)</YowkoTest>
                </PropertyGroup>  
            </When>  
            <Otherwise>  
                <PropertyGroup>  
                    <YowkoTest>ValueChanged-NoConfig</YowkoTest>
                </PropertyGroup>  
            </Otherwise>  
        </Choose>  
    </Project>
    ```

4.  指定預設值

    > 如果未指定參數值或是參數等於空字串，可以設定參數預設值

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
        <PropertyGroup>
            <Configuration Condition=" '$(Configuration)' == '' ">DEBUG</Configuration>
        </PropertyGroup> 
        <Choose>  
            <When Condition=" '$(Configuration)'=='DEBUG' ">  
                <PropertyGroup>  
                    <YowkoTest>ValueChanged - $(Configuration)</YowkoTest>
                </PropertyGroup>  
            </When>  
            <Otherwise>  
                <PropertyGroup>  
                    <YowkoTest>ValueChanged-NoConfig</YowkoTest>
                </PropertyGroup>  
            </Otherwise>  
        </Choose>  
    </Project>
    ```

## 心得

透過上面介紹的方法，就可以針對不同的 project configuration 設定不同參數值，讓多環境、多專案的 config 設定管理更加便利，也可以讓工程師能更專注於功能開發，不需要分心管理環境設定參數

經過一番研究後，發現 msbuild 原生就提供不少功能，但也因為 Visual Studio 太強大，都沒有機會好好去認識 msbuild 也就沒有真正充份發揮 msbuild，剛好趁這次機會有稍稍認識一下，下次遇到疑難雜症也許就可以派上用場了

# 參考資訊

1.  [MSBuild Conditions](https://msdn.microsoft.com/en-us/library/7szfhaft.aspx)
2.  [MSBuild Conditional Constructs](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-conditional-constructs?WT.mc_id=DOP-MVP-5002594)
3.  [專案間如何共用 config 設定 - 使用 MSBuildExtensionPack](https://blog.yowko.com/2017/08/shared-config.html)
4.  [專案間如何共用 config 設定 - 使用 MSBuild Community Tasks](https://blog.yowko.com/2017/08/config-msbuild-community-tasks.html)
