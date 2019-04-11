---
title: "JavaScript 偵測 Request 來自瀏覽器的 Go Back (回到上一頁)"
date: 2018-01-08T19:01:00+08:00
lastmod: 2018-10-02T19:01:42+08:00
draft: false
tags: ["JavaScript"]
slug: "javascript-detect-request-from-go-back"
aliases:
    - /2018/01/javascript-detect-request-from-go-back.html
---
# JavaScript 偵測 Request 來自瀏覽器的 Go Back (回到上一頁)
這是自己寫的 bug 所衍生出來的需求，大意是如果 user 在某個頁面執行 submit 動作後會 redirect 到新頁面，但如果 user 此時按下 go back 再重新 submit 一次頁面就會出現錯誤，所以打算透過 JavaScript 來偵測 request 來源，如果是使用 go back 時就做一些動作(e.x. 重新整理頁面或是提示)

## JavaScript 程式碼

```js
<script>
    $(function () {
        if (!!window.performance && window.performance.navigation.type === 2) {
            //!! 用來檢查 window.performance 是否存在
            //window.performance.navigation.type ===2 表示使用 back or forward
            console.log('Reloading');
            //window.location.reload();//或是其他動作
        }
    })
</script>
```

## 其他可用 Type

*   TYPE_NAVIGATE (0)

    > 透過 `連結`、`書籤`、`表單操作`、`script`、`直接輸入 url 開啟` 存取

*   TYPE_RELOAD (1)

    > 透過 `重新整理` 或是 `Location.reload()` 方法存取

*   TYPE_BACK_FORWARD (2)

    > 透過 `瀏覽紀錄` 存取

*   TYPE_RESERVED (255)

    > 任何其他方式

## 實際效果

*   測試程式碼

    ```js
    <script>
    $(function () {
        if (!!window.performance && window.performance.navigation.type === 0) {
            console.log('Navigate');
            //window.location.reload();
        }
        if (!!window.performance && window.performance.navigation.type === 1) {
            console.log('Reloading');
            //window.location.reload();
        }
        if (!!window.performance && window.performance.navigation.type === 2) {
            console.log('backforward');
            //window.location.reload();
        }
    
    })
    </script>
    ```

*   TYPE_NAVIGATE (0)

    ![1navigate](https://user-images.githubusercontent.com/3851540/34667850-bd884078-f4a5-11e7-8f95-110798d5f86c.png)

*   TYPE_RELOAD (1)

    ![2reload](https://user-images.githubusercontent.com/3851540/34667851-bdaef8bc-f4a5-11e7-8305-96a51b103f1a.png)

*   TYPE_BACK_FORWARD (2)

    ![3backforward](https://user-images.githubusercontent.com/3851540/34667852-bdd6fc0e-f4a5-11e7-9183-a2dffacb6075.png)

*   TYPE_RESERVED (255)

    > 這個我試不出來

## 心得

姑且不論當時想要偵測 request 的來源是因為 bug，幸虧有這次經驗才有機會學到 `PerformanceNavigation` 的操作，方便日後有其他相關需求時可以客製特定行為，只是 bug 應該要儘量避免才是 @@"

# 參考資訊

1.  [How to refresh page on back button click?](https://stackoverflow.com/questions/20899274/how-to-refresh-page-on-back-button-click)
2.  [PerformanceNavigation](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigation)
3.  [Methods to determine if an Object has a given property](https://toddmotto.com/methods-to-determine-if-an-object-has-a-given-property/)
