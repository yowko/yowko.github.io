---
title: "使用 C# 將 Excel 檔(.xlsx .xls) 轉換為 PDF"
date: 2018-03-12T21:00:00+08:00
lastmod: 2018-10-04T21:00:06+08:00
draft: false
tags: ["C#","Office"]
slug: "excel-to-pdf"
aliases:
    - /2018/03/excel-to-pdf.html
---
# 使用 C# 將 Excel 檔(.xlsx .xls) 轉換為 PDF
之前筆記 [使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF](https://blog.yowko.com/2018/01/c-sharp-word-to-pdf.html) 紀錄到該如何使用 Word 內建 API 將 Word 轉存為 PDF，後來有網友問到 Excel 及 PowerPoint 轉存的狀況，為了不要給出錯誤的答案，當然要實際測試看看才能確保正確性，而驗證的過程中發現 Excel 轉存程式跟 Word 有些落差，於是就多紀錄一篇了

## 使用 C# 將 Excel 轉 PDF

1.  NuGet 安裝 `Microsoft.Office.Interop.Excel` 套件

    > ![1nuget](https://user-images.githubusercontent.com/3851540/37274797-77a754a0-2618-11e8-8a8e-5635af07ad37.png)

2.  引用 `Microsoft.Office.Interop.Excel`

    ```cs
    using Microsoft.Office.Interop.Excel
    ```

3.  程式碼

    ```cs
    // Excel 檔案位置
    string sourcexlsx = @"D:\Downloads\003-圖表-ok.xlsx";
    // PDF 儲存位置
    string targetpdf = @"D:\Downloads\003-圖表-ok.pdf";
    
    //建立 Excel application instance
    Microsoft.Office.Interop.Excel.Application appExcel = new Microsoft.Office.Interop.Excel.Application();
    //開啟 Excel 檔案
    var xlsxDocument = appExcel.Workbooks.Open(sourcexlsx);
    //匯出為 pdf
    xlsxDocument.ExportAsFixedFormat(XlFixedFormatType.xlTypePDF, targetpdf);
    //關閉 Excel 檔
    xlsxDocument.Close();
    //結束 Excel
    appExcel.Quit();
    ```

4.  完整程式碼

    ```cs
    using Microsoft.Office.Interop.Excel;
    
    namespace xlsx2pdf
    {
        class Program
        {
            static void Main(string[] args)
            {
                // Excel 檔案位置
                string sourcexlsx = @"D:\Downloads\003-圖表-ok.xlsx";
                // PDF 儲存位置
                string targetpdf = @"D:\Downloads\003-圖表-ok.pdf";
                
                //建立 Excel application instance
                Microsoft.Office.Interop.Excel.Application appExcel = new Microsoft.Office.Interop.Excel.Application();
                //開啟 Excel 檔案
                var xlsxDocument = appExcel.Workbooks.Open(sourcexlsx);
                //匯出為 pdf
                xlsxDocument.ExportAsFixedFormat(XlFixedFormatType.xlTypePDF, targetpdf);
                //關閉 Excel 檔
                xlsxDocument.Close();
                //結束 Excel
                appExcel.Quit();
            }
        }
    }
    ```

## 實際效果

1.  原始 Excel

    > ![2excel1](https://user-images.githubusercontent.com/3851540/37274798-77d46aa8-2618-11e8-801c-cfc597b8c92b.png)

    > ![3excel2](https://user-images.githubusercontent.com/3851540/37274800-77fd8eb0-2618-11e8-81c7-73fb378e0154.png)

2.  匯出的 PDF

    > ![4pddf](https://user-images.githubusercontent.com/3851540/37274801-7825d7f8-2618-11e8-98c0-42f170cfee79.png)

*   以結果來看當然是跑版了，但如果透過預覽列印檢視，就可以發現原本就超出範圍，跑版並不是單純因為轉 PDF 而造成的

    > ![5print](https://user-images.githubusercontent.com/3851540/37274802-784e2d34-2618-11e8-9ae9-4b84da28616a.png)

## 心得

前文有提到 Word 與 Excel 的程式碼略有不同，不同內容如下

1.  namespace 不同
    *   word

        >Microsoft.Office.Interop.Word

    *   excel

        >Microsoft.Office.Interop.Excel

2.  開檔 api 不同
    *   word 使用 `Documents`

        >appWord.Documents.Open(sourcedocx);

    *   excel 使用 `Workbooks`

        >appExcel.Workbooks.Open(sourcexlsx);

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

# 參考資訊
1.  [使用 C# 將 Word 檔(.docx .doc) 轉換為 PDF](https://blog.yowko.com/2018/01/c-sharp-word-to-pdf.html)
