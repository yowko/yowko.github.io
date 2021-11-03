---
title: "傳遞參數來執行 Docker Compose"
date: 2020-05-19T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["Redis","Docker"]
slug: "docker-compose-pass-parameter"
---

## 傳遞參數來執行 Docker Compose

為了能動態決定 Docker Compose 中所使用到的參數值可以透過 `environment variable` , `.env` file 另外個人更偏好在執行 docker-compose up 時直接指定變數值，因為這樣一來不必設定 environment variable r而擔心造成汙染，也不需要多一個檔案，顯得更簡潔

這是個簡單的語法，也因為太簡單，這不知道是我第幾次查資料了，剛好最近筆記都是走這類沒營養的內容，超適合藉這個機會紀錄一下

## 基本環境說明

1. macOS Catalina 10.15.4
2. docker desktop community 2.3.0.2(45183)

    - Engine 19.03.8
    - Compose 1.25.5

## 設定方式

> 我個人最常用的就是將 host ip 傳進去 docker compose 中做為 service bind 的參數，因為 ip 可能會變動，常態性將 ip 寫死在 docker compose 中，不時會遇到 ip 變了造成 service 沒有正確啟動而多花了不少時間在 debug

- 直接寫死 ip

  - yaml

        ```yaml
        version: "3"
        services:
          test:
            image: busybox
            entrypoint: [ echo,192.168.1.112 ]
        ```

  - 啟動：`docker-compose up`

        ```bash
        docker-compose up
        ```

  - 結果

        ![1old](https://user-images.githubusercontent.com/3851540/82342682-80492980-9a24-11ea-8bc1-7c99f30484b5.jpg)

- 使用參數

  - yaml

        ```yaml
        version: "3"
        services:
          test:
            image: busybox
            entrypoint: [ echo,"${ip}" ]
        ```

  - 啟動：`docker-compose up`

        ```bash
        ip=$(ipconfig getifaddr en0) docker-compose up
        ```

  - 結果

        ![2new](https://user-images.githubusercontent.com/3851540/82342690-817a5680-9a24-11ea-8af1-f8a36909f325.jpg)

## 心得

簡單的使用方式跟流程，簡單筆記一下  減少自己查資料時間

## 參考資訊

1. [How do i pass an argument along with docker-compose up](https://stackoverflow.com/questions/35093256/how-do-i-pass-an-argument-along-with-docker-compose-up)
