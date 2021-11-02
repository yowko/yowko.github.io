---
title: "JavaScript Array 的加總"
date: 2017-12-22T03:51:00+08:00
lastmod: 2021-11-02T18:09:04+08:00
draft: false
tags: ["JavaScript"]
slug: "javascript-array-sum"
aliases:
    - /2017/12/javascript-array-sum.html
---
## JavaScript Array 的加總

最近的專案中因為前端互動性強，採用了較多的 JavaScript 來處理畫面互動性，雖然身為一名網站開發工程師，JS 並不算陌生，但實際開發時還是經常需要透過 google 以及 stackoverflow 的協助

針對這個狀況，我自己覺得問題原因主要有兩個：

1. 本身使用 JS 仍是以輔助為主非主力開發語言，造成語法熟悉度還不到信手拈來的程度
2. 再來 JS 語法日新以異，一段時間後可能就會出現更新、更簡潔的語法，讓我覺得應該要用時再查就好

不過近來想法有些不同：與其每次要用都查一次，不如我先紀錄基本語法，實際使用時再檢查是不是有更好的語法比較省時，所以我就來動手紀錄自己常用的 JS 語法囉

## 自行加總

```js
function SumData(arr){
    var sum=0;
    for (var i = 0; i < arr.length; i++) {
        sum += arr[i];
    };
    return sum;
}

var data=[1,3,5,7]
console.log(SumData(data));
```

## 使用 forEach

```js
function SumDataforEach(arr){
    var sum=0;
    arr.forEach(function(element) {
        sum+=element;
    });
    return sum;
}
var data=[1,3,5,7]
console.log(SumDataforEach(data));
```

## 使用 reduce

> 語法最簡潔

```js
function SumDatareduce(arr){
    return arr.reduce((a,b)=>a+b);  
}
var data=[1,3,5,7]
console.log(SumDatareduce(data));
```

## 物件陣列的屬性加總

> 畫面物件一般多為複雜型別，物件屬性的加總是常見需求

* 基本資料

    ```js
    var dataobj=[
        {
            name:'Yowko',
            salary:10000,
            age:30
        },{
            name:'Test',
            salary:20000,
            age:20
        },{
            name:'MVC',
            salary:15000,
            age:25
        }];
    ```

1. 使用 map 將物件屬性轉為陣列

    ```js
    var objarray=dataobj.map(el=>el.salary);
    console.log(objarray);
    ```

2. 再透過 reduce 加總

    ```js
    function SumDatareduce(arr){
        return arr.reduce((a,b)=>a+b);  
    }
    console.log(SumDatareduce(objarray));
    ```

3. 簡化 function

    ```js
    function SumDataMapReduce(arr){
    return arr.map(el=>el.salary).reduce((a,b)=>a+b);
    }
    console.log(SumDataMapReduce(dataobj));
    ```

## 動態屬性值與選擇性屬性

> 直接將物件屬性寫進 function 中就讓 function 少了共用彈性，另外是讓 fucntion 可以同時處理簡單型別及複雜型別

1. 動態屬性

    ```js
    function SumDataMapReduce(arr,prop){
        return arr.map(el=>el[prop]).reduce((a,b)=>a+b);
    }
    var propname='salary'
    console.log(SumDataMapReduce(dataobj,propname));
    ```

2. 選擇性屬性

    ```js
    function SumDataMapReduce(arr,prop){
        if(prop === undefined){
            return arr.reduce((a,b)=>a+b);  
        }
        else{
            return arr.map(el=>el[prop]).reduce((a,b)=>a+b);
        }
    }
    var propname='salary'
    console.log(SumDataMapReduce(dataobj,propname));
    var data=[1,3,5,7]
    console.log(SumDataMapReduce(data));
    ```

## 心得

仔細紀錄一下這個常見功能，再次驗證語法真的很簡單，也難怪我每次都覺得這不用特別記，寫個兩次就背起來了呀，只不過程式語言就是如此：熟悉度跟使用頻率有顯著正相關，一段時間沒用到就又忘了，就當做為下次做準備吧，至少下次要查就會快上許多

## 參考資訊

1. [Rewriting JavaScript: Sum an Array](https://medium.com/chrisburgin/rewriting-javascript-sum-an-array-dbf838996ed0)
2. [Array.prototype.forEach()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
3. [Array.prototype.reduce()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
4. [Array.prototype.map()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
