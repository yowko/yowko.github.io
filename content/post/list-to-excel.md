---
title: "使用 C# 將資料匯出成 Excel (.xlsx)"
date: 2018-04-05T22:17:00+08:00
lastmod: 2021-11-02T22:17:15+08:00
draft: false
tags: ["csharp","Office"]
slug: "list-to-excel"
aliases:
    - /2018/04/list-to-excel.html
---
## 使用 C# 將資料匯出成 Excel (.xlsx)

雖然大部份系統都會有報表相關功能，只是多數情況都無法在系統建置時就設想到所有使用者需求，加上常常功能需求的優先程度會被報表高不少，所以就會透過將資料匯出成 Excel 讓使用者自行組裝成需要的格式及內容以應付需求就成了最有效率的方式

針對匯出 excel 的需求，前幾天同事問到相關問題，才發現我連匯出 excel 的基本功能沒有做過筆記，所以立馬來補紀錄一下囉

## 1. 安裝 ClosedXML

ClosedXML 是用來處理 Excel 2007+ (.xlsx, .xlsm, etc) 非常方便的套件，除此之外套件的命名上也饒富趣味：刻意與 `Open XML` 唱反調取名為 `ClosedXML` (從 Microsoft Office 2007 開始，Office Open XML 檔案格式已經成為 Microsoft Office 預設的檔案格式，而 Office Open XML（縮寫：Open XML、OpenXML 或 OOXML），為由 Microsoft 開發的一種以 XML 為基礎並以 ZIP 格式壓縮的電子檔案規範，支援 spreadsheets, charts, presentations and word processing 等檔案格式。)

- 使用 NuGet 搜尋 `ClosedXML` 並安裝

    >![1nuget](https://user-images.githubusercontent.com/3851540/38164065-0932a1e6-3531-11e8-80d4-df1bc0c81044.png)

## 2. 建立產生 excel 用的 helper

> 這邊是主要核心及重點

- 主要程式

    ```cs
    public class XSLXHelper
    {
        /// <summary>
        /// 產生 excel
        /// </summary>
        /// <typeparam name="T">傳入的物件型別</typeparam>
        /// <param name="data">物件資料集</param>
        /// <returns></returns>
        public XLWorkbook Export<T>(List<T> data)
        {
            //建立 excel 物件
            XLWorkbook workbook = new XLWorkbook();
            //加入 excel 工作表名為 `Report`
            var sheet = workbook.Worksheets.Add("Report");
            //欄位起啟位置
            int colIdx = 1;
            //使用 reflection 將物件屬性取出當作工作表欄位名稱
            foreach (var item in typeof(T).GetProperties())
            {
                #region - 可以使用 DescriptionAttribute 設定，找不到 DescriptionAttribute 時改用屬性名稱 -
                //可以使用 DescriptionAttribute 設定，找不到 DescriptionAttribute 時改用屬性名稱
                //DescriptionAttribute description = item.GetCustomAttribute(typeof(DescriptionAttribute)) as DescriptionAttribute;
                //if (description != null)
                //{
                //    sheet.Cell(1, colIdx++).Value=description.Description;
                //    continue;
                //}
                //sheet.Cell(1, colIdx++).Value = item.Name;
                #endregion
                #region - 直接使用物件屬性名稱 -
                //或是直接使用物件屬性名稱
                sheet.Cell(1, colIdx++).Value=item.Name;
                #endregion
    
            }
            //資料起始列位置
            int rowIdx = 2;
            foreach (var item in data)
            {
                //每筆資料欄位起始位置
                int conlumnIndex = 1;
                foreach (var jtem in item.GetType().GetProperties())
                {
                    //將資料內容加上 "'" 避免受到 excel 預設格式影響，並依 row 及 column 填入
                    sheet.Cell(rowIdx, conlumnIndex).Value = string.Concat("'", Convert.ToString(jtem.GetValue(item, null)));
                    conlumnIndex++;
                }
                rowIdx++;
            }
            return workbook;
        }
    }
    ```

- 完整程式碼

    ```cs
    using ClosedXML.Excel;
    using System;
    using System.Collections.Generic;
    using System.ComponentModel;
    using System.Reflection;
    
    namespace GenXLSX
    {
        public class XSLXHelper
        {
            /// <summary>
            /// 產生 excel
            /// </summary>
            /// <typeparam name="T">傳入的物件型別</typeparam>
            /// <param name="data">物件資料集</param>
            /// <returns></returns>
            public XLWorkbook Export<T>(List<T> data)
            {
                //建立 excel 物件
                XLWorkbook workbook = new XLWorkbook();
                //加入 excel 工作表名為 `Report`
                var sheet = workbook.Worksheets.Add("Report");
                //欄位起啟位置
                int colIdx = 1;
                //使用 reflection 將物件屬性取出當作工作表欄位名稱
                foreach (var item in typeof(T).GetProperties())
                {
                    #region - 可以使用 DescriptionAttribute 設定，找不到 DescriptionAttribute 時改用屬性名稱 -
                    //可以使用 DescriptionAttribute 設定，找不到 DescriptionAttribute 時改用屬性名稱
                    //DescriptionAttribute description = item.GetCustomAttribute(typeof(DescriptionAttribute)) as DescriptionAttribute;
                    //if (description != null)
                    //{
                    //    sheet.Cell(1, colIdx++).Value=description.Description;
                    //    continue;
                    //}
                    //sheet.Cell(1, colIdx++).Value = item.Name;
                    #endregion
                    #region - 直接使用物件屬性名稱 -
                    //或是直接使用物件屬性名稱
                    sheet.Cell(1, colIdx++).Value=item.Name;
                    #endregion
    
                }
                //資料起始列位置
                int rowIdx = 2;
                foreach (var item in data)
                {
                    //每筆資料欄位起始位置
                    int conlumnIndex = 1;
                    foreach (var jtem in item.GetType().GetProperties())
                    {
                        //將資料內容加上 "'" 避免受到 excel 預設格式影響，並依 row 及 column 填入
                        sheet.Cell(rowIdx, conlumnIndex).Value = string.Concat("'", Convert.ToString(jtem.GetValue(item, null)));
                        conlumnIndex++;
                    }
                    rowIdx++;
                }
                return workbook;
            }
        }
    }
    ```

## 3. 呼叫方式

```cs
//想要匯出的資料集
var users = new List<User>() {
    new User(){Name="Yowko",Salary=10,Addr="Taipei",Birthday=new DateTime(1983,7,29) },
    new User(){Name="Test",Salary=20,Addr="USA",Birthday=new DateTime(1993,7,29) },
};
//xlsx 檔案位置
string filepath = $@"./{DateTime.Now.ToString("yyyyMMddHHmmss")}.xlsx";

//建立 xlxs 轉換物件
XSLXHelper helper = new XSLXHelper();
//取得轉為 xlsx 的物件
var xlsx=helper.Export(users);

//存檔至指定位置
xlsx.SaveAs(filepath);
```

## 4. 實際結果

- 測試 model

    ```cs
    public class User
    {
        [Description("姓名")]
        public string Name { get; set; }
        [Description("薪資")]
        public int Salary { get; set; }
        [Description("生日")]
        public DateTime Birthday { get; set; }
        public string Addr { get; set; }
    }
    ```

1. 使用 Description 搭配屬性名稱

    >![2desc](https://user-images.githubusercontent.com/3851540/38164067-095bd69c-3531-11e8-91f8-ccc0513b2be9.png)
2. 直接使用屬性名稱

    >![3property](https://user-images.githubusercontent.com/3851540/38164068-099de9ce-3531-11e8-83fa-7a4cb03c9e5d.png)

## 心得

簡單不多的程式碼讓匯出 excel 變得輕鬆，加上使用 generic 與 reflection 讓適用情境大增，非常方便

當然也可能會遇到更多需要調整的情境：像是調整 excel 寬度、底色或是自訂 attribute 來客製輸出內容在 ClosedXML 與 reflection 幫助下都可以輕易辦到的，下次有機會再來紀錄比較不同的使用方式

## 參考資訊

1. [Office Open XML](https://zh.wikipedia.org/wiki/Office_Open_XML)
2. [ClosedXML/ClosedXML](https://github.com/ClosedXML/ClosedXML)
