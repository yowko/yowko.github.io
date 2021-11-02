---
title: "Jenkins 該如何使用 SSH 存取 AD(LDAP) 驗證的 Git server"
date: 2017-05-09T23:30:00+08:00
lastmod: 2021-11-02T00:11:23+08:00
draft: false
tags: ["Git","Jenkins"]
slug: "jenkins-ssh-ad-ldap-git-server"
aliases:
    - /2017/05/jenkins-2-ssh-ad-ldap-git-server.html
    - /2017/05/jenkins-ssh-ad-ldap-git-server/
---
## Jenkins 該如何使用 SSH 存取 AD(LDAP) 驗證的 Git server

最近公司的專案正積極地從 SVN 搬遷到 GIT，所以連帶的 CI Server - Jenkins 這邊的 SCM (Source Code Management) 也需要一併調整，同事為了不想寫死帳號密碼所以打算透過 SSH 來存取 Git Server，公司為了讓大家使用上更便利，也讓大家不用記那麼多組帳號密碼，Git server 的驗證是整合 AD，而在進行 Jenkins 整合 git server 時發現網路比較少使用 AD 搭配 SSH 的做法，所以紀錄一下，另外公司目前 Git server 是使用 self-host 的 gitlab ，所以接下來的 demo ，Git server 的部份會使用 Gitlab 來呈現。

## 產生 AD 帳號的 SSH Key

1. 確認 AD 帳號

    > 最直接的方式就是去看 `.gitconfig` (通常位於 `C:\Users\{username}`) 內容, 確認對應 crendential 所設定的 username

    * 範例：

        ```config
        [credential "http://gitlab.yowko.com"]
        helper = wincred
        username = yowko.tsai
        ```

        > 表示對於 git server ("[http://gitlab.yowko.com"](http://gitlab.yowko.com%22)) AD 使用的 username 就是 `yowko.tsai`；而一般公開的 git server 服務這個 username 常常是 e-mail

2. Generate SSH key

    * 切換目錄至 `C:\Users\{username}\.ssh`

        ![1folder](https://cloud.githubusercontent.com/assets/3851540/25860223/864b24b0-3513-11e7-9fb3-05c904660a7f.png)

    * 執行 `ssh-keygen -t rsa -C "{上面拿的 AD 帳號}"`

        > `ssh-keygen -t rsa -C "yowko.tasi"`

        * 過程中會詢問三個問題 --> 直接 enter 即可

            ![2genkey](https://cloud.githubusercontent.com/assets/3851540/25860226/8662c9a8-3513-11e7-938a-514a39f2f853.png)

        * 最後會產生兩個檔案 `id_rsa(private key)`、 `id_rsa.pub(public key)`

            ![3twokey](https://cloud.githubusercontent.com/assets/3851540/25860228/86860f26-3513-11e7-88e7-1ea96e8c901c.png)

3. 檢查 SSH 是否正確

    > 這邊請參考小風的做法 [2013-04-23 Windows使用ssh對Github進行操作](https://dotblogs.com.tw/kirkchen/2013/04/23/use_ssh_to_interact_with_github_in_windows)

    * 將上面的 `.ssh` 包含產生出來的 key copy 至 `C:\Program Files (x86)\Git\` 下

    * 執行 `ssh -T {git server}`

        > `ssh -T git@gitlab.yowko.com`

        ![5testssh](https://cloud.githubusercontent.com/assets/3851540/25860220/863e45e2-3513-11e7-8222-fc23236a8dd1.png)

## 將 Public Key 綁定至 Gitlab 的帳號

1. 登入 Git Server
2. 開啟 Settins
3. 開啟 SSH tab
4. 將上面產生的 id_rsa.pub 內容複製並貼上

    ![4gitlab](https://cloud.githubusercontent.com/assets/3851540/25860229/86ea05e4-3513-11e7-8db5-5925417de0c6.png)

## Jenkins 設定

1. 加入 SSH 的 credential

    * Credentials --> System --> Global credentials (unrestricted)

        ![6addcred](https://cloud.githubusercontent.com/assets/3851540/25860227/866fefac-3513-11e7-8297-987940b7b245.png)

    * Add Credentials

        ![7addcred](https://cloud.githubusercontent.com/assets/3851540/25860222/864a3f8c-3513-11e7-8bd2-49287b6e0b88.png)

    * 設定 Credentail

        * Kind：SSH Username with private key
        * Name：自訂顯示
        * Private Key:Enter directly
            * key：將一開始產生的 private key (id_rsa) 內容完整複製貼上

                ```cert
                -----BEGIN RSA PRIVATE KEY-----
                MI---------------------------------------------fJI=
                -----END RSA PRIVATE KEY-----
                ```

        * Passphase：請保持空白
        * Id ：不會顯示出來，沒有指定時會使用 UUID

        ![8ssh](https://cloud.githubusercontent.com/assets/3851540/25860221/86494690-3513-11e7-8092-6151710e3901.png)

2. Job 設定
    * 原始碼管理 (Source Code Management) --> 指定 Git Server 的 SSH 路徑 --> 使用上面步驟加入的 credential

        ![9jobsetting](https://cloud.githubusercontent.com/assets/3851540/25860225/86628d12-3513-11e7-8536-8d923a291e5e.png)

## 參考資訊

1. [git教學（github、gitlab）](http://qbsuranalang.blogspot.tw/2015/01/gitgithubgitlab.html)
2. [2013-04-23 Windows使用ssh對Github進行操作](https://dotblogs.com.tw/kirkchen/2013/04/23/use_ssh_to_interact_with_github_in_windows)
