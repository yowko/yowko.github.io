---
title: "如何從無至有建立 React.js 開發環境"
date: 2016-12-29T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["Frontend","JavaScript"]
slug: "build-your-own-react-js-starter"
aliases:
    - /2016/12/Build-Your-Own-React-js-Starter.html
    - /2016/12/build-your-own-react-js-starter
---
# 如何從無至有建立 React.js 開發環境
React.js 有著 facebook 的效能品質保證，快速襲捲前端開發生態，很多嶄新觀念也大大地改變前端開發人員的思維，也因為快速地發展與演化讓 React.js 開發流程一直很多元，結果讓開發 React.js 的進入門檻變高，好像有好多方法可行(e.g.需不需要 jsx 語法、jsx 需要預先編譯嗎、預先編譯要用什麼？ gulp、browserify 還是 webpack+babel?)。


之前在公司分享基礎 React.js 開發時有針對簡化開發環境建置以降低門檻做簡易的比較，順手整理了心得及做法，請大家指教, 而其中最方便的做法是使用 react cli ，接著是 BYOS(Build Your Own Starter)，以下是兩種開發環境建置的流程及步驟

## React.js cli
1. 安裝 create-react-app (擇一即可)
    - `npm install -g create-react-app`
    - `yarn global add create-react-app`
        >yarn 2016/12/25 使用時有 bug , 會出現 `'create-react-app' is not recognized as an internal or external command,operable program or batch file.`

2. 建立 APP create-react-app hello-world

3. 測試(擇一即可)
    - npm
        
        ```
        cd hello-world
        npm start
        ```
    - yarn
        
        ```
        cd hello-world
        yarn start
        ```

## BYOS(Build Your Own Starter)

> - 以下操作主要目的是透過語法彙整來簡化流程，參考於 [Build Your Own Starter](http://andrewhfarmer.com/build-your-own-starter)
> - 建議先看過 [Build Your Own Starter](http://andrewhfarmer.com/build-your-own-starter) 視覺化的網頁介面大致瞭解過程的變化歷程
> - 使用 webpack+babel
> - 適合情境
>     - 簡單的環境設定(React.js cli 已經包裝過，需要客製時相關設定需要花點時間)
>     - 想要建立較小的 package
> - 適用於 windows command prompt


1. 建立工作目錄
    
    ```
    mkdir ReactJS_Demo
    cd ReactJS_Demo
    ```

2. 加入 Git 版控
    
    ```
    git init
    echo node_modules > .gitignore
    echo www/bundle.js >> .gitignore
    git add .
    git commit -m "initial commit"
    ```
3. 加入 npm 套件管理

    ```
    npm init -y
    git add .
    git commit -m "add npm package.json"
    ```

4. 加入 babel

    ```
    npm install --save babel-core
    npm install --save babel-preset-es2015
    npm install --save babel-preset-react
    echo { >> .babelrc
    echo   "presets": ["es2015", "react"] >> .babelrc
    echo } >> .babelrc
    git add .
    git commit -m "add babel"
    ```

5. 加入 webpack

    ``` 
    npm install --save webpack babel-loader
    mkdir src
    echo console.log('Hello World!'); >> src/main.js
    echo  var path = require('path'); >> webpack.config.js
    echo  var config = { >> webpack.config.js
    echo  context: path.join(__dirname, 'src'),>> webpack.config.js
    echo  entry: [>> webpack.config.js
    echo     './main.js',>> webpack.config.js
    echo  ],>> webpack.config.js
    echo output: {>> webpack.config.js
    echo     path: path.join(__dirname, 'www'),>> webpack.config.js
    echo     filename: 'bundle.js',>> webpack.config.js
    echo },>> webpack.config.js
    echo module: {>> webpack.config.js
    echo     loaders: [>> webpack.config.js
    echo     {>> webpack.config.js
    echo         test: /\.js$/,>> webpack.config.js
    echo         exclude: /node_modules/,>> webpack.config.js
    echo         loaders: ['babel'],>> webpack.config.js
    echo     },>> webpack.config.js
    echo     ],>> webpack.config.js
    echo },>> webpack.config.js
    echo resolveLoader: {>> webpack.config.js
    echo     root: [>> webpack.config.js
    echo     path.join(__dirname, 'node_modules'),>> webpack.config.js
    echo     ],>> webpack.config.js
    echo },>> webpack.config.js
    echo resolve: {>> webpack.config.js
    echo     root: [>> webpack.config.js
    echo     path.join(__dirname, 'node_modules'),>> webpack.config.js
    echo     ],>> webpack.config.js
    echo },>> webpack.config.js
    echo };>> webpack.config.js
    echo module.exports = config;>> webpack.config.js
    ```
6. 修改 package.json
    - 6-1. 需手動修改 package.json 檔案的 "scripts" 區段
        
        ``` 
        "scripts": {
        "compile": "webpack",
        "test": "echo \"Error: no test specified\" && exit 1"
        },
        ```
    - 6-12 commit packge.json
        
        ```
        git add .
        git commit -m "use webpack to compile"
        ```
7. 設定測試 server
    - 7-1. 設定使用 3100 port
        
        ``` 
        npm install --save express webpack-dev-middleware
        echo var express = require('express'); > server.js
        echo var webpackDevMiddleware = require('webpack-dev-middleware'); >>server.js
        echo var webpack = require('webpack'); >>server.js
        echo var webpackConfig = require('./webpack.config.js');>>server.js
        echo var app = express(); >>server.js
        echo var compiler = webpack(webpackConfig); >>server.js
        echo app.use(express.static(__dirname + '/www'));>>server.js
        echo app.use(webpackDevMiddleware(compiler, { >>server.js
        echo hot: true,>>server.js
        echo filename: 'bundle.js',>>server.js
        echo publicPath: '/',>>server.js
        echo stats: {>>server.js
        echo     colors: true,>>server.js
        echo },>>server.js
        echo historyApiFallback: true,>>server.js
        echo }));>>server.js
        echo var server = app.listen(3100, function() {>>server.js
        echo var host = server.address().address;>>server.js
        echo var port = server.address().port;>>server.js
        echo console.log('Example app listening at http://%s:%s', host, port);>>server.js
        echo });>>server.js
        mkdir www
        echo ^<html^> >>www/index.html
        echo ^<head^> >>www/index.html
        echo     ^<script src="/bundle.js" ^>^</script^> >>www/index.html
        echo ^</head^> >>www/index.html
        echo ^<body^> >>www/index.html
        echo     Hello World >>www/index.html
        echo ^</body^> >>www/index.html
        echo ^</html^> >>www/index.html
        ```
    - 7-1. 修改 package.json

        * 需手動修改 package.json 檔案的 "scripts" 區段
            
            ```
            "scripts": {
                "compile": "webpack",
            "start": "node server.js",
            "test": "echo \"Error: no test specified\" && exit 1"
                },
            ```
    - 7-2. commit package.json
        
        ```
        git add .
        git commit -m "add server"
        npm start
        ```
    - 7-3. 測試 server [http://localhost:3100](http://localhost:3100)
8. 加入 React

    ```
    npm install --save react react-dom
    echo import React from 'react';>src/Counter.js
    echo /**>>src/Counter.js
    echo * A counter button: tap the button to increase the count.>>src/Counter.js
    echo */>>src/Counter.js
    echo class Counter extends React.Component {>>src/Counter.js
    echo constructor() {>>src/Counter.js
    echo     super();>>src/Counter.js
    echo     this.state = {>>src/Counter.js
    echo     count: 0,>>src/Counter.js
    echo     };>>src/Counter.js
    echo }>>src/Counter.js
    echo render() {>>src/Counter.js
    echo     return (>>src/Counter.js
    echo     ^<button >>src/Counter.js
    echo         onClick={() =^> {>>src/Counter.js
    echo         this.setState({ count: this.state.count + 1 });>>src/Counter.js
    echo         }}>>src/Counter.js
    echo     ^> >>src/Counter.js
    echo         Count: {this.state.count}>>src/Counter.js
    echo     ^</button^>>>src/Counter.js
    echo     );>>src/Counter.js
    echo }>>src/Counter.js
    echo }>>src/Counter.js
    echo export default Counter;>>src/Counter.js
    ```

    - 8-1. 修改 src/main.js
        - 手動修改 src/main.js(程式進入點)
            ``` 
            import React from 'react';
            import ReactDOM from 'react-dom';
            import Counter from './Counter';
            document.addEventListener('DOMContentLoaded', function() {
                ReactDOM.render(
                    React.createElement(Counter),
                    document.getElementById('mount')
                );
            });
            ```
    - 8-2. 修改 www/index.html
        - 手動修改 www/index.html body 區段

            ```html
            <body>
                <div id="mount"></div>
            </body>
            ```
    - 8-3. test 
    
        ```
        npm start
        ```
    - 8-4. commit

        ```
        git add .
        git commit -m "add react"
        ```
## 心得 
經過一連串操作，現在終於建立起 React.js 的開發環境，是不是想用 React.js cli 就好？！繁瑣過程與設定彈性在這個案例是個 trade-off 的關係，就得視實際情況來選擇

# 參考資料
1. [create-react-app](https://github.com/facebookincubator/create-react-app)
2. [Build Your Own Starter](http://andrewhfarmer.com/build-your-own-starter)