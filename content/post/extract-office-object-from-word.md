---
title: "使用 C# 取出 Word (.docx) 中的內嵌 Office 物件"
date: 2018-01-18T02:46:00+08:00
lastmod: 2018-10-02T02:46:36+08:00
draft: false
tags: ["C#","Office"]
slug: "extract-office-object-from-word"
aliases:
    - /2018/01/extract-office-object-from-word.html
---
# 使用 C# 取出 Word (.docx) 中的內嵌 Office 物件
之前筆記 [取得 Word(.docx) 中的內嵌檔案](https://blog.yowko.com/2018/01/extract-embedded-file-in-word.html) 紀錄到如何在 word 中嵌入其他物件，也提到如何簡易地取出內嵌物件

今天則是要紀錄如何使用 C# 將 Word 中的內嵌物件取出

## C# 完整程式碼

```cs
// word 原始檔案位置
var sourcefile = @"C:\wordtest\Demo.docx";
// 內嵌檔案預計轉存目標位置
var targetfolder = @"C:\wordtest\";
//使用 System.IO.Packaging.Package 開啟 word
using (Package pkg = Package.Open(sourcefile))
{
    // 掃過 word 的所有物件
    foreach (var pkgPart in pkg.GetParts())
    {
        //取得 word 物件的完整名稱
        var oleobjectname = pkgPart.Uri.ToString();
        //物件名稱包含 `embeddings` (為內嵌物件)
        if (oleobjectname.Contains("embeddings"))
        {
            //取得內嵌物件檔案名稱
            var tmpfilename = oleobjectname.Split('/').LastOrDefault();
            //內嵌物件轉存目標完整檔名
            var targetpath = Path.Combine(targetfolder, tmpfilename);
            //建立目標轉存物件
            using (System.IO.FileStream writeStream = new System.IO.FileStream(targetpath, FileMode.Create))
            {
                //將內嵌物件內容存入目標物件
                pkgPart.GetStream().CopyTo(writeStream);
            };

        }
    }
    pkg.Close();
}
```

## 心得

透過上面紀錄的程式碼可以將 Office 物件轉存出來，但我僅測試過 excel 部份，加上 excel 內容也是為了測試而隨意輸入的內容，不確定更複雜的資料內容是否一樣可以順利完成，這點要特別留意

在查資料的過程中，相關 api 的使用方式網路上並不好找，個人不負責猜測可能是類似功能的付費套件很普遍，功能也異常強大的關係，讓多數人都願意買單所造成的，過程中我也一度想要付錢解決了事XD，幸虧最後還是找到尚可接受的做法

# 參考資訊

1.  [取得 Word(.docx) 中的內嵌檔案](https://blog.yowko.com/2018/01/extract-embedded-file-in-word.html)
2.  [c# OpenXML Example of Retrieving an embedded word doc's relationship info](https://social.msdn.microsoft.com/Forums/windows/en-US/3b061fdf-7b10-4aeb-96e6-883593aaea87/c-openxml-example-of-retrieving-an-embedded-word-docs-relationship-info?forum=oxmlsdk)
