---
title: "System.IO.FileLoadException : Could not load file or assembly (0x80131040)"
date: 2018-07-09T00:39:00+08:00
lastmod: 2018-10-07T00:39:29+08:00
draft: false
tags: ["Debug","Windows Service"]
slug: "system-io-fileloadexception-assembly"
aliases:
    - /2018/07/system-io-fileloadexception-assembly.html
---
# System.IO.FileLoadException : Could not load file or assembly
前情提要：公司有個專案需要與外部 partner 週期性進行資料交換，是常見的排程作業但 partner 沒有提供測試環境，加上 partner 的 api 環境有鎖定來源 ip，也就是所有的功能開發都需要憑空想像或是透過 production 來 debug XD 雖然不是第一次遇到這種需求，但還是覺得無力感很重呀

不過就是因為這個專案的特殊性才讓我遇到 `System.IO.FileLoadException : Could not load file or assembly` 問題，當下我花了一陣子才想到原因加上違反正常 CI/CD 流程，特別筆記一下加深印象


## 錯誤訊息
1. 訊息內容
    
    ```
    c:\>AD3ToMongoDB.exe install
    Topshelf.HostFactory Error: 0 : An exception occurred creating the host, System.IO.FileLoadException: Could not load file or assembly 'Topshelf, Version=3.2.150.0, Culture=neutral, PublicKeyToken=b800c4cfcdeea87b' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference. (Exception from HRESULT: 0x80131040)
    File name: 'Topshelf, Version=3.2.150.0, Culture=neutral, PublicKeyToken=b800c4cfcdeea87b'
    at AD3ToMongoDB.Program.<>c.<Main>b__0_1(ServiceConfigurator`1 s)
    at Topshelf.ServiceExtensions.CreateServiceBuilderFactory[TService](Action`1 callback)
    at Topshelf.ServiceExtensions.Service[TService](HostConfigurator configurator, Action`1 callback)
    at AD3ToMongoDB.Program.<>c.<Main>b__0_0(HostConfigurator x) in C:\Users\yowko.tsai\source\repos\AD3ToMongoDB\Program.cs:line 19
    at Topshelf.HostFactory.New(Action`1 configureCallback)

    WRN: Assembly binding logging is turned OFF.
    To enable assembly bind failure logging, set the registry value [HKLM\Software\Microsoft\Fusion!EnableLog] (DWORD) to 1.
    Note: There is some performance penalty associated with assembly bind failure logging.
    To turn this feature off, remove the registry value [HKLM\Software\Microsoft\Fusion!EnableLog].

    Topshelf.HostFactory Error: 0 : The service terminated abnormally, System.IO.FileLoadException: Could not load file or assembly 'Topshelf, Version=3.2.150.0, Culture=neutral, PublicKeyToken=b800c4cfcdeea87b' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference. (Exception from HRESULT: 0x80131040)
    File name: 'Topshelf, Version=3.2.150.0, Culture=neutral, PublicKeyToken=b800c4cfcdeea87b'
    at AD3ToMongoDB.Program.<>c.<Main>b__0_1(ServiceConfigurator`1 s)
    at Topshelf.ServiceExtensions.CreateServiceBuilderFactory[TService](Action`1 callback)
    at Topshelf.ServiceExtensions.Service[TService](HostConfigurator configurator, Action`1 callback)
    at AD3ToMongoDB.Program.<>c.<Main>b__0_0(HostConfigurator x) in C:\Users\yowko.tsai\source\repos\AD3ToMongoDB\Program.cs:line 19
    at Topshelf.HostFactory.New(Action`1 configureCallback)
    at Topshelf.HostFactory.Run(Action`1 configureCallback)

    WRN: Assembly binding logging is turned OFF.
    To enable assembly bind failure logging, set the registry value [HKLM\Software\Microsoft\Fusion!EnableLog] (DWORD) to 1.
    Note: There is some performance penalty associated with assembly bind failure logging.
    To turn this feature off, remove the registry value [HKLM\Software\Microsoft\Fusion!EnableLog].
    ```
2. 錯誤截圖
    
    ![1error](https://user-images.githubusercontent.com/3851540/42421830-7bd46c04-830e-11e8-8a2e-c75b0655a653.png)

## 問題釐清
1. 確認正確引用 `Topshelf` dll
    
    ![2reference](https://user-images.githubusercontent.com/3851540/42421831-7bfc41fc-830e-11e8-8f14-39303ce40fb3.png)
2. 檢查是否複製 dll 至最終 output 資料夾
    
    ![3copy](https://user-images.githubusercontent.com/3851540/42421832-7c242fb4-830e-11e8-8455-71d49d673712.png)
3. 確認 Topshelf.HostFactory 使用的 `Topshelf` dll 版本
    
    ![4topshelfinmenifest](https://user-images.githubusercontent.com/3851540/42421828-7b86870a-830e-11e8-82a8-513991e2e3e1.png)
4. 確認 `Topshelf` dll 版本
    
    ![5topshelfversion](https://user-images.githubusercontent.com/3851540/42421829-7bad539e-830e-11e8-8bd4-a75f540f685a.png)

## 解決方式
* 加入 `Topshelf` 的 bindingRedirect 
    1.  基本設定範例
        
        ```xml
        <dependentAssembly>
            <assemblyIdentity name="Topshelf" publicKeyToken="b800c4cfcdeea87b" culture="neutral" />
            <bindingRedirect oldVersion="0.0.0.0-4.0.0.0" newVersion="4.0.0.0" />
        </dependentAssembly>
        ```
    2. 完整階層範例
        
        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <configuration>
            <configSections>
                <runtime>
                    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
                        <dependentAssembly>
                            <assemblyIdentity name="Topshelf" publicKeyToken="b800c4cfcdeea87b" culture="neutral" />
                            <bindingRedirect oldVersion="0.0.0.0-4.0.0.0" newVersion="4.0.0.0" />
                        </dependentAssembly>
                    </assemblyBinding>
                </runtime>
            <configuration>
        <configSections>
        ``` 

## 心得
這是我第一次手動為 Topshelf 加上 bindingRedirect，過去雖然也常發生 dll 相依設定與實際版本不一致的狀況，但大多在進行 NuGet 套件安裝時就會自動加上 bindingRedirect，記起這件事立馬重新檢視 config 設定也才發現原來確實已自動加上，而這就是違反一般正常 CI/CS 流程所造成的主要問題：因為專案是直接透過 production 開發，在功能確認完成前並未加入標準 CI/CD 的設定中，造成直接部署時沒留意到 config 的變更而讓 bindingRedirect 的相關設定未被加入

雖然這次遇到的狀況是來自於需要配合 partner 的特殊性，不過追根究柢還是因為沒有預先建立 CI/CD 設定而造成的，再次親身體驗 CI/CD 標準化的重要性呀


# 參考資訊
1. [Running an application targeted for net461: Could not load file or assembly 'System.Runtime, Version=4.1.0.0](https://github.com/dotnet/standard/issues/295)
2. [<assemblyBinding> Element for <runtime>](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/runtime/assemblybinding-element-for-runtime?WT.mc_id=DOP-MVP-5002594)
3. [Configuring Assembly Binding Redirection](https://docs.microsoft.com/en-us/dotnet/framework/deployment/configuring-assembly-binding-redirection?WT.mc_id=DOP-MVP-5002594)
4. [Side-by-Side Execution in the .NET Framework](https://docs.microsoft.com/en-us/dotnet/framework/deployment/side-by-side-execution?WT.mc_id=DOP-MVP-5002594)
5. [Redirecting Assembly Versions](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/redirect-assembly-versions?WT.mc_id=DOP-MVP-5002594)
6. [How to: Enable and Disable Automatic Binding Redirection](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/how-to-enable-and-disable-automatic-binding-redirection?WT.mc_id=DOP-MVP-5002594)