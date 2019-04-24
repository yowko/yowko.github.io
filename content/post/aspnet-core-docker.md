---
title: "將 ASP.NET Core 部署至 Docker 中"
date: 2019-04-24T21:30:00+08:00
lastmod: 2019-04-24T21:30:31+08:00
draft: false
tags: ["asp.net core","docker"]
slug: "aspnet-core-docker"
---
# 將 ASP.NET Core 部署至 Docker 中

最近專案在 macOS 上開發 ASP.NET Core 的 Web API，完成後再透過 Docker 包成 image 準備建立 container，流程很直覺，加上 CI CD 同事都已經準備好了，所以就沒有特別花時間，直到最近要 debug 某個新功能，需要頻繁 build image，才趁這個機會紀錄一下

保哥文章 [如何將 ASP.NET Core 2.1 網站部署到 Docker 容器中](https://blog.miniasp.com/post/2018/08/25/How-to-deploy-ASPNET-Core-to-Docker-Container) 說明地很仔細 (推薦先看保哥文章理解流程，再來抄新的 dockerfile 即可)，但因為 base image 有 rename 過，為了抄起來比較快，於是我紀錄一下新的用法

## 基本環境說明

1. macOS Mojave 10.14.4
2. Docker Engine - Community 18.09.2
3. .NET Core 2.2.101
4. ASP.NET Core  2.2.0
5. 預設 Web App (Model-View-Controller) 專案範本
    
    > 將專案放於 `TestDokcer` 資料夾中 (這個資料夾名稱會影響 Dockerfile 內容)

## 專案資料夾結構

- TestDokcer
  - TestDokcer
      - TestDokcer.csproj
      - ....
  - TestDocker.sln
  - Dockerfile


![1foldstructure](https://user-images.githubusercontent.com/3851540/56672279-c0868d00-66e8-11e9-92db-9dd0aad93195.png)


![2folderstructure2](https://user-images.githubusercontent.com/3851540/56672281-c0868d00-66e8-11e9-91a4-86a36e3c8c7d.png)

## Dockerfile

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# 複製 sln csproj and restore nuget
COPY *.sln .
# 目標路徑資料夾要與來源路徑資料夾名稱(因 sln 已存在專案之 path)
COPY TestDocker/*.csproj ./TestDocker/
# nuget restore
RUN dotnet restore

# 複製其他檔案
COPY TestDocker/. ./TestDocker/
WORKDIR /app/TestDocker
# 使用 Release 建置專案並輸出至 out
RUN dotnet publish -c Release -o out


FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
# 將產出物複製至 app 下
COPY --from=build /app/TestDocker/out ./
# 啟動 TestDokcer
ENTRYPOINT ["dotnet", "TestDocker.dll"]

```

## Build image

在 Dockerfile 所在資料夾的上一層執行

- 語法

    ```bash
    docker build {folderName} -t {name}:{tag}
    ```

- 範例

    ```bash
    docker build TestDocker -t yowko/aspnetcore:v1
    ```

![3dockerbuild](https://user-images.githubusercontent.com/3851540/56672282-c11f2380-66e8-11e9-8205-269c89a38a4a.png)


## 實際使用

將本機 `8080` 指向 container 中的 `80` port (雖然上面的 dockerfile 沒有 expose 80，而是在 base image 已經先處理了)

```
docker run -d -p 8080:80 yowko/aspnetcore:v1
```

## 心得

近期專案已經全數走在 .NET Core 與 Docker 上，最近開始實際部署，這才發現雖然是簡單的幾行指令也隱藏不少細節跟眉角呀

相較於之前使用 Windows Container 的經驗，.NET Core 在 linux container 上還是順利些，啟動速度、image 大小、debug 難度都是如此，有這樣的工作環境真的該感恩呀  哈哈


# 參考資訊
1. [如何將 ASP.NET Core 2.1 網站部署到 Docker 容器中](https://blog.miniasp.com/post/2018/08/25/How-to-deploy-ASPNET-Core-to-Docker-Container)
2. [ASP.NET Core Docker Sample](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/README.md)
3. [dotnet-docker/samples/aspnetapp/Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/Dockerfile)