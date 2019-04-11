---
title: "使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF"
date: 2018-01-04T03:22:00+08:00
lastmod: 2018-10-02T03:22:42+08:00
draft: false
tags: ["C#","Office"]
slug: "c-sharp-word-to-pdf"
aliases:
    - /2018/01/c-sharp-word-to-pdf.html
---
# 使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF
同事想要將 user 上傳的 word 檔轉換為 pdf，降低內容被篡改的機會，記憶中 word 轉存成 PDF 功能的程式碼並不多，但印象模糊，於是趁這個機會順手紀錄一下，方便日後參考

**<span style="color:red">本文使用的程式碼僅適合 `已安裝` Word 的環境 </span>**

## Word 轉存為 PDF

不確定從哪一版 Word 開始(印象中從 2007 就有)，只要透過另存新檔的方式就可以將 Word 轉存為 PDF

> ![1saveas](https://user-images.githubusercontent.com/3851540/34535531-2221e8ce-f0fd-11e7-8368-d57874350be5.png)

## Word 轉 PDF 程式

1.  安裝 `Microsoft.Office.Interop.Word` NuGet 套件

    ![2nuget](https://user-images.githubusercontent.com/3851540/34535533-224e2b78-f0fd-11e7-9a64-79fdb5f17d7f.png)

2.  引用 `Microsoft.Office.Interop.Word`

    ```cs
    using Microsoft.Office.Interop.Word;
    ```

3.  程式碼

    ```cs
    // docx 檔案位置
    string sourcedocx = @"C:\sample.docx";
    // PDF 儲存位置
    string targetpdf = @"C:\output.pdf";
    
    //建立 word application instance
    Microsoft.Office.Interop.Word.Application appWord = new Microsoft.Office.Interop.Word.Application();
    //開啟 word 檔案
    var wordDocument = appWord.Documents.Open(sourcedocx);
    //匯出為 pdf
    wordDocument.ExportAsFixedFormat(targetpdf, dExportFormat.wdExportFormatPDF);
    //關閉 word 檔
    wordDocument.Close();
    //結束 word
    appWord.Quit();
    ```

4.  完整程式碼

    ```cs
    using Microsoft.Office.Interop.Word;
    
    namespace Word2PDF
    {
        class Program
        {
            static void Main(string[] args)
            {
                // word 檔案位置
                string sourcedocx = @"C:\sample.docx";
                // PDF 儲存位置
                string targetpdf = @"C:\output.pdf";
                
                //建立 word application instance
                Microsoft.Office.Interop.Word.Application appWord = new Microsoft.Office.Interop.Word.Application();
                //開啟 word 檔案
                var wordDocument = appWord.Documents.Open(sourcedocx);
                //匯出為 pdf
                wordDocument.ExportAsFixedFormat(targetpdf, WdExportFormat.wdExportFormatPDF);
                
                //關閉 word 檔
                wordDocument.Close();
                //結束 word
                appWord.Quit();
            }
        }
    }
    ```

## 出現多個 Word 背景執行作業 or 檔案使用中

1.  錯誤畫面

    ![3word](https://user-images.githubusercontent.com/3851540/34535534-22768e88-f0fd-11e7-9bfc-dc298975b229.png)

    ![4opening](https://user-images.githubusercontent.com/3851540/34535535-22a00a74-f0fd-11e7-905b-43a3ee858a95.png)

2.  問題原因：

    *  Word 背景執行 instance：未關閉 Word 元件
    *  Word 檔案使用中：Word 元件未關閉 Word 檔

3.  解決方式

    ```cs
    //關閉 word 檔
    wordDocument.Close();
    //結束 word
    appWord.Quit();
    ```

## 心得

經個人非正式測試 `.doc` 與 `.docx` 皆可正常使用，程式碼也非常簡短，只是嚴重缺點就是需要安裝 Word(印象中以前可以只安裝轉發套件，但此次測試發現無法使用，難道又是我記錯了嗎XD)，這樣一來活生生的就需要在 AP server 上安裝 office，再加上授權問題，可能不是每個團隊都適合這麼做

當然市面上有很多方便的套件可以做到 Word 轉 PDF 的功能，只是身為一個不想手動轉檔的偷懶工程師，要說服公司買套件或是自費購買安裝在公司 server 都不是件容易的事，所以擇日再來分享一下個人的免套件免安裝 office 做法

# 參考資訊

1.  [How to convert word document to pdf in C#](https://www.codeproject.com/questions/346784/how-to-convert-word-document-to-pdf-in-csharp)
