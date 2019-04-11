---
title: "為 Windows Container 安裝 Git"
date: 2019-02-18T20:40:00+08:00
lastmod: 2019-02-18T20:40:30+08:00
draft: false
tags: ["Docker","Windows","Git"]
slug: "windows-container-git"
---
# 為 Windows Container 安裝 Git

同事想要在 Windows Container 上安裝 Git 當做 base image 再用來擴充其他功能，但 Git for Windows 僅提供 .exe 版本，沒辦法用 msiexec，直接用 .exe 安裝又無法避開 popup 視窗問題

過去我好像也遇過相同問題，當時是透過 Chocolatey 解決，今天先筆記一下 Chocolatey 做法，改天再找看看有沒有其他方式

## 基本環境說明

1. Windows 10 Version 1803 (OS Build 17134.590)
2. Docker Community 18.09.1 (windows/amd64)

## Dockerfile

```
FROM mcr.microsoft.com/windows/servercore
# 安裝 Chocolatey
RUN Powershell.exe -Command Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
# 透過 Chocolatey 安裝 git
RUN choco install git.install -y
```

## 建置與實際使用
1. 建置 image

    - 格式
    
        ```cmd
        docker build {dockerfile folder} -t {repository:tag}
        ```
    
    - 範例
        ```cmd
        docker build c:\WinContainerGit -t windowswithgit:v1
        ```
        
        > `c:\WinContainerGit` 為放 `dockerfile` 檔案的資料夾

2. 建立 container 並確認成功安裝 Git

    ```cmd
    docker run -it windowswithgit:v1 git --version
    ```

    ![1gitversion](https://user-images.githubusercontent.com/3851540/52961640-601c5c00-33d6-11e9-9edc-dfb6abd45704.png)

## 心得
印象中兩、三年前想要在 Windows Container 中安裝 Git，當時並沒有找到好方法，最後想到用 Chocolatey 來安裝才成功

雖然得要多安裝 Chocolatey 讓 image size 變大，但都用 Windows Container 了應該是不差這點空間XD

不過終究是不夠漂亮，改天有時間再來找找看其他做法囉

# 參考資訊
1. [Dockerfile reference](https://docs.docker.com/engine/reference/builder)
2. [docker image build](https://docs.docker.com/engine/reference/commandline/image_build/)
3. [Installing Chocolatey](https://chocolatey.org/docs/installation)