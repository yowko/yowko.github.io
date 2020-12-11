---
title: "C# - 將 Object 的 Property 與 Value 轉換為 Dictionary"
date: 2019-09-21T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["csharp"]
slug: "csharp-object-to-dictionary"
---

## C# - 將 Object 的 Property 與 Value 轉換為 Dictionary

之前筆記 [使用 C# 存取 InfluxDB](/csharp-influxdb-curd/) 提到正在嘗試導入 InfluxDB，POC 使用到的 library - InfluxData.Net 在儲存資料時僅接受 Dictionary，如果需要將整個 c# object 儲存起來就必需做轉換，考量到專案規劃用量大，因此希望能儘量降低無謂地效能損耗，所以特別針對 c# object 的 Property 與 Value 轉型為 Dictionary 幾種方式的效能進行比較測試

我個人才疏學淺絕對是找不到最佳解，我就先紀錄一下測試的情境與方式，相信強大的同事們一定會幫忙找出更好的解法

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. NuGet package

    - BenchmarkDotNet 0.11.5
    - Bogus 28.3.1

4. 測試用基礎程式碼

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

## 轉換的備選方式

1. 使用 Reflection

    ```cs
    public List<IDictionary<string, object>> ByReflection()
    {
        var h = 0;
        var result= new List<IDictionary<string, object>>();
        for (var i = 0; i < N; i++)
        {
           var dict=_user.GetType().GetProperties().ToDictionary(x => x.Name, x => x.GetValue(_user, null));
            h = $"{h}{dict.GetHashCode()}".GetHashCode();
            result.Add(dict);
        }

        return result;
    }
    ```

2. 使用TypeDescriptor

    ```cs
    public List<IDictionary<string, object>> ByTypeDescriptor()
    {
        var result= new List<IDictionary<string, object>>();

        var h = 0;
        for (var i = 0; i < N; i++)
        {
            var dict=_user.TypeDescriptorToDictionary();
            h = $"{h}{dict.GetHashCode()}".GetHashCode();
            result.Add(dict);
        }
        return result;
    }
    ```

    - extension method

        ```cs
        public static class ObjectToDictionaryHelper
        {
            public static IDictionary<string, object> TypeDescriptorToDictionary(this object source)
            {
                return source.TypeDescriptorToDictionary<object>();
            }

            public static IDictionary<string, T> TypeDescriptorToDictionary<T>(this object source)
            {
                if (source == null) ThrowExceptionWhenSourceArgumentIsNull();

                var dictionary = new Dictionary<string, T>();
                foreach (PropertyDescriptor property in TypeDescriptor.GetProperties(source))
                {
                    object value = property.GetValue(source);
                    if (IsOfType<T>(value))
                    {
                        dictionary.Add(property.Name, (T)value);
                    }
                }
                return dictionary;
            }

            private static bool IsOfType<T>(object value)
            {
                return value is T;
            }

            private static void ThrowExceptionWhenSourceArgumentIsNull()
            {
                throw new NullReferenceException("Unable to convert anonymous object to a dictionary. The source anonymous object is null.");
            }
        }
        ```

3. 使用 Expression

    ```cs
    public List<IDictionary<string, object>> ByPocoToDictionary()
    {
        var result= new List<IDictionary<string, object>>();

        var h = 0;
        for (var i = 0; i < N; i++)
        {
            var dict=_user.ExpressionToDictionary();
            h = $"{h}{dict.GetHashCode()}".GetHashCode();
           result.Add(dict);
        }
        return result;
    }
    ```

    - extension method

        ```cs
        public static class PocoToDictionary
        {
            private static readonly MethodInfo AddToDictionaryMethod = typeof(IDictionary<string, object>).GetMethod("Add");
            private static readonly ConcurrentDictionary<Type, Func<object, IDictionary<string, object>>> Converters = new ConcurrentDictionary<Type, Func<object,  IDictionary<string, object>>>();
            private static readonly ConstructorInfo DictionaryConstructor = typeof(Dictionary<string, object>).GetConstructors().FirstOrDefault(c => c.IsPublic &&  !c.GetParameters().Any());

            public static IDictionary<string, object> ExpressionToDictionary(this object obj) => obj == null ? null : Converters.GetOrAdd(obj.GetType(), o =>
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

## 實際測試程式

- 測試用 class

    ```cs
     public class ObjectToDictionary
    {
        private const int N = 1000000;
        private readonly User _user;
        
        public ObjectToDictionary()
        {
          _user= new Faker<User>()
              .RuleFor(a => a.UserId, f => f.Random.Guid())
              .RuleFor(a => a.Name, f => f.Name.FirstName())
              .RuleFor(a => a.Birthday, f => f.Date.Past())
              .RuleFor(a => a.CurrentSalary, f => f.Random.Decimal(0M, 10000M))
              .Generate();

        }

        [Benchmark]
        public List<IDictionary<string, object>> ByReflection()
        {
            var h = 0;
            var result= new List<IDictionary<string, object>>();
            for (var i = 0; i < N; i++)
            {
               var dict=_user.GetType().GetProperties().ToDictionary(x => x.Name, x => x.GetValue(_user, null));
                h = $"{h}{dict.GetHashCode()}".GetHashCode();
                result.Add(dict);
            }

            return result;
        }

        [Benchmark]
        public List<IDictionary<string, object>> ByExpression()
        {
            var result= new List<IDictionary<string, object>>();

            var h = 0;
            for (var i = 0; i < N; i++)
            {
                var dict=_user.ExpressionToDictionary();
                h = $"{h}{dict.GetHashCode()}".GetHashCode();
               result.Add(dict);
            }
            return result;
        }
        
        [Benchmark]
        public List<IDictionary<string, object>> ByTypeDescriptor()
        {
            var result= new List<IDictionary<string, object>>();

            var h = 0;
            for (var i = 0; i < N; i++)
            {
                var dict=_user.TypeDescriptorToDictionary();
                h = $"{h}{dict.GetHashCode()}".GetHashCode();
                result.Add(dict);
            }
            return result;
        }
    }
    ```

- 執行測試

    ```cs
     var summary = BenchmarkRunner.Run<ObjectToDictionary>();
    ```

## 效能數據

|             Method |    Mean |    Error |   StdDev |
|------------------- |--------:|---------:|---------:|
|       ByReflection | 5.017 s | 0.3736 s | 1.0957 s |
|       ByExpression | 1.733 s | 0.0365 s | 0.0838 s |
|   ByTypeDescriptor | 7.087 s | 0.1385 s | 0.2982 s |

## 心得

還沒執行效能測試前就猜測應該會是 expression 勝出，雖然也想過使用 emit，但誠如忠成老師 [C# 快速動態建立物件實體技巧](https://dotblogs.com.tw/code6421/2019/08/15/csharpfastobject) 提到的可讀性，加上我也不確定 emit 寫出來後自己可以記得多久，想想還是別找自己麻煩了XD，不過如果之後有多些時間，也會做個 emit 版本的效能比較，畢竟量大的情境下  任何一點效能提升都會有幫助，對系統也都是正面的

## 參考資訊

1. [Fast Conversion of Objects to Dictionaries in C#](https://softwareproduction.eu/2018/02/28/fast-conversion-objects-dictionaries-c/)
2. [jarrettmeyer/ObjectToDictionaryHelper.cs](https://gist.github.com/jarrettmeyer/798667/a87f9bcac2ec68541511f17da3c244c0e05bdc49)
