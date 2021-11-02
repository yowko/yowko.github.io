---
title: "如何使用 Jenkins 2 建置 .NET 專案"
date: 2017-02-09T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps"]
slug: "jenkins-build-dotnet-project"
aliases:
    - /2017/02/jenkins-2-build-dotnet-project.html
---
## 如何使用 Jenkins 2 建置 .NET 專案

實際上使用 Jnekins 1 跟 Jenkins 2 在建置 .NET 專案的設定並沒有不同，依照 Jenkins 1 的設定即可，因為已改用 Jenkins 2 ，怕設定畫面不同，造成誤解，所以標題就直接使用 Jenkins 2

## 文章大綱

1. 安裝 Plugin
2. 設定 NuGet
3. 設定 MSBuild
4. 新增 project
5. prject 設定
6. Build Now

## 安裝 Plugin

- 視實際情況來安裝

    ![1plugin](https://cloud.githubusercontent.com/assets/3851540/21884148/149c17b6-d8ed-11e6-9ed9-09e435eb3afc.png)
- 必要：NuGet,MSBuild
- 版控： GitHub or Subversion or Visual Studio Team Foundation Server (TFS)

## 設定 NuGet

1. Manage Jenkins --> Configure System

    ![10NuGetsetting](https://cloud.githubusercontent.com/assets/3851540/21884152/14dba340-d8ed-11e6-85c9-3abf92584469.png)
2. NuGet 區塊
    - 指定 NuGet.exe 絕對位置
    - 可以透過 [chocolatey NuGet.CommandLine](https://chocolatey.org/packages/NuGet.CommandLine/3.4.3) 安裝 NuGet command line 工具
    - 預設值是 `${WORKSPACE}\.NuGet\NuGet.exe`

        ![11NuGet](https://cloud.githubusercontent.com/assets/3851540/21758729/4c340556-d679-11e6-8fc7-2a5220920aba.png)

## 設定 MSBuild

1. Manage Jenkins --> Global Tool Configuration

    ![7globalmsbuild](https://cloud.githubusercontent.com/assets/3851540/21884150/14d779dc-d8ed-11e6-8cd6-c0b406a33692.png)
2. MSBuild 區塊

    ![8msbuild](https://cloud.githubusercontent.com/assets/3851540/21884151/14d99834-d8ed-11e6-8ae6-b7b5dba66fdd.png)

    - 可以同時指定多個版本的 MSBuild
    - 給定識別用的 name 及 MSBuild.exe 所在資料夾路徑

        ![9msbuilds](https://cloud.githubusercontent.com/assets/3851540/21758727/4c31f32e-d679-11e6-9bca-a478d8754cdd.png)

## 新增 project

1. 加入新 job
    - `New Item` 與 `create new jobs` 都可以

        ![2create](https://cloud.githubusercontent.com/assets/3851540/21884149/14bf0e38-d8ed-11e6-93dc-ef2a3cc08eb0.png)
2. project 名稱 及類型

    ![3newporject](https://cloud.githubusercontent.com/assets/3851540/21758721/4c0eb71a-d679-11e6-9abc-e129b7bc221f.png)
    - 2-1. project 名稱
    - 2-2. 選擇 project 類型( Freestyle project)

## prject 設定

- Source Code Management

    ![4source](https://cloud.githubusercontent.com/assets/3851540/21758722/4c105c46-d679-11e6-8d9d-bfa471cddc64.png)
  - `Source Code Management` tab
  - 以 GitHub 為例
  - 指定 Repository
  - 指定 登入資訊
    - 選擇 or 新增

            ![5addcred](https://cloud.githubusercontent.com/assets/3851540/21758723/4c117b94-d679-11e6-8c7b-b9b66cd4a2da.png)
    - add credentials

            ![6cred](https://cloud.githubusercontent.com/assets/3851540/21758724/4c2d894c-d679-11e6-8df8-94c5a66170e8.png)
  - 選定 Branch

- Build

    ![12build](https://cloud.githubusercontent.com/assets/3851540/21758730/4c4f81a0-d679-11e6-9092-9d9c480d58b3.png)
  - 使用 plugin
        1. NuGet 還原
            - Add build step -->Execute Winodws batch command

                ![13NuGet](https://cloud.githubusercontent.com/assets/3851540/22652592/fc9a3c54-ecc1-11e6-9d46-ea17a4cb72e5.png)
            - 指令

                ```
                NuGet restore {projectName}.sln
                ```

            - 如上網需 proxy 時的指令(非必要)

                ```
                NuGet config -set http_proxy=http://proxyserver:port
                NuGet config -set http_proxy.user=AD\username
                NuGet config -set http_proxy.password=password
                NuGet restore {projectName}.sln 
                ```

        2. MSBuild
            - Add build step --> Build a Visual Studio project or solution using MSBuild

                ![14msbuild](https://cloud.githubusercontent.com/assets/3851540/21758732/4c53bbee-d679-11e6-9cec-a13f4535fac2.png)
            - 設定

                ![16buildsetting](https://cloud.githubusercontent.com/assets/3851540/21758733/4c54ef64-d679-11e6-8e62-d38070d70aa1.png)
                - MSBuild Version
                    - 可使用前面設定的 MSBuild 版本

                        ![15msbuildversion](https://cloud.githubusercontent.com/assets/3851540/21758716/4bcc7c1a-d679-11e6-83cd-9ecd5fb1460a.png)
                - MSBuild Build File
                    - 方案檔或是專案檔名稱
                - Command Line Arguments
                    - 指定 build 的參數
                    - build mode,build platform
                      - e.g. `/p:Configuration=Release /p:Platform="Any CPU"`

  - 直接指定路徑
        1. NuGet 還原
            - Add build step --> Execute Winodws batch command

                ![13NuGet](https://cloud.githubusercontent.com/assets/3851540/22652592/fc9a3c54-ecc1-11e6-9d46-ea17a4cb72e5.png)
            - 指令

                ```
                "C:\ProgramData\chocolatey\lib\NuGet.CommandLine\tools\NuGet.exe" restore {projectName}.sln
                ```

            - 如上網需 proxy 時的指令(非必要)

                ```
                "C:\ProgramData\chocolatey\lib\NuGet.CommandLine\tools\NuGet.exe" config -set http_proxy=http://proxyserver:port
                "C:\ProgramData\chocolatey\lib\NuGet.CommandLine\tools\NuGet.exe" config -set http_proxy.user=AD\username
                "C:\ProgramData\chocolatey\lib\NuGet.CommandLine\tools\NuGet.exe" config -set http_proxy.password=password
                "C:\ProgramData\chocolatey\lib\NuGet.CommandLine\tools\NuGet.exe" restore {projectName}.sln 
                ```

        2. MSBuild
            - Add build step --> Execute Winodws batch command

                ![13NuGet](https://cloud.githubusercontent.com/assets/3851540/22652592/fc9a3c54-ecc1-11e6-9d46-ea17a4cb72e5.png)
            - 指令

                ```
                "C:\Program Files (x86)\MSBuild\14.0\Bin\amd64\MSBuild.exe" {projectName}.sln  /p:Configuration=Release /p:Platform="Any CPU"
                ```

## Build Now

1. 執行建置(擇一)

    ![17buildnow](https://cloud.githubusercontent.com/assets/3851540/21884153/14dd092e-d8ed-11e6-98f1-a52a470085ae.png)

    ![22startbuild](https://cloud.githubusercontent.com/assets/3851540/21884158/14fe008e-d8ed-11e6-9107-d8244941cdab.png)
2. 開始建置

    ![18startbuild](https://cloud.githubusercontent.com/assets/3851540/21884154/14e070be-d8ed-11e6-8c75-5f778ef65960.png)

3. 詳細建置資訊

    ![19detail](https://cloud.githubusercontent.com/assets/3851540/21884155/14e32d86-d8ed-11e6-8596-9678e4b167dd.png)
4. 建置結果

    ![20success](https://cloud.githubusercontent.com/assets/3851540/21884156/14fb3f66-d8ed-11e6-9ae4-7d001ae06d8a.png)

    ![21result](https://cloud.githubusercontent.com/assets/3851540/21884157/14fc98c0-d8ed-11e6-915e-bec4c586d3cf.png)

## 參考資料
