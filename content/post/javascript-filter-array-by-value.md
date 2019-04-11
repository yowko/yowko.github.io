---
title: "JavaScript 依屬性值過濾陣列取得完整物件"
date: 2017-12-24T15:48:00+08:00
lastmod: 2017-12-24T15:48:19+08:00
draft: false
tags: ["JavaScript"]
slug: "javascript-filter-array-by-value"
aliases:
    - /2017/12/javascript-filter-array-by-value.html
---
# JavaScript 依屬性值過濾陣列取得完整物件
之前筆記 [JavaScript 在遞迴陣列中取得特定屬性值](https://blog.yowko.com/2017/12/javascript-recursive-map.html) 紀錄到如何將依屬性名稱取出陣列物件屬性值，當時需求是為了取得物件內包含的所有 id 值用來比較是否全選下層的依據，在文章內也提到只取出特定屬性值容易造成後續維護困難(當時是為了 treeview 畫面顯示，盡量節省資料量及物件複雜度)，而為了完整操作物件就是今天的 `依屬性值取得完整物件`

## 找出所有符合條件的物件

*   陣列範例

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
          }
        ];
    ```

1.  使用自定 function
    *   anonymous function

        ```js
        function findObjectByProporigin(arr, prop, val) {
            let result=[];
            arr.map(function(el){
                if(el[prop]===val){
                    result.push(el);
                }
            });
            return result;
        }
        findObjectByProporigin(dataobj,"type",3)
        ```

        ![1anonymou](https://user-images.githubusercontent.com/3851540/34325037-17de8898-e8c1-11e7-9265-d12ba71f8581.png)

    *   arrow function

        ```js
        function findObjectByProp(arr, prop, val) {
            let result=[];
            arr.map(el=>{
                if(el[prop]===val){
                    result.push(el);
                }
            });
            return result;
        }
        findObjectByProp(dataobj,"type",2)
        ```

        ![2arrow](https://user-images.githubusercontent.com/3851540/34325038-1808ee4e-e8c1-11e7-801d-3e72610f1771.png)

2.  使用 ES6 - `filter`

    ```js
    dataobj.filter(obj =>obj.type === 3)
    ```

    ![5filter](https://user-images.githubusercontent.com/3851540/34325041-1884322a-e8c1-11e7-9366-99ced829e7f5.png)

3.  使用 lodash - `filter`

    ```js
    _.filter(dataobj, (obj)=>obj.type === 2)
    ```

    ![6lodashfilter](https://user-images.githubusercontent.com/3851540/34325042-18af282c-e8c1-11e7-8a73-d8ad4bff05d6.png)

4.  使用 Ramda - `filter`

    ```js
    R.filter(obj=>obj.type===1, dataobj)
    ```

    ![7ramda](https://user-images.githubusercontent.com/3851540/34325043-18d96a6a-e8c1-11e7-8276-02b4cc55ead3.png)

## 心得

一口氣紀錄了好幾種寫法，包括自己刻 function、透過 ES6 特性與使用套件，大家喜歡哪個寫法勒？以前 ES6 語法尚未普及的年代(哈哈 講得好像好幾十年前，事實上也不過 3-5 年前而已)，使用 lodash、underscore 這類擴充套件可以讓開發速度及執行速度都有倍數等級的提昇，但在 ES6 多已內建在瀏覽器的今天，部份使用上引用這類外部套件就變得`不再非要不可`，當然 lodash 這類套件功能強大，絕不只今天介紹使用的 api 而已，至少相容性跟強健性都比較好，不過還是要依實際需求來衡量比較正確

# 參考資訊

1.  [JavaScript 在遞迴陣列中取得特定屬性值](https://blog.yowko.com/2017/12/javascript-recursive-map.html)
2.  [Javascript: find an object in array based on object's property](https://www.linkedin.com/pulse/javascript-find-object-array-based-objects-property-rafael/)
3.  [Array.prototype.filter()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
4.  [lodash filter](https://lodash.com/docs/4.17.4#filter)
5.  [Ramda - filter](http://ramdajs.com/docs/#filter)
