---
title: "Kibana 在 Visualize table buckets 中從 field 取部份值做為 row"
date: 2021-11-01T00:39:29+08:00
lastmod: 2021-11-01T00:39:29+08:00
draft: false
tags: ["Kibana","ELK"]
slug: "kibana-visualize-table-buckets-substring-field"
---

## Kibana 在 Visualize table buckets 中從 field 取部份值做為 row

為了後續 support 需要，想要在原本的 dashboard 中加上一個 visualize table 用來提供更詳細的資訊，原本也不是太困難的事，只是這次需要用來 group by 的資料放在類似 message 這種未切分的欄位中

## 基本環境說明

1. macOS Big Sur 11.6
2. docker desktop 3.6.0(67351)
3. docker images
    - elasticsearch:7.11.2
    - yowko/fluentd-elasticsearch:1.0.0
    - fluent/fluentd:v1.12.0-debian-1.0
    - kibana:7.11.2
4. sample log

    ```log
    2021-11-01 09:00:38.192 [172][ERR][Yowko.DomainService.Test.Application.Utilities.ExceptionInterceptor]UserId:TW001|Name:Yowko|DepartmentId:D01
    traceId:b9e680d30a025e7a
    ```

## 設定方式

1. 新增 Visualize (Data table)

    ![1visualize](https://user-images.githubusercontent.com/3851540/139638627-88412d5c-7ffc-4c24-90ca-f4b6b6593f98.png)

    ![2createvisualize](https://user-images.githubusercontent.com/3851540/139638636-7e8a0c3b-cf4e-497a-97d9-e698b7c51e24.png)

    ![3aggrgation](https://user-images.githubusercontent.com/3851540/139638639-670dd6cb-4537-43e4-b221-714a1c9df596.png)

    ![4datatable](https://user-images.githubusercontent.com/3851540/139638644-d6448881-4af1-42cd-9d83-23f31eac7836.png)

    ![5datasource](https://user-images.githubusercontent.com/3851540/139638646-5f2fb3b3-616c-4cb7-b358-636a01b01fa1.png)

2. 以 `DepartmentId` 為 row 切分基準

    - `Spilt rows`

        ![6spiltrows](https://user-images.githubusercontent.com/3851540/139638647-fec0a812-6ed9-4375-ad01-e3b4891f8108.png)

    - `Aggregation` : `Terms`

        ![7terms](https://user-images.githubusercontent.com/3851540/139638652-ea91776b-2f1a-454d-9965-45f3784c1d3a.png)

    - `Field` : `message.keyword`

        ![8field](https://user-images.githubusercontent.com/3851540/139638654-e997994a-ef13-4fb7-9cab-3ebabcee8cb7.png)

    - `Advanced` --> `JSON Input`

        ![9advanced](https://user-images.githubusercontent.com/3851540/139638658-5710d52c-c9e4-41d3-a8ad-1d57a007dc74.png)

        ```json
        {
            "script": "_value.substring(_value.lastIndexOf('|')).replace('|DepartmentId:','')"
        }
        ```

        > JSON Input 中的 `_value` 也可以使用 `doc['message.keyword'].value`

3. 以 `UserId` 為 row 切分基準

    > 重複上述步驟，僅最後 `JSON Input` 改為

    ```json
    {
        "script": "_value.substring(0, _value.indexOf('|')).replace('UserId:','')"
    }
    ```

## 心得

google 的過程中一直找到 `Painless script`、`scripted field` 但似乎都是在 index level 直接多個欄位，但我並非所有的 record 都需要切欄位，最後還是請教同事搞定的

另外我在開發時有遇到 `Error executing runtime field or scripted field on index pattern` 跟 `Error executing Painless script` 後來查到是因為並非所有 record 都有對應的資料內容造成 parse fail

- 實際效果

    ![10result](https://user-images.githubusercontent.com/3851540/139638934-c51f099a-29df-4515-945a-75d8a34baee9.png)

## 參考資訊

1. [Elasticsearch failed to execute script](https://stackoverflow.com/a/66212210)
