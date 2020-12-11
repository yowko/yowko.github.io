---
title: "Windows Dockerfile 如何指定 VOLUME"
date: 2017-09-26T02:18:00+08:00
lastmod: 2020-12-11T02:18:08+08:00
draft: false
tags: ["Docker"]
slug: "windows-dockerfile-volume"
aliases:
    - /2017/09/windows-dockerfile-volume.html
---
# Windows Dockerfile 如何指定 VOLUME
同事在參考 [使用 dockerfile 建立 Windows Container 版 Jenkins](/2017/08/dockerfile-windows-container-jenkins.html) 後，打算透過指定 volume 的方式將 Jenkins 相關設定儲存在 host 環境上不用隨 container 異動重新設定，經過一番嘗試，終於找出正確設定，還督促著我要記得將筆記補齊 XD

所以為了讓其他同事在實作時也能更順利完成，就來看看該如何設定吧

## Docker Volume 的用途

container 本身是個用完即丟的概念，而 container 中的相關檔案當然也會跟著 container 被刪除而消失，為了讓 container 的檔案可以被保存以及與其他 container 分享資訊，docker 提出 volume 的做法，透過指定 volume 的方式讓 container 將檔案儲存至 host 上，詳細內容可以參考 [深入理解Docker Volume（一）](http://dockone.io/article/128)

## 使用 docker volume

1.  建立 container volume (在 container 中加入資料夾並自動指定 host mapping)

    > `docker run -it -v C:/tsst microsoft/windowsservercore cmd`

    *   取得自動對應至 host 的資料夾位置

        > `docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' container_id_or_name`

        ![1getvolumemapping](https://user-images.githubusercontent.com/3851540/30823731-cfc94496-a25f-11e7-8ebe-c26501ac82d4.png)

    *   在 host 加入檔案，container 就可以看到

        ![2addfile](https://user-images.githubusercontent.com/3851540/30823735-cfd03f1c-a25f-11e7-82e0-8fb041cee17e.png)

2.  將 host folder 指定給 container volume

    > `docker run -it -v C:/testvolume:C:\volume microsoft/windowsservercore cmd`

    ![3volumemapping](https://user-images.githubusercontent.com/3851540/30823732-cfcd6a62-a25f-11e7-9cc6-d7e553b998c8.png)

## Windows Dockerfile 的 VOLUME 注意事項

> 指定 container 內的 volume 位置必需符合下列條件

1.  必需是空資料夾或是不存在的資料夾

    ![4notempty](https://user-images.githubusercontent.com/3851540/30823736-d0109abc-a25f-11e7-838c-05f866bde5bd.png)

2.  只能是 `c:`

    ![5notc](https://user-images.githubusercontent.com/3851540/30823733-cfcf0840-a25f-11e7-88fb-a64450771c2f.png)

## Dockerfile 中指定 Volume

以 [使用 dockerfile 建立 Windows Container 版 Jenkins](/2017/08/dockerfile-windows-container-jenkins.html) 為例

```dockerfile
#指定基礎 os image
FROM microsoft/windowsservercore

#將 jenkins 安裝檔複製到 container 中
ADD ./setup c:/jenkins

#安裝 jenkins
RUN ["msiexec.exe", "/i", "C:\\jenkins\\jenkins.msi", "/qn"]

#移除 container 中的安裝檔(讓 image 保持乾淨)
RUN Powershell.exe -Command remove-item c:/jenkins -Recurse

#對外使用 8080 port
EXPOSE 8080  

#如果有 slave 需加上 50000 port
EXPOSE 50000
```

> 路徑中的空白需用 `%20` 取代，符號可以用 `\` 跳脫或是使用 ascii code

*   指定特定資料夾

    *   寫法一：使用 ascii code

        > `VOLUME "C:\Program%20Files%20%28x86%29\Jenkins\jobs"`

    *   寫法二：使用 `\`

        > `VOLUME C:\\Program%20Files%20\(x86\)\\Jenkins\\jobs`

*   指定多個資料夾(使用空白符號區隔)

    *   寫法一：使用 ascii code

        > `VOLUME "C:\Program%20Files%20%28x86%29\Jenkins\jobs" "C:\Program%20Files%20%28x86%29\Jenkins\logs"`

    *   寫法二：使用 `\`

        > `VOLUME C:\\Program%20Files%20\(x86\)\\Jenkins\\jobs C:\\Program%20Files%20\(x86\)\\Jenkins\\logs`

*   完整範例

    ```dockerfile
    #指定基礎 os image
    FROM microsoft/windowsservercore
    
    # image 維護者資訊
    MAINTAINER yowko@yowko.com
    
    #將 jenkins 安裝檔複製到 container 中
    ADD ./setup c:/jenkins
    
    #安裝 jenkins
    RUN ["msiexec.exe", "/i", "C:\\jenkins\\jenkins.msi", "/qn"]
    
    #移除 container 中的安裝檔(讓 image 保持乾淨)
    RUN Powershell.exe -Command remove-item c:/jenkins -Recurse
    
    # 指定 volume
    VOLUME "C:\Program%20Files%20%28x86%29\Jenkins\jobs" "C:\Program%20Files%20%28x86%29\Jenkins\logs"
    
    #對外使用 8080 port
    EXPOSE 8080  
    
    #如果有 slave 需加上 50000 port
    EXPOSE 50000
    
    # 預設執行動作，可用來避免 container 自動停止
    CMD [ "ping localhost -t" ]
    ```
    
## 心得

windows 的 volume 用法跟 linux 不太相同，不同的點在於對符號的處理方式需要特別留意，windows 的 dockerfile 文件也相對不好查，但終究是找到正確的做法了

# 參考資訊

1.  [Dockerfile on Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile?WT.mc_id=DOP-MVP-5002594)
2.  [使用 dockerfile 建立 Windows Container 版 Jenkins](/2017/08/dockerfile-windows-container-jenkins.html)
3.  [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
4.  [深入理解Docker Volume（一）](http://dockone.io/article/128)
