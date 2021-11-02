---
title: "讓 NLog 發送 Mail "
date: 2018-07-17T23:03:00+08:00
lastmod: 2021-11-02T23:03:31+08:00
draft: false
tags: ["套件","NLog"]
slug: "nlog-mail"
aliases:
    - /2018/07/nlog-mail.html
---
## 讓 NLog 發送 Mail

曾經在之前筆記 [讓 log4net 收到指定錯誤 Level 發送 mail](/log4net-mail) 紀錄到透過 log4net 的 appender 設定讓出現指定 log Level 時自動發送 mail，當時就覺得 NLog 在設定的便利性上與文件的可讀性上有不少優勢

最近手邊另個專案也希望在程式補捉到 Error 或是 Fatal 等級的訊息時發送 mail，不同的是這個專案採用 NLog，過程中發現部份設定記憶日益模糊，趁著專案需要順手整理一下

## 基本語法

```xml
<targets>
  <target xsi:type="Mail"
          name="String"
          header="Layout"
          footer="Layout"
          layout="Layout"
          html="Boolean"
          addNewLines="Boolean"
          replaceNewlineWithBrTagInHtml="Boolean"
          encoding="Encoding"
          subject="Layout"
          to="Layout"
          bcc="Layout"
          cc="Layout"
          from="Layout"
          body="Layout"
          smtpUserName="Layout"
          enableSsl="Boolean"
          secureSocketOption="None|Auto|SslOnConnect|StartTls|StartTlsWhenAvailable"*
          smtpPassword="Layout"
          smtpAuthentication="Enum"
          smtpServer="Layout"
          smtpPort="Integer"
          useSystemNetMailSettings="Boolean"
          deliveryMethod="Enum"
          pickupDirectoryLocation="String"
          timeout="Integer"
          skipCertificateValidation="Boolean"
 />
</targets>
```

## 屬性說明

> 以下 `Layout` 格式，代表可以使用內建語法指令(e.g. `${message}`, `${level}` etc)，詳細說明請參考 [Layout Renderers](https://github.com/NLog/NLog/wiki/Layout-Renderers)

- xsi:type="Mail"

    > 指定 target 類型為 `Mail`

  - 接受的類型清單可以參考 [Targets](https://github.com/NLog/NLog/wiki/Targets)
        > 常見類型有：`File`,`Database`,`Console`,`Mail`
- name

    > target 的識別用名稱，用來讓 logger 指定輸出使用

  - 接受 `Layout` 格式
- header

    > email 的 header

  - 接受 `Layout` 格式
- footer

    > email 的 footer

  - 接受 `Layout` 格式
- layout

    > 輸出訊息內容

  - 接受 `Layout` 格式
  - 預設值：`${message}${newline}`
  - 與 `body` 用途相同，後者設定會覆蓋前者
- html

    > email 內容使用 HTML 或是純文字

  - 接受 `Boolean` 格式
  - 預設值：`false`
- addNewLines

    > 是否在每筆 log 間插入換行符號

  - 接受 `Boolean` 格式
  - 預設值：`false`
- replaceNewlineWithBrTagInHtml

    > 是否在 body 中將 `NewLine 符號` 轉換為 `<br/>`

  - 接受 `Boolean` 格式
  - 預設值：`false`
- encoding

    > 指定 email 的編碼格式，詳細清單請參考 [Encoding Class](https://docs.microsoft.com/he-il/dotnet/api/system.text.encoding?view=netframework-4.7&?WT.mc_id=DOP-MVP-5002594#Remarks), 使用 Encoding 的 name

  - 接受 `Encoding` 格式
  - 預設值：`UTF-8`
- subject

    > email 的標題

  - 接受 `Layout` 格式
  - 預設值：`Message from NLog on ${machinename}`
- to

    > 收件者，多個收件者以 `,` 分隔

  - 接受 `Layout` 格式
  - NLog 4.0 開始如果有指定 `bcc` 或是 `cc` 則非必填
- bcc

    > 密件副本收件者，多個收件者以 `,` 分隔

  - 接受 `Layout` 格式
- cc

    > 副本收件者，多個收件者以 `,` 分隔

  - 接受 `Layout` 格式
- from

    > 寄件者

  - 接受 `Layout` 格式
- body

    > 輸出訊息內容

  - 接受 `Layout` 格式
  - 預設值：`${message}${newline}`
  - 與 `layout` 用途相同，後者設定會覆蓋前者
- smtpUserName

    > 帳號

  - 接受 `Layout` 格式
  - `SmtpAuthentication` 設為 `basic` 時使用
- enableSsl

    > 是否啟用 `SSL`

  - 接受 `Boolean` 格式
  - 預設值：`false`
  - `port 465` 無法使用 SSL
- secureSocketOption="None|Auto|SslOnConnect|StartTls|StartTlsWhenAvailable"
  - 只有在 `NLog.Mailkit 2.1` 以上版本可指定 `SSL` or `TLS`
  - 預設值：`StartTlsWhenAvailable`
  - `enableSsl` 設為 `true` 時會值是 `SslOnConnect`
- smtpPassword

    > 密碼

  - 接受 `Layout` 格式
  - `SmtpAuthentication` 設為 `basic` 時使用
- smtpAuthentication="None|Basic|Ntlm"

    > SMTP 驗證模式

  - Basic - 使用 `username` 與 `password`
  - None - 沒有驗證
  - Ntlm - 使用 NT LAN Manager (NTLM)驗證通訊協定
- smtpServer

    > SMTP server

  - 接受 `Layout` 格式
- smtpPort

    > SMTP port

  - 接受 `Integer` 格式\
  - 預設值：`25`
  - `port 465` 無法在啟用 SSL 下使用
- useSystemNetMailSettings

    > 強制使用 system.net/mailSettings

    ```xml
    <system.net>
        <mailSettings>
            <smtp from="mail@domain.com" deliveryMethod="SpecifiedPickupDirectory">
            <network host="localhost" port="25"/>
            <specifiedPickupDirectory pickupDirectoryLocation="C:/Temp/Email"/>
            </smtp>
        </mailSettings>
    </system.net>
    ```

  - 接受 `Boolean` 格式
  - 預設值：`false`  
- deliveryMethod="Network|PickupDirectoryFromIis|SpecifiedPickupDirectory"

    > 指定 email 的發送處理方式

  - 我沒用過，文件上名稱不一致：`smtpDeliveryMethod`
  - 允許值：
    - Network - Email 透過 network 傳送至 smtp
    - PickupDirectoryFromIis - Email 複製至本機 IIS 用的 pickup 資料夾供傳送.
    - SpecifiedPickupDirectory - Email 複製至 `PickupDirectoryLocation` 指定的資料夾供外部應用程式傳送
  - `NLog 4.2` 開始導入
  - 預設值：`Network`
- pickupDirectoryLocation

    > 設定待處理 mail 的儲存位置

  - 接受 `String` 格式
  - `NLog 4.2` 開始導入
- timeout

    > 設定 SMTP 逾時放棄時間

  - 接受 `Integer` 格式
  - 預設值：`10000` (10 秒)
- skipCertificateValidation

    > 是否忽略 SSL 證書檢查，專屬 `NLog.MailKit` 的設定

  - 接受 `Boolean` 格式
  - `NLog.MailKit 1.1` 開始導入

## 使用範例

1. nlog.config

    ```xml
    <configuration>
        <configSections>
            <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
        </configSections>
        <nlog>
            <targets async="true">
                <target type="Mail" name="email" header="HEADER ${newline}" footer="${newline} FOOTER" html="true" addNewLines="true" replaceNewlineWithBrTagInHtml="true" subject="${var:subject}" to="yowko@yowko.com" bcc="yowko@yowko.com" cc="yowko@yowko.com" from="yowko@yowko.com" smtpServer="127.0.0.1" smtpPort="25" />
            </targets>
            <rules>
                <logger name="*" minlevel="Error" writeTo="email" />
            </rules>
        </nlog>
    </configuration>
    ```

2. 實際使用

    ```cs
    var logger = LogManager.GetCurrentClassLogger();
    LogManager.Configuration.Variables["subject"] = "Test Error occur";
    logger.Error("Yowko Test mail");
    ```

3. 實際效果

    ![1result](https://user-images.githubusercontent.com/3851540/42806484-26b9e2ce-89e1-11e8-8b80-85a6dc3a0dab.png)

## 心得

NLog 在使用上不僅方便，文件又清楚，雖然有個屬性 (deliveryMethod|smtpDeliveryMethod) 在不同位置有不同的名稱，但瑕不掩瑜呀  整理及測試的過程都沒遇到什麼困難，明顯與 log4net 差很多，還是 NLog 對我的胃口呀

## 參考資訊

1. [讓 log4net 收到指定錯誤 Level 發送 mail](/log4net-mail)
2. [Mail target](https://github.com/NLog/NLog/wiki/Mail-Target)
3. [Layout Renderers](https://github.com/NLog/NLog/wiki/Layout-Renderers)
4. [Targets](https://github.com/NLog/NLog/wiki/Targets)
