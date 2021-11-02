---
title: "JavaScript 在遞迴陣列中取得特定屬性值"
date: 2017-12-24T00:59:00+08:00
lastmod: 2021-11-02T00:59:31+08:00
draft: false
tags: ["JavaScript"]
slug: "javascript-recursive-map"
aliases:
    - /2017/12/javascript-recursive-map.html
---
## JavaScript 在遞迴陣列中取得特定屬性值

之前筆記 [JavaScript Array 的加總](/javascript-array-sum) 紀錄到 int Array 的加總及 object Array 依屬性名稱加總內容的做法，今天則是要來紀錄如何將遞迴陣列中的特定屬性取出

## 一般物件陣列

* 範例物件

    ```js
    var objArray = [
        { id: 0, name: 'Object 0', otherProp: '321' },
        { id: 1, name: 'O1', otherProp: '648' },
        { id: 2, name: 'Another Object', otherProp: '850' },
        { id: 3, name: 'Almost There', otherProp: '046' },
        { id: 4, name: 'Last Obj', otherProp: '984' }
    ];
    ```

* 使用 `map`

    ```js
    objArray.map(el=>el.id)
    ```

* 結果

    ![1map](https://user-images.githubusercontent.com/3851540/34321219-9f329df2-e844-11e7-925d-96c7b4423e00.png)

## 遞迴物件陣列

* 範例物件

    ```js
    var bues=[
      {
        Id:'RD1',
        Name:'研發部',
        SubTeam:[{
            Id:'RD11',
            Name:'產品團隊',
            SubTeam:[{
                Id:'RD111',
                Name:'專案經理',
                SubTeam:[]
            },{
                Id:'RD112',
                Name:'系統分析',
                SubTeam:[]
            },{
                Id:'RD113',
                Name:'開發工程師',
                SubTeam:[]
            }
            ]
       },{
            Id:'RD12',
            Name:'專案團隊',
            SubTeam:[{
                Id:'RD121',
                Name:'專案一部',
                SubTeam:[{
                    Id:'RD1211',
                    Name:'PM',
                    SubTeam:[]
                    },{
                    Id:'RD1212',
                    Name:'RD',
                    SubTeam:[]
                    }]
            },{
                Id:'RD122',
                Name:'專案二部',
                SubTeam:[{
                    Id:'RD1221',
                    Name:'PM',
                    SubTeam:[]
                    },{
                    Id:'RD1222',
                    Name:'RD',
                    SubTeam:[]
                    }]
            }
           ]
       }]
      },{
        Id:'SL1',
        Name:'業務部',
        SubTeam:[]
      }
      ];
    ```

* 使用 `map` 僅能取得第一層內容

    ```js
    bues.map(el=>el.Id)
    ```

    ![2recursivemap](https://user-images.githubusercontent.com/3851540/34321220-9f643d44-e844-11e7-9d59-34f481fbc8be.png)

* 使用 `JSON.stringify`

    1. anonymous function

        ```js
        var allbu=[];
        JSON.stringify(bues, function(k, v) {
            if (k === 'Id') allbu.push(v);
            return v;
        });
        ```

        ![3anonymous](https://user-images.githubusercontent.com/3851540/34321221-9f93ade0-e844-11e7-8b55-e0f83b534b1c.png)

    2. arrow function

        ```js
        var allbuid=[];
        JSON.stringify(bues,(k,v)=>{if(k==='Id')allbuid.push(v);return v;});
        ```

        ![4arrow](https://user-images.githubusercontent.com/3851540/34321222-9fc70f96-e844-11e7-8cd4-f189327127f2.png)

* 使用自訂遞迴 map

    ```js
    function deepmap(arr,prop){
        let props=[];
        arr.map(el=>{
            Object.keys(el).map(el2=>{
            if(el2===prop){
                props.push(el[el2]);
            }
            if(Array.isArray(el[el2])){
                let _result=deepmap(el[el2],prop);
                if(_result.length>0){
                    _result.map(el3=>props.push(el3));
                }
            }
            });
        });
        return props;
    }
    ```

    * 實際使用

        ```js
        deepmap(bues,"Id")
        ```

    * 結果

        ![5deepmap](https://user-images.githubusercontent.com/3851540/34321223-9ffd2392-e844-11e7-86fd-74f4305251ed.png)

## 心得

現在回頭紀錄之前的需求感覺有點怪，主要是站在客觀的角度來看文章內容很難推敲出為什麼會有這樣的需求：原因是想取得 object 中所有 id 值，用來檢查比對物件在畫面(treeview)上是否被全選。那為什麼不使用完整物件：較簡潔也較省資源

另外在自行撰寫 deepmap 時遇到一個問題：無法使用 `concat` 來 merge array，後來改用 map 搭配 push 解決，但還是覺得不太對勁，有點疙瘩

## 參考資訊

1. [JavaScript Array 的加總](/javascript-array-sum)
2. [Check if object is array?](https://stackoverflow.com/questions/4775722/check-if-object-is-array)
3. [Object.keys()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)
