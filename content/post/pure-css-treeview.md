---
title: "使用 Pure CSS 製作 Treeview (樹狀圖)"
date: 2017-12-10T16:42:00+08:00
lastmod: 2018-09-30T16:42:03+08:00
draft: false
tags: ["Frontend"]
slug: "pure-css-treeview"
aliases:
    - /2017/12/pure-css-treeview.html
---
# 使用 Pure CSS 製作 Treeview (樹狀圖)
新專案有個功能需要用到樹狀圖 (Treeview)，算是個常見需求，難度也不算高，但更仔細了解細節後，發現似乎又有點不同，加上之前並沒有筆記跟備份專案的習慣，完成的 treeview 功能已經忘記做法，於是就趁著這個機會再來做一個，順便筆記一下，以節省日後開發的時間，順便讓以後的自己在檢視過往做法時可以多個依據

## 需求

1.  階層間縮排要明確
2.  上下左右階層間需要有導引線
3.  階層數目不定
4.  僅使用 CSS 不使用 Javascript

    *   效果示意圖

        ![](https://www.jqueryscript.net/images/Animated-Tree-View-Plugin-For-jQuery-Bootstrap-3-MultiNestedLists.jpg)

## 實際製作

*   基於 Bootstrap css 撰寫
*   線上範例請參考 [jsfiddle 線上範例](https://jsfiddle.net/r1qdpx3w/1/)


*   Html

    ```html
    <li class="root">執行長
    <ul>
        <li>研發部
        <ul>
            <li>產品團隊
            <ul>
            <li>專案經理</li>
            <li>系統分析</li>
            <li>開發工程師</li>
            </ul>

                    </li>
            <li>專案團隊
            <ul>
                <li>專案一部
                <ul>
                    <li>PM</li>
                    <li>RD</li>  
                </ul>
                </li>
                <li>專案二部
                <ul>
                    <li>PM</li>
                    <li>RD</li>
                </ul>
                </li>
            </ul>
            </li>
            <li>訓練團隊
            </li>
        </ul>
        </li>
        <li>業務團隊</li>
    </ul>
    </li>
    </ul>
    ```

*   CSS (使用 SCSS)

    ```scss
    ul {
    margin: 0px 0px 0px 5px;
    list-style: none;
    line-height: 2em;
    font-family: Arial;

        li {
            font-size: 16px;
            position: relative;

            &:before {
                position: absolute;
                left: -15px;
                top: 0px;
                content: '';
                display: block;
                border-left: 1px solid #ddd;
                height: 1em;
                border-bottom: 1px solid #ddd;
                width: 10px;
            }

            &:after {
                position: absolute;
                left: -15px;
                bottom: -7px;
                content: '';
                display: block;
                border-left: 1px solid #ddd;
                height: 100%;
            }

            &.root {
                margin: 0px 0px 0px -5px;

                &:before {
                    display: none;
                }

                &:after {
                    display: none;
                }
            }

        
            &:last-child {
                &:after {
                    display: none;
                }
            }
        }
    }
    ```

*   實際效果

    ![1result](https://user-images.githubusercontent.com/3851540/33803340-bd3a7e9a-ddc8-11e7-9c6f-2f7930a43aeb.png)

## 心得

因為使用該 treeview 的畫面會套用 Vue.js 所以不打算再為了 treeview 功能使用 JQuery 套件，一時找不到合適的寫法最後決定參考網上範例改成自訂樣式，修修改改好久終於達成目標，特以此紀錄懷念團隊有專業前端工程師配合的日子

# 參考資訊

1.  [Pure CSS Tree Menu Framework](http://cssdeck.com/labs/pure-css-tree-menu-framework)
2.  [5 Pure CSS Treeview Menu Examples](http://www.designerslib.com/pure-css-treeview-menu/)
