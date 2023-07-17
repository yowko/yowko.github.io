---
title: "APT 安裝指定版本套件"
date: 2023-07-12T00:30:00+08:00
lastmod: 2023-07-12T00:30:31+08:00
draft: false
tags: ["linux","ubuntu"]
slug: "apt-specific-version"
---

## APT 安裝指定版本套件

原則上服務的版本升級應該都是樂見的，降低安全漏洞曝露風險、增加新的功能、避免日後跳版升級困難，除非有什麼特殊因素不能升級，最近安裝 RabbitMQ 時遇到的狀況就是如此：預設安裝 RabbitMQ 會安裝最新版本 `3.12.1` 搭配 Erlang 最新版本 `26`，結果官方 .NET library 無法成功連線需要降版 Erlang 至 `25`、RabbitMQ plugin 還未全面支援 `3.12` 需降版 RabbitMQ 至 `3.11.x`

但實際執行時卻發現目標套件是正確版本 `25`，但其他相依套件都直接安裝成最新版本 `26` 而 RabbitMQ 因為沒有安裝對應版本的 Erlang 而 fail

## 基本環境說明

1. Azure VM Standard B2s (2 vcpu，4 GiB 記憶體)
2. Linux (ubuntu 22.04)
3. erlang-nox 1:25.3.2.2-1
4. apt 設定

    > 這是從 [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html) 上抄來的

    ```bash
    #!/bin/sh

    sudo apt-get install curl gnupg apt-transport-https -y

    ## Team RabbitMQ's main signing key
    curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor | sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null
    ## Community mirror of Cloudsmith: modern Erlang repository
    curl -1sLf https://ppa1.novemberain.com/gpg.E495BB49CC4BBE5B.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null
    ## Community mirror of Cloudsmith: RabbitMQ repository
    curl -1sLf https://ppa1.novemberain.com/gpg.9F4587F226208342.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg > /dev/null

    ## Add apt repositories maintained by Team RabbitMQ
    sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
    ## Provides modern Erlang/OTP releases
    ##
    deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main
    deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main

    ## Provides RabbitMQ
    ##
    deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
    deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
    EOF

    ## Update package indices
    sudo apt-get update -y
    ```

5. 安裝 erlang 25

    ```bash
    sudo apt install -y erlang-nox=1:25.3.2.2-1
    ```

    ![1wrongversion](https://github.com/yowko/picsbed/assets/3851540/5a7ecc49-de81-47b6-9fbb-fb84bc84a3ed)

## 設定方式

1. 建立 preferences 檔案：`/etc/apt/preferences.d/erlang`

    ```txt
    Package: erlang*
    Pin: version 1:25.3.2.2-1
    Pin-Priority: 1000
    ```

2. 更新 apt cache 套用 preference

    ```bash
    sudo apt policy
    ```

3. 重新安裝指定版本 erlang

    ```bash
    sudo apt install -y erlang-nox=1:25.3.2.2-1
    ```

    ![2correctversion](https://github.com/yowko/picsbed/assets/3851540/ecbdc009-7551-4aa7-a0c1-9c021824a406)

## 心得

這次遇到的狀況是成功安裝指定版本的套件，但相依套件都安裝最新版本的，我快速看了一下 apt 的依賴寫法，應該是套件本身要處理好相依套件的版本 (Depends 的指定版本範圍)，但我個人猜測可能是因為相依套件不一定會跟著升級，所以就不能寫得太死

我以為這個問題很常見，結果資源比想像中難找很多，我也還不是很清楚有沒有其他解決方式，先紀錄一版囉

## 參考資訊

1. [RabbitMQ 官網：Installing on Debian and Ubuntu](https://www.rabbitmq.com/install-debian.html)
2. [How do I see what packages are installed on Ubuntu Linux?](https://www.cyberciti.biz/faq/apt-get-list-packages-are-installed-on-ubuntu-linux/)
3. [How to Install a Specific Version of a Package in Ubuntu Linux](https://trendoceans.com/install-a-specific-version-of-a-package-in-ubuntu/)
4. [Install Specific Package Version With Apt Command in Ubuntu](https://itsfoss.com/apt-install-specific-version/)
5. [Managing Debian Software with APT (apt-get etc)](https://www.linuxtopia.org/online_books/linux_system_administration/managing_debian_software_with_apt/ch-apt-get.en_009.html)
