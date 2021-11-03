---
title: "Enum in C#"
date: 2017-01-29T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "enum-in-c-sharp"
aliases:
    - /2017/01/enum-in-c-sharp.html
---
## Enum in C\#

最近跟同事討論 enum 相關應用時剛好同事問到 enum 可以用什麼型別，無奈學藝不精沒有答出完整正確答案XD，剛好趁這個機會順便整理一下。

## 基本特性

1. 名稱中不得包含空白字元
2. 預設值為 `0`

    >建議可以將商業邏輯上最適合的值設為 `0` 或是 直接指定  `Default=0`

3. 未指定下，Value 以 `1` 累加
4. 預設型別為 `int` , 可透過 `:` 指定使用其他整數數字型別(Numeric Type)

    >enum Days:byte { Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday };

5. Value 可以有計算式

    ```cs
    enum MachineState
    {
        PowerOff = 0,
        Running = 5,
        Sleeping = 10,
        Hibernating = Sleeping + 5
    }
    ```

## Enum 取代數字的好處

1. 可以初步過濾掉無效值(ex：`星期` 欄位可以控制不出現 `8` 這種無效數值)
2. IntelliSense 提示

## 允許使用的資料型別

- 只要是整數數字型別(Numeric Type)都是被允許的

    Allow Type List

    No.|Type|Range|Size|.NET Framework 型別|特殊說明|宣告方式
    ---|---|---|---|---|---|---
    1| [byte](https://msdn.microsoft.com/zh-tw/library/5bdb6693.aspx)|0 至 255|不帶正負號的 8 位元整數|System.Byte|-|byte myByte = 255;
    2| [sbyte](https://msdn.microsoft.com/zh-tw/library/d86he86x.aspx)|-128 至 127|帶正負號的 8 位元整數|System.SByte|-|sbyte sByte1 = 127;
    3| [short](https://msdn.microsoft.com/zh-tw/library/ybs77ex4.aspx)|-32,768 至 32,767|    帶正負號的 16 位元整數|System.Int16|-|short x = 32767;
    4| [ushort](https://msdn.microsoft.com/zh-tw/library/cbf1574z.aspx)|0 至 65,535|不帶正負號的 16 位元整數|System.UInt16|-|ushort myShort = 65535;
    5| [int](https://msdn.microsoft.com/zh-tw/library/5kzh1b5w.aspx)|-2,147,483,648 至 2,147,483,647|帶正負號的 32 位元整數|System.Int32|預設值為 0|int i = 123;
    6| [uint](https://msdn.microsoft.com/zh-tw/library/x0sksh43.aspx)|0 至 4,294,967,295|    不帶正負號的 32 位元整數|    System.UInt32|***注意：uint 型別不符合 CLS 標準。 請儘可能使用 int***|uint myUint = 4294967290;//4294967290u//4294967290U
    7| [long](https://msdn.microsoft.com/zh-tw/library/ctetwysk.aspx)|-9,223,372,036,854,775,808 至 9,223,372,036,854,775,807|    帶正負號的 64 位元整數|    System.Int64|-|long long1 = 4294967296;//4294967296L
    8| [ulong](https://msdn.microsoft.com/zh-tw/library/t98873t4.aspx)|0 至 18,446,744,073,709,551,615|    不帶正負號的 64 位元整數|    System.UInt64|-|ulong uLong = 9223372036854775808; //`L` or `I` 會依大小決定是 `long` or `ulong`;`U` or `u`會依大小決定是 `unit` or `ulong`;`UL`、`ul`、`Ul`、`uL`、`LU`、`lu`、`Lu` 或 `lU` 則會是 `ulong`

## 其他用法

1. Bit Flag(位元旗標)
    - 位元運算效能較佳
    - `0` 表示未設定
    - 在 enum 套用 `System.FlagsAttribute` 屬性
    - 可以用一個欄位儲存數個選項
        - 範例以下列 Days 為例，要表示星期一、三、五，需用 `1,3,5` (變成字串操作)

            ```cs
            enum Days { Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday };
            ```

        - 如果改用 Days2 ,星期一、三、五，只要直接存 `30` ((Monday = 0x2) + (Wednesday = 0x8) + (Friday = 0x20))

            ```cs
            [Flags]
            enum Days2
            {
                None = 0x0,//0;// 0 << 0
                Sunday = 0x1,//1;// 1 << 0
                Monday = 0x2,//2;// 1 << 1
                Tuesday = 0x4,//4;//  1 << 2
                Wednesday = 0x8,//8;//  1 << 3
                Thursday = 0x10,//16;//  1 << 4
                Friday = 0x20,//32;//  1 << 5
                Saturday = 0x40//64;//  1 << 6
            }
            ```

    - 使用`&`(AND)、`|`(OR)、`~`(NOT) 和 `^`(XOR) 位元運算子
        - 範例

            ```cs
            // 使用 `OR`初始化
            Days2 meetingDays = Days2.Tuesday | Days2.Thursday;
            // 20
            
            // 使用 `OR` 增加 friday.
            meetingDays = meetingDays | Days2.Friday;
            // 52
            
            // 使用 `AND` 取得 Tuesday,  Friday
            meetingDays&(Days2.Tuesday | Days2.Friday)
            //36
            
            // 使用 `XOR` 移除 Tuesday.
            meetingDays = meetingDays ^ Days2.Tuesday;
            //48
            ```

2. 使用 `System.Enum` 的型別 `Method`
    - 指定 Value 取得名稱

        >Enum.GetName(typeof(Days), 4)

    - 取得所有 Value 陣列

        >Enum.GetValues(typeof(Days))

    - 取得所有 Name 陣列

        >Enum.GetNames(typeof(Days))

## 參考資料

1. [enum (C# 參考)](https://msdn.microsoft.com/zh-tw/library/sbbt4032.aspx)
2. [列舉類型 (C# 程式設計手冊)](https://msdn.microsoft.com/zh-tw/library/cc138362.aspx)
