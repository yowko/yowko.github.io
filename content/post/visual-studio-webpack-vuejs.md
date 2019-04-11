---
title: "在 Visual Studio 中使用 WebPack 來編譯 Vue.js"
date: 2017-11-13T02:45:00+08:00
lastmod: 2018-09-28T02:45:38+08:00
draft: false
tags: ["套件","Frontend","JavaScript","Visual Studio"]
slug: "visual-studio-webpack-vuejs"
aliases:
    - /2017/11/visual-studio-webpack-vuejs.html
---
# 在 Visual Studio 中使用 WebPack 來編譯 Vue.js
筆記 [使用 Webpack 來編譯 Vue.js Single File Components (.vue)](https://blog.yowko.com/2017/11/webpack-vue-file.html) 紀錄著使用 webpack 來編譯 .vue 檔案的做法，而我平常主要還是使用 Visual Studio 開發，所以還是得將 WebPack 及 Visual Studio 做個整合，來看看該如何設定吧

## 基本設定

1.  安裝 vue

    *   專案上按右鍵 --> Quick Install Package...

        ![1quickinstall](https://user-images.githubusercontent.com/3851540/32702052-c7162f4e-c81b-11e7-9099-964fb620c75a.png)

    *   選擇安裝工具 --> 輸入 `vue` --> 預設會加上 `--save-dev`

        ![2installvue](https://user-images.githubusercontent.com/3851540/32702053-c740344c-c81b-11e7-98b2-61f8c95b63bf.png)

    *   工具有其他選擇

         ![3othertool](https://user-images.githubusercontent.com/3851540/32702054-c769370c-c81b-11e7-9758-908a6de96eeb.png)

    *   預設 `--save-dev` 可以編輯修改的

         ![4change](https://user-images.githubusercontent.com/3851540/32702056-c793127a-c81b-11e7-9559-9b3a48bdcf66.png)

2.  安裝其他相關套件

    *   專案上按右鍵 --> Quick Install Package...

        ![1quickinstall](https://user-images.githubusercontent.com/3851540/32702052-c7162f4e-c81b-11e7-9099-964fb620c75a.png)

    *   選擇安裝工具 --> 輸入 `webpack babel-loader babel-core css-loader vue-loader vue-template-compiler`

        ![5otherpackage](https://user-images.githubusercontent.com/3851540/32702057-c7bb5ba4-c81b-11e7-8912-98e86c1207e4.png)

## 安裝 WebPack Task Runner

1.  Visual Studio 主選單 Tools --> Extensions and Updates...

    ![6extension](https://user-images.githubusercontent.com/3851540/32702058-c7e3a5b4-c81b-11e7-96e6-a8fdeee73282.png)

2.  Online tab --> 搜尋 `WebPack Task Runner` --> 安裝

    ![7webpackrunner](https://user-images.githubusercontent.com/3851540/32702059-c80ab4ce-c81b-11e7-96dd-691409d44a42.png)

## 加入網頁及 js

1.  HomeController --> index

    ```html
    @{
        Layout = null;
    }
    
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Vue.js webpack .vue</title>
    </head>
    <body>
    <div id="app">
        <msg></msg>
    </div>
    <script src="~/Scripts/dist/build.js"></script>
    </body>
    </html>
    ```

2.  在 `Scripts/src` 資料夾中加入 `msg.vue`

    ```html
    <template lang="html">
        <div class="message">
            {{ message }}
        </div>
    </template>
            
    <script>
    export default {
    data () {
        return {
        message: 'Helo, .vue file'
        }
    }
    }
    </script>
    
    <style lang="css">
        .message {
            color: blue;
        }
    </style>
    ```

3.  在 `Scripts/src` 資料夾中加入 `index.js`

    ```js
    import Vue from 'vue'
    import Msg from './msg.vue'
    new Vue({
        el: '#app',
        components: { Msg }
    })
    ```

## 設定 WebPack

1.  在網站根目錄加入 `webpack.config.js`
    *   專案上按右鍵 --> Add --> New Item...

        ![8newitem](https://user-images.githubusercontent.com/3851540/32702060-c83635ea-c81b-11e7-9c01-7aac44a44ec8.png)

    *   Web --> 選擇 WebPack Configuration File

        ![9addwebpackconfig](https://user-images.githubusercontent.com/3851540/32702062-c8831b58-c81b-11e7-9101-5cc0625b386e.png)

2.  設定 `webpack.config.js` 編譯內容

    ```js
    "use strict";
    
    module.exports = {
        entry: "./Scripts/src/index.js",
        output: {
            filename: "./Scripts/dist/build.js"
        },
        module: {
            loaders: [
                {
                    test: /\.js?$/,
                    loader: "babel-loader"
                },
                {
                    test: /\.vue$/,
                    loader: 'vue-loader'
                }
            ]
        },
        resolve: {
            extensions: ['.js', '.vue'],
            alias: {
                'vue': 'vue/dist/vue.js',
            }
        }
    };
    ```

## 執行編譯

1.  開啟 Task Runner Explorer

    *   Visual Studio 主選單 View --> Other Windows --> Task Runner Explorer

        ![10taskrunner](https://user-images.githubusercontent.com/3851540/32702063-c8aa278e-c81b-11e7-862c-2c8d572cc5cb.png)

2.  Run - Development 上按右鍵 --> Run

    ![11run](https://user-images.githubusercontent.com/3851540/32702064-c8d17df2-c81b-11e7-9951-297bb6fca0c7.png)

3.  編譯成功

    > code 0 為成功

    ![12buildpass](https://user-images.githubusercontent.com/3851540/32702065-c8f94666-c81b-11e7-8aa4-4d5e62fcf0be.png)

## 心得

經過之前筆記 [使用 Webpack 來編譯 Vue.js Single File Components (.vue)](https://blog.yowko.com/2017/11/webpack-vue-file.html) 的經驗，在 Visual Studio 上的設定並沒有遇到太多問題，對於前端框架及工具的支援度很好，只是網路上相對介紹文章略少，如果沒有之前 [使用 Webpack 來編譯 Vue.js Single File Components (.vue)](https://blog.yowko.com/2017/11/webpack-vue-file.html) 的經驗，我想一定又少不了一番折騰

另外 Task Runner Explorer 還有一些設定可以調整，就待日後用到再紀錄囉

# 參考資訊

1.  [How To Integrate Webpack Into Visual Studio 2015](https://sochix.ru/how-to-integrate-webpack-into-visual-studio-2015/)
2.  [使用 Webpack 來編譯 Vue.js Single File Components (.vue)](https://blog.yowko.com/2017/11/webpack-vue-file.html)
3.  [Visual Studio 2015 工作執行器 (Task Runners) 簡介](https://blogs.msdn.microsoft.com/msdntaiwan/2016/01/06/visual-studio-2015-task-runners/)
