---
title: "如何指定 container 或是 .NET application 的時區"
date: 2023-10-26T00:30:00+08:00
lastmod: 2023-10-26T00:30:31+08:00
draft: false
tags: ["csharp","container","dotnet"]
slug: "timezone-in-container-or-dotnet"
---

## 如何指定 container 或是 .NET application 的時區

最近經手的一個專案，使用到外部 partner 提供的 NuGet library，但這個 library 有個問題，就是 `DateTime.UtcNow` 與 `DateTime.Now` 同時都有用到，而且沒有提供任何的設定方式，造成某些情境下的時間就會出錯而造成 bug，所以只好想辦法讓整個 container 或是 .NET application 的時區改成 UTC+0。

原本以為要改 os 設定會讓整個設定流程複雜很多，但出乎意外地簡單，加上同事也問了相同問題，所以快速紀錄一下設定方式

## 基本環境說明

1. macOS Ventura 13.3
2. OrbStack 1.0.1(1697)
3. .NET SDK 6.0.413
4. docker images

    - mcr.microsoft.com/dotnet/aspnet:8.0

## 設定方式與實際效果：透過指定 environment variable `TZ` 來設定時區

1. container

    > 這邊我們指定為 `Asia/Taipei` (`mcr.microsoft.com/dotnet/aspnet:8.0` 預設時區為 `UTC`)

    - 透過 `date +"%Z %z"` 指令來確認
    - `docker run -it --rm -e TZ=Asia/Taipei mcr.microsoft.com/dotnet/aspnet:8.0 date +"%Z %z"`

    ![1container](https://github.com/yowko/picsbed/assets/3851540/b1719384-4c87-4548-9f96-acae2bf0f95d)

2. .NET application

    > 這邊我們指定為 `Etc/GMT` (預設使用 os 時區，我的 macbook pro 設定為 `Asia/Taipei`)

    - `TimeZoneInfo.Local.DisplayName` 可以在 .NET application 中取得目前時區
    - `TZ=Etc/GMT dotnet run --project TimestampTest/TimestampTest.csproj`
    - 指定 environment variable `TZ` 也是相同效果

    ![2dotnetrun](https://github.com/yowko/picsbed/assets/3851540/b454aaea-0ec7-4f31-8c2f-0d53c105d4c7)

## 心得

TZ ID 可以參考 [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

如果 application 部署是使用 container，可以在 dockerfile 中指定時區，這樣 application 本來就會 os 的時區設定

## 參考資訊

1. [How do I change timezone in a docker container?](https://stackoverflow.com/questions/57607381/how-do-i-change-timezone-in-a-docker-container)
2. [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
