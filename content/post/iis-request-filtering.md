---
title: "如何透過 IIS 的 Request Filtering 功能限制存取特定檔案或附檔名"
date: 2017-04-19T01:39:00+08:00
lastmod: 2021-11-02T17:53:12+08:00
draft: false
tags: ["IIS"]
slug: "iis-request-filtering"
aliases:
    - /2017/04/iis-request-filtering.html
---
## 如何透過 IIS 的 Request Filtering 功能限制存取特定檔案或附檔名

同事因為某個專案使用 Share code with Add as Link 的功能，讓 xml 格式的設定檔直接被放在根目錄下，如此一來爬蟲或是知道檔名的人就可以直接在網路上取得檔案內容，而這樣的設定檔一般都是用來存取其他資源的重要敏感資訊，相關資訊外流是非常嚴重的問題

因此希望在不改程式架構的前提下，避免相關內容被直接存取，打算先治標改天再治本

## 知道檔案路徑後可以直接開啟

* 機敏資訊馬上就裸奔了

    ![1xml](https://cloud.githubusercontent.com/assets/3851540/25116475/96d71936-243f-11e7-9328-e2a532bc972c.png)

## 安裝 Request Filtering 功能

* Windows Server 2016
  * 伺服器管理 --> 新增角色及功能

        ![1manage](https://cloud.githubusercontent.com/assets/3851540/25144288/dcab8bbc-249f-11e7-978b-6e9cbeb22f45.png)

  * 在您開始前

        ![2intro](https://cloud.githubusercontent.com/assets/3851540/25144290/dcb51614-249f-11e7-9ca4-5ef934353def.png)

  * 安裝類型

        ![3type](https://cloud.githubusercontent.com/assets/3851540/25144294/dcd7148a-249f-11e7-8c37-65d7a5243735.png)

  * 伺服器選取項目

        ![4targetserver](https://cloud.githubusercontent.com/assets/3851540/25144295/dcd8f05c-249f-11e7-999b-2717a6fa7dda.png)

  * 伺服器角色(IIS)

        ![5iis](https://cloud.githubusercontent.com/assets/3851540/25144293/dcd5317e-249f-11e7-8640-09dc26733166.png)

        ![6iis](https://cloud.githubusercontent.com/assets/3851540/25144291/dcd35566-249f-11e7-9bb7-b5d0720be521.png)

    * 角色服務

            ![7filter](https://cloud.githubusercontent.com/assets/3851540/25144283/dc66089e-249f-11e7-8b71-35dbf5c37493.png)

  * 確認

        ![8confirm](https://cloud.githubusercontent.com/assets/3851540/25144284/dc8ee192-249f-11e7-8b60-bce39369ef4b.png)

  * 結果

        ![9install](https://cloud.githubusercontent.com/assets/3851540/25144289/dcac2d06-249f-11e7-9723-4dd14b5f4726.png)

        ![10success](https://cloud.githubusercontent.com/assets/3851540/25144286/dcaa10f2-249f-11e7-9a5d-61b847a52415.png)

* Window 7、Windows 10

  * 開啟 Programs and Features

        ![1win10program](https://cloud.githubusercontent.com/assets/3851540/25144285/dca92f8e-249f-11e7-9758-8e2f068fe83b.png)

  * 開啟

        ![2feature](https://cloud.githubusercontent.com/assets/3851540/25144287/dcabb060-249f-11e7-9845-58b9acc5bae7.png)

  * Internet Information Services --> World Wide Web Services --> Security --> Request Filtering

        ![3win10install](https://cloud.githubusercontent.com/assets/3851540/25144292/dcd3d6bc-249f-11e7-9b2e-43ad50833396.png)

## Request Filtering 支援的設定

![3requestfilter](https://cloud.githubusercontent.com/assets/3851540/25116476/96f7b718-243f-11e7-9896-3c841f84df79.png)

![4filterfunction](https://cloud.githubusercontent.com/assets/3851540/25116477/96f92cce-243f-11e7-8964-cd1490e8fc34.png)

1. File Name Extensions
2. Rules
3. Hidden Segments
4. URL
5. HTTP Verbs
6. Headers
7. Query Strings

## 可以用來限制檔案存取的功能

1. File Name Extensions

    * 限制特定附檔名
    * Deny File Extension... --> File name extension

        ![5Denyfilename](https://cloud.githubusercontent.com/assets/3851540/25116480/97187b60-243f-11e7-850a-94384bc6b361.png)

    * 設定後會自動在 web.config 的 `system.webServer` 區段加上下列內容

        ```xml
        <security>
            <requestFiltering>
                <fileExtensions>
                    <add fileExtension=".xml" allowed="false" />
                </fileExtensions>
            </requestFiltering>
        </security>
        ```

    * 畫面結果：HTTP Error 404.7 - Not Found

        ![6filenameresult](https://cloud.githubusercontent.com/assets/3851540/25116479/97180676-243f-11e7-9b26-d82ad742aa22.png)

2. Rules
    * 限制特定檔名，附檔名限制加可不加
    * Add Filtering Rule... --> Scan url --> Applies To (這填附檔名) --> Deny Strings (這填檔名)

        ![7rulesetting](https://cloud.githubusercontent.com/assets/3851540/25116481/9718c7e6-243f-11e7-86f4-d84a35fcd89e.png)

    * 設定後會自動在 web.config 的 `system.webServer` 區段加上下列內容

        ```xml
        <security>
            <requestFiltering>
                <filteringRules>
                    <filteringRule name="xml" scanUrl="true" scanQueryString="false">
                        <appliesTo>
                            <add fileExtension=".xml" />
                        </appliesTo>
                        <denyStrings>
                            <add string="web" />
                        </denyStrings>
                    </filteringRule>
                </filteringRules>
            </requestFiltering>
        </security>
        ```

    * 畫面結果：HTTP Error 404.19 - Not Found

        ![8ruleresult](https://cloud.githubusercontent.com/assets/3851540/25116478/97184758-243f-11e7-8834-2dc5971fdf5b.png)

3. Hidden Segments
    * 限制 URL 上的完整檔名 (檔名+附檔名)
    * Add Hidden Segment... --> Hidden segment

        ![9hiddenseg](https://cloud.githubusercontent.com/assets/3851540/25116482/971abe0c-243f-11e7-8c74-139454860983.png)

    * 設定後會自動在 web.config 的 `system.webServer` 區段加上下列內容

        ```xml
        <security>
            <requestFiltering>
                <hiddenSegments>
                    <add segment="web.xml" />
                </hiddenSegments>
            </requestFiltering>
        </security>
        ```

    * 畫面結果：HTTP Error 404.8 - Not Found

        ![10hiddenseg](https://cloud.githubusercontent.com/assets/3851540/25116483/9720637a-243f-11e7-9a69-bb2489bf64b4.png)

4. URL
    * 限制 URL 上的完整檔名
    * Deny Sequence... --> URL sequence

        ![11denysequence](https://cloud.githubusercontent.com/assets/3851540/25116484/973b190e-243f-11e7-9566-1e1926b15ab8.png)

    * 設定後會自動在 web.config 的 `system.webServer` 區段加上下列內容

        ```xml
        <security>
            <requestFiltering>
                <denyUrlSequences>
                    <add sequence="web.xml" />
                </denyUrlSequences>
            </requestFiltering>
        </security>
        ```

    * 畫面結果：HTTP Error 404.5 - Not Found

        ![12sequenceresult](https://cloud.githubusercontent.com/assets/3851540/25116485/973dce42-243f-11e7-801f-689429ffa7c5.png)

## 參考資訊

1. [Request Filtering <requestfiltering></requestfiltering>](https://www.iis.net/configreference/system.webserver/security/requestfiltering)
2. [IIS 7要求篩選規則](http://www.lijyyh.com/2012/04/iis-7.html)
