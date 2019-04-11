---
title: "WebPack 在 Visual Studio 無法正確編譯，在 Command Prompt 卻正常"
date: 2017-11-14T23:21:00+08:00
lastmod: 2018-09-28T23:21:36+08:00
draft: false
tags: ["Frontend","Tools","Visual Studio"]
slug: "webpack-error-in-visual-studio"
aliases:
    - /2017/11/webpack-error-in-visual-studio.html
---
# WebPack 在 Visual Studio 無法正確編譯，在 Command Prompt 卻正常
在之前筆記 [在 Visual Studio 中使用 WebPack 來編譯 Vue.js](https://blog.yowko.com/2017/11/visual-studio-webpack-vuejs.html) 紀錄到如何在 Visual Studio 使用 WebPack 來編譯 Vue.js，設定上步驟並不多，使用上也相當方便，只是又踩到雷了XD 不過我在家中電腦與公司電腦都有遇到相同問題，而且公司電腦還遇到兩次一樣的狀況(應該不算是人品問題吧 >"<)：Visual Studio 的 WebPack 無法順利完成編譯，查了好久，也試著把有專案重建 都無法解決問題，後來才發現在 Command Prompt 中是正常的 @@"

就來看看問題發生原因跟解決方式吧

## 錯誤訊息

1.  訊息內容

    ```
    C:\Users\YowkoTsai\Documents\Visual Studio 2017\Projects\TestWebpack\TestWebpack> cmd /c SET NODE_ENV=development&& webpack --color --display-error-details
    Hash: d15ba0238639a7c76803
    Version: webpack 3.8.1
    Time: 793ms
                    Asset    Size  Chunks                    Chunk Names
    ./Scripts/Build/dist.js  301 kB       0  [emitted]  [big]  main
    [0] (webpack)/buildin/global.js 488 bytes {0} [built]
    [1] ./Scripts/VueSrc/main.js 221 bytes {0} [built]
    [6] ./Scripts/VueSrc/app.vue 2.74 kB {0} [built] [failed] [1 error]
        + 4 hidden modules
    ERROR in ./Scripts/VueSrc/app.vue
    Module build failed: SyntaxError: Unexpected token {
        at exports.runInThisContext (vm.js:53:16)
        at Module._compile (module.js:373:25)
        at Object.Module._extensions..js (module.js:404:10)
        at Module.load (module.js:343:32)
        at Function.Module._load (module.js:300:12)
        at Module.require (module.js:353:17)
        at require (internal/module.js:12:17)
        at Object.<anonymous> (C:\Users\YowkoTsai\Documents\Visual Studio 2017\Projects\TestWebpack\TestWebpack\node_modules\vue-loader\index.js:1:80)
        at Module._compile (module.js:397:26)
        at Object.Module._extensions..js (module.js:404:10)
        at Module.load (module.js:343:32)
        at Function.Module._load (module.js:300:12)
        at Module.require (module.js:353:17)
        at require (internal/module.js:12:17)
        at loadLoader (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\node_modules\loader-runner\lib\loadLoader.js:13:17)
        at iteratePitchingLoaders (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\node_modules\loader-runner\lib\LoaderRunner.js:169:2)
        at runLoaders (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\node_modules\loader-runner\lib\LoaderRunner.js:362:2)
        at NormalModule.doBuild (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModule.js:182:3)
        at NormalModule.build (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModule.js:275:15)
        at Compilation.buildModule (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\Compilation.js:151:10)
        at factoryCallback (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\Compilation.js:344:12)
        at C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModuleFactory.js:241:5
        at C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModuleFactory.js:94:13
        at C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\node_modules\tapable\lib\Tapable.js:268:11
        at NormalModuleFactory.<anonymous> (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\CompatibilityPlugin.js:52:5)
        at NormalModuleFactory.applyPluginsAsyncWaterfall (C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\node_modules\tapable\lib\Tapable.js:272:13)
        at C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModuleFactory.js:69:10
        at C:\Users\YowkoTsai\AppData\Roaming\npm\node_modules\webpack\lib\NormalModuleFactory.js:194:7
        at nextTickCallbackWith0Args (node.js:452:9)
        at process._tickCallback (node.js:381:13)
    @ ./Scripts/VueSrc/main.js 2:0-28
    Process terminated with code 2.
    ```

    > 我另台電腦遇到的錯誤是 `>` arrow function 無法編譯的問題， error code 8，但解法相同

2.  錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/32787249-0f387888-c991-11e7-9c32-233c0cdeeef4.png)

3.  在 Command Prompt 卻一切正常

    ![2cmd](https://user-images.githubusercontent.com/3851540/32787252-0f6e84f0-c991-11e7-963c-68fe34c5dc71.png)

## 解決方式

*   將 Visual Studio 使用的 WebPack 設定與 Command Prompt 相同 --> 吃環境變數中的設定
    *   Visual Studio 主選單 Tools --> Options...

        ![6tools](https://user-images.githubusercontent.com/3851540/32787256-1036c1ae-c991-11e7-9a20-fe6b14fe8d1a.png)

    *   Projects and Solutions --> Web Package Management --> External Web Tools

        ![7externaltools](https://user-images.githubusercontent.com/3851540/32787734-4249c078-c992-11e7-8d5b-79b33fe9714e.png)

    *   將 `$(PATH)` 移至第一個位置

        ![3change1](https://user-images.githubusercontent.com/3851540/32787253-0fa5753c-c991-11e7-988d-62bc02f0f615.png)

        ![4cahnge2](https://user-images.githubusercontent.com/3851540/32787254-0fdc2258-c991-11e7-8547-177e81c43f7d.png)

*   設定成功

    ![5buildok](https://user-images.githubusercontent.com/3851540/32787255-10091100-c991-11e7-8196-da1b687d42b2.png)

## 心得

這個問題我每台電腦、每個 Visual Studio (我同時測試過 Visual Studio 2015 及 Visual Studio 2017 ) 都需要調整設定，我傾向認定應該是微軟 Visual Studio 團隊經過考量所做的選擇而非 bug 畢竟不同 Visual Studio 間的設定都相同，也都會出現問題 XD

讓人不解的是做這樣的設計不是很容易出現 Visual Studio 與 Command Prompt 行為不同的情況嗎，一般前端工具的使用者應該不會透過 Visual Studio ，一定直接操作 global 環境的呀，，好在靈光乍現，想到用 Command Prompt 測試才找出問題

# 參考資訊

1.  [Visual Studio Task Runner console throwing error](https://github.com/FormidableLabs/webpack-dashboard/issues/97)
