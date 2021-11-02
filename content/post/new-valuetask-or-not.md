---
title: "需要在 return 前自行 new ValueTask 嗎？"
date: 2019-08-11T21:30:00+08:00
lastmod: 2021-11-02T21:30:31+08:00
draft: false
tags: ["csharp"]
slug: "new-valuetask-or-not"
---

## 需要在 return 前自行 new ValueTask 嗎？

上個月黑大的 [閱讀筆記 - 使用 .NET Async/Await 的常見錯誤](https://blog.darkthread.net/blog/common-async-await-mistakes/) 跟同事討論起 ValueTask 的用法，大意是 ValueTask 在使用上只需要將方法簽章改為 `ValueTask<T>` 就好嗎？  還是應該在回傳時 `new ValueTask<T>()`

網路上的文章兩種寫法都有，但較有公信心的內容看似都有 new 的動作，雖然有用過 ValueTask 但我就是勉強稱得上用過的程度，經不起強大同事針針見血地的問題，為了避免錯誤的意見，我重新檢視了相關文章跟資料，覺得好像應該要 `new` 才對？？  不過想要搞清楚的想法還是在我心裡悄悄地發芽了，只是左思右想還是沒有非常確定答案，乾脆先紀錄一下目前發現，待其他高手提點再來補充囉

## 基本環境說明

1. macOS Mojave 10.14.6
2. .NET Core SDK 2.2.301 (.NET Core Runtime 2.2.6)
3. 測試程式碼

    ```cs
    static async Task Main()
    {
        await TestTask();
        await TestValueTask();
        await TestValueTaskWithNew();
    }

    private static async Task<int> TestTask()
    {
        await Task.Delay(100);

        return 100;
    }

    private static async ValueTask<int> TestValueTask()
    {
        await Task.Delay(100);

        return 10;
    }

    private static ValueTask<int> TestValueTaskWithNew()
    {
        return new ValueTask<int>(0);
    }
    ```

## IL - Intermediate Language

先說個人發現，有興趣的朋友可以再看看詳細 IL 內容

1. `Task<int>` 與 `async ValueTask<int>`

    >1. `Task<int>` 使用 __***TaskAwaiter***__; 而 `async ValueTask<int>` 使用 __***ValueTaskAwaiter***__
    >2. `Task<int>` 回傳 __***class***__ ; `async ValueTask<int>` 回傳 __***valuetype***__

2. `async ValueTask<int>` 與 `ValueTask<int>` 的差異

    >1. 僅 `async ValueTask<int>` 實作 __***IAsyncStateMachine***__; `ValueTask<int>` 未實作 __***IAsyncStateMachine***__
    >2. 僅 `async ValueTask<int>` 存在 __***MoveNext()***__ 方法

- `Task<int>`

    <details><summary>Collapse or Expand</summary>
    <p>

    ```txt
    // Type: ValueTaskResearch.Program 
    // Assembly: ValueTaskResearch, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
    // MVID: 99E6E072-258E-433E-84E8-C626124D37C8
    // Location: /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.dll
    // Sequence point data from /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.pdb

    .class private auto ansi beforefieldinit
    ValueTaskResearch.Program
        extends [System.Runtime]System.Object
    {

    .class nested private sealed auto ansi beforefieldinit
        '<Main>d__0'
        extends [System.Runtime]System.Object
        implements [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

        .field public int32 '<>1__state'

        .field public valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder '<>t__builder'

        .field private valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32> '<>u__1'

        .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
        {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

        } // end of method '<Main>d__0'::.ctor

        .method private final hidebysig virtual newslot instance void
        MoveNext() cil managed
        {
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::MoveNext()
        .maxstack 3
        .locals init (
            [0] int32 V_0,
            [1] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32> V_1,
            [2] class ValueTaskResearch.Program/'<Main>d__0' V_2,
            [3] class [System.Runtime]System.Exception V_3
        )

        IL_0000: ldarg.0      // this
        IL_0001: ldfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0006: stloc.0      // V_0
        .try
        {

            IL_0007: ldloc.0      // V_0
            IL_0008: brfalse.s    IL_000c
            IL_000a: br.s         IL_000e
            IL_000c: br.s         IL_0047

            // [8 9 - 8 10]
            IL_000e: nop

            // [9 13 - 9 30]
            IL_000f: call         class [System.Runtime]System.Threading.Tasks.Task`1<int32> ValueTaskResearch.Program::TestTask()
            IL_0014: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<!0/*int32*/> class [System.Runtime]System.Threading.Tasks.Task`1<int32>::GetAwaiter()
            IL_0019: stloc.1      // V_1

            IL_001a: ldloca.s     V_1
            IL_001c: call         instance bool valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32>::get_IsCompleted()
            IL_0021: brtrue.s     IL_0063
            IL_0023: ldarg.0      // this
            IL_0024: ldc.i4.0
            IL_0025: dup
            IL_0026: stloc.0      // V_0
            IL_0027: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_002c: ldarg.0      // this
            IL_002d: ldloc.1      // V_1
            IL_002e: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0033: ldarg.0      // this
            IL_0034: stloc.2      // V_2
            IL_0035: ldarg.0      // this
            IL_0036: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_003b: ldloca.s     V_1
            IL_003d: ldloca.s     V_2
            IL_003f: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::AwaitUnsafeOnCompleted<valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32>, class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32>*/&, !!1/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
            IL_0044: nop
            IL_0045: leave.s      IL_0099
            IL_0047: ldarg.0      // this
            IL_0048: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_004d: stloc.1      // V_1
            IL_004e: ldarg.0      // this
            IL_004f: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0054: initobj      valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32>
            IL_005a: ldarg.0      // this
            IL_005b: ldc.i4.m1
            IL_005c: dup
            IL_005d: stloc.0      // V_0
            IL_005e: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_0063: ldloca.s     V_1
            IL_0065: call         instance !0/*int32*/ valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter`1<int32>::GetResult()
            IL_006a: pop
            IL_006b: leave.s      IL_0085
        } // end of .try
        catch [System.Runtime]System.Exception
        {

            IL_006d: stloc.3      // V_3
            IL_006e: ldarg.0      // this
            IL_006f: ldc.i4.s     -2 // 0xfe
            IL_0071: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_0076: ldarg.0      // this
            IL_0077: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_007c: ldloc.3      // V_3
            IL_007d: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetException(class [System.Runtime]System.Exception)
            IL_0082: nop
            IL_0083: leave.s      IL_0099
        } // end of catch

        // [13 9 - 13 10]
        IL_0085: ldarg.0      // this
        IL_0086: ldc.i4.s     -2 // 0xfe
        IL_0088: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'

        IL_008d: ldarg.0      // this
        IL_008e: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0093: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetResult()
        IL_0098: nop
        IL_0099: ret

        } // end of method '<Main>d__0'::MoveNext

        .method private final hidebysig virtual newslot instance void
        SetStateMachine(
            class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine stateMachine
        ) cil managed
        {
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerHiddenAttribute::.ctor()
            = (01 00 00 00 )
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::SetStateMachine(class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine)
        .maxstack 8

        IL_0000: ret

        } // end of method '<Main>d__0'::SetStateMachine
    } // end of class '<Main>d__0'

    .class nested private sealed auto ansi beforefieldinit
        '<TestTask>d__1'
        extends [System.Runtime]System.Object
        implements [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

        .field public int32 '<>1__state'

        .field public valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> '<>t__builder'

        .field private valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter '<>u__1'

        .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
        {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

        } // end of method '<TestTask>d__1'::.ctor

        .method private final hidebysig virtual newslot instance void
        MoveNext() cil managed
        {
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::MoveNext()
        .maxstack 3
        .locals init (
            [0] int32 V_0,
            [1] int32 V_1,
            [2] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter V_2,
            [3] class ValueTaskResearch.Program/'<TestTask>d__1' V_3,
            [4] class [System.Runtime]System.Exception V_4
        )

        IL_0000: ldarg.0      // this
        IL_0001: ldfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'
        IL_0006: stloc.0      // V_0
        .try
        {

            IL_0007: ldloc.0      // V_0
            IL_0008: brfalse.s    IL_000c
            IL_000a: br.s         IL_000e
            IL_000c: br.s         IL_0049

            // [16 9 - 16 10]
            IL_000e: nop

            // [17 13 - 17 35]
            IL_000f: ldc.i4.s     100 // 0x64
            IL_0011: call         class [System.Runtime]System.Threading.Tasks.Task [System.Runtime]System.Threading.Tasks.Task::Delay(int32)
            IL_0016: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter [System.Runtime]System.Threading.Tasks.Task::GetAwaiter()
            IL_001b: stloc.2      // V_2

            IL_001c: ldloca.s     V_2
            IL_001e: call         instance bool [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::get_IsCompleted()
            IL_0023: brtrue.s     IL_0065
            IL_0025: ldarg.0      // this
            IL_0026: ldc.i4.0
            IL_0027: dup
            IL_0028: stloc.0      // V_0
            IL_0029: stfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'
            IL_002e: ldarg.0      // this
            IL_002f: ldloc.2      // V_2
            IL_0030: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestTask>d__1'::'<>u__1'
            IL_0035: ldarg.0      // this
            IL_0036: stloc.3      // V_3
            IL_0037: ldarg.0      // this
            IL_0038: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
            IL_003d: ldloca.s     V_2
            IL_003f: ldloca.s     V_3
            IL_0041: call         instance void valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::AwaitUnsafeOnCompleted<valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter, class ValueTaskResearch.Program/'<TestTask>d__1'>(!!0/*valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter*/&, !!1/*class ValueTaskResearch.Program/'<TestTask>d__1'*/&)
            IL_0046: nop
            IL_0047: leave.s      IL_00a1
            IL_0049: ldarg.0      // this
            IL_004a: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestTask>d__1'::'<>u__1'
            IL_004f: stloc.2      // V_2
            IL_0050: ldarg.0      // this
            IL_0051: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestTask>d__1'::'<>u__1'
            IL_0056: initobj      [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter
            IL_005c: ldarg.0      // this
            IL_005d: ldc.i4.m1
            IL_005e: dup
            IL_005f: stloc.0      // V_0
            IL_0060: stfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'
            IL_0065: ldloca.s     V_2
            IL_0067: call         instance void [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::GetResult()
            IL_006c: nop

            // [19 13 - 19 24]
            IL_006d: ldc.i4.s     100 // 0x64
            IL_006f: stloc.1      // V_1
            IL_0070: leave.s      IL_008c
        } // end of .try
        catch [System.Runtime]System.Exception
        {

            IL_0072: stloc.s      V_4
            IL_0074: ldarg.0      // this
            IL_0075: ldc.i4.s     -2 // 0xfe
            IL_0077: stfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'
            IL_007c: ldarg.0      // this
            IL_007d: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
            IL_0082: ldloc.s      V_4
            IL_0084: call         instance void valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::SetException(class [System.Runtime]System.Exception)
            IL_0089: nop
            IL_008a: leave.s      IL_00a1
        } // end of catch

        // [20 9 - 20 10]
        IL_008c: ldarg.0      // this
        IL_008d: ldc.i4.s     -2 // 0xfe
        IL_008f: stfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'

        IL_0094: ldarg.0      // this
        IL_0095: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
        IL_009a: ldloc.1      // V_1
        IL_009b: call         instance void valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::SetResult(!0/*int32*/)
        IL_00a0: nop
        IL_00a1: ret

        } // end of method '<TestTask>d__1'::MoveNext

        .method private final hidebysig virtual newslot instance void
        SetStateMachine(
            class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine stateMachine
        ) cil managed
        {
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerHiddenAttribute::.ctor()
            = (01 00 00 00 )
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::SetStateMachine(class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine)
        .maxstack 8

        IL_0000: ret

        } // end of method '<TestTask>d__1'::SetStateMachine
    } // end of class '<TestTask>d__1'

    .method private hidebysig static class [System.Runtime]System.Threading.Tasks.Task
        Main() cil managed
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.AsyncStateMachineAttribute::.ctor(class [System.Runtime]System.Type)
        = (
            01 00 24 56 61 6c 75 65 54 61 73 6b 52 65 73 65 // ..$ValueTaskRese
            61 72 63 68 2e 50 72 6f 67 72 61 6d 2b 3c 4d 61 // arch.Program+<Ma
            69 6e 3e 64 5f 5f 30 00 00                      // in>d__0..
        )
        // type(class ValueTaskResearch.Program/'<Main>d__0')
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 2
        .locals init (
        [0] class ValueTaskResearch.Program/'<Main>d__0' V_0,
        [1] valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder V_1
        )

        IL_0000: newobj       instance void ValueTaskResearch.Program/'<Main>d__0'::.ctor()
        IL_0005: stloc.0      // V_0
        IL_0006: ldloc.0      // V_0
        IL_0007: call         valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Create()
        IL_000c: stfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0011: ldloc.0      // V_0
        IL_0012: ldc.i4.m1
        IL_0013: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0018: ldloc.0      // V_0
        IL_0019: ldfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_001e: stloc.1      // V_1
        IL_001f: ldloca.s     V_1
        IL_0021: ldloca.s     V_0
        IL_0023: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Start<class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
        IL_0028: ldloc.0      // V_0
        IL_0029: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_002e: call         instance class [System.Runtime]System.Threading.Tasks.Task [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::get_Task()
        IL_0033: ret

    } // end of method Program::Main

    .method private hidebysig static class [System.Runtime]System.Threading.Tasks.Task`1<int32>
        TestTask() cil managed
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.AsyncStateMachineAttribute::.ctor(class [System.Runtime]System.Type)
        = (
            01 00 28 56 61 6c 75 65 54 61 73 6b 52 65 73 65 // ..(ValueTaskRese
            61 72 63 68 2e 50 72 6f 67 72 61 6d 2b 3c 54 65 // arch.Program+<Te
            73 74 54 61 73 6b 3e 64 5f 5f 31 00 00          // stTask>d__1..
        )
        // type(class ValueTaskResearch.Program/'<TestTask>d__1')
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 2
        .locals init (
        [0] class ValueTaskResearch.Program/'<TestTask>d__1' V_0,
        [1] valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> V_1
        )

        IL_0000: newobj       instance void ValueTaskResearch.Program/'<TestTask>d__1'::.ctor()
        IL_0005: stloc.0      // V_0
        IL_0006: ldloc.0      // V_0
        IL_0007: call         valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<!0/*int32*/> valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::Create()
        IL_000c: stfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
        IL_0011: ldloc.0      // V_0
        IL_0012: ldc.i4.m1
        IL_0013: stfld        int32 ValueTaskResearch.Program/'<TestTask>d__1'::'<>1__state'
        IL_0018: ldloc.0      // V_0
        IL_0019: ldfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
        IL_001e: stloc.1      // V_1
        IL_001f: ldloca.s     V_1
        IL_0021: ldloca.s     V_0
        IL_0023: call         instance void valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::Start<class ValueTaskResearch.Program/'<TestTask>d__1'>(!!0/*class ValueTaskResearch.Program/'<TestTask>d__1'*/&)
        IL_0028: ldloc.0      // V_0
        IL_0029: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestTask>d__1'::'<>t__builder'
        IL_002e: call         instance class [System.Runtime]System.Threading.Tasks.Task`1<!0/*int32*/> valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1<int32>::get_Task()
        IL_0033: ret

    } // end of method Program::TestTask

    .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
    {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

    } // end of method Program::.ctor

    .method private hidebysig static specialname void
        '<Main>'() cil managed
    {
        .entrypoint
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 1
        .locals init (
        [0] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter V_0
        )

        IL_0000: call         class [System.Runtime]System.Threading.Tasks.Task ValueTaskResearch.Program::Main()
        IL_0005: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter [System.Runtime]System.Threading.Tasks.Task::GetAwaiter()
        IL_000a: stloc.0      // V_0
        IL_000b: ldloca.s     V_0
        IL_000d: call         instance void [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::GetResult()
        IL_0012: ret

    } // end of method Program::'<Main>'
    } // end of class ValueTaskResearch.Program
    ```

    </p>
    </details>

- `async ValueTask<int>`

    <details><summary>Collapse or Expand</summary>
    <p>

    ```txt
    // Type: ValueTaskResearch.Program 
    // Assembly: ValueTaskResearch, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
    // MVID: EA74DFC9-6E5E-4DC8-B7FC-A47B369C42E7
    // Location: /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.dll
    // Sequence point data from /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.pdb

    .class private auto ansi beforefieldinit
    ValueTaskResearch.Program
        extends [System.Runtime]System.Object
    {

    .class nested private sealed auto ansi beforefieldinit
        '<Main>d__0'
        extends [System.Runtime]System.Object
        implements [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

        .field public int32 '<>1__state'

        .field public valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder '<>t__builder'

        .field private valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> '<>u__1'

        .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
        {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

        } // end of method '<Main>d__0'::.ctor

        .method private final hidebysig virtual newslot instance void
        MoveNext() cil managed
        {
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::MoveNext()
        .maxstack 3
        .locals init (
            [0] int32 V_0,
            [1] valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> V_1,
            [2] valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32> V_2,
            [3] class ValueTaskResearch.Program/'<Main>d__0' V_3,
            [4] class [System.Runtime]System.Exception V_4
        )

        IL_0000: ldarg.0      // this
        IL_0001: ldfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0006: stloc.0      // V_0
        .try
        {

            IL_0007: ldloc.0      // V_0
            IL_0008: brfalse.s    IL_000c
            IL_000a: br.s         IL_000e
            IL_000c: br.s         IL_004a

            // [8 9 - 8 10]
            IL_000e: nop

            // [10 13 - 10 35]
            IL_000f: call         valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32> ValueTaskResearch.Program::TestValueTask()
            IL_0014: stloc.2      // V_2
            IL_0015: ldloca.s     V_2
            IL_0017: call         instance valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<!0/*int32*/> valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32>::GetAwaiter()
            IL_001c: stloc.1      // V_1

            IL_001d: ldloca.s     V_1
            IL_001f: call         instance bool valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>::get_IsCompleted()
            IL_0024: brtrue.s     IL_0066
            IL_0026: ldarg.0      // this
            IL_0027: ldc.i4.0
            IL_0028: dup
            IL_0029: stloc.0      // V_0
            IL_002a: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_002f: ldarg.0      // this
            IL_0030: ldloc.1      // V_1
            IL_0031: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0036: ldarg.0      // this
            IL_0037: stloc.3      // V_3
            IL_0038: ldarg.0      // this
            IL_0039: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_003e: ldloca.s     V_1
            IL_0040: ldloca.s     V_3
            IL_0042: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::AwaitUnsafeOnCompleted<valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>, class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>*/&, !!1/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
            IL_0047: nop
            IL_0048: leave.s      IL_009e
            IL_004a: ldarg.0      // this
            IL_004b: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0050: stloc.1      // V_1
            IL_0051: ldarg.0      // this
            IL_0052: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0057: initobj      valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>
            IL_005d: ldarg.0      // this
            IL_005e: ldc.i4.m1
            IL_005f: dup
            IL_0060: stloc.0      // V_0
            IL_0061: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_0066: ldloca.s     V_1
            IL_0068: call         instance !0/*int32*/ valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>::GetResult()
            IL_006d: pop
            IL_006e: leave.s      IL_008a
        } // end of .try
        catch [System.Runtime]System.Exception
        {

            IL_0070: stloc.s      V_4
            IL_0072: ldarg.0      // this
            IL_0073: ldc.i4.s     -2 // 0xfe
            IL_0075: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_007a: ldarg.0      // this
            IL_007b: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_0080: ldloc.s      V_4
            IL_0082: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetException(class [System.Runtime]System.Exception)
            IL_0087: nop
            IL_0088: leave.s      IL_009e
        } // end of catch

        // [12 9 - 12 10]
        IL_008a: ldarg.0      // this
        IL_008b: ldc.i4.s     -2 // 0xfe
        IL_008d: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'

        IL_0092: ldarg.0      // this
        IL_0093: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0098: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetResult()
        IL_009d: nop
        IL_009e: ret

        } // end of method '<Main>d__0'::MoveNext

        .method private final hidebysig virtual newslot instance void
        SetStateMachine(
            class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine stateMachine
        ) cil managed
        {
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerHiddenAttribute::.ctor()
            = (01 00 00 00 )
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::SetStateMachine(class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine)
        .maxstack 8

        IL_0000: ret

        } // end of method '<Main>d__0'::SetStateMachine
    } // end of class '<Main>d__0'

    .class nested private sealed auto ansi beforefieldinit
        '<TestValueTask>d__1'
        extends [System.Runtime]System.Object
        implements [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

        .field public int32 '<>1__state'

        .field public valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> '<>t__builder'

        .field private valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter '<>u__1'

        .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
        {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

        } // end of method '<TestValueTask>d__1'::.ctor

        .method private final hidebysig virtual newslot instance void
        MoveNext() cil managed
        {
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::MoveNext()
        .maxstack 3
        .locals init (
            [0] int32 V_0,
            [1] int32 V_1,
            [2] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter V_2,
            [3] class ValueTaskResearch.Program/'<TestValueTask>d__1' V_3,
            [4] class [System.Runtime]System.Exception V_4
        )

        IL_0000: ldarg.0      // this
        IL_0001: ldfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'
        IL_0006: stloc.0      // V_0
        .try
        {

            IL_0007: ldloc.0      // V_0
            IL_0008: brfalse.s    IL_000c
            IL_000a: br.s         IL_000e
            IL_000c: br.s         IL_0049

            // [22 9 - 22 10]
            IL_000e: nop

            // [23 13 - 23 35]
            IL_000f: ldc.i4.s     100 // 0x64
            IL_0011: call         class [System.Runtime]System.Threading.Tasks.Task [System.Runtime]System.Threading.Tasks.Task::Delay(int32)
            IL_0016: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter [System.Runtime]System.Threading.Tasks.Task::GetAwaiter()
            IL_001b: stloc.2      // V_2

            IL_001c: ldloca.s     V_2
            IL_001e: call         instance bool [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::get_IsCompleted()
            IL_0023: brtrue.s     IL_0065
            IL_0025: ldarg.0      // this
            IL_0026: ldc.i4.0
            IL_0027: dup
            IL_0028: stloc.0      // V_0
            IL_0029: stfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'
            IL_002e: ldarg.0      // this
            IL_002f: ldloc.2      // V_2
            IL_0030: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>u__1'
            IL_0035: ldarg.0      // this
            IL_0036: stloc.3      // V_3
            IL_0037: ldarg.0      // this
            IL_0038: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
            IL_003d: ldloca.s     V_2
            IL_003f: ldloca.s     V_3
            IL_0041: call         instance void valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::AwaitUnsafeOnCompleted<valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter, class ValueTaskResearch.Program/'<TestValueTask>d__1'>(!!0/*valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter*/&, !!1/*class ValueTaskResearch.Program/'<TestValueTask>d__1'*/&)
            IL_0046: nop
            IL_0047: leave.s      IL_00a1
            IL_0049: ldarg.0      // this
            IL_004a: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>u__1'
            IL_004f: stloc.2      // V_2
            IL_0050: ldarg.0      // this
            IL_0051: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>u__1'
            IL_0056: initobj      [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter
            IL_005c: ldarg.0      // this
            IL_005d: ldc.i4.m1
            IL_005e: dup
            IL_005f: stloc.0      // V_0
            IL_0060: stfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'
            IL_0065: ldloca.s     V_2
            IL_0067: call         instance void [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::GetResult()
            IL_006c: nop

            // [25 13 - 25 23]
            IL_006d: ldc.i4.s     10 // 0x0a
            IL_006f: stloc.1      // V_1
            IL_0070: leave.s      IL_008c
        } // end of .try
        catch [System.Runtime]System.Exception
        {

            IL_0072: stloc.s      V_4
            IL_0074: ldarg.0      // this
            IL_0075: ldc.i4.s     -2 // 0xfe
            IL_0077: stfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'
            IL_007c: ldarg.0      // this
            IL_007d: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
            IL_0082: ldloc.s      V_4
            IL_0084: call         instance void valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::SetException(class [System.Runtime]System.Exception)
            IL_0089: nop
            IL_008a: leave.s      IL_00a1
        } // end of catch

        // [26 9 - 26 10]
        IL_008c: ldarg.0      // this
        IL_008d: ldc.i4.s     -2 // 0xfe
        IL_008f: stfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'

        IL_0094: ldarg.0      // this
        IL_0095: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
        IL_009a: ldloc.1      // V_1
        IL_009b: call         instance void valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::SetResult(!0/*int32*/)
        IL_00a0: nop
        IL_00a1: ret

        } // end of method '<TestValueTask>d__1'::MoveNext

        .method private final hidebysig virtual newslot instance void
        SetStateMachine(
            class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine stateMachine
        ) cil managed
        {
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerHiddenAttribute::.ctor()
            = (01 00 00 00 )
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::SetStateMachine(class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine)
        .maxstack 8

        IL_0000: ret

        } // end of method '<TestValueTask>d__1'::SetStateMachine
    } // end of class '<TestValueTask>d__1'

    .method private hidebysig static class [System.Runtime]System.Threading.Tasks.Task
        Main() cil managed
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.AsyncStateMachineAttribute::.ctor(class [System.Runtime]System.Type)
        = (
            01 00 24 56 61 6c 75 65 54 61 73 6b 52 65 73 65 // ..$ValueTaskRese
            61 72 63 68 2e 50 72 6f 67 72 61 6d 2b 3c 4d 61 // arch.Program+<Ma
            69 6e 3e 64 5f 5f 30 00 00                      // in>d__0..
        )
        // type(class ValueTaskResearch.Program/'<Main>d__0')
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 2
        .locals init (
        [0] class ValueTaskResearch.Program/'<Main>d__0' V_0,
        [1] valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder V_1
        )

        IL_0000: newobj       instance void ValueTaskResearch.Program/'<Main>d__0'::.ctor()
        IL_0005: stloc.0      // V_0
        IL_0006: ldloc.0      // V_0
        IL_0007: call         valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Create()
        IL_000c: stfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0011: ldloc.0      // V_0
        IL_0012: ldc.i4.m1
        IL_0013: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0018: ldloc.0      // V_0
        IL_0019: ldfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_001e: stloc.1      // V_1
        IL_001f: ldloca.s     V_1
        IL_0021: ldloca.s     V_0
        IL_0023: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Start<class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
        IL_0028: ldloc.0      // V_0
        IL_0029: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_002e: call         instance class [System.Runtime]System.Threading.Tasks.Task [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::get_Task()
        IL_0033: ret

    } // end of method Program::Main

    .method private hidebysig static valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32>
        TestValueTask() cil managed
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.AsyncStateMachineAttribute::.ctor(class [System.Runtime]System.Type)
        = (
            01 00 2d 56 61 6c 75 65 54 61 73 6b 52 65 73 65 // ..-ValueTaskRese
            61 72 63 68 2e 50 72 6f 67 72 61 6d 2b 3c 54 65 // arch.Program+<Te
            73 74 56 61 6c 75 65 54 61 73 6b 3e 64 5f 5f 31 // stValueTask>d__1
            00 00                                           // ..
        )
        // type(class ValueTaskResearch.Program/'<TestValueTask>d__1')
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 2
        .locals init (
        [0] class ValueTaskResearch.Program/'<TestValueTask>d__1' V_0,
        [1] valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> V_1
        )

        IL_0000: newobj       instance void ValueTaskResearch.Program/'<TestValueTask>d__1'::.ctor()
        IL_0005: stloc.0      // V_0
        IL_0006: ldloc.0      // V_0
        IL_0007: call         valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<!0/*int32*/> valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::Create()
        IL_000c: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
        IL_0011: ldloc.0      // V_0
        IL_0012: ldc.i4.m1
        IL_0013: stfld        int32 ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>1__state'
        IL_0018: ldloc.0      // V_0
        IL_0019: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
        IL_001e: stloc.1      // V_1
        IL_001f: ldloca.s     V_1
        IL_0021: ldloca.s     V_0
        IL_0023: call         instance void valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::Start<class ValueTaskResearch.Program/'<TestValueTask>d__1'>(!!0/*class ValueTaskResearch.Program/'<TestValueTask>d__1'*/&)
        IL_0028: ldloc.0      // V_0
        IL_0029: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32> ValueTaskResearch.Program/'<TestValueTask>d__1'::'<>t__builder'
        IL_002e: call         instance valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<!0/*int32*/> valuetype [System.Runtime]System.Runtime.CompilerServices.AsyncValueTaskMethodBuilder`1<int32>::get_Task()
        IL_0033: ret

    } // end of method Program::TestValueTask

    .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
    {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

    } // end of method Program::.ctor

    .method private hidebysig static specialname void
        '<Main>'() cil managed
    {
        .entrypoint
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 1
        .locals init (
        [0] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter V_0
        )

        IL_0000: call         class [System.Runtime]System.Threading.Tasks.Task ValueTaskResearch.Program::Main()
        IL_0005: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter [System.Runtime]System.Threading.Tasks.Task::GetAwaiter()
        IL_000a: stloc.0      // V_0
        IL_000b: ldloca.s     V_0
        IL_000d: call         instance void [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::GetResult()
        IL_0012: ret

    } // end of method Program::'<Main>'
    } // end of class ValueTaskResearch.Program

    ```

     </p>
    </details>

- `ValueTask<int>`

    <details><summary>Collapse or Expand</summary>
    <p>

    ```txt
    // Type: ValueTaskResearch.Program 
    // Assembly: ValueTaskResearch, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
    // MVID: 16376FF2-3379-4D36-AE5C-65DF961109C2
    // Location: /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.dll
    // Sequence point data from /Users/yowko.tsai/ValueTaskResearch/ValueTaskResearch/bin/Debug/netcoreapp2.2/ValueTaskResearch.pdb

    .class private auto ansi beforefieldinit
    ValueTaskResearch.Program
        extends [System.Runtime]System.Object
    {

    .class nested private sealed auto ansi beforefieldinit
        '<Main>d__0'
        extends [System.Runtime]System.Object
        implements [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
        = (01 00 00 00 )

        .field public int32 '<>1__state'

        .field public valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder '<>t__builder'

        .field private valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> '<>u__1'

        .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
        {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

        } // end of method '<Main>d__0'::.ctor

        .method private final hidebysig virtual newslot instance void
        MoveNext() cil managed
        {
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::MoveNext()
        .maxstack 3
        .locals init (
            [0] int32 V_0,
            [1] valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> V_1,
            [2] valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32> V_2,
            [3] class ValueTaskResearch.Program/'<Main>d__0' V_3,
            [4] class [System.Runtime]System.Exception V_4
        )

        IL_0000: ldarg.0      // this
        IL_0001: ldfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0006: stloc.0      // V_0
        .try
        {

            IL_0007: ldloc.0      // V_0
            IL_0008: brfalse.s    IL_000c
            IL_000a: br.s         IL_000e
            IL_000c: br.s         IL_004a

            // [8 9 - 8 10]
            IL_000e: nop

            // [11 13 - 11 42]
            IL_000f: call         valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32> ValueTaskResearch.Program::TestValueTaskWithNew()
            IL_0014: stloc.2      // V_2
            IL_0015: ldloca.s     V_2
            IL_0017: call         instance valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<!0/*int32*/> valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32>::GetAwaiter()
            IL_001c: stloc.1      // V_1

            IL_001d: ldloca.s     V_1
            IL_001f: call         instance bool valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>::get_IsCompleted()
            IL_0024: brtrue.s     IL_0066
            IL_0026: ldarg.0      // this
            IL_0027: ldc.i4.0
            IL_0028: dup
            IL_0029: stloc.0      // V_0
            IL_002a: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_002f: ldarg.0      // this
            IL_0030: ldloc.1      // V_1
            IL_0031: stfld        valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0036: ldarg.0      // this
            IL_0037: stloc.3      // V_3
            IL_0038: ldarg.0      // this
            IL_0039: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_003e: ldloca.s     V_1
            IL_0040: ldloca.s     V_3
            IL_0042: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::AwaitUnsafeOnCompleted<valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>, class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>*/&, !!1/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
            IL_0047: nop
            IL_0048: leave.s      IL_009e
            IL_004a: ldarg.0      // this
            IL_004b: ldfld        valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0050: stloc.1      // V_1
            IL_0051: ldarg.0      // this
            IL_0052: ldflda       valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32> ValueTaskResearch.Program/'<Main>d__0'::'<>u__1'
            IL_0057: initobj      valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>
            IL_005d: ldarg.0      // this
            IL_005e: ldc.i4.m1
            IL_005f: dup
            IL_0060: stloc.0      // V_0
            IL_0061: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_0066: ldloca.s     V_1
            IL_0068: call         instance !0/*int32*/ valuetype [System.Runtime]System.Runtime.CompilerServices.ValueTaskAwaiter`1<int32>::GetResult()
            IL_006d: pop
            IL_006e: leave.s      IL_008a
        } // end of .try
        catch [System.Runtime]System.Exception
        {

            IL_0070: stloc.s      V_4
            IL_0072: ldarg.0      // this
            IL_0073: ldc.i4.s     -2 // 0xfe
            IL_0075: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
            IL_007a: ldarg.0      // this
            IL_007b: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
            IL_0080: ldloc.s      V_4
            IL_0082: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetException(class [System.Runtime]System.Exception)
            IL_0087: nop
            IL_0088: leave.s      IL_009e
        } // end of catch

        // [12 9 - 12 10]
        IL_008a: ldarg.0      // this
        IL_008b: ldc.i4.s     -2 // 0xfe
        IL_008d: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'

        IL_0092: ldarg.0      // this
        IL_0093: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0098: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::SetResult()
        IL_009d: nop
        IL_009e: ret

        } // end of method '<Main>d__0'::MoveNext

        .method private final hidebysig virtual newslot instance void
        SetStateMachine(
            class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine stateMachine
        ) cil managed
        {
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerHiddenAttribute::.ctor()
            = (01 00 00 00 )
        .override method instance void [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine::SetStateMachine(class [System.Runtime]System.Runtime.CompilerServices.IAsyncStateMachine)
        .maxstack 8

        IL_0000: ret

        } // end of method '<Main>d__0'::SetStateMachine
    } // end of class '<Main>d__0'

    .method private hidebysig static class [System.Runtime]System.Threading.Tasks.Task
        Main() cil managed
    {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.AsyncStateMachineAttribute::.ctor(class [System.Runtime]System.Type)
        = (
            01 00 24 56 61 6c 75 65 54 61 73 6b 52 65 73 65 // ..$ValueTaskRese
            61 72 63 68 2e 50 72 6f 67 72 61 6d 2b 3c 4d 61 // arch.Program+<Ma
            69 6e 3e 64 5f 5f 30 00 00                      // in>d__0..
        )
        // type(class ValueTaskResearch.Program/'<Main>d__0')
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 2
        .locals init (
        [0] class ValueTaskResearch.Program/'<Main>d__0' V_0,
        [1] valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder V_1
        )

        IL_0000: newobj       instance void ValueTaskResearch.Program/'<Main>d__0'::.ctor()
        IL_0005: stloc.0      // V_0
        IL_0006: ldloc.0      // V_0
        IL_0007: call         valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Create()
        IL_000c: stfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_0011: ldloc.0      // V_0
        IL_0012: ldc.i4.m1
        IL_0013: stfld        int32 ValueTaskResearch.Program/'<Main>d__0'::'<>1__state'
        IL_0018: ldloc.0      // V_0
        IL_0019: ldfld        valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_001e: stloc.1      // V_1
        IL_001f: ldloca.s     V_1
        IL_0021: ldloca.s     V_0
        IL_0023: call         instance void [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::Start<class ValueTaskResearch.Program/'<Main>d__0'>(!!0/*class ValueTaskResearch.Program/'<Main>d__0'*/&)
        IL_0028: ldloc.0      // V_0
        IL_0029: ldflda       valuetype [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder ValueTaskResearch.Program/'<Main>d__0'::'<>t__builder'
        IL_002e: call         instance class [System.Runtime]System.Threading.Tasks.Task [System.Threading.Tasks]System.Runtime.CompilerServices.AsyncTaskMethodBuilder::get_Task()
        IL_0033: ret

    } // end of method Program::Main

    .method private hidebysig static valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32>
        TestValueTaskWithNew() cil managed
    {
        .maxstack 1
        .locals init (
        [0] valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32> V_0
        )

        // [29 9 - 29 10]
        IL_0000: nop

        // [30 13 - 30 42]
        IL_0001: ldc.i4.0
        IL_0002: newobj       instance void valuetype [System.Runtime]System.Threading.Tasks.ValueTask`1<int32>::.ctor(!0/*int32*/)
        IL_0007: stloc.0      // V_0
        IL_0008: br.s         IL_000a

        // [31 9 - 31 10]
        IL_000a: ldloc.0      // V_0
        IL_000b: ret

    } // end of method Program::TestValueTaskWithNew

    .method public hidebysig specialname rtspecialname instance void
        .ctor() cil managed
    {
        .maxstack 8

        IL_0000: ldarg.0      // this
        IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
        IL_0006: nop
        IL_0007: ret

    } // end of method Program::.ctor

    .method private hidebysig static specialname void
        '<Main>'() cil managed
    {
        .entrypoint
        .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerStepThroughAttribute::.ctor()
        = (01 00 00 00 )
        .maxstack 1
        .locals init (
        [0] valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter V_0
        )

        IL_0000: call         class [System.Runtime]System.Threading.Tasks.Task ValueTaskResearch.Program::Main()
        IL_0005: callvirt     instance valuetype [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter [System.Runtime]System.Threading.Tasks.Task::GetAwaiter()
        IL_000a: stloc.0      // V_0
        IL_000b: ldloca.s     V_0
        IL_000d: call         instance void [System.Runtime]System.Runtime.CompilerServices.TaskAwaiter::GetResult()
        IL_0012: ret

    } // end of method Program::'<Main>'
    } // end of class ValueTaskResearch.Program
    ```

    </p>
    </details>

## 心得

看完 IL 後是不是一切豁然開朗 ？！  說實話我個人是沒有這種感覺啦 XD

所幸在 [C# Language ValueTask](https://riptutorial.com/csharp/example/28612/valuetask-t-) 找到我個人可以接受的說明：

> 過去的在實作 async 卻希望使用同步呼叫時會需要使用 `Task.Run` 或 `Task.FromResult`，無形之中就造成效能耗損，而 `ValueTask` 可同時用於 __***同步實作 (Synchronous implementation)***__ 與 __***非同步實作 (Asynchronous implementation)***__

1. `同步實作 (Synchronous implementation)`

    > 同步方法適用，方法簽章中沒有 `async`，方法中沒有用到 `await`，就需要 `new ValueTask` 來傳遞回傳值

    ```cs
    private static ValueTask<int> TestValueTaskWithNew()
    {
        return new ValueTask<int>(0);
    }
    ```

2. `非同步實作 (Asynchronous implementation)`

    > 非同步方法適用，方法簽章中有 `async`，方法中有用到 `await`，可以直傳回傳 value type，不需要 `new ValueTask`

    ```cs
    private static async ValueTask<int> TestValueTask()
    {
        await Task.Delay(100);

        return 10;
    }
    ```

記了這麼多，好像都是廢話，但小弟資質駑鈍，就麻煩各位大大提點了 ~~~~

## 參考資訊

1. [閱讀筆記 - 使用 .NET Async/Await 的常見錯誤](https://blog.darkthread.net/blog/common-async-await-mistakes/)
2. [Correcting Common Async/Await Mistakes in .NET - Brandon Minnick](https://www.youtube.com/watch?v=J0mcYVxJEl0)
3. [async 與 await by Huanlin學習筆記](https://www.huanlintalk.com/2016/01/async-and-await.html)
4. [有關Task ConfigureAwait() 的一些事情| 微軟開發工具資訊分享- 點部落](https://dotblogs.com.tw/aspnetshare/2018/06/03/configureawaiter)
5. [C# Language ValueTask](https://riptutorial.com/csharp/example/28612/valuetask-t-)
