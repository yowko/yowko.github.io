---
title: "[Benchmark] DB 物件對映至 C# class 的做法"
date: 2019-01-01T00:42:34+08:00
lastmod: 2019-01-01T00:42:34+08:00
draft: false
tags: ["C#","Dapper","套件","Benchmark"]
slug: "object-relation-mapping"
---
# [Benchmark] DB 物件對映至 C# class 的做法
跟同事討論到 ORM 的優劣，當然各有擁護的對象，但相同的目標卻很一致：`速度快`，而在 `速度快` 這個基本前提下，我個人覺得還有討論空間： 

1. 程式執行速度高
2. 系統開發速度快

經過幾年工作後我自己遇到絕大多數的 user 都想同時滿足，但不幸的是兩者有著極高度 trade-off 的關係，可能得要視不同情境或是時機並提供完整資訊才能讓 user 做出正確選擇

剛好最近正在建構新系統的 DB access layer，所以比較了三種 ORM 方式

1. ADO.NET + 逐一手動 mapping
2. Dapper
3. ADO.NET + FastMember

，紀錄一下實際執行的效能數據、讓團隊擁有較多資訊才能做出較正確選擇

## 前提說明
1. .NET Core 2.2.101
2. 資料來源為 SQL Server 2014 範例資料庫 AdventureWorks2014 中的 Person.Person

    - 連線字串
    
        ```cs
        const string constr = "Data Source=.;Integrated Security=SSPI;Initial Catalog=AdventureWorks2014;";
        ```
    
    - model
        ```cs
        public class Person
        {
            public int BusinessEntityID { get; set; }

            public string PersonType { get; set; }

            public bool NameStyle { get; set; }

            public string Title { get; set; }

            public string FirstName { get; set; }

            public string MiddleName { get; set; }

            public string LastName { get; set; }

            public string Suffix { get; set; }

            public int EmailPromotion { get; set; }

            public string AdditionalContactInfo { get; set; }

            public string Demographics { get; set; }

            public Guid rowguid { get; set; }

            public DateTime ModifiedDate { get; set; }

        }
        ```

3. Dapper 1.50.5
4. FastMember 1.1.0 (如欲使用 BenchmarkDotNet 跑 benchmark 需自行產生 release 版，NuGet 上的為 Debug build)
5. 透過 BenchmarkDotNet 0.11.3 產生執行效率數據

## 做法一：ADO.NET + 逐一手動 mapping

```cs
public List<Person> MunualMapping()
{
    List<Person> result = new List<Person>();
    using (var conn = new SqlConnection(constr))
    using (var command = new SqlCommand("SELECT * FROM [AdventureWorks2014].[Person].[Person]", conn))
    {
        conn.Open();
        SqlDataReader reader = command.ExecuteReader();
        if (reader.HasRows)
        {
            while (reader.Read())
            {
                result.Add(ConvertToPerson(reader));
            }
        }
        reader.Close();
    }
    return result;
    
    //用來逐一轉換
    Person ConvertToPerson(SqlDataReader dr)
    {
        var _person = new Person()
        {
            BusinessEntityID = Convert.ToInt32(dr["BusinessEntityID"]),
            PersonType = Convert.ToString(dr["PersonType"]),
            NameStyle = Convert.ToBoolean(dr["NameStyle"]),
            Title = Convert.ToString(dr["Title"]),
            FirstName = Convert.ToString(dr["FirstName"]),
            MiddleName = Convert.ToString(dr["MiddleName"]),
            LastName = Convert.ToString(dr["LastName"]),
            Suffix = Convert.ToString(dr["Suffix"]),
            EmailPromotion = Convert.ToInt32(dr["EmailPromotion"]),
            AdditionalContactInfo = Convert.ToString(dr["AdditionalContactInfo"]),
            Demographics = Convert.ToString(dr["Demographics"]),
            rowguid = new Guid(Convert.ToString(dr["rowguid"])),
            ModifiedDate = Convert.ToDateTime(dr["ModifiedDate"])
        };
        return _person;
    }
}
```



## 使法二：Dapper

```cs
public List<Person> DapperMapping()
{
    List<Person> result = new List<Person>();
    using (IDbConnection db = new SqlConnection(constr))
    {
        result = db.Query<Person>("SELECT * FROM [AdventureWorks2014].[Person].[Person]").ToList();
    }
    return result;
}
```

## 使法三：ADO.NET + FastMember

```cs
public List<Person> FastMemberMapping()
{
    List<Person> result = new List<Person>();
    using (var conn = new SqlConnection(constr))
    using (var command = new SqlCommand("SELECT * FROM [AdventureWorks2014].[Person].[Person]", conn))
    {
        conn.Open();
        SqlDataReader reader = command.ExecuteReader();
        if (reader.HasRows)
        {
            while (reader.Read())
            {
                //透過 fastmember 建立 Person 類型的存取器
                var accessor = TypeAccessor.Create(typeof(Person));
                var _person = new Person();
                for (int i = 0; i < reader.FieldCount; i++)
                {
                    //取得 DB 欄位名稱
                    string propName = reader.GetName(i);
                    //取得 DB 值
                    var dbvalue = reader.GetValue(i);
                    //以下需依實際 DB 內容做調整
                    if (dbvalue is DBNull)
                    {
                        accessor[_person, propName] = null;
                    }
                    else
                    {
                        accessor[_person, propName] = dbvalue;
                    }
                }
                result.Add(_person);
            }
        }
        reader.Close();
    }
    return result;
}
```

## 執行效率數據

1. 第一次

    Method |     Mean |    Error |   StdDev |
    ------------------|---------:|---------:|---------:|
    MunualMapping | 434.6 ms | 7.724 ms | 7.225 ms |
    DapperMapping | <span style="color:lightgreen">419.0 ms</span> | 5.063 ms | 4.488 ms |
    FastMemberMapping | <span style="color:red">436.2 ms</span> | 4.996 ms | 4.172 ms |

2. 第二次

    Method |     Mean |    Error |   StdDev |
    ------------------ |---------:|---------:|---------:|
    MunualMapping | 433.8 ms | 6.329 ms | 5.920 ms |
    DapperMapping | <span style="color:red">436.4 ms</span> | 3.076 ms | 2.877 ms |
    FastMemberMapping | <span style="color:lightgreen">417.4 ms</span> | 7.937 ms | 8.493 ms |

3. 第三次

    Method |     Mean |    Error |    StdDev |
    ------------------ |---------:|---------:|----------:|
    MunualMapping | <span style="color:red">444.8 ms</span> | 8.834 ms | 20.996 ms |
    DapperMapping | 438.6 ms | 8.653 ms | 12.410 ms |
    FastMemberMapping | <span style="color:lightgreen">428.6 ms</span> | 3.509 ms |  2.930 ms |

4. 第四次

    Method |     Mean |    Error |   StdDev |
    ------------------ |---------:|---------:|---------:|
    MunualMapping | 434.4 ms | 5.808 ms | 5.432 ms |
    DapperMapping | <span style="color:red">441.7 ms </span>| 3.372 ms | 3.154 ms |
    FastMemberMapping | <span style="color:lightgreen">426.5 ms </span>| 5.486 ms | 5.132 ms |

5. 第五次

    Method |     Mean |    Error |   StdDev |
    ------------------ |---------:|---------:|---------:|
    MunualMapping | <span style="color:red">435.5 ms</span> | 5.696 ms | 5.328 ms |
    DapperMapping | 431.4 ms | 3.146 ms | 2.943 ms |
    FastMemberMapping | <span style="color:lightgreen">427.4 ms</span> | 3.687 ms | 3.448 ms |

## 心得
前幾年看過 Dapper 做的 ORM benchmark，印象中是手動 mapping 最快，Dapper 僅排第三，最近再看一次 ORM benchmark 排名大改，多了不少之前沒用過的 ORM framework，這待日後有機會再來研究

這次專案不考慮 Dapper 以外的 ORM framework 主要是穩定性及開發上的熟悉度，擔心冒然使用新的 ORM 反而出現部份情境不支援的問題或是底層的 bug ，造成開發時程延宕

就這次測試結果來看

1. 程式執行速度

    >我會將三者執行效率視為接近或是一致，速度差距並不顯著，尤其加上考慮標準差影響後更是如此 
2. 系統開發速度

    > Dapper 明顯程式碼少很多，也沒有手動 mapping 可能出現的打錯字問題及 FastMember 需要自行處理型別特性的狀況

# 參考資訊
1. [Fastest way to map result of SqlDataReader to object](https://stackoverflow.com/a/44853182)