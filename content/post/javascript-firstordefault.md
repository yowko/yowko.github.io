---
title: "JavaScript 從陣列中取出符合條件的第一個物件 (C# LINQ 的 FirstOrDefault)"
date: 2017-12-25T13:25:00+08:00
lastmod: 2021-11-02T23:36:38+08:00
draft: false
tags: ["JavaScript"]
slug: "javascript-firstordefault"
aliases:
    - /2017/12/javascript-firstordefault.html
---
## JavaScript 從陣列中取出符合條件的第一個物件 (C# LINQ 的 FirstOrDefault)

之前筆記 [JavaScript 依屬性值過濾陣列取得完整物件](/javascript-filter-array-by-value) 紀錄到如何將陣列中的符合特定屬性值條件的物件取出，與本文要介紹的用法相近，只是本文僅取出一筆，最後回傳值是 object，而前文則是回傳 array，當然可以透過前文技巧取出陣列後再使用 index 為 0 取出第一筆資料，只是既然要過濾條件了直接取得最後需要的物件不僅較省資源也較直覺，就來看看該怎麼做吧

## 找出第一個符合條件的物件

* 陣列範例

    ```js
    var dataobj=[
          {
            name:'ASP.NET MVC',
            type:3,
            salary:15000,
            headcount:25
          },{
            name:'Java',
            type:3,
            salary:25000,
            headcount:75
          },{
            name:'Python',
            type:3,
            salary:22000,
            headcount:50
          },{
            name:'React',
            type:1,
            salary:22000,
            headcount:30
          },{
            name:'Angular',
            type:1,
            salary:20000,
            headcount:15
          },{
            name:'Oracle',
            type:2,
            salary:28000,
            headcount:10
          },{
            name:'Sql Server',
            type:2,
            salary:18000,
            headcount:8
          },{
            name:'ASP.NET MVC',
            type:3,
            salary:25000,
            headcount:5
          }
        ];
    ```

1. 使用自定 function
    * 使用 for

        ```js
        function findObjectByProporigin(arr, prop, val) {
            for (var i = 0; i < arr.length; i++) {
                if (arr[i][prop] === val) {
                    return arr[i];
                }
            }
            return null;
        }
        findObjectByPropfor(dataobj,"name","ASP.NET MVC")
        ```

        ![1for](https://user-images.githubusercontent.com/3851540/34333293-2316e718-e976-11e7-8983-1df65305b8c7.png)

    * 使用 `for...in`

        ```js
        function findObjectByPropforin(arr, prop, val) {
            for(var index in arr){
                if (arr[index][prop] === val) {
                    return arr[index];
                }
            }
            return null;
        }
        findObjectByPropforin(dataobj,'name','ASP.NET MVC')
        ```

        ![4forin](https://user-images.githubusercontent.com/3851540/34333297-23909612-e976-11e7-9f37-c0895cb11183.png)

    * 使用 `for...of`

        ```js
        function findObjectByPropforof(arr, prop, val) {
            for(var el of arr){
                if (el[prop] === val) {
                    return el;
                }
            }
            return null;
        }
        findObjectByPropforof(dataobj,'name','ASP.NET MVC')
        ```

        ![5forof](https://user-images.githubusercontent.com/3851540/34333298-23b81282-e976-11e7-8df6-005eeaa20ecc.png)

    * forEach 與 map 無法中途終止會造成 LastOrDefault 的效果

        * forEach

            ```js
            function findObjectByPropforEach(arr, prop, val) {
                var result=null;
                arr.forEach(function (el) {
                    if (el[prop] === val) {
                        result= el;
                    }
                });
                return result;
            };
            findObjectByPropforEach(dataobj,"name","ASP.NET MVC")
            ```

            ![2foreach](https://user-images.githubusercontent.com/3851540/34333295-233f7e58-e976-11e7-9867-515b79f94993.png)

        * map

            ```js
            function findObjectByPropmap(arr, prop, val) {
                var result = null;
                arr.map(el => {
                    if (el[prop] === val) {
                        result = el;
                        return true;
                    }
                });
                return result;
            }
            findObjectByPropmap(dataobj,"name","ASP.NET MVC")
            ```

            ![3map](https://user-images.githubusercontent.com/3851540/34333296-23670fea-e976-11e7-91a3-c734ef39d2fd.png)

2. 使用 ES6 - `find`

    ```js
    dataobj.find(obj=>obj.name==='ASP.NET MVC')
    ```

    ![6find](https://user-images.githubusercontent.com/3851540/34333299-23e23aa8-e976-11e7-904d-4337a641c2e0.png)

3. 使用 lodash - `find`

    ```js
    _.find(dataobj,obj=>obj.name==='ASP.NET MVC')
    ```

    ![7lodashfind](https://user-images.githubusercontent.com/3851540/34333301-240c9d02-e976-11e7-999a-55e05e1ab179.png)

4. 使用 Ramda - `find`

    ```js
    R.find(obj=>obj.name==='ASP.NET MVC', dataobj)
    ```

    ![8ramda](https://user-images.githubusercontent.com/3851540/34333303-24620436-e976-11e7-8098-acc30f046abe.png)

## 心得

原本以為程式碼與之前筆記 [JavaScript 依屬性值過濾陣列取得完整物件](/2017/12/javascript-filter-array-by-value.html) 會非常相似，想不到測試撰寫過程才知道是我想得太簡單了，更進一步證實我的 JavaScript 技能相當薄弱呀(這時才發現 `forEach` 與 `map` 無法中途終止 - 沒有 `break` 跟 `continue`)，幸虧本身就不是靠 js 吃穿的，覺得還過得去 哈哈

不過經過一番測試與查閱資料後，已經比較清楚其中差異，相信下次一定可以比較快解決問題，至少有地方可以<del>抄</del>參考

## 遞迴陣列

1. [JavaScript 依屬性值過濾陣列取得完整物件](/javascript-filter-array-by-value)
2. [Javascript: find an object in array based on object's property](https://www.linkedin.com/pulse/javascript-find-object-array-based-objects-property-rafael/)
3. [Array.prototype.find()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
4. [JavaScript 循環中斷研究與終結，前端 er 必背！](https://juejin.im/entry/5884717a1b69e6005919f0d3)
5. [lodash - find](https://lodash.com/docs/4.17.4#find)
6. [Ramda - find](http://ramdajs.com/docs/#find)
