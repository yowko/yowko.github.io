---
title: "將不同瀏覽器的上傳元件(input type='file')樣式統一"
date: 2017-01-25T00:42:34+08:00
lastmod: 2018-09-10T00:42:34+08:00
draft: false
tags: ["Frontend","Html"]
slug: "customize-file-input-style"
aliases:
    - /2017/01/customize-file-input-style.html
---
# 將不同瀏覽器的上傳元件(input type='file')樣式統一
最近在公司舊系統上做了一個 Excel 上傳用來新增資料的功能，公司 QA 測試時發現不同瀏覽器有不一樣的顯示。聽到這樣的描述，大家一定也馬上聯想到是瀏覽器預設樣式所造成的，解法就是載入`Reset css`，搞定？！立馬找個 CDN 加到專案確認一下，果然我的上傳元件統一樣式了，但網站其他東西卻大跑版@@"

馬上理解到系統上原本就沒用 `Reset css`，連 `Reset css` 都沒有，果然也沒有 `bootstrap` ，只能手刻，立馬來回味一下全手工 html 與 css 的傳統風格吧

## 原本樣式不同
- 線上範例：[jsbin](http://jsbin.com/xuzeqopuce/edit?html,css,js,output)

    ```html
    <div class="x-form-item">
    	<label class="inputlabel">Choose a file:</label>
    	<div class="x-form-element">
    		<input id="file" name="file" class="x-form-text x-form-field inputtext {required:true}" size="40" type="file">
    		<button id="submitForm" style="vertical-align:middle;">Submit</button>
    	</div>
    </div>
    ```


1. IE 11
    
    ![IE11](https://cloud.githubusercontent.com/assets/3851540/22191361/e517ec9c-e164-11e6-9bda-44f6491e827e.png)

2. Chrome
    
    ![CHROME](https://cloud.githubusercontent.com/assets/3851540/22191357/e4db23de-e164-11e6-8e77-bbbac2c15f37.png)

3. firefox
    
    ![firefox](https://cloud.githubusercontent.com/assets/3851540/22191360/e5178c98-e164-11e6-8ac4-33301dd44d1b.png)


## 修改後樣式

> 客製不是好解法，但我想以後要遇到這樣問題的機會應該也不多，個人覺得應該複用經過公開驗證的元件比自己刻來得好

- 在畫面上放個透明的 `<input type="file"/>` 再來用 `textbox` 跟 `button` 來統一畫面
- 線上範例：[jsbin](http://jsbin.com/yagohizuci/1/edit?html,css,js,output)

    ```html
    <style>
    .filediv{ width:500px; height:25px;  position:relative;}
    #filetxt{position:absolute;right:100px;height:19px!important;width:173px}
    #filebtn{ position:absolute; left:155px;width:70px;height:25px; }
    #file{ width:245px; position:absolute; left:155px; top:0px; opacity:0; outline: none;height: 25px;}
    #submitForm{position:absolute;right:40px;width:60px;height:25px;}
    </style>
    <div class="x-form-item filediv">
    	<label class="inputlabel fileContainer">Choose a file:</label>
    	<button id="filebtn" >SELECT</button>
    	<input id="filetxt" type="text" value="  please choose a file" />
        <input id='file' type="file" name="file" onfocus="this.blur()" class="x-form-text x-form-field inputtext {required:true}" size="40"/>
    	<button id="submitForm" style="vertical-align:middle;">Submit</button>
    </div>
    <script type="javascript/text">
    $('#file').change(function(){
    				$('#filetxt').val($('#file').val());
    			});
    </script>
    ```

1. IE 11
    
    ![IE11](https://cloud.githubusercontent.com/assets/3851540/22191362/e519d5de-e164-11e6-978e-5c7e98c8f4bd.png)

2. Chrome
    
    ![CHROME](https://cloud.githubusercontent.com/assets/3851540/22191358/e4ffbbc2-e164-11e6-8d08-159e573da610.png)

3. firefox
    
    ![firefox](https://cloud.githubusercontent.com/assets/3851540/22191359/e517869e-e164-11e6-9f94-0073a912b99e.png)


# 參考資料
1. [自訂 input type="file" 格式](http://audi.tw/blog/css/input.type=file.asp)