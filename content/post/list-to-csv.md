---
title: "使用 C# 將資料匯出為 CSV"
date: 2018-04-04T15:21:00+08:00
lastmod: 2018-10-04T15:21:11+08:00
draft: false
tags: ["C#"]
slug: "list-to-csv"
aliases:
    - /2018/04/list-to-csv.html
---
# 使用 C# 將資料匯出為 CSV
最近有個需求是將部份資料內容倒進其他系統中，主要計劃是打算透過目標系統所開發的 restful api 來 insert 資料，但在實際透過 api 交換資料之前最重要的工作當然就是確保資料格式跟內容都是正確的，經過討論後決定將資料匯出成 CSV 讓目標系統相關人員 review 匯出的資料內容是否合乎要求

收到這個需求時，雖然知道難度應該是不高，只是沒有很確定的想法，還試著找過有沒有好用的套件，最後還是索性自己動手，順手紀錄一下

## 前提基本設定

1.  基本 model

    ```cs
    class UserData
    {
        public string Name { get; set; }
        public int Salary { get; set; }
        public DateTime Birth { get; set; }
    }
    ```

2.  製造假資料

    ```cs
    List<UserData> users = new List<UserQuery.UserData>() {
        new UserData(){Name="Test",Salary=100,Birth=new DateTime(2000,1,1)},
        new UserData(){Name="Demo",Salary=200,Birth=new DateTime(2001,7,31)},
    };
    ```

## 實際使用 C# 將資料匯出為 CSV

1.  方法一：手動組裝寫入內容

    *   手動組裝方法

        ```cs
        void WriteToCSV(string FilePath, List<UserData> data)
        {
            using (var file = new StreamWriter(FilePath))
            {
                foreach (var item in data)
                {
                    file.WriteLineAsync($"{item.Name},{item.Salary},{item.Birth}");
                }
            }
        }
        ```

    *   呼叫方式

        ```cs
        string filepath = @"D:\tmp\write.csv";
        WriteToCSV(filepath,users);
        ```

2.  方法二：覆寫 model 的 ToString
    *   覆寫 model

        ```cs
        class UserData
        {
            public string Name { get; set; }
            public int Salary { get; set; }
            public DateTime Birth { get; set; }
            public override string ToString()
            {
                return $"{this.Name},{this.Salary},{this.Birth}";
            }
        }
        ```

    *   寫入檔案時直接將 object ToString 即可

        ```cs
        void ModelToCSV(string FilePath, List<UserData> data)
        {
            using (var file = new StreamWriter(FilePath))
            {
                foreach (var item in data)
                {
                    file.WriteLineAsync(item.ToString());
                }
            }
        }
        ```

    *   呼叫方式

        ```cs
        string filepath = @"D:\tmp\model.csv";
        ModelToCSV(filepath, users);
        ```

3.  方法三：新增使用 generic 及 reflection 自動組裝的 helper

    *   helper 內容

        ```cs
        /// <summary>
        /// CSV Generator
        /// </summary>
        /// <param name="genColumn">output data property name</param>
        /// <param name="FilePath">target CSV path</param>
        /// <param name="data"> List of T</param>
        void CSVGenerator<T>(bool genColumn, string FilePath, List<T> data)
        {
            using (var file = new StreamWriter(FilePath))
            {
                Type t = typeof(T);
                PropertyInfo[] propInfos = t.GetProperties(BindingFlags.Public | BindingFlags.Instance);
                //是否要輸出屬性名稱
                if (genColumn)
                {
                    file.WriteLineAsync(string.Join(",", propInfos.Select(i => i.Name)));
                }
                foreach (var item in data)
                {
                    file.WriteLineAsync(string.Join(",", propInfos.Select(i => i.GetValue(item))));
                }
            }
        }
        ```

    *   呼叫方式

        ```cs
        string filepath = @"D:\tmp\helper.csv";
        CSVGenerator<UserData>(true, filepath, users);
        ```

## 心得

雖然這個需求來得很急，一度真的考慮自己手工組產出的內容，但最後還是過不了自己這關，最後採取使用 generic 及 reflection 來處理這個需求，寫完後發現在目標 model 屬性很多的情況下反而還比手工一個個屬性串接還來得快，加上程式碼也相對整潔乾淨，再次證實自己的堅持是正確的

# 參考資訊

1.  [Export List<t> to a CSV</t>](https://social.msdn.microsoft.com/Forums/vstudio/en-US/732a1e19-5055-4b07-b1dc-86d5a9fe6fef/export-listt-to-a-csv?forum=csharpgeneral)
2.  [Create a CSV File from a .NET Generic List](http://www.csharptocsharp.com/generate-csv-from-generic-list)
