---
title: "Visual Studio Build fail - Exception has been thrown by the target of an invocation"
date: 2017-10-11T22:06:00+08:00
lastmod: 2018-9-27T22:06:51+08:00
draft: false
tags: ["Visual Studio","Debug"]
slug: "visual-studio-build-fail-exception"
aliases:
    - /2017/10/visual-studio-build-fail-exception.html
---
# Visual Studio Build fail - Exception has been thrown by the target of an invocation
這是在本機利用 Visual Studio 2017 debug 程式時遇到的問題，明明程式修改只改 View 竟然引發了 build fail，我又沒有啟用 build view 功能@@"，把修改還原還是一樣無法解決問題，雖然很快就排除問題，但過程中一開始完全沒有方向，而最終解決方式又相當簡單，情緒落差之大足以筆記一篇XD

## 錯誤訊息

1.  訊息內容

    ```
    Exception has been thrown by the target of an invocation.
    ```

2.  錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/31444696-7dd51c1a-aecf-11e7-9741-00bef8a83a84.png)

## 解決方式

以下解決方式需搭配以下錯誤

1.  注意錯誤訊息中伴隨著多個無法從 `packages` 複製檔案至 `bin\roslyn\` 的錯誤

2.  程式正在執行中

    > IIS Express 咬住

    ![2iisexpress](https://user-images.githubusercontent.com/3851540/31444697-7dfe5e2c-aecf-11e7-9f93-bda4a410dc2e.png)

*   解決方式：<span style="color:red">關閉 IIS Express</span>

    ![3exitiisexpress](https://user-images.githubusercontent.com/3851540/31444698-7e241e82-aecf-11e7-9ae0-3844942f90e4.png)

## 心得

看完解決方式是不是覺得遜遜的？！ 我也有一樣的想法，一直考慮是不是需要特別筆記，但後來想想這麼簡單的動作還搞不定，再來一次我一定會吐血，所以下定決心筆記一下加深印象

# 參考資訊
