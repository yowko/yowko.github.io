---
title: "如何讓 ASP.NET Core 3.1 以 amd64 image 在 arm 晶片 (M1) 上執行"
date: 2022-04-03T00:30:00+08:00
lastmod: 2022-04-03T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","Linux","arm","Docker"]
slug: "aspdotnet-core-31-amd64-arm"
---

## 如何讓 ASP.NET Core 3.1 以 amd64 image 在 arm 晶片 (M1) 上執行

公司電腦準備做周期性汰換，所以開始評估搭載 arm cpu (M1) 的 macbook pro，經過一輪測試後，絕大部份工具都能正常使用，而團隊目前大量使用的開發媒介 - docker 基本也沒問題，不過團隊原有的 application image (linux/amd64) 在 arm 上執行時卻無法正常運作，錯誤資訊如下

- 錯誤訊息

    ```log
    Unhandled exception. System.IO.IOException: Function not implemented
       at System.IO.FileSystemWatcher.StartRaisingEvents()
       at System.IO.FileSystemWatcher.StartRaisingEventsIfNotDisposed()
       at System.IO.FileSystemWatcher.set_EnableRaisingEvents(Boolean value)
       at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.TryEnableFileSystemWatcher()
       at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.CreateFileChangeToken(String filter)
       at Microsoft.Extensions.FileProviders.PhysicalFileProvider.Watch(String filter)
       at Microsoft.Extensions.Configuration.FileConfigurationProvider.<.ctor>b__1_0()
       at Microsoft.Extensions.Primitives.ChangeToken.ChangeTokenRegistration`1..ctor(Func`1 changeTokenProducer, Action`1 changeTokenConsumer, TState state)
       at Microsoft.Extensions.Primitives.ChangeToken.OnChange(Func`1 changeTokenProducer, Action changeTokenConsumer)
       at Microsoft.Extensions.Configuration.FileConfigurationProvider..ctor(FileConfigurationSource source)
       at Microsoft.Extensions.Configuration.Json.JsonConfigurationSource.Build(IConfigurationBuilder builder)
       at Microsoft.Extensions.Configuration.ConfigurationBuilder.Build()
       at Microsoft.Extensions.Hosting.HostBuilder.BuildAppConfiguration()
       at Microsoft.Extensions.Hosting.HostBuilder.Build()
       at grpc31.Program.Main(String[] args) in /source/grpc31/Program.cs:line 18
    qemu: uncaught target signal 6 (Aborted) - core dumped
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/160317383-61ed949d-8501-490e-8b50-f4e1f6a5a869.png)

想要使用 `linux/amd64` 是希望本機開發與 prod 使用的 image 架構是一樣的，避免因為不同的平台出現不同的行為而沒辦法在開發階段就預先發現問題

## 基本環境說明

1. macOS Monterey 12.2.1 (M1)
2. docker desktop 4.5.0(74594)
3. .NET SDK 6.0.200
4. 指定 netcoreapp3.1 framework 並使用 gRPC default project template 建立專案
5. NuGet packages

    - Microsoft.Extensions.Hosting 6.0.1

6. dockerfile

    > 參考自 [dotnet/dotnet-docker](https://github.com/dotnet/dotnet-docker/tree/main/samples/aspnetapp)

    ```dockerfile
    # https://hub.docker.com/_/microsoft-dotnet
    FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
    WORKDIR /source
    
    # copy csproj and restore as distinct layers
    COPY *.sln .
    COPY grpc31/*.csproj ./grpc31/
    #RUN dotnet restore -r linux-x64
    
    # copy everything else and build app
    COPY grpc31/. ./grpc31/
    WORKDIR /source/grpc31
    #RUN dotnet publish -c release -o /app -r linux-x64 --self-contained false --no-restore
    RUN dotnet publish -c release -o /app -r linux-x64 --self-contained true
    
    # final stage/image
    FROM mcr.microsoft.com/dotnet/aspnet:6.0-bullseye-slim-amd64
    WORKDIR /app
    COPY --from=build /app ./
    ENTRYPOINT ["./grpc31"]
    ```

7. 指定 amd64 架構來啟動 container

    ```bash
    docker run -it --platform linux/amd64 yowko:grpc31
    ```

## 問題分析

- 從錯誤訊息最後一行 `qemu: uncaught target signal 6 (Aborted) - core dumped` 可以看得出來是 qemu 所拋出的錯誤

    ![3fromqemu](https://user-images.githubusercontent.com/3851540/160317397-a23ba701-62b6-41ba-bb59-15126c0049bf.png)

- [Docker Desktop for Apple silicon Known issues](https://docs.docker.com/desktop/mac/apple-silicon/#known-issues) 有提到 `filesystem change notification APIs (inotify)` 沒辦法在 qemu 模擬器下正常運作

    ![2inotifynotwork](https://user-images.githubusercontent.com/3851540/160317394-ddf206b7-73d9-4bc6-8d9c-bdacaddcd053.png)

- [ASP.NET Docker Gotchas and Workarounds](https://khalidabuhakmeh.com/aspnet-docker-gotchas-and-workarounds#configuration-reloads-and-filesystemwatcher) 提到兩個重點

    1. 許多 config provider 都支援 hot-reload
    2. .NET Core hot-reload 使用的 `FileSystemWatcher` 在 linux 上的實作是 `inotify`

    ![4confighotreload](https://user-images.githubusercontent.com/3851540/160317400-379ec4af-9359-44b5-8023-914629c2bbdd.png)

- 結論：

    1. linux/amd64 架構的 ASP.NET Core container 在 arm cpu 主機上是透過 qemu 模擬器執行
    2. ASP.NET Core 的 config hot reload 機制是使用 `inotify` 實作
    3. qemu 不支援 `inotify`

## 解決方式

1. 將 project target framework 升至 .NET 5 以上

    [Microsoft Docs:Disable app configuration reload on change](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#disable-app-configuration-reload-on-change) 提到在 ASP.NET Core 5.0 調整了讀取 appsettings.json 的做法：允許使用 `執行參數` 或是 `環境變數` 來關閉 config hot-reload 的功能 (預設 `啟用 hot reload`)

    ![5disablehotreload](https://user-images.githubusercontent.com/3851540/160317402-aac5aeed-53fb-4a68-af53-00cdc725de9b.png)

2. 無法升級

    [GitHub:HostingHostBuilderExtensions](https://github.com/dotnet/runtime/blob/main/src/libraries/Microsoft.Extensions.Hosting/src/HostingHostBuilderExtensions.cs) 可以看到修改的方式：讀取 `hostBuilder:reloadConfigOnChange` 做為是否 hot reload 的設定值，既然無法升級 project 的 target framework 加上這個 change 存在 `Microsoft.Extensions.Hosting` 這個 package 中，那只升級這個 package 行不行呢？答案是 `可以`

    - 升級 `Microsoft.Extensions.Hosting` 至 `6.0.1`

        > 理論上應該升級至 `5` 以後都可以，但我沒測試

    - 傳遞正確環境變數可以正常啟動

        1. `hostBuilder:reloadConfigOnChange`

            ```bash
            docker run -it --platform --env DOTNET_hostBuilder__reloadConfigOnChange=false linux/amd64 yowko:grpc31_hosting
            ```

            ```bash
            docker run -it --platform --env ASPNETCORE_hostBuilder__reloadConfigOnChange=false linux/amd64 yowko:grpc31_hosting
            ```

        2. `DOTNET_USE_POLLING_FILE_WATCHER`

            ```bash
            docker run -it --platform --env DOTNET_USE_POLLING_FILE_WATCHER=1 linux/amd64 yowko:grpc31_hosting
            ```

## 心得

測試過一輪後，參數設定方式結論整理如下

設定方式\參數|`hostBuilder:reloadConfigOnChange`|`USE_POLLING_FILE_WATCHER`
:---|:---|:---
大小寫|沒有限制|只能全大寫
prefix|`DOTNET_` ,`ASPNETCORE_`,`dotnet_` 與 `aspnetcore_` 皆可|僅能使用 `DOTNET`
連線符號|`:` 與 `__` 皆可| 沒有用到 `:`

- `DOTNET_` ,`ASPNETCORE_` 是官方文件 [Microsoft Docs:Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#environment-variables) 上明確說明可以用於設定 environment variables 的 prefix，我看程式碼 [GitHub:HostingHostBuilderExtensions](https://github.com/dotnet/runtime/blob/main/src/libraries/Microsoft.Extensions.Hosting/src/HostingHostBuilderExtensions.cs) 沒提到可以使用 `小寫`，而我實際使用上 `dotnet_` 與 `aspnetcore_` 皆能正確運作，不過僅限於 `hostBuilder:reloadConfigOnChange`，`USE_POLLING_FILE_WATCHER` 就不允許小寫(包含 prefix 與 參數本身皆不允許使用小寫)
- 官方文件 [Microsoft Docs:Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#environment-variables) 也提到使用 `:` 設定 environment variables 會有問題，應該要使用 `__` 取代，但實際使用我沒遇到問題

雖然 ASP.NET Core 3.1 已經有些過時，團隊也逐漸將 project 升級至 ASP.NET Core 6，相同問題應該不會再遇到，但有陣子沒有解決 .NET 相關 issue，趁這個機會回味逐一釐清問題與脈絡並找到解決方式的整個過程還是相當過癮呀

## 參考資訊

1. [dotnet/dotnet-docker](https://github.com/dotnet/dotnet-docker/tree/main/samples/aspnetapp)
2. [ASP.NET Docker Gotchas and Workarounds](https://khalidabuhakmeh.com/aspnet-docker-gotchas-and-workarounds)
3. [Docker Desktop for Apple silicon](https://docs.docker.com/desktop/mac/apple-silicon/#known-issues)
4. [qemu exception throwing on x64 emulator for docker with .NET6/.NET Core on the Apple M1 chip](https://blackie1019.gitlab.io/2022/02/22/qemu-exception-throwing-on-x64-emulator-for-docker-with-NET6-NET-Core-on-the-Apple-M1-chip/)
5. [Microsoft Docs:Disable app configuration reload on change](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#disable-app-configuration-reload-on-change)
6. [Microsoft Docs:Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594#environment-variables)
