---
title: "Jenkins 2 如何使用 PowerShell 以及自定 build fail (指定 exit code)"
date: 2017-02-12T00:42:34+08:00
lastmod: 2021-11-02T00:42:34+08:00
draft: false
tags: ["Jenkins","DevOps","PowerShell"]
slug: "jenkins-powershell-plugin"
aliases:
    - /2017/02/jenkins2-powershell-plugin.html
---
## Jenkins 2 如何使用 PowerShell 以及自定 build fail (指定 exit code)

Jenkins 除了用來做為 CI(持續性整合) 工具外，也可以與其他 plugin 配合達成其他目的(e.g.IIS restart、檔案壓縮備份...)，今天就來看看可以怎麼與 PowerShell 整合執行 PowerShell 指令

## 文章大綱

1. 安裝 PowerShell plugin
2. 設定 Powershell plugin
3. PowerShell 丟出 build fail

## 1. 安裝 PowerShell plugin

1. Manage Jenkins --> Manage Plugins

    ![1plugins](https://cloud.githubusercontent.com/assets/3851540/22322569/e4ad8e7c-e3d7-11e6-961b-a638e2e8c81c.png)

2. Available --> Filter

    ![2install](https://cloud.githubusercontent.com/assets/3851540/22322567/e4acf1a6-e3d7-11e6-9268-6c3dec063e8c.png)

## 2. 設定 Powershell plugin

1. Build --> ADD BUILD STEP --> Windows PowerShell

    ![3powershellsetting](https://cloud.githubusercontent.com/assets/3851540/22322568/e4adad08-e3d7-11e6-8a2a-5aabf2094305.png)

2. Command

    > 直接寫 Powershell 語法

    ![4commnad](https://cloud.githubusercontent.com/assets/3851540/22322570/e4af87d6-e3d7-11e6-9451-a82bd90e3358.png)

## 3. PowerShell 丟出 build fail

預設情況 Jenkins 只要有執行 PowerShell，不論是否正確執行皆會視為 `SUCCESS`，所以需要手動拋出 build fail

>![5alwayserror](https://cloud.githubusercontent.com/assets/3851540/22322571/e4b23cb0-e3d7-11e6-9c69-7953c771c5ec.png)

* 手動拋出錯誤(使用 try catch 為例)
  * 將 PowerShell 實際執行的 command 用 try catch 包
  * catch 區段 丟出 `exit 1` 以通知 Jenkins 拋出 build fail

        ```ps1
        Try
        {
            Get-Content C:\securestringa.txt  -ErrorAction Stop
        }
        Catch
        {
            write-output "get data fail!"
            exit 1
        }
        ```

        ![6error](https://cloud.githubusercontent.com/assets/3851540/22322572/e4b41e7c-e3d7-11e6-9592-acb443b6e459.png)

## 參考資料

1. [PowerShell Plugin](https://wiki.jenkins-ci.org/display/JENKINS/PowerShell+Plugin)
