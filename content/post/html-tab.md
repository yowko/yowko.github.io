---
title: "在網頁中按 tab 鍵，為什麼亂跳？！"
date: 2016-12-08T00:42:34+08:00
lastmod: 2018-09-05T00:42:34+08:00
draft: false
tags: ["Frontend","Html"]
slug: "html-tab"
aliases:
    - /2016/12/tab.html
---
# 在網頁中按 tab 鍵，為什麼亂跳？！
網路輸入資料也許每天都要發生個好幾次，在輸入表單上按下`tab`鍵讓輸入焦點可以轉移到下一個輸入框，需要修改時按下`shift+tab`也可以回到上一個輸入框，這節省了我們需要在鍵盤與滑鼠間轉換的時間，也讓我們看起來專業許多(誤XD)，但有時候難免會遇到按下`tab`鍵，輸入框卻亂跳，要多按幾下才會到預期的位置，我們就來看看該如何正確設定吧

# 哪些 html element 是 `tab` 的作用範圍
1. 有 `href` 屬性的 element
    - 1-1. [HTMLAnchorElement](http://www.w3schools.com/tags/tag_a.asp)
        
        > 就是 超連結

    - 1-2. [HTMLAreaElement](http://www.w3schools.com/tags/tryit.asp?filename=tryhtml_areamap)
        
        > image map(圖片定位)的超連結

2. 表單項目
會跳過設定為 `disabled` 的element

    - 2-1. [HTMLInputElement](http://www.w3schools.com/TAgs/tag_input.asp)
        
        > 輸入項
    - 2-2. [HTMLSelectElement](http://www.w3schools.com/TAgs/tag_select.asp) 
        
        > 下接選單
    - 2-3. [HTMLTextAreaElement](http://www.w3schools.com/TAgs/tag_textarea.asp) 
        
        > 跨行輸入框
    - 2-4. [HTMLButtonElement](http://www.w3schools.com/TAgs/tag_button.asp) 
        
        > 按鈕

3. HTMLIFrameElement
 iframe 內的 element 順序與其他相同

    - [HTMLIFrameElement](http://www.w3schools.com/TAgs/tag_iframe.asp)
        
        > iframe


4. 影音相關
會先選取到 audio/video element，接著會到 控制項目

    - 4-1. [HTMLVideoElement](http://www.w3schools.com/TAgs/tag_video.asp)
    - 4-2. [HTMLAudioElement](http://www.w3schools.com/TAgs/tag_audio.asp)

5. 其他有 `tabindex` 的 element
tabindex < 0 會略過

# 順序
1. 在有 `tabindex` >0 的 element 上 按`tab` 鍵
    - 會依 `tabindex` 順序移動
    - `tabindex` 結束，會依 element 在 html 上的順序，由上而下 、由左至右
    - `tabindex` =0 的 element ，就如同未設定，不會列入 `tabindex` 的順序
    - `tabindex` < 0 的 element ，會跳過

2. 尚未選取特定元件
    - element 在 html 上的順序，由上而下 、由左至右

# 測試用 [jsbin](http://jsbin.com/qeluxen/1/edit?html,css,console,output)

# 參考資料
1. [HTML <area> Tag](http://www.w3schools.com/tags/tag_area.asp)
2. [Which HTML elements can receive focus?](http://stackoverflow.com/questions/1599660/which-html-elements-can-receive-focus)