---
title: "利用 POSTMAN 中的 collection 來快速測試 application 功能是否正常"
date: 2017-03-07T01:42:34+08:00
lastmod: 2020-12-11T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "postman-collection-runner"
aliases:
    - /2017/03/postman-collection-runner.html
---
# 利用 POSTMAN 中的 collection 來快速測試 application 功能是否正常
無意中聽到 frontend team 的同事在抱怨 frontend 畫面出現錯誤提示，得花點時間查 log 才能發現是 api 有狀況。當下聯想到之前很多專案也都缺乏 api 的監控機制，常見的是 MIS 針對 server 進行 cpu 、 memory、hd 甚至網路流量進行監控，當然有愈來愈多注重服務的團隊會進行 application 等級的心跳監控，但 application 功能愈來愈多，監控項目是不是也隨著增加呢？ 

同事討論的問題發生原因也是這麼來的，application 本體正常運作，但因為新功能 api 未回傳正確資料使得 frontend 部份功能異常，所以出現監控系統回應正常但有些功能卻無法 work 的問題

在跨團隊介接時，postman 的 collection 是個方便的溝通方式(當然 swagger 也是)，今天就來介紹如何利用 postman 的 collection 做基本的測試，可以當做快速定位異常的方法之一

## 準備 collection
將需要基本監控的 request 準備專屬的 collection，當然也可以將開發用的 collection 拿來用，但需要注意不同環境間的參數是否可通用，建議使用參數化寫法，可以參考 [如何在 POSTMAN 中使用參數化寫法來進行不同環境的測試](/2017/03/postman-parameter-test.html), 我就延用其中的範例來進行後續的 demo

1. 逐一將 request 加入 collection 中
    - 按下 `Save` or `Save to collection`
        
        ![1save](https://cloud.githubusercontent.com/assets/3851540/23577622/454ee08e-00ff-11e7-849d-9164944bebba.png) 
        
        ![1savetocollection](https://cloud.githubusercontent.com/assets/3851540/23577623/4551e220-00ff-11e7-8875-a82fb78393fa.png)
    - 填寫基本資料
        - 為 request 命名(使用好辨識的名稱)
        - 加上 request 相關描述(optional)
        - 加入已存在的 collection 或是 建立新的 collection
            
            ![2savetoloeection](https://cloud.githubusercontent.com/assets/3851540/23577625/45522c58-00ff-11e7-8cb6-33558584b429.png)
2. 重複上個步驟直到所需 request 皆加入為止
    
    ![3collection](https://cloud.githubusercontent.com/assets/3851540/23577624/4551e7ac-00ff-11e7-8c4e-80b0f742d9df.png)

## 為每個 request 寫驗證邏輯
沒有寫驗證會直接被判定為 pass ，這樣就失去執行驗證測試的意義了

1. 開啟 request 的 `Tests` tab
2. 依實際情境撰寫 response 驗證邏輯
    - 有 snippets 可以輔助撰寫
    - 驗證方式可以參考 [Testing examples](https://www.getpostman.com/docs/testing_examples)
        
        ![4test](https://cloud.githubusercontent.com/assets/3851540/23577627/4552ed82-00ff-11e7-9fcf-38a6ba19ebe7.png)
3. 測試名稱可以自訂
    - 讓測試結果更一目瞭然 
        
        ![5testname](https://cloud.githubusercontent.com/assets/3851540/23577626/4552e44a-00ff-11e7-8be5-1e482bb32c50.png)
4. 實際範例
    - 指定環境變數
        
        >`postman.setEnvironmentVariable("Id", "3");` 
    - 驗證 response 的 http status code 為 200
        
        >`tests["Status code is 200"] = responseCode.code === 200;` 
    - 驗證 response body 中包含 "3"
        
        >`tests["Body matches string"] = responseBody.has("3");`
    - 驗證 response body 等於 "3"
        
        >`tests["Body is correct"] = responseBody ==='"3"';` 
    - 驗證 response body 的內容是一個字串
        
        ```js
        var _id=postman.getEnvironmentVariable("Id");
        var _return="post_return:"+_id
        var jsonData = JSON.parse(responseBody);
        tests["Body is correct"] = jsonData === _return;
        ```

* 可以透過 console.log 的方式來為撰寫的 test 偵錯，設定方式請參考 [如何對 POSTMAN desktop app 偵錯](/2017/03/debug-postman-test.html)
    

## 使用 postman runner 來執行測試
1. 開啟 postman Collection Runner
    
    ![6runner](https://cloud.githubusercontent.com/assets/3851540/23577628/4573829a-00ff-11e7-8e6d-0000a6862c43.png) 
2. 設定 Runner
    - 選擇要執行的 collection
    - 指定環境變數
    - 指定執行次數
    - 指定每個 request 間的間隔毫秒數
    - 指定資料來源 可以使用 CSV/JSON
        
        ![7runnersetting](https://cloud.githubusercontent.com/assets/3851540/23577630/4579e252-00ff-11e7-8aeb-16f11db54fa5.png) 
3. 按下 Start Test 開啟測試

## 測試結果
![7RESULT](https://cloud.githubusercontent.com/assets/3851540/23577629/45772bf2-00ff-11e7-95dc-b778c98797be.png)


# 參考資料
1. [Running a collection](https://www.getpostman.com/docs/running_collections)
2. [Testing examples](https://www.getpostman.com/docs/testing_examples)
3. [如何對 POSTMAN desktop app 偵錯](http://blog.yowko.com/2017/03/debug-postman-test.html)