---
title: "讓 EntityFramework 綁定自定 Enum 型別"
date: 2017-02-05T00:42:34+08:00
lastmod: 2018-09-11T00:42:34+08:00
draft: false
tags: ["C#","Entity Framework"]
slug: "entityframework-enum-binding-class"
aliases:
    - /2017/02/entityframework-enum-binding-class.html
---
# 讓 EntityFramework 綁定自定 Enum 型別
讓 EntityFramework 可以綁定自定 Enum 有兩個好處：

1. 不用再自行轉換 int 跟 Enum 
2. 透過 scaffolding 可以直接綁定 dropdownlist，這讓我們在開發上節省不少工作。

今天的筆記是針對 EntityFramework Database First 模式(因為 Code First 的類別本來就是自定的)，透過 EntityFramework 的設定，可以讓 Class 產出時直接綁定 Enum 型別，存取資料時也能自動完成轉換。

## 資料欄位定義
- WeekDay 為 `int`
	
    ![1table](https://cloud.githubusercontent.com/assets/3851540/22261604/8af273d2-e2a8-11e6-8f86-d5f1d6ff839c.png)

## 自訂 Enum

```cs
public enum WeekDaysEnum
{
    Default,
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday
}
```

## 未設定前
1. Class
	- int 型別
		
        ![2class](https://cloud.githubusercontent.com/assets/3851540/22261603/8af14f16-e2a8-11e6-8069-a077317ab5bb.png)

2. scaffolding
	- 開放式 input
		
        ![3form](https://cloud.githubusercontent.com/assets/3851540/22261608/8b0254aa-e2a8-11e6-848f-084af56466b2.png)

## EntityFramework 設定綁定 Enum
1. 開啟 .edmx
	
    ![4edmx](https://cloud.githubusercontent.com/assets/3851540/22261605/8af4a4ea-e2a8-11e6-871b-4c8757315b91.png)

2. 開啟 Model Browser
	- 1-1. 在 edmx diagram 隨處按右鍵
    - 1-2. `Show in Model Browser`
		
        ![5modelbrowser](https://cloud.githubusercontent.com/assets/3851540/22261606/8af73746-e2a8-11e6-8b31-a320790ef2da.png)

3. Model Browser 加入自訂 Enum
	
    ![6model](https://cloud.githubusercontent.com/assets/3851540/22261607/8afbfae2-e2a8-11e6-9205-d42cdab5116e.png)

    - 3-1. 新增自訂 Enum
        - Enum Types 右鍵
        - Add New Enum Type
        
            ![7addnew](https://cloud.githubusercontent.com/assets/3851540/22261596/8ac9b46a-e2a8-11e6-91de-e6192f8036f3.png)

    - 3-2. 新增方式有兩種
        - a. 直接輸入
            - 會新增 .cs 在 .edmx 所在 folder 下
            - 依輸入內容建立 enum
    		
                ![8inputenum](https://cloud.githubusercontent.com/assets/3851540/22261600/8acf1086-e2a8-11e6-8030-7a71894f2055.png)
        - b. 綁定既有 enum
            - 指定 enum 名稱
            - 輸入既有 enum 型別名稱(建議加上 namespace)
   			
                ![9existedenum](https://cloud.githubusercontent.com/assets/3851540/22261597/8aca9880-e2a8-11e6-8733-01f06232d87b.png)

    - 3-3. 欄位綁定 Enum 型別
        - 開啟欄位屬性視窗
            - 在欲綁定的欄位按 右鍵--> properties 或 F4
        - Type 選擇剛剛加入的 enum 型別
 		    
             ![10binding](https://cloud.githubusercontent.com/assets/3851540/22261602/8ad46bd0-e2a8-11e6-81a2-87a90045cfae.png)

    - 3-4. 記得存檔及編譯

## 完成結果
1. Class
	- 自定 enum  型別
		
        ![11calss](https://cloud.githubusercontent.com/assets/3851540/22261598/8acbed34-e2a8-11e6-88bf-184aea48789d.png)

2. scaffolding
	- 綁定 dropdownlist
		
        ![12dropdownlist](https://cloud.githubusercontent.com/assets/3851540/22261601/8ad0766a-e2a8-11e6-947b-f74547253e92.png)

## 心得
綁定後就可初步避免手誤填錯的問題(如果擔心有心人士搗蛋還是需要自行實作檢查機制)，搭配上 scaffolding 自動產生對應的 dropdownlist ，另外還有再也不用手動轉換，這些都讓開發效率可以大大提昇。

# 參考資訊