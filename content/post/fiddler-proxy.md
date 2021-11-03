---
title: "為 fiddler 設定需驗證的 proxy"
date: 2016-12-23T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "fiddler-proxy"
aliases:
    - /2016/12/fiddler-proxy-with-authentication.html
---
## 為 fiddler 設定需驗證的 proxy

不管是第三方介接或是 mobile device 開發，不免都會使用到外部 API，除了使用 postman 進行 API 內容確認外，fiddler 更是用來驗證執行內容與除錯的的一大利器

## 1. 開啟設定頁面

* Tools --> Telerik Fiddler Options...

    ![step1](https://trello-attachments.s3.amazonaws.com/5801e1bda8b80f14cec7e83f/600x337/49d65dc2bbbaa9e797be35e2f4e29a4f/fiddler1-5-1_%E7%BB%93%E6%9E%9C.png)

## 2.代理伺服器(proxy)相關設定

* Gatway --> Manual Proxy Configuration:

  * 2-1. 設定代理伺服器(proxy)

        ```  
        http=proxyserver:proxyport;https=proxyserver:proxyport;
        ```

        ![2proxysetting](https://trello-attachments.s3.amazonaws.com/581164a17a360d6e6a151489/813x552/ffbadc5a0c2b81b9bfafd01b2cf435fb/_output_2proxysetting.png)

  * 2-2. 指定排除的 ip(不使用proxy)
    * 可以使用 domain 或 ip

            ![1fiddlerbypass](https://trello-attachments.s3.amazonaws.com/581164a17a360d6e6a151489/813x552/17563aa1c1f94b1c5d8cecf36ae363fe/_output_1fiddlerbypass.png)

## 3. 直接在 Request 內容中加入認證(crednetial)

* Rules --> Customize Rules...

    ![step3](https://trello-attachments.s3.amazonaws.com/5801e1bda8b80f14cec7e83f/512x393/69f4b48f814dcba62e76f0b0f02ff2a4/fiddler1_%E7%BB%93%E6%9E%9C.png)

* 在 OnBeforeRequest 加入 `oSession.oRequest["Proxy-Authorization"] = "Basic QURcdXNlcm5hbWU6cGFzc3dvcmQ=";`

    ![rule](https://trello-attachments.s3.amazonaws.com/581164a17a360d6e6a151489/1200x272/f12f591ff108ed75146eb41726e42748/fiddler5_%E7%BB%93%E6%9E%9C.png)

## 4. 如何取得 crednetial 內容

* Tools --> TextWizard

    ![STEP4](https://trello-attachments.s3.amazonaws.com/5801e1bda8b80f14cec7e83f/600x383/b90390c44aa8402f078db98c8b989148/fiddler2_%E7%BB%93%E6%9E%9C.png)

* 4-1. 加密
  * 貼上欲加密內容-->`Transfrom` 選 `To Base64` --> click `Send output to input`
  * `AD\username:password`-->`QURcdXNlcm5hbWU6cGFzc3dvcmQ=`

        ![encrypt](https://trello-attachments.s3.amazonaws.com/581164a17a360d6e6a151489/738x451/5646219abc755866eef1ac4c8f0af0e3/fiddler3_%E7%BB%93%E6%9E%9C.png)

* 4-2. 解密
* 貼上欲解密內容-->`Transfrom` 選 `From Base64` --> click `Send output to input`
* `QURcdXNlcm5hbWU6cGFzc3dvcmQ=`-->`AD\username:password`

    ![decrypt](https://trello-attachments.s3.amazonaws.com/581164a17a360d6e6a151489/738x451/ba43e2c15c16be42f6a66b607a292f3f/fiddler4_%E7%BB%93%E6%9E%9C.png)

## 參考資料

1. [stackoverflow](http://stackoverflow.com/questions/2989466/configuring-fiddler-to-use-company-networks-proxy)
