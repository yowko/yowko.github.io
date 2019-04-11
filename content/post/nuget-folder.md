---
title: "指定 NuGet packages 存放位置"
date: 2017-06-15T21:00:00+08:00
lastmod: 2018-09-22T21:00:39+08:00
draft: false
tags: ["NuGet","Visual Studio"]
slug: "nuget-folder"
aliases:
    - /2017/06/nuget-folder.html
---
# 指定 NuGet packages 存放位置
同事負責的 team 有個專案，因為歷史緣故因素裡面有超過二十個 projects，實際上各個專案間並沒有相依或是關連性，正確的做法應該要把各個專案拆分開來，但拆分的動作不僅影響開發流程，連 CI Server 這邊也需要一併調整，冒然行事可能會造成問題，所以還需要完整規劃該如何拆分。

只是目前遇到的問題還是要先解決，這二十來個 projects 有各自獨立的 NuGet 參考，packages 是分開的，問題發生在 clone 這個專案時會需要一併 clone 這全部二十來個 projects，packages 也會一併被 clone，加上這二十幾個 projects 都是 WEB Api 專案，NuGet 參考大多數都一樣，造成空間被無謂的佔用，平均一個 project 的 packages 約需 200 MB，二十來個就超過 4 GB，這在開發人員電腦上可能還好，但在 CI Server 上就是個大負擔，硬碟空間跟網路 IO 都是，所以打算做些調整，至少讓這二十來個 projects 可以共用 NuGet 套件參考，順利的話應該可以省下 95% 的空間XD 接下來就讓我們看看該如何設定吧

## 加入 `nuget.config`

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>  
  <config>
    <add key="repositoryPath" value="..\Packages" />
  </config>
</configuration>
```
*   可以指定 `絕對路徑` 或是 `相對路徑`
*   nuget.config 可以是 solution level 或是 project level

    *   solution level，請將 `nuget.config` 跟 `.sln` 放在一起
    *   project level，請將 `nuget.config` 跟 `.csproj` 放在一起



## 修改 `.csproj` 檔

因為修改了 NuGet 套件的存放位置雖然 NuGet restore 會重新下載相關套件，但因 .csproj 已經紀錄了套件位置會導致專案 Reference 都跑掉，所以需要重新指定 dll 位置

*   Reference 遺失

    ![1referencemiss](https://user-images.githubusercontent.com/3851540/27176097-de88a98e-51f3-11e7-8742-74bec5f93fc5.png)

* 卸載專案

    > 專案上按右鍵 --> Unload Project

    ![2unloadproject](https://user-images.githubusercontent.com/3851540/27176099-deaae166-51f3-11e7-8894-a3ade743ec32.png)

*   編輯 .csproj 檔

    > 已卸載專案上按右鍵 --> Edit `.csproj`

    ![3editcsproj](https://user-images.githubusercontent.com/3851540/27176100-deaf4ec2-51f3-11e7-960f-e20882e76c9d.png)

*   修改 Reference 的參考路徑

    > 將 Reference 的 HintPath 改至正確位置

    ![4hintpath](https://user-images.githubusercontent.com/3851540/27176101-deb6f672-51f3-11e7-8073-518274783b4e.png)

*   重新載入專案

    > 已卸載專案上按右鍵 --> Reload `.csproj`

    ![5reloadproj](https://user-images.githubusercontent.com/3851540/27176096-de880d6c-51f3-11e7-8775-0aa6b24da29a.png)

*   Reference 正常

    ![6referenceok](https://user-images.githubusercontent.com/3851540/27176098-de897314-51f3-11e7-86c7-5fdbd688dc91.png)

## 心得

修改的工並不多，但測試過程還滿麻煩的： nuget.config 修改後，需要關閉專案重開才會生效，不然 nuget.config 會有 cache。實際修改專案時相對路徑常會有階層填錯的問題，算是苦工呀

# 參考資訊

1.  [Setting up a common nuget packages folder for all solutions when some projects are included in multiple solutions](https://stackoverflow.com/questions/18376313/setting-up-a-common-nuget-packages-folder-for-all-solutions-when-some-projects-a)
