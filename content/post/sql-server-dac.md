---
title: "你認識 SQL Server 的資料層應用程式(Data-tier Applications - DAC)嗎?"
date: 2017-04-14T02:39:00+08:00
lastmod: 2018-09-17T02:39:51+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-dac"
aliases:
    - /2017/04/sql-server-dac.html
---
# 你認識 SQL Server 的資料層應用程式(Data-tier Applications - DAC)嗎?
DAC 首次出現於 SQL Server 2008 R2 ，但直至今日經過了這麼久時間還是很少聽到其他人討論這個功能，雖然曾經用過但還是認識不深，最近新專案在部署至 Azure 時選擇使用 DAC 相關技術，所以花了時間暸解內容，順手筆記

## 資料層應用程式(Data-tier Applications - DAC) 是什麼？

資料層應用程式(Data-tier Applications - DAC) 包含所有 SQL Server 物件 (e.g. table、view、stored procedure) 定義，可以用來進行資料庫部署

依內容分成兩種檔案：

1.  DACPAC
    *   僅資料庫結構描述
    *   適合用來部署

2.  BACPAC
    *   資料庫結構描述 + 資料內容
    *   適合用來還原資料庫

  

## DAC 所需權限

*   Deploy ：dbmanager 或擁有 CREATE DATABASE 權限
*   Delete：dbmanager  或擁有 DROP DATABASE 權限


## DACPAC 相關操作
1.  Extract
    
    > 將資料庫結構封裝為 DACPAC

    * 目標資料庫右鍵 --> Tasks --> Extract Data-tier Application...
        
        ![1extract](https://cloud.githubusercontent.com/assets/3851540/25018436/e993754a-20b9-11e7-911e-bb9a3d53975e.png)

    * Introduction
        
        ![2intro](https://cloud.githubusercontent.com/assets/3851540/25018438/e995d920-20b9-11e7-8a23-177de209b74b.png)

    * Set Properties
        *   Appliaction name(名稱需相同才能進行升級)
        *   Version(版本必需不同)
        *   Description
        *   Save tp DAC package file
            
            ![3info](https://cloud.githubusercontent.com/assets/3851540/25018437/e9945ab4-20b9-11e7-9720-5060d4395934.png)

    * Validation and Summary
        
        ![4validation](https://cloud.githubusercontent.com/assets/3851540/25018439/e997e332-20b9-11e7-9903-023b8f0c3488.png)

    * Build Package
        
        ![5inprogress](https://cloud.githubusercontent.com/assets/3851540/25018440/e9a0e69e-20b9-11e7-8e3b-76b201075bfb.png)

        ![6success](https://cloud.githubusercontent.com/assets/3851540/25018441/e9a636bc-20b9-11e7-9a07-2c15c12a987b.png)

2.  Deploy
    
    > 使用 DACPAC 建立資料庫, 預設將資料庫註冊為 DAC

    * 資料庫資料夾右鍵 --> Deploy Data-tier Application...

        ![7deploy](https://cloud.githubusercontent.com/assets/3851540/25018443/e9bf86a8-20b9-11e7-9712-88c66cf9aef3.png)

    * Intrduction

        ![8info](https://cloud.githubusercontent.com/assets/3851540/25018442/e9bb755e-20b9-11e7-9fb9-dc1b38b9bda4.png)

    * Select Package

        ![9filelocation](https://cloud.githubusercontent.com/assets/3851540/25018445/e9c0f948-20b9-11e7-8910-82f10bbfe913.png)

    * Update Configuration

        ![10dbname](https://cloud.githubusercontent.com/assets/3851540/25018444/e9c09520-20b9-11e7-9fd1-95364d546773.png)

    * Summary

        ![11summary](https://cloud.githubusercontent.com/assets/3851540/25018446/e9cfc37e-20b9-11e7-903b-2c43f0ea5b96.png)

    * Deploy DAC

        ![12inprogress](https://cloud.githubusercontent.com/assets/3851540/25018448/e9d1228c-20b9-11e7-91cd-2046bb725afc.png)

        ![13sucess](https://cloud.githubusercontent.com/assets/3851540/25018449/e9e81992-20b9-11e7-870b-39cb58252128.png)

3.  Register

    > 將資料庫註冊為 DAC(在 SQL Server 中建立版控)

    * 目標資料庫右鍵 --> Tasks --> Register as Data-tier Application...

        ![14register](https://cloud.githubusercontent.com/assets/3851540/25018454/e9fffb02-20b9-11e7-8aa4-748f76a59051.png)

    * Introduction

        ![15intro](https://cloud.githubusercontent.com/assets/3851540/25018450/e9faa4d6-20b9-11e7-9ded-f0de6a1a19d8.png)

    * Set Properties
        *   DAC instance name
        *   Application name
        *   Version
        *   Description

        ![16properties](https://cloud.githubusercontent.com/assets/3851540/25018451/e9fe1940-20b9-11e7-9ca0-720b3668484c.png)

    * Validation and Summary

        ![17validation](https://cloud.githubusercontent.com/assets/3851540/25018452/e9fe3d8a-20b9-11e7-914f-3ac846ab7dbe.png)

    * Register DAC

        ![18inpreogress](https://cloud.githubusercontent.com/assets/3851540/25018453/e9ff9630-20b9-11e7-838a-abe6ba5bdd8b.png)

        ![19success](https://cloud.githubusercontent.com/assets/3851540/25018455/ea1bcdaa-20b9-11e7-96ea-6f6928059a4f.png)

4.  Upgrade

    > 使用 DACPAC 來更新資料庫

    * 目標資料庫 --> Tasks --> Upgrade Data-tier Application...

        ![20upgrade](https://cloud.githubusercontent.com/assets/3851540/25018457/ea33d6fc-20b9-11e7-9f4b-919f68af78c2.png)

    * Introduction

        ![21intro](https://cloud.githubusercontent.com/assets/3851540/25018459/ea3715c4-20b9-11e7-99ae-d6c448996dac.png)

    * Select Package

        ![22select](https://cloud.githubusercontent.com/assets/3851540/25018460/ea3760c4-20b9-11e7-8281-e88fb00b56f8.png)

    * Detect Package

        ![23detect](https://cloud.githubusercontent.com/assets/3851540/25018456/ea33a876-20b9-11e7-92cf-c0f39de3189e.png)

        *   顯示 no cahnges ?  這應該是 bug
            
            ![24nochanges](https://cloud.githubusercontent.com/assets/3851540/25018458/ea34231e-20b9-11e7-8155-6af3552b9965.png)

    * Options
        *   失敗是否 rollback
            
            ![25option](https://cloud.githubusercontent.com/assets/3851540/25018461/ea530ac2-20b9-11e7-93f1-31a8c9e5090d.png)

    * Review Upgrade Plan

        ![26review](https://cloud.githubusercontent.com/assets/3851540/25018464/ea65e598-20b9-11e7-8e94-d01bc7f19665.png)

        *   可能造成資料遺失

            ![27dataloss](https://cloud.githubusercontent.com/assets/3851540/25018462/ea6434d2-20b9-11e7-8b9a-a3328b299bb6.png)

    * Summary

        ![28summary](https://cloud.githubusercontent.com/assets/3851540/25018463/ea650d30-20b9-11e7-9644-54caa0d1a946.png)

    * Upgrade DAC

        ![29inprogress](https://cloud.githubusercontent.com/assets/3851540/25018466/ea6a0e0c-20b9-11e7-9c73-f235ed8bcfab.png)

        ![30success](https://cloud.githubusercontent.com/assets/3851540/25018465/ea66df84-20b9-11e7-8b2d-91a90497bc58.png)

5.  Delete

    > 僅刪除在 SQL Server 的版本資訊

    *   目標資料庫右鍵 --> Tasks --> Delete Data-tier Application...

        ![31delete](https://cloud.githubusercontent.com/assets/3851540/25018467/ea7ae3ee-20b9-11e7-92dd-7e5fd38aa2e0.png)

    *   Introduction

        ![32intro](https://cloud.githubusercontent.com/assets/3851540/25018468/ea8c04e4-20b9-11e7-83e7-390a806ec154.png)

    *   Summary

        ![33summary](https://cloud.githubusercontent.com/assets/3851540/25018470/ea92d0e4-20b9-11e7-84a1-6ddb8c0abfeb.png)

    *   Delete DAC

        ![34success](https://cloud.githubusercontent.com/assets/3851540/25018471/ea9fad78-20b9-11e7-888a-40fcf20f863b.png)

## BACPAC 相關操作

1.  Export

    > 將資料庫 schema 及資料封裝

    * 目標資料庫右鍵 --> Tasks --> Export Data-tier Application...

        ![35export](https://cloud.githubusercontent.com/assets/3851540/25018469/ea8e9c86-20b9-11e7-9f19-63abe89eb86f.png)

    * Introduction

        ![36intro](https://cloud.githubusercontent.com/assets/3851540/25018475/eab9f7a0-20b9-11e7-8668-e489065c49da.png)

    * Export Settings

        * Settings

            > 選擇儲存位置及檔名

            ![37settings](https://cloud.githubusercontent.com/assets/3851540/25018472/eaae4fa4-20b9-11e7-8e6c-e7d35c7f8a5d.png)

        * Advanced

            > 選擇匯出內容

            ![38advanced](https://cloud.githubusercontent.com/assets/3851540/25018473/eab5a588-20b9-11e7-803d-7f9571b9d624.png)

    * Summary

        ![39summary](https://cloud.githubusercontent.com/assets/3851540/25018474/eab81386-20b9-11e7-8d01-4ea2d525db65.png)

    * Results

        ![40inprogress](https://cloud.githubusercontent.com/assets/3851540/25018477/ead12b0a-20b9-11e7-8380-19fe6d4cf242.png)

        ![41success](https://cloud.githubusercontent.com/assets/3851540/25018476/eacbe2b2-20b9-11e7-8900-fa754148f4aa.png)

2.  Import

    > 使用 BACPAC 將資料庫 schema 及資料匯入

    * 資料庫資料夾右鍵 --> Import Data-tier Application...

        ![42import](https://cloud.githubusercontent.com/assets/3851540/25018478/eadf5798-20b9-11e7-9a3b-8675ae9c79c6.png)

    * Introduction

        ![43INTRO](https://cloud.githubusercontent.com/assets/3851540/25018433/e96db396-20b9-11e7-8a09-0d348c6d00e7.png)

    * Import Settings

        > 選擇 BACPAC 檔案位置

        ![44filelocation](https://cloud.githubusercontent.com/assets/3851540/25018430/e96c73be-20b9-11e7-8975-cd8bbd6759b4.png)

    * Database Settings

        > 設定 db 名稱及相關檔案儲放路徑

        ![45dbsetting](https://cloud.githubusercontent.com/assets/3851540/25018435/e9710190-20b9-11e7-857f-cebe749ed028.png)

    * Summary

        ![46summary](https://cloud.githubusercontent.com/assets/3851540/25018434/e96e2ee8-20b9-11e7-8894-97b8ec604436.png)

    * Results

        ![47inprogress](https://cloud.githubusercontent.com/assets/3851540/25018432/e96d40c8-20b9-11e7-932e-b5adafd63f1e.png)

        ![48success](https://cloud.githubusercontent.com/assets/3851540/25018431/e96ce9ac-20b9-11e7-8c52-c6fce60dab35.png)


# 參考資訊
1.  [Data-tier Applications](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications?WT.mc_id=DOP-MVP-5002594)
2.  [SQL Server 資料庫版本控管](https://channel9.msdn.com/Series/SQL-PASS-TAIWAN/SQL-Server-realase-management)
