---
title: "如何對 POSTMAN desktop app 偵錯"
date: 2017-03-07T01:42:34+08:00
lastmod: 2018-09-13T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "debug-postman"
aliases:
    - /2017/03/debug-postman-test.html
---
# 如何對 POSTMAN desktop app 偵錯
最近剛好在撰寫 postman 的 test，過程當然不像文章上寫的那麼容易XD，所以就需要 debug 寫的 test，猜想應該也會有其他人需要，不然就當做之後重灌時用來節省時間的個人筆記也不賴，紀錄一下該如何使用

## 1. 開啟 chrome 的設定頁面
* 在 chrome 網址列輸入 `chrome://flags` 按下 enter
    
    ![1setting](https://cloud.githubusercontent.com/assets/3851540/23577197/4da7d160-00f4-11e7-9947-a1656aea5796.png)

## 2. 搜尋 `packed`
* 按下 `ctrl+f` 尋找 `debug-packed-apps` 的設定
* 中文是 `封裝應用程式偵錯` 
    
    ![2packed](https://cloud.githubusercontent.com/assets/3851540/23577196/4da7b162-00f4-11e7-93d4-5cf607ff9b7c.png)

## 3. 啟用 `封裝應用程式偵錯` 
* 按下 `啟用` 需重新啟動 chrome 才會生效
    
    ![3enable](https://cloud.githubusercontent.com/assets/3851540/23577198/4da81f1c-00f4-11e7-9243-b84cc764f304.png)

## 效果
* 啟用後在 postman 中隨處按右鍵就會出現相關選單
    
    ![4result](https://cloud.githubusercontent.com/assets/3851540/23577199/4dace5f6-00f4-11e7-8107-a11abf2d609e.png)

## 心得
* postman desktop app 也是用 electron 包的
* postman 官方的快捷鍵 (ctrl+alt+c) 沒有作用
* 右鍵選單的快捷鍵 (ctrl+shift+i) 也沒有作用


# 參考資料
1. [The Postman Console](http://blog.getpostman.com/2016/08/26/the-postman-console/)
2. [Enabling Chrome Developer Tools inside Postman](http://blog.getpostman.com/2014/01/27/enabling-chrome-developer-tools-inside-postman/)