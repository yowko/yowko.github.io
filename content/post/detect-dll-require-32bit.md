---
title: "如何知道使用的 dll 是否需以 32 位元模式執行"
date: 2018-05-08T09:57:00+08:00
lastmod: 2021-11-03T09:57:26+08:00
draft: false
tags: ["csharp","IIS","PowerShell","Tools"]
slug: "detect-dll-require-32bit"
aliases:
    - /2018/05/detect-dll-require-32bit.html
---
## 如何知道使用的 dll 是否需以 32 位元模式執行

最近因為年度計畫預計做些 Windows server OS 的升級作業，將大部份 OS 升級為 Windows Server 2016，也順便整理 server 上的 application，發現仍有不少 application 執行在 32 位模式下，過去主要的原因是 `Oracle.DataAccess.dll`，原以為只要搞定 `Oracle.DataAccess.dll` 就可以，想不到後續又遇到其他參考的 dll 需要執行在 32 bit 模式下，所以找了幾個方式來試著不需要反覆部署至 IIS 上進行測試即可得知特定的 dll 是否需要執行在 32 bit 的相容執行環境

網路上分享的方法有很多種，今天會以本次遇到問題的 dll 中的 `Oracle.DataAccess.dll` 與 `System.Data.dll`當做範例來進行測試，立馬來看看哪個方式比較方便吧

## 使用 dumpbin：<span style="color:red">無法識別是否需要 32 位元模式</span>

> 曾經在之前筆記 [如何看程式是 32 bit 還是 64 bit](/binary-is-32-bit-64-bit) 紀錄到，詳細的使用方式可以參考原文 [如何看程式是 32 bit 還是 64 bit](/binary-is-32-bit-64-bit)

- 使用方式

    ```bash
    dumpbin.exe /headers {檔案}
    ```

- 實際使用
    - `System.Data.dll`
        - 需要 32 位元模式

            ```cmd
            $ "G:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64\dumpbin.exe" /headers D:\Test\System.Data.dll
            Microsoft (R) COFF/PE Dumper Version 12.00.40629.0
            Copyright (C) Microsoft Corporation.  All rights reserved.


            Dump of file D:\Test\System.Data.dll

            PE signature found

            File Type: DLL

            FILE HEADER VALUES
                        14C machine (x86)
                        5 number of sections
                    471EBF27 time date stamp Wed Oct 24 11:42:31 2007
                        0 file pointer to symbol table
                        0 number of symbols
                        E0 size of optional header
                        2102 characteristics
                            Executable
                            32 bit word machine
                            DLL

            OPTIONAL HEADER VALUES
                        10B magic # (PE32)
                        8.00 linker version
                    2A4800 size of code
                    40800 size of initialized data
                        0 size of uninitialized data
                    2A4630 entry point (65114630)
                        1000 base of code
                    2A6000 base of data
                    64E70000 image base (64E70000 to 6515BFFF)
                        1000 section alignment
                        200 file alignment
                        5.00 operating system version
                        8.00 image version
                        4.10 subsystem version
                        0 Win32 version
                    2EC000 size of image
                        400 size of headers
                    2EDF46 checksum
                        3 subsystem (Windows CUI)
                        0 DLL characteristics
                    100000 size of stack reserve
                        1000 size of stack commit
                    100000 size of heap reserve
                        1000 size of heap commit
                        0 loader flags
                        10 number of directories
                    2A5580 [     1A2] RVA [size] of Export Directory
                    2A478C [      B4] RVA [size] of Import Directory
                    2AE000 [   35F40] RVA [size] of Resource Directory
                        0 [       0] RVA [size] of Exception Directory
                        0 [       0] RVA [size] of Certificates Directory
                    2E4000 [    4BCC] RVA [size] of Base Relocation Directory
                        1330 [      1C] RVA [size] of Debug Directory
                        0 [       0] RVA [size] of Architecture Directory
                        0 [       0] RVA [size] of Global Pointer Directory
                        0 [       0] RVA [size] of Thread Storage Directory
                    116D0 [      40] RVA [size] of Load Configuration Directory
                        0 [       0] RVA [size] of Bound Import Directory
                        1000 [     2D8] RVA [size] of Import Address Table Directory
                        0 [       0] RVA [size] of Delay Import Directory
                    110C4 [      48] RVA [size] of COM Descriptor Directory
                        0 [       0] RVA [size] of Reserved Directory
            ```

            ![1dumpbinsystemdata](https://user-images.githubusercontent.com/3851540/39715721-c55a85ea-5260-11e8-895c-1374bca2dfa7.png)

        - 不需 32 位元模式

            ```log
            Dump of file D:\Test\System.Data_ok.dll

            PE signature found

            File Type: DLL

            FILE HEADER VALUES
                        14C machine (x86)
                        3 number of sections
                    4BA1E072 time date stamp Thu Mar 18 16:12:34 2010
                        0 file pointer to symbol table
                        0 number of symbols
                        E0 size of optional header
                        2022 characteristics
                            Executable
                            Application can handle large (>2GB) addresses
                            DLL

            OPTIONAL HEADER VALUES
                        10B magic # (PE32)
                    10.00 linker version
                    142600 size of code
                        800 size of initialized data
                        0 size of uninitialized data
                    142A7A entry point (615B2A7A)
                        2000 base of code
                    146000 base of data
                    61470000 image base (61470000 to 615B9FFF)
                        2000 section alignment
                        200 file alignment
                        4.00 operating system version
                        0.00 image version
                        4.00 subsystem version
                        0 Win32 version
                    14A000 size of image
                        200 size of headers
                    14ED98 checksum
                        3 subsystem (Windows CUI)
                        140 DLL characteristics
                            Dynamic base
                            NX compatible
                    100000 size of stack reserve
                        1000 size of stack commit
                    100000 size of heap reserve
                        1000 size of heap commit
                        0 loader flags
                        10 number of directories
                        0 [       0] RVA [size] of Export Directory
                    142A28 [      4F] RVA [size] of Import Directory
                    146000 [     434] RVA [size] of Resource Directory
                        0 [       0] RVA [size] of Exception Directory
                    143000 [    1758] RVA [size] of Certificates Directory
                    148000 [       C] RVA [size] of Base Relocation Directory
                        0 [       0] RVA [size] of Debug Directory
                        0 [       0] RVA [size] of Architecture Directory
                        0 [       0] RVA [size] of Global Pointer Directory
                        0 [       0] RVA [size] of Thread Storage Directory
                        0 [       0] RVA [size] of Load Configuration Directory
                        0 [       0] RVA [size] of Bound Import Directory
                        2000 [       8] RVA [size] of Import Address Table Directory
                        0 [       0] RVA [size] of Delay Import Directory
                        2008 [      48] RVA [size] of COM Descriptor Directory
                        0 [       0] RVA [size] of Reserved Directory
            ```

            ![2dumpbinsystemdata32](https://user-images.githubusercontent.com/3851540/39715722-c583e520-5260-11e8-8ae9-8fd9f2e60a17.png)
    - `Oracle.DataAccess.dll`
        - 需要 32 位元

            ```cmd
            $ "G:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64\dumpbin.exe" /headers D:\Test\Oracle.DataAccess.dll
            Microsoft (R) COFF/PE Dumper Version 12.00.40629.0
            Copyright (C) Microsoft Corporation.  All rights reserved.


            Dump of file D:\Test\Oracle.DataAccess.dll

            PE signature found

            File Type: DLL

            FILE HEADER VALUES
                        14C machine (x86)
                        3 number of sections
                    48B315B4 time date stamp Tue Aug 26 04:27:32 2008
                        0 file pointer to symbol table
                        0 number of symbols
                        E0 size of optional header
                        210E characteristics
                            Executable
                            Line numbers stripped
                            Symbols stripped
                            32 bit word machine
                            DLL

            OPTIONAL HEADER VALUES
                        10B magic # (PE32)
                        8.00 linker version
                    DE000 size of code
                        2000 size of initialized data
                        0 size of uninitialized data
                    DFD1E entry point (110DFD1E)
                        2000 base of code
                    E0000 base of data
                    11000000 image base (11000000 to 110E3FFF)
                        2000 section alignment
                        1000 file alignment
                        4.00 operating system version
                        0.00 image version
                        4.00 subsystem version
                        0 Win32 version
                    E4000 size of image
                        1000 size of headers
                    E5C54 checksum
                        3 subsystem (Windows CUI)
                        540 DLL characteristics
                            Dynamic base
                            NX compatible
                            No structured exception handler
                    100000 size of stack reserve
                        1000 size of stack commit
                    100000 size of heap reserve
                        1000 size of heap commit
                        0 loader flags
                        10 number of directories
                        0 [       0] RVA [size] of Export Directory
                    DFCC4 [      57] RVA [size] of Import Directory
                    E0000 [     460] RVA [size] of Resource Directory
                        0 [       0] RVA [size] of Exception Directory
                        0 [       0] RVA [size] of Certificates Directory
                    E2000 [       C] RVA [size] of Base Relocation Directory
                        0 [       0] RVA [size] of Debug Directory
                        0 [       0] RVA [size] of Architecture Directory
                        0 [       0] RVA [size] of Global Pointer Directory
                        0 [       0] RVA [size] of Thread Storage Directory
                        0 [       0] RVA [size] of Load Configuration Directory
                        0 [       0] RVA [size] of Bound Import Directory
                        2000 [       8] RVA [size] of Import Address Table Directory
                        0 [       0] RVA [size] of Delay Import Directory
                        2008 [      48] RVA [size] of COM Descriptor Directory
                        0 [       0] RVA [size] of Reserved Directory
            ```

            ![3dumpbinoracle](https://user-images.githubusercontent.com/3851540/39715723-c5ab92e6-5260-11e8-8b7f-a868594996c6.png)

        - 不需要 32 位元

            ```cmd
            $ "G:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\amd64\dumpbin.exe" /headers D:\Test\Oracle.DataAccess_ok.dll
            Microsoft (R) COFF/PE Dumper Version 12.00.40629.0
            Copyright (C) Microsoft Corporation.  All rights reserved.


            Dump of file D:\Test\Oracle.DataAccess_ok.dll

            PE signature found

            File Type: DLL

            FILE HEADER VALUES
                        8664 machine (x64)
                        2 number of sections
                    4B9A20B6 time date stamp Fri Mar 12 19:08:38 2010
                        0 file pointer to symbol table
                        0 number of symbols
                        F0 size of optional header
                        212E characteristics
                            Executable
                            Line numbers stripped
                            Symbols stripped
                            Application can handle large (>2GB) addresses
                            32 bit word machine
                            DLL

            OPTIONAL HEADER VALUES
                        20B magic # (PE32+)
                        8.00 linker version
                    F6000 size of code
                        1000 size of initialized data
                        0 size of uninitialized data
                        0 entry point
                        2000 base of code
                    11000000 image base (0000000011000000 to 00000000110F9FFF)
                        2000 section alignment
                        1000 file alignment
                        4.00 operating system version
                        0.00 image version
                        4.00 subsystem version
                        0 Win32 version
                    FA000 size of image
                        1000 size of headers
                    1036BB checksum
                        3 subsystem (Windows CUI)
                        540 DLL characteristics
                            Dynamic base
                            NX compatible
                            No structured exception handler
                    400000 size of stack reserve
                        4000 size of stack commit
                    100000 size of heap reserve
                        2000 size of heap commit
                        0 loader flags
                        10 number of directories
                        0 [       0] RVA [size] of Export Directory
                        0 [       0] RVA [size] of Import Directory
                    F8000 [     460] RVA [size] of Resource Directory
                        0 [       0] RVA [size] of Exception Directory
                        0 [       0] RVA [size] of Certificates Directory
                        0 [       0] RVA [size] of Base Relocation Directory
                        0 [       0] RVA [size] of Debug Directory
                        0 [       0] RVA [size] of Architecture Directory
                        0 [       0] RVA [size] of Global Pointer Directory
                        0 [       0] RVA [size] of Thread Storage Directory
                        0 [       0] RVA [size] of Load Configuration Directory
                        0 [       0] RVA [size] of Bound Import Directory
                        0 [       0] RVA [size] of Import Address Table Directory
                        0 [       0] RVA [size] of Delay Import Directory
                        2000 [      48] RVA [size] of COM Descriptor Directory
                        0 [       0] RVA [size] of Reserved Directory
            ```

            ![4dumpbinoracle](https://user-images.githubusercontent.com/3851540/39715724-c5d5b88c-5260-11e8-89a8-6035c798d279.png)

## 使用 CorFlags：<span style="color:red">個人測試下不一定可以識別是否需要 32 位元模式</span>

> `CorFlags.exe` 是 .net framework 的內建工具

- 判斷基準

    platform|PE|32BITREQ
    :---|:---|:---
    Any CPU|PE32| 0
    x86| PE32| 1
    x64|PE32+| 0

- 實際使用
    - `System.Data.dll`
        - 需要 32 位元模式

            ```cmd
            $ "C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.7.1 Tools\CorFlags.exe" d:\TEST\System.Data.dll
            Microsoft (R) .NET Framework CorFlags Conversion Tool.  Version  4.7.2558.0
            Copyright (c) Microsoft Corporation.  All rights reserved.

            Version   : v2.0.50727
            CLR Header: 2.5
            PE        : PE32
            CorFlags  : 0x18
            ILONLY    : 0
            32BITREQ  : 0
            32BITPREF : 0
            Signed    : 1 
            ```

            ![5corflagssystemdata](https://user-images.githubusercontent.com/3851540/39715725-c5fc2440-5260-11e8-91b7-2a1fe5456262.png)
        - 不需要 32 位元模式

            ```cmd
            $ "C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.7.1 Tools\CorFlags.exe" d:\TEST\System.Data_ok.dll
            Microsoft (R) .NET Framework CorFlags Conversion Tool.  Version  4.7.2558.0
            Copyright (c) Microsoft Corporation.  All rights reserved.

            Version   : v4.0.30319
            CLR Header: 2.5
            PE        : PE32
            CorFlags  : 0x9
            ILONLY    : 1
            32BITREQ  : 0
            32BITPREF : 0
            Signed    : 1
            ```

            ![6corflagsystemdata](https://user-images.githubusercontent.com/3851540/39715727-c624472c-5260-11e8-9e4b-46da12ef7119.png)
    - `Oracle.DataAccess.dll`
        - 需要 32 位元模式

            ```cmd
            $ "C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.7.1 Tools\CorFlags.exe" d:\TEST\Oracle.DataAccess.dll
            Microsoft (R) .NET Framework CorFlags Conversion Tool.  Version  4.7.2558.0
            Copyright (c) Microsoft Corporation.  All rights reserved.

            Version   : v2.0.50727
            CLR Header: 2.5
            PE        : PE32
            CorFlags  : 0xb
            ILONLY    : 1
            32BITREQ  : 1
            32BITPREF : 0
            Signed    : 1
            ```

            ![7corflagoracle](https://user-images.githubusercontent.com/3851540/39715728-c64d8812-5260-11e8-9f18-7ca879c4ab9b.png)
        - 不需要 32 位元模式

            ```cmd
            $ "C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.7.1 Tools\CorFlags.exe" d:\TEST\Oracle.DataAccess_ok.dll
            Microsoft (R) .NET Framework CorFlags Conversion Tool.  Version  4.7.2558.0
            Copyright (c) Microsoft Corporation.  All rights reserved.

            Version   : v2.0.50727
            CLR Header: 2.5
            PE        : PE32+
            CorFlags  : 0x9
            ILONLY    : 1
            32BITREQ  : 0
            32BITPREF : 0
            Signed    : 1
            ```

            ![8corflagoracle](https://user-images.githubusercontent.com/3851540/39715730-c683bcac-5260-11e8-9cb4-8c461d341e92.png)

## 使用 powershell：<span style="color:red">勉強可識別是否需要 32 位元模式</span>

- ProcessorArchitecture

    ProcessorArchitecture|說明
    :---|:---
    Amd64|僅 64-bit AMD 處理器適用
    Arm    | ARM 處理器適用
    IA64    |僅 64-bit Intel 處理器適用.
    MSIL    |不限處理器
    None    |無法辨識處理器
    X86    |32-bit Intel 處理器, 或是 64-bit 平台上 Windows on Windows 環境(WOW64).

- 實際使用
    - `System.Data.dll`
        - 需要 32 位元模式

            ```ps1
            PS C:\Users\yowko.tsai> $dllpath="d:\TEST\System.Data.dll"

            [reflection.assemblyname]::GetAssemblyName($dllpath) | fl


            Name                  : System.Data
            Version               : 2.0.0.0
            CultureInfo           : 
            CultureName           : 
            CodeBase              : file:///d:/TEST/System.Data.dll
            EscapedCodeBase       : file:///d:/TEST/System.Data.dll
            ProcessorArchitecture : X86
            ContentType           : Default
            Flags                 : PublicKey
            HashAlgorithm         : SHA1
            VersionCompatibility  : SameMachine
            KeyPair               : 
            FullName              : System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089
            ```

            ![9pssystemdata](https://user-images.githubusercontent.com/3851540/39715731-c6af75a4-5260-11e8-9b1a-632b027f90e4.png)

        - 不需要 32 位元模式

            ```ps1
            PS C:\Users\yowko.tsai> $dllpath="d:\TEST\System.Data_ok.dll"

            [reflection.assemblyname]::GetAssemblyName($dllpath) | fl


            Name                  : System.Data
            Version               : 4.0.0.0
            CultureInfo           : 
            CultureName           : 
            CodeBase              : file:///d:/TEST/System.Data_ok.dll
            EscapedCodeBase       : file:///d:/TEST/System.Data_ok.dll
            ProcessorArchitecture : None
            ContentType           : Default
            Flags                 : PublicKey
            HashAlgorithm         : SHA1
            VersionCompatibility  : SameMachine
            KeyPair               : 
            FullName              : System.Data, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089

            ```

            ![10pssystemdata](https://user-images.githubusercontent.com/3851540/39715733-c6d750b0-5260-11e8-82ad-41dc242598f5.png)

    - `Oracle.DataAccess.dll`
        - 需要 32 位元模式

            ```ps1
            PS C:\Users\yowko.tsai> $dllpath="d:\TEST\Oracle.DataAccess.dll"

            [reflection.assemblyname]::GetAssemblyName($dllpath) | fl


            Name                  : Oracle.DataAccess
            Version               : 2.111.7.0
            CultureInfo           : 
            CultureName           : 
            CodeBase              : file:///d:/TEST/Oracle.DataAccess.dll
            EscapedCodeBase       : file:///d:/TEST/Oracle.DataAccess.dll
            ProcessorArchitecture : X86
            ContentType           : Default
            Flags                 : PublicKey
            HashAlgorithm         : SHA1
            VersionCompatibility  : SameMachine
            KeyPair               : 
            FullName              : Oracle.DataAccess, Version=2.111.7.0, Culture=neutral, PublicKeyToken=89b483f429c47342
            ```

            ![11psoracle](https://user-images.githubusercontent.com/3851540/39715734-c7248056-5260-11e8-8998-c9a7ac0c115e.png)
        - 不需要 32 位元模式

            ```ps1
            PS C:\Users\yowko.tsai> $dllpath="d:\TEST\Oracle.DataAccess_ok.dll"

            [reflection.assemblyname]::GetAssemblyName($dllpath) | fl


            Name                  : Oracle.DataAccess
            Version               : 2.112.1.0
            CultureInfo           : 
            CultureName           : 
            CodeBase              : file:///d:/TEST/Oracle.DataAccess_ok.dll
            EscapedCodeBase       : file:///d:/TEST/Oracle.DataAccess_ok.dll
            ProcessorArchitecture : Amd64
            ContentType           : Default
            Flags                 : PublicKey
            HashAlgorithm         : SHA1
            VersionCompatibility  : SameMachine
            KeyPair               : 
            FullName              : Oracle.DataAccess, Version=2.112.1.0, Culture=neutral, PublicKeyToken=89b483f429c47342
            ```

            ![12psoracle](https://user-images.githubusercontent.com/3851540/39715735-c74f7766-5260-11e8-9d7b-86f1ba68190f.png)

## 使用 c#：<span style="color:red">可識別是否需要 32 位元模式</span>

- 實際使用
    - `System.Data.dll`
        - 需要 32 位元模式

            ```cs
            string dllpath = @"D:\Test\System.Data.dll";
            Assembly assembly = Assembly.ReflectionOnlyLoadFrom(dllpath);
            assembly.ManifestModule.GetPEKind(out PortableExecutableKinds peKind, out ImageFileMachine machine);
            peKind.Dump();
            ```

            ![13cssystemdata](https://user-images.githubusercontent.com/3851540/39715736-c77b7848-5260-11e8-9911-8863abee38f7.png)
        - 不需要 32 位元模式

            ```cs
            string dllpath = @"D:\Test\System.Data_ok.dll";
            Assembly assembly = Assembly.ReflectionOnlyLoadFrom(dllpath);
            assembly.ManifestModule.GetPEKind(out PortableExecutableKinds peKind, out ImageFileMachine machine);
            peKind.Dump();
            ```

            ![14cssystemdata](https://user-images.githubusercontent.com/3851540/39715737-c7a18740-5260-11e8-83ea-82a9edad16d7.png)
    - `Oracle.DataAccess.dll`
        - 需要 32 位元模式

            ```cs
            string dllpath = @"D:\Test\Oracle.DataAccess.dll";
            Assembly assembly = Assembly.ReflectionOnlyLoadFrom(dllpath);
            assembly.ManifestModule.GetPEKind(out PortableExecutableKinds peKind, out ImageFileMachine machine);
            peKind.Dump();
            ```

            ![15csoracle](https://user-images.githubusercontent.com/3851540/39715740-c7cb025a-5260-11e8-8f45-7f339d698ab7.png)
        - 不需要 32 位元模式

            ```cs
            string dllpath = @"D:\Test\Oracle.DataAccess_ok.dll";
            Assembly assembly = Assembly.ReflectionOnlyLoadFrom(dllpath);
            assembly.ManifestModule.GetPEKind(out PortableExecutableKinds peKind, out ImageFileMachine machine);
            peKind.Dump();
            ```

            ![16csoracle](https://user-images.githubusercontent.com/3851540/39715742-c7f29586-5260-11e8-9c49-668d4a1de62d.png)

## 心得

方法乍看之下有好幾個，但實際使用上可以正確判讀出是否需要 32 bit 模式的並不多，嚴格說起來只有 PowerShell 跟 c# 的方法可以滿足要求，CorFlag 實際使用上有的 dll 可以正確判讀有的則不行，而 c# 與 PowerShell 相比，PowerShell 的訊息仍然相對模糊，c# 整體的訊息非常直覺，但 PowerShell 不用編譯的優點則是 C# 望塵莫及的

透過將幾個方式測試一輪順便紀錄，到時有需要就視情況取用即可，雖然花了一些時間，但日後再次用到時就划算了(應該)

## 參考資訊

1. [如何看程式是 32 bit 還是 64 bit](/binary-is-32-bit-64-bit)
2. [Check whether a .NET dll is built for Any CPU, x86, or x64](https://malvinly.com/2016/11/16/check-whether-a-net-dll-is-built-for-any-cpu-x86-or-x64/)
3. [ProcessorArchitecture 列舉](https://msdn.microsoft.com/library/system.reflection.processorarchitecture)
4. [How to determine if a .NET assembly was built for x86 or x64?](https://stackoverflow.com/questions/270531/how-to-determine-if-a-net-assembly-was-built-for-x86-or-x64)
5. [PortableExecutableKinds 列舉](https://msdn.microsoft.com/zh-tw/library/system.reflection.portableexecutablekinds%28v=vs.110%29.aspx)
