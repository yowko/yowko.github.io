---
title: "使用 XSD 為 XML 客製 Intellisense 輸入選單"
date: 2017-07-02T22:53:00+08:00
lastmod: 2019-07-23T22:53:29+08:00
draft: false
tags: ["Visual Studio","XML"]
slug: "xml-intellisense-xsd"
aliases:
    - /2017/07/xml-intellisense-xsd.html
---
# 使用 XSD 為 XML 客製 Intellisense 輸入選單
同事負責的專案中有個自訂的 XML，主要用來紀錄一些不同 partner 的設定資訊。因為 partner 很多，常有新增或是調整這個 XML 的需求，為了避免人為輸入資料錯誤，想要在編輯 XML 時加上 intellisense 清單選擇提示

印象中我好像沒有這麼做過，但直覺上應該可行，畢竟 Visual Studio 編輯 web.config 也有類似功能，就來看看可以怎麼做吧

## 自訂 XML 內容

```xml
<?xml version="1.0" encoding="utf-8" ?>
<YowkoList>
  <YowkoItem YowkoCloth="1">
    <YowkoType>Paints</YowkoType>
  </YowkoItem>
</YowkoList>
```

## 加入 XSD

1.  XSD 可以從外部匯入也可以直接新增在專案中(視是否要與外部共用)
2.  以新增至專案為例：新增位置 按右鍵 --> Add --> New Item...

    ![1addxsd](https://user-images.githubusercontent.com/3851540/27770803-f6d519be-5f77-11e7-8cc6-c5ce5668d175.png)

3.  XML Schema --> 給個名字

    ![2xsdname](https://user-images.githubusercontent.com/3851540/27770804-f6f6319e-5f77-11e7-84e0-52943cc0eec6.png)

4.  預設畫面

    ![3designer](https://user-images.githubusercontent.com/3851540/27770805-f6fc6f32-5f77-11e7-968e-c7de76d5cde4.png)

## 編輯 XSD

1.  使用 XML Editor 來編輯

    ![4xmleditor](https://user-images.githubusercontent.com/3851540/27770806-f71d2650-5f77-11e7-9c01-e93cd488302b.png)

2.  預設 XSD 內容

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema id="Test"
        targetNamespace="http://tempuri.org/Test.xsd"
        elementFormDefault="qualified"
        xmlns="http://tempuri.org/Test.xsd"
        xmlns:mstns="http://tempuri.org/Test.xsd"
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
    >
    </xs:schema>
    ```

3.  將 `id`、`targetNamespace`、`xmlns`、`xmlns:mstns` 修改易於識別的內容

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema id="YowkoSchema"
        targetNamespace="http://yowko.com/YowkoSchema.xsd"
        elementFormDefault="qualified"
        xmlns="http://yowko.com/YowkoSchema.xsd"
        xmlns:mstns="http://yowko.com/YowkoSchema.xsd"
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
    >
    </xs:schema>
    ```

    *   `id` 實測下不影響功能
    *   `targetNamespace` 則需與 `xmlns:xs` 內容需一致

4.  加上 XSD 定義
    *   定義 intellisense 選單

        > 清單叫為 `YowkoCloth`，是 integer 格式

        ```xml
        <xs:simpleType name="YowkoCloth">
            <xs:restriction base="xs:integer">
            <xs:enumeration value="1"></xs:enumeration>
            <xs:enumeration value="5"></xs:enumeration>
            <xs:enumeration value="10"></xs:enumeration>
            <xs:enumeration value="11"></xs:enumeration>
            </xs:restriction>
        </xs:simpleType>
        ```

    *   定義 intellisense 選單使用方式

        > 定義為 `YowkoElement` 中的 `YowkoCloth` attribute

        ```xml
        <xs:complexType name="YowkoElement">
            <xs:attribute name="YowkoCloth" type="YowkoSize" use="required"></xs:attribute>
        </xs:complexType>
        ```

    *   定義 YowkoElement 使用方式

        > 根物件 `YowkoList` 有個類型為 `YowkoElement` 的 `YowkoItem` element

        ```xml
        <xs:element name="YowkoList">
            <xs:complexType>
            <xs:sequence>
                <xs:element name="YowkoItem" type="YowkoElement" minOccurs="0" maxOccurs="unbounded">
                </xs:element>
            </xs:sequence>
            </xs:complexType>
        </xs:element>
        ```

    *   完整 XSD

        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <xs:schema id="YowkoSchema"
                targetNamespace="http://yowko.com/YowkoSchema.xsd"
                elementFormDefault="qualified"
                xmlns="http://yowko.com/YowkoSchema.xsd"
                xmlns:mstns="http://yowko.com/YowkoSchema.xsd"
                xmlns:xs="http://www.w3.org/2001/XMLSchema"
        >
            <xs:simpleType name="YowkoSize">
                <xs:restriction base="xs:integer">
                <xs:enumeration value="1"></xs:enumeration>
                <xs:enumeration value="5"></xs:enumeration>
                <xs:enumeration value="10"></xs:enumeration>
                <xs:enumeration value="11"></xs:enumeration>
                </xs:restriction>
            </xs:simpleType>
                        <xs:complexType name="YowkoElement">
                <xs:sequence>
                <xs:element name="YowkoType" type="xs:string"></xs:element>
                </xs:sequence>
                <xs:attribute name="YowkoCloth" type="YowkoSize" use="required"></xs:attribute>
            </xs:complexType>
                        <xs:element name="YowkoList">
                <xs:complexType>
                <xs:sequence>
                    <xs:element name="YowkoItem" type="YowkoElement" minOccurs="0" maxOccurs="unbounded">
                    </xs:element>
                </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:schema>
        ```

## XML 加上使用自訂 XSD

1.  在根 element 加上 namesspace 引用

    ```xml
    xmlns="http://yowko.com/YowkoSchema.xsd"
    ```

2.  完整 XML

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <YowkoList xmlns="http://yowko.com/YowkoSchema.xsd">
        <YowkoItem YowkoCloth="1">
            <YowkoType>Paints</YowkoType>
        </YowkoItem>
    </YowkoList>
    ```

## 實際效果

1.  編輯提示(會出現提示選單供選擇)

    ![5intellisenseresult](https://user-images.githubusercontent.com/3851540/27770809-f71df9a4-5f77-11e7-8fa4-505e56d0758b.png)

2.  輸入 `<` 也會有 element 提示

    ![8element](https://user-images.githubusercontent.com/3851540/27770807-f71d829e-5f77-11e7-983b-eff1b535e1a7.png)

3.  輸入未在清單之值會出現錯誤提示

    ![6xsderror](https://user-images.githubusercontent.com/3851540/27770808-f71dccc2-5f77-11e7-98f6-dd37a444e9f8.png)

4.  輸入未在清單之值也會出現在 build warning 中

    ![7xsdwarning](https://user-images.githubusercontent.com/3851540/27770810-f71eafde-5f77-11e7-81cf-996084a4e815.png)

## 心得

實際流程不多，但網路上資源並不好找，我猜想可能對這個功能的需求不多，畢竟編輯 XML 的大多數是工程師，XML 格式是固定的，與其花這個時間做 XML intellisense 大家應該都會選擇把時間拿出開發其他功能吧

# 參考資訊

1.  [XML 編輯器 IntelliSense 功能](https://msdn.microsoft.com/zh-tw/library/ms255811.aspx)
2.  [Intellisense for custom XML in Visual Studio](http://blogs.lessthandot.com/index.php/desktopdev/mstech/vs2012/intellisense-for-custom-xml-in/)
