---
title: "使用 Docker 版 Nginx 建立靜態頁面網站"
date: 2020-01-29T14:30:00+08:00
lastmod: 2020-01-29T14:30:31+08:00
draft: false
tags: ["Docker","Nginx"]
slug: "nginx-docker-static-html"
---

## 使用 Docker 版 Nginx 建立靜態頁面網站

最近經手的小專案，需要一個呈現靜態頁面的網站，因為需要動態改變網站上顯示頁面的 routing 所以第一時間還是想到透過 ASP.NET Core 的機制來處理，不過仔細想想只為了 routing 就用 ASP.NET Core 莫名增加後續維護的成本及門檻實在沒必要，於是就打算透過 Nginx 來實現，過去沒有使用 Docker 版 Nginx 經驗，剛好趁這個機會試試看，隨手紀錄一下囉

情境說明：平常情況下網站會顯示 `A` 網頁，如果遇到特殊情況 (特賣活動、緊急公告...etc) 就顯示 `B` 網頁

## 基本環境說明

1. macOS Mojave 10.15.2
2. docker desktop community 2.2.0.0(42247)
3. Docker Engine 19.03.5
4. nginx 1.17.8

## 設定步驟

1. 準備不同顯示內容網頁

    > 以下僅示範用途，將 html 檔案儲存在 `/nginx/content` 中，請依個別實際情況調整

    - A.html

        ```html
        <html>
            <h1>Common Page</h1>
              <p>This is A</p>
        </html>
        ```

    - B.html

        ```html
        <html>
            <h1>Special Page</h1>
              <p>This is B</p>
        </html>
        ```

2. 準備 nginx 用 conf

    > 示範用途將 conf 儲存在 `/nginx/config/web.conf`，請依實際情況調整

    ```conf
    server{
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  yowkotest.com;
        root         /usr/share/nginx/html;
        index        A.html;#B.html
        charset utf-8;
        access_log /var/log/nginx/access_log;
        error_log /var/log/nginx/error_log;
    }
    ```

3. 啟動 container

    ```bash
    docker run --rm --name nginx -p 8080:80 -v /nginx/config/web.conf:/etc/nginx/conf.d/default.conf:ro -v /nginx/content:/usr/share/nginx/html:ro -d nginx
    ```

    - 將自訂的 nginx conf mount 至 container 中取代預設 nginx conf
    - 將想要顯示的網頁也 mount 至 container 中

4. 手動調整想要顯示的頁面

    ```conf
    server{
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  yowkotest.com;
        root         /usr/share/nginx/html;
        index        B.html;
        charset utf-8;
        access_log /var/log/nginx/access_log;
        error_log /var/log/nginx/error_log;
    }
    ```

5. 套用新的 conf

    > 下列兩個方法擇一即可，會進行 hot reload 不會讓 nginx 有 downtime

    - 使用 nginx cli

        ```bash
        docker exec -it nginx /etc/init.d/nginx reload
        ```

        ![1nginxreload](https://user-images.githubusercontent.com/3851540/73337269-ef044a80-42ae-11ea-855c-7cfc6268906e.png)

    - 使用 service 服務

        ```bash
        docker exec -it nginx service nginx reload
        ```

        ![2servicereload](https://user-images.githubusercontent.com/3851540/73337271-ef044a80-42ae-11ea-98cd-8c12c55b4c8c.png)

## 心得

之前使用 nginx 時沒有留下筆記，結果這次卡到好幾個點，搞得我都分不清楚是我記錯設定還是 docker 所引起的 config 差異，幸虧終究是解決了，雖然只是個簡單的設定但感覺真不錯，三不五時就有不同需求跟挑戰可以嘗試不同做法

## 參考資訊

1. [nginx](https://hub.docker.com/_/nginx)
2. [Hosting a static site using Nginx web server inside Docker container](https://medium.com/code-to-express/https-medium-com-kumarnitish-hosting-static-site-using-nginx-web-server-in-docker-container-167b31df70bb)
3. [Nginx config reload without downtime](https://serverfault.com/questions/378581/nginx-config-reload-without-downtime)
