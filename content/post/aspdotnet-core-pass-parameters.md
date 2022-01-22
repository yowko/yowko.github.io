---
title: "啟動 ASP.NET Core 時傳入參數"
date: 2022-01-22T00:30:00+08:00
lastmod: 2022-01-22T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp"]
slug: "aspdotnet-core-pass-parameters"
---

## 啟動 ASP.NET Core 時傳入參數

團隊中有多個專案都有使用相同 source code，不過可以依據 config 不同而執行著不同任務的特性

sre 在處理這種類型的 application 時都是 build 一份 image，在部署時透過給不同的 env 來指定 container 用途

最近因為實驗性專案需求，想要在同個 container 中執行多個 application：同一份程式碼，透過不同 process 來執行不同任務
在這樣條件下，修改 env 跟調整 config file 都有些限制

- 修改 env

    > 我想到的就是動態修改 env，但又不想汙染整個 container 的 enviroment 參數A

- 調整 config file

    > 因為是同份 code，所以 config file 也是同一份，除非是產生兩份 config，但這的讓 build flow 變複雜

## 基本環境說明

1. macOS Monterey 12.1
2. .NET SDK 6.0.100
3. 程式語法

    ```cs
    Console.WriteLine(Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"));
    Console.WriteLine(Environment.GetEnvironmentVariable("mode"));
    ```

## 使用方式

- 確認可用

    - 語法

        ```bash
        {env_name1}={env_value1} {env_name2}={env_value2} dotnet {application}.dll
        ```

    - 範例

        ```bash
        mode=test ASPNETCORE_ENVIRONMENT=Prod dotnet ArrayConfig.dll
        ```

    - 效果

        ![1ok](https://user-images.githubusercontent.com/3851540/150645567-2cdc9053-818f-418b-a652-4c557ed1fe08.png)

- 其他嘗試 (失敗)

    - 使用 `,`

        > 所有內容都會指定成第一個 env 的 value

        ![2comma](https://user-images.githubusercontent.com/3851540/150645569-ab4ebcde-90f1-4ba6-9b60-6fc64696760c.png)

    - 使用 `;`

        > 只有最接近 `dotnet` 的 env 設定生效

        ![3semicolon](https://user-images.githubusercontent.com/3851540/150645570-29f4b530-bdf4-41aa-a962-8f10be49fc94.png)

    - 使用 `&&`

        > 只有最接近 `dotnet` 的 env 設定生效

        ![4andand](https://user-images.githubusercontent.com/3851540/150645571-cad8988a-28a0-49a2-8a6d-25a9722a6e09.png)

    - 使用 `&`

        > 無效的設定方式

        ![5and](https://user-images.githubusercontent.com/3851540/150645572-a18d0662-051e-41c9-9c83-ed6189806a1e.png)

## 心得

我覺得這個應該是 shell 的使用技巧，但我關鍵字下得不好 沒能找到直接的說明

之前也遇過這個使用情境，當時也試了一輪才找到正確的設定方式，最近又試了一輪XD  雖然內容簡單但還是決定簡單紀錄一下

## 參考資訊

1. [Setting an environment variable before a command in Bash is not working for the second command in a pipe](https://stackoverflow.com/questions/10856129/setting-an-environment-variable-before-a-command-in-bash-is-not-working-for-the)
