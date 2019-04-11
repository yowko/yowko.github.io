---
title: "增加 Release Mode(Release Build) 時的偵錯資訊"
date: 2016-12-14T00:42:34+08:00
lastmod: 2018-09-06T00:42:34+08:00
draft: false
tags: ["Visual Studio"]
slug: "release-mode-with-pdb"
aliases:
    - /2016/12/debug-in-release-mode-with-pdb.html
---
# 增加 Release Mode(Release Build) 時的偵錯資訊
 相信大家都遇過 Release Build 的網站出現錯誤卻因為資訊過少而難以偵錯，但又不可能為了除錯把 Debug Build 的檔案丟上去

 這個時候我們就可以將 build 完的成品與對應的 `.pdb` 檔一併部署至正式環境中，到時真的不幸遇到問題時就可以得到較詳細的錯誤資料(ex.行號)，立馬來看看該如何設定吧


# 有無 .pdb 的實際差異
1. 一般狀況(沒有行號資訊)
 
 ![withoutline](https://cloud.githubusercontent.com/assets/3851540/21704905/4106a866-d3f7-11e6-9037-b8f20abb4880.png)

1. 加上 .pdb (有行號資訊)

![withline](https://cloud.githubusercontent.com/assets/3851540/21704909/410a23c4-d3f7-11e6-8400-a1444ee1f5d5.png)


# 首先確認 Release Build 的設定會產出 pdb
1. 開啟專案屬性設定
    
    ![prperties](https://cloud.githubusercontent.com/assets/3851540/21704904/4105d99a-d3f7-11e6-91f8-08f2bec63f37.png)

2. 確認 Configuration Mode
    - 選擇 `Build` --> 確認 `Configuration`
        
        ![configuration](https://cloud.githubusercontent.com/assets/3851540/21704902/40c47284-d3f7-11e6-944d-766c248d4be2.png)
    
3. 檢查 Output
    - 在 `Output` 區段，按下 `Advanced...`
        
        ![output](https://cloud.githubusercontent.com/assets/3851540/21704910/4128a83a-d3f7-11e6-94f4-b579cd6b8b5e.png)

4. 確認 Debug info
    - 在 `Output` 區段，`Debug info` 設定為 `pdb-only`
        
        ![pdb-only](https://cloud.githubusercontent.com/assets/3851540/21704907/41083af0-d3f7-11e6-857a-8aea6355347d.png)


5. 專案中的 `obj/Release` 資料夾中有產生對應的 pdb

    ![pdb](https://cloud.githubusercontent.com/assets/3851540/21704903/40e5f5f8-d3f7-11e6-84c6-1b62dc5f06fa.png)


# 單鍵部署(publish)時沒有 pdb?
* 預設 <font style="color:red">排除</font> debug 符號檔

    ![default](https://cloud.githubusercontent.com/assets/3851540/21704906/4107fa36-d3f7-11e6-86d4-4f9c7ebbb0a1.png)

* 取消排除 debug 符號檔
    
    ![uneched](https://cloud.githubusercontent.com/assets/3851540/21704908/4109780c-d3f7-11e6-98bd-43fcfd138fd5.png)


# 參考資料
1. [Include .pdb files in Web Application Publish for Release mode (VS2012)](http://blog.deltacode.be/2012/09/26/include-pdb-files-in-web-application-publish-for-release-mode-vs2012/)