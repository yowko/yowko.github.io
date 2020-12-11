---
title: "C# - Property 與 Value 的 Dictionary 轉為 Object"
date: 2019-12-09T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["csharp"]
slug: "property-value-dictionary-to-object"
---

## C# - Property 與 Value 的 Dictionary 轉為 Object

之前筆記 [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary/) 紀錄到將 C# object 的 property name 與 value 透過 dictionary 的資料型態存放，當時主要是為了配合 InfluxDB 的 insert 而做的筆記 ，想不到時隔兩個月，竟然想要透過 property - value 的 dictionary 來轉回 C# object，有趣的是來源不是 InfluxDB 而是 Redis

## 基本環境說明

1. macOS Catalina 10.15.1
2. .NET Core SDK 3.1.100
3. 測試用程式碼

    - model

        ```cs
        public class User
        {
            public int UserId { get; set; }

            public string Name { get; set; }

            public string Email { get; set; }
        }
        ```

    - 假資料

        ```cs
        User[] _user = new User[]
            {
                new User
                {
                    UserId = 1,
                    Name = "yowko",
                    Email = "yowko@yowko.com"
                },
                new User
                {
                    UserId = 3,
                    Name = "test",
                    Email = "test@test.com"
                },
                new User
                {
                    UserId = 5,
                    Name = "poc",
                    Email = "poc@poc.com"
                },
            };
        ```

    - 將 object 轉為 dictionary

        > 之前筆記 [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary/) 中的程式只能轉換 reference type，這邊小改一下，讓 value type 也可以順利轉換 (但有 boxing 的效能損耗)

        ```cs
        public static class PocoToDictionary
        {
            private static readonly MethodInfo AddToDictionaryMethod = typeof(IDictionary<string, object>).GetMethod("Add");

            private static readonly ConcurrentDictionary<Type, Func<object, IDictionary<string, object>>> Converters =
                new ConcurrentDictionary<Type, Func<object, IDictionary<string, object>>>();

            private static readonly ConstructorInfo DictionaryConstructor = typeof(Dictionary<string, object>)
                .GetConstructors().FirstOrDefault(c => c.IsPublic && !c.GetParameters().Any());

            public static IDictionary<string, object> ExpressionToDictionary(this object obj) => obj == null
                ? null
                : Converters.GetOrAdd(obj.GetType(), o =>
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
                        from prop in inputType.GetProperties(BindingFlags.Instance | BindingFlags.Public |
                                                            BindingFlags.FlattenHierarchy)
                        where prop.CanRead && (prop.PropertyType.IsPrimitive || prop.PropertyType == typeof(string))
                        let getExpression = Expression.Property(typedInputExpression, prop.GetMethod)
                        select Expression.Call(outputVariable, AddToDictionaryMethod, Expression.Constant(prop.Name),
                            Expression.Convert(getExpression, typeof(object))));
                    body.Add(Expression.Return(returnTarget, outputVariable));
                    body.Add(Expression.Label(returnTarget, Expression.Constant(null, outputType)));
                    var lambdaExpression = Expression.Lambda<Func<object, IDictionary<string, object>>>(
                        Expression.Block(new[] {outputVariable}, body),
                        inputExpression);

                    return lambdaExpression.Compile();
                })(obj);
        }
        ```

    - 將各個 object 轉為 Dictionary list

        ```cs
        public static List<IDictionary<string, object>> ByPocoToDictionary()
        {
            var result = new List<IDictionary<string, object>>();
            var h = 0;
            foreach (var item in _user)
            {
                var dict = item.ExpressionToDictionary();
                h = $"{h}{dict.GetHashCode()}".GetHashCode();
                result.Add(dict);
            }
            return result;
        }
        ```

## 將 Dictionary 轉為 object

1. 轉換程式

    ```cs
    private static T DictionaryToObject<T>(IDictionary<String, Object> dictionary) where T : class, new()
    {
        // 取得 T 所有 property
        var myPropertyInfo = typeof(T).GetProperties();
        // 將 property 的 name 轉為小寫當 key，value 為原始大小寫，讓傳入的資料無論大小寫皆可轉換
        var properties = myPropertyInfo
            .Select(a => new KeyValuePair<string, string>(a.Name.ToLowerInvariant(), a.Name))
            .ToDictionary(a => a.Key, a => a.Value);

        // 建立 T 實體
        var instance = Activator.CreateInstance<T>();

        //處理所有欄位
        foreach (var (key, value) in dictionary)
        {
            var name = key.ToLowerInvariant();

            //欄位名稱不存在就換下一個
            if (!properties.TryGetValue(name, out var property)) continue;

            var prop = typeof(T).GetProperty(property);

            //依據不同型別來做轉換，只意思寫 int 與 string，請自行擴充
            switch (prop.PropertyType)
            {
                case { } intType when intType == typeof(int):
                    prop.SetValue(instance, Convert.ToInt32(value), null);

                    break;
                case { } stringType when stringType == typeof(string):
                    prop.SetValue(instance, Convert.ToString(value), null);

                    break;
            }
        }

        return instance;
    }
    ```

2. 實際使用

    ```cs
    // 取得 users 的 dictionary list
    var result = ByPocoToDictionary();
    // 將 dictionary 再轉回 user
    var outputResult = result.Select(DictionaryToObject<User>);
    ```

## 心得

其實上次紀錄 [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary/) 時，我就覺得有些荒謬了，覺得這樣的需求沒什麼道理，使用情境很是侷限，想不到這次竟然還需要來個逆向操作，不過所幸有上次的經驗，讓這次的問題可以快速被解決，雖然可能會因為上次的既定印象反而限制了思考的方向而造成用了效能不佳的方法，但終究是多了解了一個可以解決問題的方法，不論好壞還是學到了

## 參考資訊

1. [C# - 將 Object 的 Property 與 Value 轉換為 Dictionary](/csharp-object-to-dictionary/)
2. [Expression of type 'System.Int32' cannot be used for return type 'System.Object'](https://stackoverflow.com/questions/43080505/c-sharp-7-0-switch-on-system-type)
