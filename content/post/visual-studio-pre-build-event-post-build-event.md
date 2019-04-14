---
title: "Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event)"
date: 2017-01-10T00:42:34+08:00
lastmod: 2018-09-09T00:42:34+08:00
draft: false
tags: ["Visual Studio","MSBuild"]
slug: "visual-studio-pre-build-event-post-build-event"
aliases:
    - /2017/01/visual-studio-pre-build-event-post-build-event.html
    - /2017/01/visual-studio-pre-build-event-post-build-event/
---
# Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event)
同事在`被`參考的專案中新增一個 config 檔案，並且設為 `Copy always`，設定後 config 檔案會在發行時自動加入 bin 資料夾，但考慮到可能日後會直接修改線上設定，不想直接修改位於 bin 資料夾內的 config 檔案而引發 IIS recycle，所以希望可以把 config 搬離 bin 資料夾，所以打算透過 Visual Studio 的 Build Events 來將 config 檔案移出 bin 資料夾，需要讀取該檔案的位置再透過相對路徑來讀取

雖然我覺得解決方式不是非常正統，但舊系統我想可以讓需求獲得滿足是最重要的，就來看看如何使用 Visual Studio 建置前事件( Pre-build Event ) / 建置後事件( Post-build Event) 吧

## 編譯設定
1. 專案右鍵 --> Properties
    
    ![1property](https://cloud.githubusercontent.com/assets/3851540/21706186/182b59f6-d400-11e6-88c4-22c1530eabdf.png)

2. Build Events
    
    ![2buildevents](https://cloud.githubusercontent.com/assets/3851540/21706192/1873ca38-d400-11e6-8fac-6ffc118be372.png)

## 使用巨集
![6showeditor](https://cloud.githubusercontent.com/assets/3851540/21706190/186b412e-d400-11e6-8a8a-88ad50a5a21b.png)


## 巨集指令(Macros)
![5monaco](https://cloud.githubusercontent.com/assets/3851540/21706191/186e1d4a-d400-11e6-8692-7ccab6853bfd.png)

巨集|描述
---|---
$(OutDir)|	編譯後的輸出路徑。 結尾會有反斜線 `\` (e.g. `bin\`) ![3outputpath](https://cloud.githubusercontent.com/assets/3851540/21706189/186ab43e-d400-11e6-9ffe-d562f269bc16.png)
$(ConfigurationName)|	build mode (e.g. `Debug`,`Release`)
$(ProjectName)|	專案名稱 (e.g.`Opserver`)
$(TargetName)|	建置後輸出 dll 的檔名 (e.g. `StackExchange.Opserver`)<br/>![4targetname](https://cloud.githubusercontent.com/assets/3851540/21706188/1869bf34-d400-11e6-8818-663be504a457.png)
$(TargetPath)|	建置後輸出 dll 的絕對路徑及檔名 (定義為磁碟機 + 路徑 + 主檔名 + 副檔名) e.g. `D:\Downloads\Opserver-overhaul\Opserver\bin\StackExchange.Opserver.dll`
$(ProjectPath)|	專案檔的絕對路徑 (定義為磁碟機 + 路徑 + 主檔名 + 副檔名) e.g. `D:\Downloads\Opserver-overhaul\Opserver\Opserver.csproj`
$(ProjectFileName)|	專案完整檔名 e.g. `Opserver.csproj`
$(TargetExt)|	建置主要輸出檔的副檔名。 在副檔名之前包括一個 `.` (e.g. `.dll`)
$(TargetFileName)|	建置後輸出 dll 的完整檔名 (定義為主檔名 + 副檔名) e.g. `StackExchange.Opserver.dll`
$(DevEnvDir)|	Visual Studio 安裝目錄 ;結尾會有反斜線 `\` (e.g. `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\`)
$(TargetDir)|	建置後輸出目錄 (定義為磁碟機 + 路徑)。結尾會有反斜線 `\` (e.g. `D:\Downloads\Opserver-overhaul\Opserver\bin\`)
$(ProjectDir)|	專案位置路徑 (定義為磁碟機 + 路徑)，結尾會有反斜線 `\` (e.g. `D:\Downloads\Opserver-overhaul\Opserver\`)
$(SolutionFileName)|	方案完整檔名 (e.g. `{方案檔案名稱}.sln`) e.g. `Opserver.sln`
$(SolutionPath)|	方案檔的絕對路徑 (定義為磁碟機 + 路徑 + 主檔名 + 副檔名) e.g. `D:\Downloads\Opserver-overhaul\Opserver.sln`
$(SolutionDir)|	方案的目錄路徑 (定義為磁碟機 + 路徑)，結尾會有反斜線 `\`  e.g. `D:\Downloads\Opserver-overhaul\`
$(SolutionName)|	方案的檔名。(e.g. `Opserver`)
$(PlatformName)|編譯的目標平台。 (e.g. `AnyCPU`)
$(ProjectExt)|	專案檔的副檔名。 在副檔名之前包括一個 `.` (e.g. `.cs[roj`)
$(SolutionExt)|	方案的副檔名。 在副檔名之前包括一個 `.`。(e.g. `.sln`)

## 語法
1. 使用 command prompt script (e.g. xcopy,robocopy)
2. 巨集可以搭配相對目錄 (e.g. `$(SolutionDir)..\..\` 表示 `SolutionDir` 往上兩層)


## 建置後事件( Post-build Event) 執行時機
1. Always
    
    > 無論編譯成功與否皆執行

2. On successful build
    
    > 僅在編譯成功時執行

3. When the build updates the project output
    
    > 只有編譯需要更新輸出內容時執行

## VS 在 robocopy 使用時出現錯誤
- `The command "robocopy D:\JenkinsData D:\my-app /s *.json" exited with code 1.`
    
    ![7robocopyerror](https://cloud.githubusercontent.com/assets/3851540/21706187/184e96aa-d400-11e6-8a05-0ac7abd13a64.png)

- robocopy 執行代碼
    - 代碼可累加 (e.g. `7` 代表 `1` + `2` + `4` 皆發生)
    
        exit code | 說明
        ---|:---
        0|沒有錯誤，也沒有複製任何檔案，來源與目標已同步
        1|檔案已成功複製
        2|偵測到目標有額外檔案(檔案比來源目錄還多)
        4|檔案或是資料夾不一致
        8|檔案或是資料夾無法被複製(複製出錯且重試次數過多)
        16|錯誤，無法複製任何檔案(使用方法錯誤或是權限不足)

- 解決方式
    - exit code 小於等於 `7` 都算是成功，輸出 `0`
    - 如果 大於等於 `8` 就算失敗，輸出 `1`
        
        ```
        IF %ERRORLEVEL% LEQ 7 exit /b 0
        Rem ELSE
        exit /b 1
        ```

# 參考資料
1. [Pre-build Event/Post-build Event Command Line Dialog Box](https://msdn.microsoft.com/en-us/library/42x5kfw4.aspx)
2. [建置前事件/建置後事件命令列對話方塊](https://msdn.microsoft.com/zh-tw/library/42x5kfw4.aspx)
3. [Post build events using relative paths do not work in VS 2013](http://stackoverflow.com/questions/30773158/post-build-events-using-relative-paths-do-not-work-in-vs-2013)
4. [ROBOCOPY Exit Codes](http://ss64.com/nt/robocopy-exit.html)