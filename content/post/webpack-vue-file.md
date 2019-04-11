---
title: "使用 Webpack 來編譯 Vue.js Single File Components (.vue)"
date: 2017-11-12T22:24:00+08:00
lastmod: 2018-09-28T22:24:18+08:00
draft: false
tags: ["套件","Frontend","JavaScript"]
slug: "webpack-vue-file"
aliases:
    - /2017/11/webpack-vue-file.html
---
# 使用 Webpack 來編譯 Vue.js Single File Components (.vue)
最近因為新專案的部份頁面有較多的互動功能，所以打算透過 Vue.js 來處理，其中某些功能在多個頁面間可以共用，就使用了 Vue.js 的 Single File Components (.vue) - 單一檔案元件來模組化

只是要使用 .Vue 檔案需要經過編譯，雖然可以透過 http-vue-loader (詳細內容請參考 Kuro 文章 - [不需編譯也能載入 .vue 元件檔: 使用 http-vue-loader](https://kuro.tw/posts/2017/07/11/%E4%B8%8D%E9%9C%80%E7%B7%A8%E8%AD%AF%E4%B9%9F%E8%83%BD%E8%BC%89%E5%85%A5-vue-%E5%85%83%E4%BB%B6%E6%AA%94-%E4%BD%BF%E7%94%A8-http-vue-loader/))做到直接在網頁載入 .Vue 檔案，不過效能上還是會有耗損的情況，因此還是透過正統方式來編譯及打包

之前用到前端框架的專案，都有強大的前端工程師支援，後端工程師的工作大多寫寫 API 提供資料而已，對於編譯打包的設定多數不了解，我也是如此XD，雖然曾經設定過，但時間久遠加上前端工具日新月異，所以趁這個機會好好做個紀錄

## 基本環境

1.  建立新工作目錄

    ```
    mkdir vuetest
    ```

2.  使用新工作目錄

    ```
    cd vuetest
    ```

3.  加入使用 npm

    ```
    npm init -y
    ```

    > 會自動加入 `package.json` 並產生基本內容

4.  安裝 vue.js (最後需要輸出)

    ```
    npm install vue -P
    ```

    > `-P` 是預設值可以不加，會加至 `dependencies` 其他還有 `-D` - 只在 dev(devDependencies) 跟 `-O`(optionalDependencies)

5.  安裝相關套件

    ```
    npm install webpack babel-loader babel-core css-loader vue-loader vue-template-compiler -D
    ```

## 建立網頁並加入 vue.js 功能 (用來確認 vue.js 功能正常 - 可略過)

1.  加入 `index.html` 檔案

    ```html
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
            <div class="message">
                {{ message }}
            </div>
        </div>
    </body>
    </html>
    ```

2.  引用 vue.js

    ```html
    <script src="./node_modules/vue/dist/vue.js"></script>
    ```

3.  撰寫 vue.js 功能

    ```html
    <script type="text/javascript">
        new Vue({
            el: '#app',
            data: {
                message: "Hello Vue"
            }
        })
    </script>
    ```

4.  index.html 完整程式碼

    ```html
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
            <div class="message">
                {{ message }}
            </div>
        </div>
        <script src="./node_modules/vue/dist/vue.js"></script>
        <script type="text/javascript">
            new Vue({
                el: '#app',
                data: {
                    message: "Hello Vue"
                }
            })
        </script>
    </body>
    </html>
    ```

## 將 vue.js 功能抽出 html (用來逐步製作 .vue 中 - 可略過)

1.  建立 index.js

    ```js
    new Vue({
        el: '#app',
        data: {
            message: "Hello Vue"
        }
    })
    ```

2.  index.html 改引用 index.js

    ```html
    <script src="./src/index.js"></script>
    ```

## 統一管理 javascript (用來逐步製作 .vue 中可略過)

> javascript 散落各處不好管理

1.  修改 index.js 加入引用 vue

    ```js
    import Vue from 'vue'
    new Vue({
        el: '#app',
        data: {
            message: "Hello Vue123"
        }
    })
    ```

2.  設定 webpack 編譯 index.js

    *   根目錄加入 `webpack.config.js`
    *   設定 `webpack.config.js`

        ```js
        "use strict";
        module.exports = {
            entry: "./src/index.js",//指定要編譯的檔案位置
            output: {
                filename: "./dist/build.js"//指定編譯後結果檔案位置
            },
            module: {
                loaders: [
                    {
                        test: /\.js$/,
                        loader: "babel-loader",//使用 babel-loader 來編譯 .js
                        exclude: /node_modules/
                    }
                ]
            },
            resolve: {
                extensions: ['.js', '.vue'],
                alias: {
                    'vue':'vue/dist/vue.js',//指定 vue 對應使用的真實 js 檔案
                }
            }
        };
        ```

3.  執行編譯

    > webpack

    ![1webpack](https://user-images.githubusercontent.com/3851540/32699865-5bfb316a-c7f7-11e7-9ea2-bbe974117f0d.png)

4.  修改 js 引用

    *   移除 index.html 相關 js 引用
    *   加入編譯後檔案引用

        ```html
        <script src="./dist/build.js"></script>
        ```

    *   完整 index.html

        ```html
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
                <div class="message">
                    {{ message }}
                </div>
            </div>
            <script src="./dist/build.js"></script>
        </body>
        </html>
        ```

## 使用 .vue 來模組輸出 (重點在此)

1.  建立 `msg.vue`

    > 將 html、javascript、css 部份全部移至 `msg.vue`

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

2.  在 index.js 加入引用 msg.vue 並指定 components

    ```js
    import Vue from 'vue'
    import Msg from './msg.vue'
    new Vue({
        el: '#app',
        components:{Msg}
    })
    ```

3.  webpack.config.js 加入編譯 `.vue` 設定
    *   module --> loaders 加入 vue-loader

        ```
        {
            test: /\.vue$/,
            loader: 'vue-loader'//使用 babel-loader 來編譯 .vue
        }
        ```

    *   webpack.config.js 完整版

        ```js
        "use strict";
        module.exports = {
            entry: "./src/index.js",
            output: {
                filename: "./dist/build.js"
            },
            module: {
                loaders: [
                    {
                        test: /\.js$/,
                        loader: "babel-loader",
                        exclude: /node_modules/
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
                    'vue':'vue/dist/vue.js',
                }
            }
        };
        ```

4.  index.html 使用自訂 tag

    ```html
    <div id="app">
    <msg></msg>
    </div>
    ```

    *   index.html 完整程式碼

        ```html
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
            <script src="./dist/build.js"></script>
        </body>
        </html>
        ```

5.  重新編譯

    > webpack

6.  完整專案結構

    ![2folderstructure](https://user-images.githubusercontent.com/3851540/32699866-5c2efd56-c7f7-11e7-81b7-d4807d059a3b.png)

## 心得

回顧起來內容不多，但可是花了我不少時間找解法也踩了不少雷才終於完整，當然難度來自於不熟悉，只是再次印證我堅信的名言：`難就是難在不會，會就不難了`，經過這次整理對一些細節比較有印象也比較清楚整個流程，只是我覺得前端工程真的滿辛苦，這真的應該力行專業分工，不然很難做得好做得快

# 參考資訊

1.  [不需編譯也能載入 .vue 元件檔: 使用 http-vue-loader](https://kuro.tw/posts/2017/07/11/%E4%B8%8D%E9%9C%80%E7%B7%A8%E8%AD%AF%E4%B9%9F%E8%83%BD%E8%BC%89%E5%85%A5-vue-%E5%85%83%E4%BB%B6%E6%AA%94-%E4%BD%BF%E7%94%A8-http-vue-loader/)
2.  [npm-install](https://docs.npmjs.com/cli/install)
3.  [vue + webpack 起手式](https://segmentfault.com/a/1190000005363030)
4.  [Single File Components](https://vuejs.org/v2/guide/single-file-components.html)
