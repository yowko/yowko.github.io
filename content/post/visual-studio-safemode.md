---
title: "Visual Studio 的無痕模式"
date: 2016-12-04T00:42:34+08:00
lastmod: 2018-09-05T00:42:34+08:00
draft: false
tags: ["Visual Studio"]
slug: "visual-studio-safemode"
aliases:
    - /2016/12/visual-studio-safe-mode.html
    - /2016/12/visual-studio-safemode
---
# Visual Studio 的無痕模式
>Visual Studio SafeMode (乾淨模式)，只載入預設環境及服務，不載入第三方套件

有遇過下面的狀況嗎？

1. 同樣的 OS 、同一個版本的 Visual Studio、同一個專案，你開啟專案時會遇到一些奇怪的問題(ex.不能 build、不能 逐步偵錯....etc)，但你隔壁的同事卻順暢無比？！雖然不願意承認，但卻也讓你默默地懷疑起是不是自己的人品出了問題呢？！
2. 按下 Visual Studio 捷徑，過了好久卻還是開不到 Visual Studio 首頁？！ 讓你考慮是不是該重灌了嗎？！


如果出現以上狀況，你可以先試著確認是不是 Visual Studio plugin 造成的


# Visual Studio 2013 開啟 SafeMode

## 開啟方式
1. command line
    ``` 
	C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\devenv.exe /safemode
    ```
    ![commnad](https://cloud.githubusercontent.com/assets/3851540/23978898/5e689368-0a31-11e7-825e-bde23dfc24e4.png)

2. 修改捷徑的目標
    - 捷徑圖示右鍵 --> 在 `Visual Studio 2013` 按右鍵 --> 選 `properties` --> 在 `Target` 最後面加上 `/safemode`
    
        ![target](https://cloud.githubusercontent.com/assets/3851540/23978783/d4f68158-0a30-11e7-8615-338658d8afe2.png)

        ![properties](https://cloud.githubusercontent.com/assets/3851540/23978785/d4fb73d4-0a30-11e7-8d7d-d77c8a5d610d.png)

    - <font style="color:red">這會造成以後開啟 Visual Studio 都進入 `SafeMode`</font>，記得測完要改回來

3. 由捷徑圖示右鍵選單
    - 捷徑圖示右鍵 --> 點選 `Safe Mode`
    
        ![rightclick](https://cloud.githubusercontent.com/assets/3851540/23978784/d4f8eace-0a30-11e7-9d99-69ef7ff5cb02.png)

## 效果比較
1. 一般模式
    
    ![2013normalmode](https://cloud.githubusercontent.com/assets/3851540/23978790/d51b8c46-0a30-11e7-9d6a-36d386c85893.png)

2. SafeMode
    - 套件上顯示 `[Disabled]`，有些套件顯示顏色也不同
    
        ![2013safemode](https://cloud.githubusercontent.com/assets/3851540/23978791/d5203192-0a30-11e7-9942-93069e656384.png)

# Visual Studio 2015 開啟 SafeMode
## 開啟方式
1. command line
    ``` 
	C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\devenv.exe /safemode
    ```
    
    ![commnad](https://cloud.githubusercontent.com/assets/3851540/23978899/5e6bf328-0a31-11e7-8176-f45836b4de76.png)

2. 修改捷徑的目標
    - 捷徑圖示右鍵 --> 在 `Visual Studio 2015` 按右鍵 --> 選 `properties` --> 在 `Target` 最後面加上 `/safemode`

    ![target](https://cloud.githubusercontent.com/assets/3851540/23978788/d500168c-0a30-11e7-9757-a8186fdb1ab3.png)

    ![properties](https://cloud.githubusercontent.com/assets/3851540/23978787/d4ff4306-0a30-11e7-8e6e-742b5cf86c6b.png)

    - <font style="color:red">這會造成以後開啟 Visual Studio 都進入 `SafeMode`</font>，記得測完要改回來


## 效果比較

1. 一般模式
    ![2015normalmode](https://cloud.githubusercontent.com/assets/3851540/23978786/d4fd495c-0a30-11e7-9979-b6cb2c5d008e.png)

2. SafeMode
    - 套件上顯示 `[Disabled]`，有些套件顯示顏色也不同
    
        ![2015safemode](https://cloud.githubusercontent.com/assets/3851540/23978789/d51a7d24-0a30-11e7-938b-e6e829754d67.png)




# 參考資料
1. [Devenv Command Line Switches](https://msdn.microsoft.com/en-us/library/ms241278.aspx)
2. [Visual Studio Safe Mode](https://blogs.msdn.microsoft.com/zainnab/2010/11/15/visual-studio-safe-mode/)