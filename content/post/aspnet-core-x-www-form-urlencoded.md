---
title: "讓 ASP.NET Web API 與 ASP.NET Core 可以支援 x-www-form-urlencoded"
date: 2018-10-01T02:44:00+08:00
lastmod: 2021-11-03T02:44:00+08:00
draft: false
tags: ["ASP.NET Core","ASP.NET Web API"]
slug: "aspnet-core-x-www-form-urlencoded"
---
## 讓  ASP.NET Web API 與 ASP.NET Core 可以支援 x-www-form-urlencoded

最近某個專案需要將資料(前端)傳送至其他平台 API 進行處理，跟一般需求相同，只是資料傳輸格式不是使用過去較熟悉的 `application/json` 而是使用 `application/x-www-form-urlencoded`

雖然 `application/x-www-form-urlencoded` 是 POST 的預設格式，但單純 key-value 的格式在複雜情境下不免捉襟見肘，所以過去絕大部份專案都使用 `application/json`

用慣了 `application/json` 遇到 `application/x-www-form-urlencoded` 卻換成是我捉襟見肘了XD 為了確認負責的前端專案可以正確與外部平台串接，於是嘗試架設可以用來 debug 的 API ，有些設定沒有用過，順手筆記一下

## 前提設定

1. POST model

    ```cs
    public class loginmodel
    {
        public string username { get; set; }
        public string password { get; set; }
        public string type { get; set; }
    }
    ```

2. 使用 postman 進行測試

    > 以下為 postman 產出的 curl 內容，url 記得換為目標 api url

    ```cmd
    curl -X POST \
    {url} \
    -H 'Content-Type: application/x-www-form-urlencoded' \
    -d 'username=yowko&password=password&type=basic'
    ```

## ASP.NET Web API

> 沒有特殊設定，只要直接建立 controller 以及 action 即可直接使用

1. 程式碼

    > 參數屬性 `[FromBody]` 加不加都可以正確 binding

    ```cs
    public class ValuesController : ApiController
    {
        [HttpPost]
        //public string Post([FromBody]loginmodel data)
        public string Post(loginmodel data)
        {
            var value = data;
            return $"{Request.Content.Headers.ContentType}_{value.username}_{value.password}_{value.type}";
        }
    }
    ```

2. 實際結果
    * 使用 `application/json`

        > 可正確取得資料

        ![1webapijson](https://user-images.githubusercontent.com/3851540/46751630-aad1ed00-cced-11e8-8fc0-db1b9b07a37f.png)
    * 使用 `application/x-www-form-urlencoded`

        > 可正確取得資料

        ![2webapiurlencode](https://user-images.githubusercontent.com/3851540/46751631-ab6a8380-cced-11e8-90cf-f85fc8ae86b1.png)

## ASP.NET Core

 > `[FromForm]` 參數屬性是主要的重點

1. 程式碼

    > `[Consumes("application/x-www-form-urlencoded")]` 加不加都不影響 binding，但加上後在遇到錯誤 content-type 時回應較明確

    ```cs
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        [HttpPost]
        [Consumes("application/x-www-form-urlencoded")]
        public string Post([FromForm]loginmodel data)
        {
            return $"{Request.Headers["Content-Type"]}_{data.username}_{data.password}_{data.type}";
        }
    }
    ```

2. 實際結果
    * 使用 `application/json` 且未加上 `[Consumes("application/x-www-form-urlencoded")]`

        > 無法正確取得資料

        ![3corejosnempty](https://user-images.githubusercontent.com/3851540/46751632-ab6a8380-cced-11e8-8cd7-9c0dc3377a87.png)

    * 使用 `application/json` 並加上 `[Consumes("application/x-www-form-urlencoded")]`

        > 無法正確取得資料並得到 `415 Unsupported Media Type` 錯誤

        ![4corejson415](https://user-images.githubusercontent.com/3851540/46751633-ab6a8380-cced-11e8-95c0-01e395402c51.png)

    * 使用 `application/x-www-form-urlencoded`

        > 可正確取得資料

        ![5coreurlencoded](https://user-images.githubusercontent.com/3851540/46751634-ac031a00-cced-11e8-9dd5-574c7ab37d9d.png)

## 心得

ASP.NET Web API 太方便了呀，什麼設定都不用就可以通吃

ASP.NET Core 雖然相對麻煩了些，但讓實際開發人員處理這類細微差異反而更能讓程式碼有更正確的回應，設計理念比較符合現代思維

以這次的經驗來說，預期應該只有 `application/x-www-form-urlencoded` 的 request 可以通過資料傳送

* ASP.NET Web API 卻自行轉換
* ASP.NET Core 沒有加上 `[Consumes("application/x-www-form-urlencoded")]` 的版本還無法正確 binding 資料

## 參考資訊

1. [Accept x-www-form-urlencoded in Web API .Net Core](https://stackoverflow.com/a/49042444/3600583)
2. [ASP.NET Web API 中傳送 HTML 表單資料： form-urlencoded 資料](https://docs.microsoft.com/zh-tw/aspnet/web-api/overview/advanced/sending-html-form-data-part-1?WT.mc_id=DOP-MVP-5002594)
