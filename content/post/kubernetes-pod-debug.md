---
title: "如何 Debug Kubernetes Pod"
date: 2024-09-13T00:30:00+08:00
lastmod: 2024-09-13T00:30:31+08:00
draft: false
tags: ["csharp","kubernetes"]
slug: "kubernetes-pod-debug"
---

## 如何 Debug Kubernetes Pod

團隊從 .NET 8 問市之後就開始逐步將舊有的專案升級到 .NET 8，隨著所有專案都升級完成之後，團隊便開始套用 Microsoft PM - Richard Lander 發表的 [Announcing .NET Chiseled Containers](https://devblogs.microsoft.com/dotnet/announcing-dotnet-chiseled-containers/?WT.mc_id=DOP-MVP-5002594) 做法：將 base image 中改為 `mcr.microsoft.com/dotnet/aspnet:8.0-jammy-chiseled` 這樣可以讓 image 更小，並且更安全。

在整理資料的過程發現我對於 image 的大小與 container 安全性的了解已經過時，剛好趁著這個機會更新一下

- rootless 的 container：不要使用 root 來執行 container，以限制 process 的權限
- shellless：image 中不包含 shell
- binaryless：image 中除了由我們自行建立的 binary 之外不包含其他 binary
- distroless：image 中除了由我們自行建立的 binary 之外不包含其他程式、套件以及系統文件

[.NET Chiseled Containers](https://devblogs.microsoft.com/dotnet/announcing-dotnet-chiseled-containers/?WT.mc_id=DOP-MVP-5002594) 就是 Google Distroless 的應用，讓 image 除了我們自行建立的 binary 之外不包含任何其他東西，這樣可以減少 image 的大小，也可以減少攻擊面，不過提高安全性就不免會犧牲便利性，所以今天就來紀錄一下如何 Debug Kubernetes 中使用這類 shellless、binaryless、distroless image 或是 chiseled container 的 Pod，以下紀錄主要以工作上用到的 .NET chiseled image 為例

## 基本環境說明

- macOS Sonoma 14.6.1 (Apple M2 Pro)
- OrbStack 1.7.2 (17389)
- container images

    - mcr.microsoft.com/dotnet/samples:aspnetapp-chiseled
    - nicolaka/netshoot

- 使用 `mcr.microsoft.com/dotnet/samples:aspnetapp-chiseled` 模擬 .NET chiseled image 建立的 container

    ```bash
    kubectl run test --image=mcr.microsoft.com/dotnet/samples:aspnetapp-chiseled
    ```

## 面臨狀況與 Debug 方式

- 無法透過 `kubectl exec` 進入 container 執行 debug (確認網路或是檢查設定....)

    ![1shellless](https://github.com/user-attachments/assets/1678e977-3427-46ce-9d4b-e5947d80a761)

- 使用暫時性 debug 用 container

    - 透過 `--target`：直接進入目標 container 執行 debug

        > 適用在 pod 仍在 running

        - 語法

            ```bash
            kubectl debug -it {pod name} --image={image name} --target={container name}
            ```

        - 範例

            ```bash
            kubectl debug -it test --image=nicolaka/netshoot --target=test
            ```

            ![2debug](https://github.com/user-attachments/assets/d468d5c3-a640-4bd3-aad1-7c20797df204)

            ![3localhost](https://github.com/user-attachments/assets/7a55d494-7f7f-4ae3-8a52-178abcc9911e)

    - 透過 `--copy-to`：複製目標 container 並進入新的 container 執行 debug

        > 適用在 pod 無法正確啟動

        - 語法

            ```bash
            kubectl debug -it {pod name} --image={image name} --copy-to={new container name}
            ```

        - 範例

            ```bash
            kubectl debug -it test --image=nicolaka/netshoot --copy-to=test-debug
            ```

            ![4copyto](https://github.com/user-attachments/assets/46b76027-0770-4bad-bc88-3d15d45c43f0)

## 心得

- kubectl debug 如果未指定 container 名稱 (使用 `--container`) 會自動產生
- kubectl debug 的 `-i` 會自動連線至新的 container，可以透過 `--attach=false` 來避免自動連線 (但這樣就無法 debug，我沒想到什麼情境會用到)
- 使用 `exit` 退出 debug container 會自動刪除 container
- 如果 session 斷開，但 container 還在執行，可以透過 `kubectl attach` 重新連線
- `--share-processes` 可以讓新的 container 與原本的 container 共享 process namespace，這樣就可以看到 pod 中其他 container 的 process

## 參考資料

1. [Announcing .NET Chiseled Containers](https://devblogs.microsoft.com/dotnet/announcing-dotnet-chiseled-containers/?WT.mc_id=DOP-MVP-5002594)
2. [shellless 容器、binaryless 容器以及 distroless 容器](https://mozillazg.com/2021/05/security-use-shell-less-and-binary-less-distroless-container-with-root-less-container.html)
3. [Secure your .NET cloud apps with rootless Linux Containers](https://devblogs.microsoft.com/dotnet/securing-containers-with-rootless/?WT.mc_id=DOP-MVP-5002594)
4. [當遇到 Distroless Container 除錯要什麼沒什麼該怎麼辦? 你的好朋友 kubectl debug](https://blog.pichuang.com.tw/20230713-distroless-container-debug.html)
5. [GitHub:nicolaka/netshoot](https://github.com/nicolaka/netshoot)
6. [GitHub:GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless)
