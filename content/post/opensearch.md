---
title: "怎麼讓網站在輸入網址後按 tab 就可以直接搜尋網站內容(OpenSearch)"
date: 2016-12-26T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["Tools"]
slug: "opensearch"
aliases:
    - /2016/12/Add-OpenSearch-to-website.html
---
# 怎麼讓網站在輸入網址後按 tab 就可以直接搜尋網站內容(OpenSearch)
無意間注意到 Chrome 在網址列輸入 `youtube` 時會提示按下 `tab`, 按下 `tab` 後 網址列會出現 `搜尋 YouTube 影片搜尋`  的前綴，接著就可以直接搜尋 `youtube`  上的影片，覺得滿方便的，所以就來看看可以怎麼做吧


# youtube 效果

![1youtubetab](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/699x98/093bd9129604cf2a0e812c0acd567237/_output_1youtubetab.png)

- 直接搜尋 `yoututbe` 影片
    ![youtubetab](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/688x106/cec7322bf0e46f0f20279ab6bcc0bda1/_output_youtubesearch.png)

# 1. 程式加上可接受查詢字串
- 讓網站有位置可以接住帶進來的查詢字串
- 這邊先不做事，查詢字串內容直接顯示出來
    
    ![searchfunction](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/466x368/de8eb4c8f6fc493b7f75150c048683a9/_output_searchfunction.png)

# 2. 設定 OpenSearch 描述文件
- 2-1. 新增檔案
    - 專案右鍵--> Add --> New Item...
        
        ![newfile](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/687x355/325e541cbd8d86558568859369d222f8/_output_newfile.png)

- 2-2. 選擇 XML 檔
    - Data--> XML File
        
        ![newxml](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/955x660/3f99c0af35530de0d8c4d8118677eb3c/_output_newxml.png)

- 2-3. 加入 OpenSearch 描述內容
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
     <OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
       <ShortName>yowko Search</ShortName>
       <Description>Use localhost to search the Web.</Description>
       <Tags>example web</Tags>
       <Contact>yowko@yowko.com</Contact>
       <Url type="text/html"
           template="http://localhost:18735/search/index?q={searchTerms}"/>
     </OpenSearchDescription>
    ```

    ![editxml](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/681x231/9f5dceb4355081332d1b1cb9e3d4ab02/_output_editxml.png)

# 3. 引用 OpenSearch 描述文件
- 在 head 區域中加上 
    
    ```html
    <link rel="search"
              type="application/opensearchdescription+xml"
              href="http://localhost:18735/opensearch.xml"
              title="Content search" />
    ```
    ![include](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/1200x365/18247e0e1c1244a52c12590af92e736f/_output_INCLUDE.png)

# 4. 效果
- 4-1. 網址列輸入`關鍵字`
    
    ![tab](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/444x41/ea686d926e7dfa2fe0fe942921fae8e9/_output_tab.png)

- 4-2. 可直接搜尋
    
    ![opensearch](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/345x52/e716c359ed63470984ece3505cf4a953/_output_opensearch.png)

- 4-3. 搜尋結果
    
    ![result](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/552x121/7c5cd57ac28590e2795bfb844fdeb2ff/_output_result.png)


# 5. 其他設定
- 5-1. 編輯搜尋引擎
    - 網址列右鍵 --> 編輯搜尋引擎
        
        ![rightclick](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/464x289/f54d19a10cd2f0206f8a55d5d470446b/_output_rightclick.png)

- 5-2. 修改關鍵字
    - 可以修改成喜歡的字
        
        ![searchengine](https://trello-attachments.s3.amazonaws.com/584002d11a92a77dc7a5b452/1048x829/4dbbea518b308c150ba09d99d2d63856/_output_searchengine.png)

整個設定的工並不多，但提升方便性非常多，太划算了！！


# 參考資料
1. [在Chrome網址列搜尋Youtube](https://www.ptt.cc/bbs/Google/M.1270417836.A.B07.html)
2. [opensearch.org](http://www.opensearch.org/Home)
3. [For developers Specifications](http://www.opensearch.org/Specifications/OpenSearch/1.1#Overview_2)