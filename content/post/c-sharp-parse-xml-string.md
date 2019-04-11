---
title: "C# 解析 XML 字串(C# Parse XML string)"
date: 2016-12-20T00:42:34+08:00
lastmod: 2018-09-07T00:42:34+08:00
draft: false
tags: ["C#"]
slug: "c-sharp-parse-xml-string"
aliases:
    - /2016/12/c-sharp-parse-xml-string.html
---
# C# 解析 XML 字串(C# Parse XML string)
最近在介紹第三方金流服務時，廠商使用 XML 回傳，有段時間沒處理 XML，剛收到時還愣了一下，突然間想不起來該怎麼做，看來有年紀後就不適合再靠記憶力，馬上來紀錄一下。

# 整理 XML string
1. 收到的內容
    
    ```
    "\t\t\t\t\t\t\t\t\t\t\t\t<?xml version=\"1.0\" encoding=\"UTF-8\"?><documents><Resp><Id>486441f5-7fa6-4676-b37d-ef29cfbdb1fe</Id><accountId>yowko</accountId><orderNo>Yowko16111800018</orderNo><mtaTransId>8800000006293669</mtaTransId><transAmt>3</transAmt><result>0000</result><respCode>0</respCode><transTime>20161118143324</transTime><completeTime>20161118150716</completeTime></Resp></documents>";
    ```
    - 看起來是 XML ，但又好像不太一樣？！, 立馬驗證一下，[XML Validator from w3schools](http://www.w3schools.com/xml/xml_validator.asp)
        
        ![validatorfail](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x424/fd5e04a974bbb6f4fffc1a4371765e46/output_validatorfail.png)

    - `\t` 讓我覺得應該經過 html encode，所以先 `html decode`

2. decode xml via html decode
    - `HttpUtility.HtmlDecode(xml);`

        > 經過 decode 後，果然看來就正常多了
    
        ![decode](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x263/ab06c009af4475c7e13ca65069d2e13d/output_decoded.png)
    
        ![decodevalidfail](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x604/27a6e50bab7fd4331ec66fbebde31d16/output_decodevalidfail.png)

    > 錯誤訊息明白的指出 XML 宣告只允許在文件開始處，接著使用 `trim` 

3. trim
    -  `HttpUtility.HtmlDecode(xml).Trim();`

        ![trim](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x269/2a99de0f8a164b23d33e95905e3d2e64/output_trim.png)

        ![validate](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x665/a41b247f682232c910ae5e155b075c8b/output_validaok.png)


# 開始解析 XML to object
1. 將 xml 先轉成 class
	- Visual Studio 主選單 `Edit` --> `Paste Special` --> `Paste XML as Classes`
    
        ![paste](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1070x434/43af6ba4ea111e207a35401143b765bf/output_pasteXML.png)

2. xml 解析成 object

- Namespace:`System.Xml.Serialization`

- 程式說明:
    
    ```cs
	var reader = new StringReader(xmlstring);//xmlstring 是傳入 XML 格式的 string
	var serializer = new XmlSerializer(typeof(documents));//documents 是 paste xml as class 來的類別
	var instance = (documents)serializer.Deserialize(reader);
    ```
- 實際使用：
    
    ```cs
    string xml = "\t\t\t\t\t\t\t\t\t\t\t\t<?xml version=\"1.0\" encoding=\"UTF-8\"?><documents><Resp><Id>486441f5-7fa6-4676-b37d-ef29cfbdb1fe</Id><accountId>yowko</accountId><orderNo>Yowko16111800018</orderNo><mtaTransId>8800000006293669</mtaTransId><transAmt>3</transAmt><result>0000</result><respCode>0</respCode><transTime>20161118143324</transTime><completeTime>20161118150716</completeTime></Resp></documents>";
    var reader = new StringReader(HttpUtility.HtmlDecode(xml).Trim());
    var serializer = new XmlSerializer(typeof(documents));
    var instance = (documents)serializer.Deserialize(reader);
    ```

    ![serilizeobject](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/833x884/6f30a4e40c76fe6344ef17dbd9309a32/output_pastetoobject.png)

# 取得特定 node 資料
1. 使用 `XmlDocument`
    - Namaspace:`System.Xml`

    - 程式說明:

        ```cs 
        XmlDocument doc = new XmlDocument();
        doc.LoadXml(xmlstring);//xmlstring 是傳入 XML 格式的 string
        doc.GetElementsByTagName("result")[0]?.InnerText;//取得第一個 result 節點內的值
        ```
    
    - 實際使用：

        ```cs
        string xml = "\t\t\t\t\t\t\t\t\t\t\t\t<?xml version=\"1.0\" encoding=\"UTF-8\"?><documents><Resp><Id>486441f5-7fa6-4676-b37d-ef29cfbdb1fe</Id><accountId>yowko</accountId><orderNo>Yowko16111800018</orderNo><mtaTransId>8800000006293669</mtaTransId><transAmt>3</transAmt><result>0000</result><respCode>0</respCode><transTime>20161118143324</transTime><completeTime>20161118150716</completeTime></Resp></documents>";

        XmlDocument doc = new XmlDocument();
        doc.LoadXml(HttpUtility.HtmlDecode(xml).Trim());
        doc.GetElementsByTagName("result")[0]?.InnerText;
        ```
        
        ![xmldocOK](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/789x416/29e253b808db6d1362fd0cf64ab4a4dd/output_xmldoc_ok.png)

2. 使用 `XDocument`
    - Namespace:`System.Xml.Linq`
    - 程式說明:

        ```cs 
        XDocument doc = XDocument.Parse(xmlstring);//xmlstring 是傳入 XML 格式的 string
        doc.Descendants().FirstOrDefault(e => e.Name.ToString().ToLower().Contains("result"))?.Value;//取得第一個 result 節點內的值
        ```
    - 實際使用：
        ```cs
        string xml = "\t\t\t\t\t\t\t\t\t\t\t\t<?xml version=\"1.0\" encoding=\"UTF-8\"?><documents><Resp><Id>486441f5-7fa6-4676-b37d-ef29cfbdb1fe</Id><accountId>yowko</accountId><orderNo>Yowko16111800018</orderNo><mtaTransId>8800000006293669</mtaTransId><transAmt>3</transAmt><result>0000</result><respCode>0</respCode><transTime>20161118143324</transTime><completeTime>20161118150716</completeTime></Resp></documents>";

        XDocument doc = XDocument.Parse(HttpUtility.HtmlDecode(xml).Trim());
        DOC.Descendants().FirstOrDefault(e => e.Name.ToString().ToLower().Contains("result"))?.Value;
        ```
        
        ![xdoc_ok](https://trello-attachments.s3.amazonaws.com/582ee075a22645f48fd4fd8f/1200x405/2101bbdca4bdbe52a452490eef175891/output_xdoc_ok.png)


# 參考資料
1. [XML Validator from w3schools](http://www.w3schools.com/xml/xml_validator.asp)
2. [XDocument 類別](https://msdn.microsoft.com/zh-tw/library/system.xml.linq.xdocument.aspx)
3. [XmlDocument 類別](https://msdn.microsoft.com/zh-tw/library/system.xml.xmldocument.aspx)