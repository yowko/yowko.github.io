---
title: "使用指令 (jenkins cli) 來執行 Jenkins 動作：安裝 plugin、重新啟動"
date: 2017-10-26T08:53:00+08:00
lastmod: 2021-11-02T08:53:19+08:00
draft: false
tags: ["Jenkins"]
slug: "jenkins-cli"
aliases:
    - /2017/10/jenkins-cli.html
---
## 使用指令 (jenkins cli) 來執行 Jenkins 動作：安裝 plugin、重新啟動

一般情境我們都是使用 Jenkins GUI 來操作來安裝套件或是其他動作，可說是簡單又方便幾乎沒有進入門檻，對於想要學習 Jenkins 的人非常友善，也是推薦做法，今天則是要來介紹在無法使用 GUI 的情境 - 全自動化 script 安裝或是 docker 安裝時只能透過指令來執行 Jenkins 動作

## 下載 Jenkins CLI

Jenkins CLI 是跟著 Jenkins 版本而有不同的，如果 Jenkins 有更新就需要重路下載，載點就是在 Jenkins 中

* 下載連結格式

    > `https://{jenkis_url}/jnlpJars/jenkins-cli.jar`

* 下載連結範例

    > `https://localhost:8080/jnlpJars/jenkins-cli.jar`

## Jenkins CLI 指令介紹頁面

* 說明 URL 格式

    > `http://{jenkins_url}/cli/`

* 說明 URL 範例

    > `http://localhost:8081/cli/`

* 說明實例

    ![1jenkinscli](https://user-images.githubusercontent.com/3851540/32029886-e3ae22ba-ba29-11e7-8d7b-df6492dd2491.png)

## 執行 Jenkins CLI 指令

* 需要安裝 java 並加入環境變數中

    > 在 Jenkins 安裝機器上，安裝 Jenkins 時即會預設安裝 Java

* 指令格式

    > `java -jar jenkins-cli.jar [-s JENKINS_URL] command [options...] [arguments...]`

* 指令範例

    > `java -jar jenkins-cli.jar -s http://localhost:8081 version`

* 指令實例

    ![3comamnd](https://user-images.githubusercontent.com/3851540/32029888-e3f99682-ba29-11e7-966c-d0f74eee79d4.png)

## Jenkins CLI 特定指令說明

* 說明指令格式

    > `java -jar jenkins-cli.jar -s {jenkins_url} help {command}`

* 說明指令範例

    > `java -jar jenkins-cli.jar -s http://localhost:8081 help install-plugin`

* 設明指令實例

    ![2helpcommand](https://user-images.githubusercontent.com/3851540/32029887-e3d5264e-ba29-11e7-88c8-28fbceaf52d7.png)

## 執行 Jenkins CLI 指令需認證

因為 Jenkins 重要性很高，一般都會有認驗證檢查做第一層保護，避免重要資訊外洩或是惡意操作造成問題，Jenkins CLI 也有相同機制

* 錯誤訊息
  * 一般指令

        ```log
        ERROR: anonymous is missing the Overall/Read permission
        ```

  * help 指令

        ```log
        ERROR: You must authenticate to access this Jenkins.
        Jenkins CLI
        Usage: java -jar jenkins-cli.jar [-s URL] command [opts...] args...
        Options:
        -s URL       : the server URL (defaults to the JENKINS_URL env var)
        -http        : use a plain CLI protocol over HTTP(S) (the default; mutually exclusive with -ssh and -remoting)
        -ssh         : use SSH protocol (requires -user; SSH port must be open on server, and user must have registered a public key)
        -remoting    : use deprecated Remoting channel protocol (if enabled on server; for compatibility with legacy commands or command modes only)
        -i KEY       : SSH private key file used for authentication (for use with -ssh or -remoting)
        -p HOST:PORT : HTTP proxy host and port for HTTPS proxy tunneling. See https://jenkins.io/redirect/cli-https-proxy-tunnel
        -noCertificateCheck : bypass HTTPS certificate check entirely. Use with caution
        -noKeyAuth   : dont try to load the SSH authentication private key. Conflicts with -i
        -user        : specify user (for use with -ssh)
        -strictHostKey : request strict host key checking (for use with -ssh)
        -logger FINE : enable detailed logging from the client
        -auth [ USER:SECRET | @FILE ] : specify username and either password or API token (or load from them both from a file);
        for use with -http, or -remoting but only when the JNLP agent port is disabled
        
        The available commands depend on the server. Run the help command to
        see the list.
        ```

* 錯誤畫面
  * 一般指令

        ![4permission](https://user-images.githubusercontent.com/3851540/32029889-e41e63fe-ba29-11e7-9391-2376f2649c05.png)

  * help 指令

        ![5helppermission](https://user-images.githubusercontent.com/3851540/32029890-e4485664-ba29-11e7-8f67-39288139a267.png)

* 解決方式
    > 我個人使用指定帳號密碼的方式示範，但文件上較推薦使用 ssh 方式

  * 說明指令格式

        > `java -jar jenkins-cli.jar -s {jenkins_url} {command} --username {username} --password {password}`

  * 說明指令範例

        > `java -jar jenkins-cli.jar -s http://localhost:8081 help install-plugin --username yowko --password pass.123`

  * 設明指令實例

        ![6withauth](https://user-images.githubusercontent.com/3851540/32029891-e46df9fa-ba29-11e7-832f-d86053d7d3d6.png)

## 心得

因為想要透過 docker 來建立 Jenkins 服務，為了完全自動化作業所以想到利用 command script 來安裝需要的 plugin，進而發現原來 Jenkins 提供非常完整的 CLI 支援，順手紀錄一下。

想到一個 Jenkins 就有這麼多東西要學習，真的是學無止盡呀

## 參考資訊

1. [Jenkins CLI](https://wiki.jenkins.io/display/JENKINS/Jenkins+CLI)
2. [Support --username and --password as parameters to jenkins-cli.jar](https://github.com/chef-cookbooks/jenkins/issues/299)
