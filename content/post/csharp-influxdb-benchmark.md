---
title: "[Benchmark] 使用 C# 對 InfluxDB insert 操作的效能數據"
date: 2019-09-22T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["csharp","InfluxDB","Benchmark"]
slug: "csharp-influxdb-benchmark"
---

## [Benchmark] 使用 C# 對 InfluxDB insert 操作的效能數據

之前筆記 [使用 C# 存取 InfluxDB](/csharp-influxdb-curd/) 紀錄了 C# 在 InfluxDB 的基本 CRUD，也提到新專案可能會使用 InfluxDB 儲存資料，在了解 C# 的基本用法後接著就來確認一下 C# 與 InfluxDB 這個組合在效能是否可以跟得上規劃中的系統需求

今天測試的 InfluxDB benchmark 是之前筆記 [[Benchmark] 使用 C# 對 NoSQL insert 操作的效能數據](/nosql-insert-benchmark/) 的延伸，大致上測試方式情境都會承襲於之前內容

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. Docker for Mac 2.1.0.2 (37877)
4. InfluxDB v1.7.8
5. NuGet package

    - InfluxData.Net 8.0.1
    - BenchmarkDotNet 0.11.5
    - Bogus 28.3.1

6. 測試用程式碼

    - User.cs

        ```cs
        public class User
        {
            public Guid UserId { get; set; }
            public string Name { get; set; }
            public DateTime Birthday { get; set; }
            public decimal CurrentSalary { get; set; }
        }
        ```

    - NoSQLAbstract.cs

        ```cs
        public abstract class NoSQLAbstract
        {
            [Params(100, 10000, 1000000)]
            public int Times { get; set; }
            public abstract Task InsertTest();
        }
        ```

    - InfluxDBTest.cs

        ```cs
        public class InfluxDBTest: NoSQLAbstract
        {
            [Benchmark]
            public override async Task InsertTest()
            {
                var influxDbClient = new InfluxDbClient("http://localhost:8086/", "admin", "pass.123", InfluxDbVersion.Latest);

                var dbName = "benchmark";
               // var response = await influxDbClient.Database.CreateDatabaseAsync(dbName);

                var fakesUsers = UserUtility.GetFakeUsers(Times);

                foreach (var userList in UserUtility.SpiltBySize(UserUtility.GetFakeUsers(Times), 10000))
                {
                    var pointToWrites =
                        userList.Select(a => new Point()
                        {
                            Name = "users",

                            //Tags = a.ToDictionary(),
                            Fields = a.ToDictionary()
                        });

                    await influxDbClient.Client.WriteAsync(pointToWrites, dbName);
                }
            }
        }
        ```

    - PocoToDictionary.cs

        > 這是之前筆記 [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary) 測試轉型做法，找出較佳的做法

        ```cs
        public static class PocoToDictionary
        {
            private static readonly MethodInfo AddToDictionaryMethod = typeof(IDictionary<string, object>).GetMethod("Add");
            private static readonly ConcurrentDictionary<Type, Func<object, IDictionary<string, object>>> Converters = new ConcurrentDictionary<Type, Func<object,  IDictionary<string, object>>>();
            private static readonly ConstructorInfo DictionaryConstructor = typeof(Dictionary<string, object>).GetConstructors().FirstOrDefault(c => c.IsPublic &&  !c.GetParameters().Any());

            public static IDictionary<string, object> ToDictionary(this object obj) => obj == null ? null : Converters.GetOrAdd(obj.GetType(), o =>
            {
                var outputType = typeof(IDictionary<string, object>);
                var inputType = obj.GetType();
                var inputExpression = Expression.Parameter(typeof(object), "input");
                var typedInputExpression = Expression.Convert(inputExpression, inputType);
                var outputVariable = Expression.Variable(outputType, "output");
                var returnTarget = Expression.Label(outputType);
                var body = new List<Expression>
                {
                    Expression.Assign(outputVariable, Expression.New(DictionaryConstructor))
                };
                body.AddRange(
                    from prop in inputType.GetProperties(BindingFlags.Instance | BindingFlags.Public | BindingFlags.FlattenHierarchy)
                    where prop.CanRead && (prop.PropertyType.IsPrimitive || prop.PropertyType == typeof(string))
                    let getExpression = Expression.Property(typedInputExpression, prop.GetMethod)
                    select Expression.Call(outputVariable, AddToDictionaryMethod, Expression.Constant(prop.Name), getExpression));
                body.Add(Expression.Return(returnTarget, outputVariable));
                body.Add(Expression.Label(returnTarget, Expression.Constant(null, outputType)));

                var lambdaExpression = Expression.Lambda<Func<object, IDictionary<string, object>>>(
                    Expression.Block(new[] { outputVariable }, body),
                    inputExpression);

                return lambdaExpression.Compile();
            })(obj);
        }
        ```

    - UserUtility.cs

        ```cs
        public static class UserUtility
        {
            public static List<User> GetFakeUsers(int times)
            {
                var fakesUsers = new Faker<User>()
                    .RuleFor(a => a.UserId, f => f.Random.Guid())
                    .RuleFor(a => a.Name, f => f.Name.FirstName())
                    .RuleFor(a => a.Birthday, f => f.Date.Past())
                    .RuleFor(a => a.CurrentSalary, f => f.Random.Decimal(0M, 10000M))
                    .Generate(times);

                return fakesUsers;
            }

            public static List<T>[] SpiltBySize<T>(List<T> list, int size)
            {
                if (list == null)
                    throw new ArgumentNullException(nameof(list));

                if (size < 1)
                    throw new ArgumentOutOfRangeException("totalPartitions");

                var count = (int) Math.Ceiling(list.Count / (double) size);
                var partitions = new List<T>[count];

                var k = 0;
                for (var i = 0; i < partitions.Length; i++)
                {
                    partitions[i] = new List<T>(size);
                    for (var j = k; j < k + size; j++)
                    {
                        if (j >= list.Count)
                            break;
                        partitions[i].Add(list[j]);
                    }

                    k += size;
                }

                return partitions;
            }
        }
        ```

    - 執行 benchmark

        ```cs
        BenchmarkRunner.Run<InfluxDBTest>();
        ```

## 實際測試數據

執行五次測試，每次測試包含 insert `100`,`10,000`,`1,000,000` 筆資料，測試結果如下

1. 第一次

    |     Method |   Times |        Mean |       Error |      StdDev |
    |----------- |-------: |------------:|------------:|------------:|
    | InsertTest |     100 |    10.68 ms |   0.2112 ms |   0.3226 ms |
    | InsertTest |   10000 |    98.28 ms |   2.8324 ms |   7.9425 ms |
    | InsertTest | 1000000 | 9,049.60 ms | 172.0483 ms | 176.6809 ms |

2. 第二次

    |     Method |   Times |        Mean |       Error |      StdDev |
    |----------- |-------: |------------:|------------:|------------:|
    | InsertTest |     100 |    10.88 ms |   0.2165 ms |   0.5103 ms |
    | InsertTest |   10000 |    93.86 ms |   1.8598 ms |   4.4915 ms |
    | InsertTest | 1000000 | 9,239.67 ms | 181.5064 ms | 186.3937 ms |

3. 第三次

    |     Method |   Times |        Mean |       Error |      StdDev |
    |----------- |-------: |------------:|------------:|------------:|
    | InsertTest |     100 |    10.88 ms |   0.2165 ms |   0.5103 ms |
    | InsertTest |   10000 |    93.86 ms |   1.8598 ms |   4.4915 ms |
    | InsertTest | 1000000 | 9,239.67 ms | 181.5064 ms | 186.3937 ms |

4. 第四次

    |     Method |   Times |        Mean |       Error |     StdDev |
    |----------- |-------: |------------:|------------:|-----------:|
    | InsertTest |     100 |    11.71 ms |   0.6246 ms |   1.832 ms |
    | InsertTest |   10000 |    94.54 ms |   2.1136 ms |   6.098 ms |
    | InsertTest | 1000000 | 9,372.18 ms | 185.2235 ms | 265.642 ms |

5. 第五次

    |     Method |   Times |        Mean |       Error |      StdDev |
    |----------- |-------: |------------:|------------:|------------:|
    | InsertTest |     100 |    10.45 ms |   0.2332 ms |   0.3766 ms |
    | InsertTest |   10000 |    93.05 ms |   1.8342 ms |   4.3946 ms |
    | InsertTest | 1000000 | 9,103.24 ms | 134.9402 ms | 126.2232 ms |

## 心得

比照一下之前筆記 [[Benchmark] 使用 C# 對 NoSQL insert 操作的效能數據](/nosql-insert-benchmark/)，可以發現 InfluxDB 在 insert 的效能只輸給 PostgreSQL -json，甚至比 MongoDB 還要來得快 - 不排除 `.NET Core 2.2.101` 與 `.NET Core SDK 2.2.301` 或是 SDK 版本的差異，但總體趨勢相信是正確的

經過簡單的測試後暫時不用擔心 InfluxDB 在 insert 上會跟不上系統需求

## 參考資訊

1. [使用 C# 存取 InfluxDB](/csharp-influxdb-curd/)
2. [[Benchmark] 使用 C# 對 NoSQL insert 操作的效能數據](/nosql-insert-benchmark/)
3. [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary)
