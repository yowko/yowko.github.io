---
title: "Vue.js 2.0 基礎 API 與 Directives 用法範例"
date: 2016-12-22T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["Frontend","JavaScript"]
slug: "vue-js-2-example"
aliases:
    - /2016/12/Vue-js-2-API-and-Directives-sample.html
    - /2016/12/vue-js-2-example
---
# Vue.js 2.0 基礎 API 與 Directives 用法範例
Vue.js 因為方便性及效能突出，備受矚目，雖然發展迅速，但整體的說明文件仍然相當不足，讓使用時的便利降低不少，相信過段時間一定會大幅改善的。剛好前些日子有粗淺地嘗試過，順手紀錄一下自己的使用範例

# 使用方式
1. 在 HTML 中加入 `Vue.js` 引用
	
    ```html
    <script src="http://vuefe.cn/js/vue.js"></script>
    ```

2. 在 HTML 上建立 Container
	
    ```html
    <div id='demo'>
        <p>{{message}}</p>
        <input v-model='message'>
    </div>
    ```

3. 在 JavaScript 中宣告 Vue
	
    ```js
    var demo =new Vue({
        el:'#demo',
        data:{
            message:'Hello Vue.js'
        }
    })
    ```

# API
1. el
	- DOM 元素
	- 提供 Vue.js 執行目標
	- CSS 選擇器 or  HTMLElement instance
	- [線上範例](http://jsbin.com/diruluv/edit?html,js,output)
	- Javascript
    	
        ```js
        new Vue({
          	el:'#test'
        })
        ```
	- Html
    	```
        <div id='test'></div>
        ```

2. template
	- 模版
	- 用來替換 el 的內容
	- [線上範例](http://jsbin.com/vapiki/edit?html,js,output)
	- Javascript
    	```
        new Vue({
          	el:'#test',
      		template:'<span>rrr</span>'
        })
        ```
	- Html
    	```
        <div id='test'></div>
        ```

3. render
	- 不同於 template ，可以用 JavaScript 建立 VNode
	- [線上範例](http://jsbin.com/hagape/edit?html,js,output)
	- Javascript
    	```
        new Vue({
          	el:'#test',
      		render:function(elementCreate){
                return elementCreate('h1',"hello world")
              }
        })
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

4. data
	- 為 Vue instance 提供資料
	- [線上範例](http://jsbin.com/bosaqi/edit?js,output)
	- Javascript
    	
        ```js
        var d={a:'Hello'};
        new Vue({
              el:'#test',
              template:'<span>{{a}}</span>',
              data:d
        })
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

5. methods
	- 可以直接在 directive 中使用或是直接透 Vue instance 叫用
	- [線上範例](http://jsbin.com/bosaqi/edit?js,output)
	- Javascript
    	
        ```js
        var d={a:1,b:2};
        new Vue({
              el:'#test',
              data:d,
              template:'<child>{{plus()}}</child>',
              methods:{
                  plus:function(){
                    return this.a+this.b;
                  }
              }
        })
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

6. watch
	- key 是 watch 對象, value 是執行動作
	- value 可以是方法實體、方法名稱，也可以 json (深度 watch)
	- [線上範例](http://jsbin.com/jusuyob/edit?html,js,output)
	- Javascript
    	
        ```js
        var d={a: 1,b: 2,c: 3};
        var vm =new Vue({
              el:'#test',
              data:d,
              template:'<div>{{a+b}}</div>',
              watch:{
                a: function (val, oldVal) {
                   console.log('new:'+  val+', old: '+ oldVal);
                },
                // 方法名
                b: 'someMethod',
                // 深度 watcher
                c: {
                  handler: function (val, oldVal) { /* ... */ },
                  deep: true
                }
          }
        })
        vm.a=3;
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

7. props
	- Array or Object
	- 用來接收父組件的資料
	- [線上範例](http://jsbin.com/fulebaf/edit?html,js,output)
	- Javascript
    	
        ```js
        Vue.component('child', {
              props: [ 'message1'],
              template: '<span>{{ message1 }}</span>'
        })
    
        new Vue({
              el:'#test',
              data:{message:"Hello Vue"},
              template:'<child v-bind:message1=message></child>'
        })
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

8. computed
	- 有 `get` 跟 `set` 方法 --> 會自動 binding
	- [線上範例](http://jsbin.com/bogowan/edit?html,js,output)
	- Javascript
    	
        ```js
        var d={a:1,b:2};
        var vm=new Vue({
              el:'#test',
              data:d,
              template:'<child>{{a_b}}</child>',
              computed:{
                // 僅讀取
                a_b:function(){
                  return this.a*2+this.b*2;
                },
                // 讀取和設置
                aPlus: {
                  get: function () {
                    return this.a + 1
                  },
                  set: function (v) {
                    this.a = v
                  }
                }
              }
        })
    
        console.log(vm.aPlus);//2
        console.log(vm.aPlus=5);//5
        ```
	- Html
    	
        ```html
        <div id='test'></div>
        ```

# Directives
1. v-text
	- 更新 Vue component 的 textcontent
	- `<span v-text="msg"></span>` 等同 `<span>{{msg}}</span>`
	- [線上範例](http://jsbin.com/romezab/edit?html,js,output)
    	
        ```html
        <span>{{msg}}</span>
        <br/>
        <span v-text="msg"></span>
        ```

2. v-html
	- 更新 component 的 innerHTML
	- 可使用 HTML 語法
	- [線上範例](http://jsbin.com/bugawi/edit?html,js,output)
    	
        ```js
        new Vue({
              el:'#test',
              data:{msg:'<span style="color:red">Hello Vue.js</span>'},
              template:'<div v-html="msg"></div>'
        })
        ```

3. v-if/v-else
	- 依判斷式結果來決定是否 render,
	- 直接影響 html 是否輸出或是銷毀
	- [線上範例](http://jsbin.com/wokuget/edit?html,js,output)
    	
        ```html
        new Vue({
              el:'#test',
              data:{showif:false},
              template:"<div><h1 v-if='showif'>Yes</h1>"+
                        "<h1 v-else>No</h1></div>"
        })
        ```

4. v-show
	- 透過判斷式來切換 component 的 display 屬性
    - [線上範例](http://jsbin.com/ceqedi/edit?html,js,output)
    	
        ```js
        new Vue({
              el:'#test',
              data:{showif:false,isshow:true},
              template:"<div v-show='isshow'>Hello Vue.js</div></div>"
        })
        ```

6. v-for
	- 用來重複 render
	- [線上範例](http://jsbin.com/papagaq/edit?html,js,output)
    	
        ```html
        <ul id="example-1">
            <li v-for="item in items">
              {{ item.message }}
            </li>
        </ul>
        ```
7. v-on
	- 用來監聽事件
	- 普通 HTML Element 只能監聽 `原生 DOM 事件`
	- 自定組件可以監聽 `自定義事件`
	- 縮寫：`@`
	- ` v-on:click="doThis"` 與 `@click="doThis` 相同
    - [線上範例](http://jsbin.com/seliwof/edit?html,js,output)
    	
        ```html
        <button v-on:click="doThis">Test1</button>
        <button @click="doThis">test2</button>
        ```

8. v-bind
	- 動態屬性 binding
	- 縮寫：`:`
	- `v-bind:href="targetUrl"` 與 `:href="targetUrl"` 相同
    - [線上範例](http://jsbin.com/fosefe/edit?html,js,output)
        
        ```html
        <a v-bind:href="targetUrl" target="_blank">yahoo1</a>
        <a :href="targetUrl" target="_blank">yahoo2</a>
        ```

9. v-model
	- 表單的雙向 binding
	- [線上範例](http://jsbin.com/fogefi/edit?html,js,output)
    	
        ```html
        <input v-model="message" placeholder="edit me"/>
        <p>Message is: {{ message }}</p>
        ```

10. v-pre
	- 標示為不經由 Vue 編譯
    - [線上範例](http://jsbin.com/wazoco/edit?html,js,output)
        
        ```html
        <span >{{ msg }}</span>
        <hr>
        <span v-pre>{{ msg }}</span>
        ```

11. v-cloak
	- Vue 編譯完成前，元素不顯示
	- CSS 也需要配合
	- 可以透過下面範例，先刪除 CSS 來比較差異
	- [線上範例](http://jsbin.com/kajeram/edit?html,css,js,output)
	- CSS
    	
        ```css
        [v-cloak] {
     		 display: none;
    	}
        ```
    - HTML
        
        ```html
        <p v-cloak>Message is: {{ msg }}</p>
        ```
    - JavaScript (模擬 call api 延遲現象)
        
        ```js
        $(function(){
            setTimeout("new Vue({el:'#test',data:{msg:'Hello Vue.js'}})",1000);
        })
        ```
12. v-once
	- 只 binding 一次，之後就會略過
	- 可以透過下面範例 ，在 console 中輸入 `msg=123` 來瞭解效果
	- [線上範例](http://jsbin.com/namamup/edit?html,js,console,output)
    	
        ```html
        <span>{{msg}}</span>
        <br/>
        <span v-once>{{msg}}</span>
        ```

# 參考資料
1. [Vue.js](https://vuefe.cn/)
2. [Vue.js](http://vuejs.org/)
3. [Vue.js](https://vuejs.org.cn/)