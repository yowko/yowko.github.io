---
title: "匯出 Excel 時使用多國語系 Resource 當做欄位名稱"
date: 2018-04-06T21:53:00+08:00
lastmod: 2020-12-11T21:53:14+08:00
draft: false
tags: ["C#","Office"]
slug: "reflection-resource-culture"
aliases:
    - /2018/04/reflection-resource-culture.html
---
# 匯出 Excel 時使用多國語系 Resource 當做欄位名稱
之前筆記 [使用 C# 將資料匯出成 Excel (.xlsx)](/2018/04/list-to-excel.html) 紀錄到使用 ClosedXML 搭配 generic 與 reflection 匯出 excel，方便使用者自行調整資料報表

剛好有個系統需要支援多國語系，連帶地使用者對要求 excel 也需要有多國語系，雖然資料內容無法隨意改變，但 excel 的欄位名稱就有調整的空間，就來看看可以如何處理吧

## 使用 Resource
1. 加入 Resource
    - 專案上按右鍵 --> Add --> New Item...
        
        >![1newitem](https://user-images.githubusercontent.com/3851540/38173737-cadffab8-35f5-11e8-86eb-3fa7de6b0175.png) 
    - Visual C# Items --> Resource File --> 指定 .resx 名稱
        
        >![2addresx](https://user-images.githubusercontent.com/3851540/38173738-cb0ac6da-35f5-11e8-9db0-8d61b494dbde.png)
2. 使用 ResXManager 來管理多國語系文字
    - 搜尋並安裝 `ResXManager` 
        
        >![3addnuget](https://user-images.githubusercontent.com/3851540/38173739-cb34d83a-35f5-11e8-957c-78bbf92fe25c.png) 
    - VS 主選單 Tools --> ResX Manager
        
        >![4openresxmanager](https://user-images.githubusercontent.com/3851540/38173740-cb615acc-35f5-11e8-86a9-db9f14ecf26e.png) 
    - 加入新語言
        
        >![5addlang](https://user-images.githubusercontent.com/3851540/38173741-cb8bbbaa-35f5-11e8-9528-e4a627ac6971.png)

        >![6selectlang](https://user-images.githubusercontent.com/3851540/38173742-cbb57058-35f5-11e8-8da8-a1fe67e59407.png)
    - 加入新字串
        
        >![7addkey](https://user-images.githubusercontent.com/3851540/38173743-cbdfc5d8-35f5-11e8-8dff-985eb9ff01be.png)
        
        >![9resxmanager](https://user-images.githubusercontent.com/3851540/38173745-cc4ad2ba-35f5-11e8-8b9b-468e062d66f7.png)
        
        - 第一次填寫不同語言時會提示建立 Resource 檔案
            
            >![8addresxfile](https://user-images.githubusercontent.com/3851540/38173744-cc23041a-35f5-11e8-9514-4e72c1561946.png)
        
## 將多國語系 Resource 加入 Excel 匯出
1. excel helper 直接取用 resource 內容
    - 範本
        
        ```cs
        Resource.ResourceManager.GetString({resource key}, {target CultureInfo};
        ```
    - 實例
        
        ```cs
        Resource.ResourceManager.GetString("Name", new CultureInfo("zh-TW"));
        ```
     
2. 將 resource 相關資訊傳入 excel helper
    - 調整 excel helper
        - 允許可傳入 ResourceManager 及 CultureInfo
            
            ```cs
            ResourceManager _rm;
            CultureInfo _ci;
            public XSLXHelper() : this(null, null)
            {
            }
            public XSLXHelper(ResourceManager rm, CultureInfo ci)
            {
                _rm = rm;
                _ci = ci;
            }
            ```
        - 調整取欄位邏輯
            
            ```cs
            if (_rm != null)
            {
                //依 resource key 及 CultureInfo 取得 resource 內容
                string resourceName = _ci == null ? _rm.GetString(item.Name) : _rm.GetString(item.Name, _ci);
                //如 resource key 取不到資料則使用屬性名稱
                sheet.Cell(1, colIdx++).Value = string.IsNullOrWhiteSpace(resourceName) ? item.Name : resourceName;
            }
            else
                sheet.Cell(1, colIdx++).Value = item.Name;
            ```
        - 程式呼叫
            
            ```cs
            XSLXHelper helper = new XSLXHelper(new System.Resources.ResourceManager(typeof(Resource)), new CultureInfo("zh-TW"));
            ```
        - 完整程式碼
            
            ```cs
            using ClosedXML.Excel;
            using System;
            using System.Collections.Generic;
            using System.Globalization;
            using System.Resources;
            
            namespace TestHelper
            {
                public class XSLXHelper
                {
                    ResourceManager _rm;
                    CultureInfo _ci;
                    public XSLXHelper() : this(null, null)
                    {
                    }
                    public XSLXHelper(ResourceManager rm, CultureInfo ci)
                    {
                        _rm = rm;
                        _ci = ci;
                    }
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
                            #region - 直接使用物件屬性名稱 -
                            if (_rm != null)
                            {
                                //依 resource key 及 CultureInfo 取得 resource 內容
                                string resourceName = _ci == null ? _rm.GetString(item.Name) : _rm.GetString(item.Name, _ci);
                                //如 resource key 取不得資料則使用屬性名稱
                                sheet.Cell(1, colIdx++).Value = string.IsNullOrWhiteSpace(resourceName) ? item.Name : resourceName;
                            }
                            else
                                sheet.Cell(1, colIdx++).Value = item.Name;
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
            
            ```cs
            //想要匯出的資料集
            var users = new List<User>() {
                new User(){Name="Yowko",Salary=10,Address="Taipei",Birthday=new DateTime(1983,7,29) },
                new User(){Name="Test",Salary=20,Address="USA",Birthday=new DateTime(1993,7,29) },
            };
            //xlsx 檔案位置
            string filepath = $@"./{DateTime.Now.ToString("yyyyMMddHHmmss")}.xlsx";

            //建立 xlxs 轉換物件
            XSLXHelper helper = new XSLXHelper(new System.Resources.ResourceManager(typeof(Resource)), new CultureInfo("zh-TW"));
            //取得轉為 xlsx 的物件
            var xlsx=helper.Export(users);

            //存檔至指定位置
            xlsx.SaveAs(filepath);
            ```

## 心得
在 excel helper 中直接依 CultureInfo 取得 resource 內容適用於 excel helper 與 Resource 在同個專案中，如果像同事遇到的狀況： excel helper 獨立存在於 library 專案中沒有 resource 概念時，就可以透過將 ResourceManager 與 CultureInfo 傳入來處理

重新 review 程式碼，實際的程式碼並不多，概念與難度也不難，但卻花好幾個小時才找到真正的做法，我想主要原因是過往工作缺乏多國語系需求的關係，明顯經驗與相關知識都不足，而這就是做專案的趣味所在，常常有意外的需求來增加工作樂趣跟自我挑戰

# 參考資訊
1. [多國語系 - 使用ResXManager管理資源檔](https://dotblogs.com.tw/wasichris/2015/06/19/151598)
2. [ResourceManager 類別](https://msdn.microsoft.com/zh-tw/library/system.resources.resourcemanager%28v=vs.110%29.aspx)
3. [Get Resources with string](https://stackoverflow.com/questions/17828774/get-resources-with-string)