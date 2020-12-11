---
title: "用 C# 將 PowerPoint 檔(.pptx .ppt) 轉換為 PDF"
date: 2018-03-13T01:30:00+08:00
lastmod: 2020-12-11T11:00:34+08:00
draft: false
tags: ["C#","Office"]
slug: "ppt-to-pdf"
aliases:
    - /2018/03/ppt-to-pdf.html
---
# 用 C# 將 PowerPoint 檔(.pptx .ppt) 轉換為 PDF
之前筆記 [使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF](/2018/01/c-sharp-word-to-pdf.html) 紀錄到該如何使用 Word 內建 API 將 Word 轉存為 PDF，後來有網友問到 Excel 及 PowerPoint 轉存的狀況，為了不要給出錯誤的答案，當然要實際測試看看比較準囉，其中 Excel 轉存 PDF 程式碼已紀錄在 [使用 C# 將 Excel 檔(.xlsx .xls) 轉換為 PDF](/2018/03/excel-to-pdf.html)，接著就是來看看 C# 在處理 PowerPoint 轉 PDF 有什麼不同吧

## 使用 C# 將 PowerPoint 轉為 PDF

1.  安裝 NuGet 套件 `Microsoft.Office.Interop.PowerPoint`

    > ![1nuget](https://user-images.githubusercontent.com/3851540/37319754-5aeb3532-26ab-11e8-90bd-09f9d48f56ee.png)

2.  引用 `Microsoft.Office.Interop.PowerPoint`

    ```cs
    using Microsoft.Office.Interop.PowerPoint;
    ```

3.  程式碼

    ```cs
    // pptx 檔案位置
    string sourcepptx = @"D:\Downloads\Docker.pptx";
    // PDF 儲存位置
    string targetpdf = @"D:\Downloads\Docker.pdf";
    
    //建立 PowerPoint application instance
    Microsoft.Office.Interop.PowerPoint.Application appPPT = new Microsoft.Office.Interop.PowerPoint.Application();
    
    //開啟 PowerPoint 檔案
    var pptSlide = appPPT.Presentations.Open(sourcepptx);
    
    //匯出為 pdf
    pptSlide.ExportAsFixedFormat(targetpdf, PpFixedFormatType.ppFixedFormatTypePDF);
    //關閉 PowerPoint 檔
    pptSlide.Close();
    //結束 PowerPoint
    appPPT.Quit();
    ```

4.  完整程式碼

    ```cs
    using Microsoft.Office.Interop.PowerPoint;
    
    namespace pptx2pdf
    {
        class Program
        {
            static void Main(string[] args)
            {
                // pptx 檔案位置
                string sourcepptx = @"D:\Downloads\Docker.pptx";
                // PDF 儲存位置
                string targetpdf = @"D:\Downloads\Docker.pdf";
                
                //建立 PowerPoint application instance
                Microsoft.Office.Interop.PowerPoint.Application appPPT = new Microsoft.Office.Interop.PowerPoint.Application();
                
                //開啟 PowerPoint 檔案
                var pptSlide = appPPT.Presentations.Open(sourcepptx);
                
                //匯出為 pdf
                pptSlide.ExportAsFixedFormat(targetpdf, PpFixedFormatType.ppFixedFormatTypePDF);
                //關閉 PowerPoint 檔
                pptSlide.Close();
                //結束 PowerPoint
                appPPT.Quit();
            }
        }
    }
    ```

## 出現 `MsoTriState` 沒有加入參考錯誤

1.  錯誤訊息

    ```
    The type 'MsoTriState' is defined in an assembly that is not referenced. You must add a reference to assembly 'office, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c'.
    ```

2.  錯誤截圖

    > ![2notreference](https://user-images.githubusercontent.com/3851540/37319755-5b1523b0-26ab-11e8-985c-b8a776437e63.png)

3.  加入 `office.dll` 參考

    > ![3addref](https://user-images.githubusercontent.com/3851540/37319756-5b4c5722-26ab-11e8-9fb8-184bed4d2e61.png)

    *   DLL 位置

        ```
        C:\Windows\assembly\GAC_MSIL\office\15.0.0.0__71e9bce111e9429c
        ```

## 心得

前文有提到 Word 、 Excel 、PowerPoint 的程式碼略有不同，不同內容如下

1.  namespace 不同
    *   word

        >Microsoft.Office.Interop.Word

    *   excel

        >Microsoft.Office.Interop.Excel

    *   powerpoint

        >Microsoft.Office.Interop.PowerPoint

2.  開檔 api 不同
    *   word 使用 `Documents`

        >appWord.Documents.Open(sourcedocx);

    *   excel 使用 `Workbooks`

        >appExcel.Workbooks.Open(sourcexlsx);

    *   powerpoint 使用 `Presentations`

        >appPPT.Presentations.Open(sourcepptx);

3.  export function 參數不同
    *   word
        *   先指定 pdf 儲存路徑
        *   再使用 `dExportFormat.wdExportFormatPDF`

        ```cs
        wordDocument.ExportAsFixedFormat(targetpdf, dExportFormat.wdExportFormatPDF);
        ```

    *   excel
        *   先指定匯出格式 `XlFixedFormatType.xlTypePDF`
        *   再指定 pdf 儲存路徑

        ```cs
        xlsxDocument.ExportAsFixedFormat(XlFixedFormatType.xlTypePDF, targetpdf);
        ```
    *   powerpoint
        *   先指 pdf 儲存路徑
        *   再指定匯出格式 `PpFixedFormatType.ppFixedFormatTypePDF`

        ```cs
        pptSlide.ExportAsFixedFormat(targetpdf, PpFixedFormatType.ppFixedFormatTypePDF);
        ```

*   測試下來，只有 PowerPoint 需要加入 office.dll 的參考


依我個人的實際測試下來， PowerPoint 跟 Word 都沒有跑版問題(可能我的測試 case 沒用到太特殊的物件)，而 Excel 跑版狀況就滿嚴重的，可能是 Excel 的使用情境本來就容易出現橫向延伸的狀況

# 參考資訊

1.  [使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF](/2018/01/c-sharp-word-to-pdf.html)
2.  [使用 C# 將 Excel 檔(.xlsx .xls) 轉換為 PDF](/2018/03/excel-to-pdf.html)
