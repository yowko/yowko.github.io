---
title: "SQL Server 字串分割 function 的效能調校"
date: 2017-04-21T02:08:00+08:00
lastmod: 2018-09-17T10:35:44+08:00
draft: false
tags: ["SQL Server"]
slug: "sql-server-split-string-function"
aliases:
    - /2017/04/sql-server-split-string-function.html
---
# SQL Server 字串分割 function 的效能調校
過去曾經參加某個效能調校的研討會，相關資訊已經不可考，就連講師是誰都沒有印象，但卻對其提出的效能調校 guideline 有很深的體悟，他說效能調校的首要任務就是讓技術工具正確地各司其職，當下沒有什麼特別想法，隨著日後踏雷經驗增加，慢慢才悟到這句的奧妙與困難

普遍來說工程師都會特別擅長一至兩種程式語言，所以自然會用使用熟悉的技術來解決問題，雖然可以正確解決問題但卻不一定是最正確的方式，像是 db 就適合做資料儲存、AP 就適合用來做運算，情境簡單時，我們可以單獨使用 db 或 ap 來搞定;一旦情境複雜了或是資料量變龐大，原本單一技術的限制就會浮現，所以我們就該讓 db 專職在資料存取、ap 則專注於資料的運算

今天想分享的就是其中一個案例：將需要運算處理的資料交給 ap 運算後再傳給 db，讓 db 進行大批量資料異動，以減少 network io ，也讓 db 少去不擅長運算的工作，以提升效能，而 db 收到大批 id(key) 資料需要拆解成 db 可使用的格式，這就是 `字串分割 function` 的用途，在之前工作中曾經遇到效能瓶頸，最近在使用這個方式時，特別請同事 Colin 做了效能改善，因為這個技巧常用，所以做個筆記

## 使用 function return table

```sql
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE FUNCTION [dbo].[fn_SplitString] 
(
 @source varchar(max),
 @spiltor varchar(1)
)
RETURNS 
@result TABLE 
(
 Id int PRIMARY KEY
)
AS
BEGIN
 DECLARE @index int
SET @index = -1

WHILE (LEN(@source) > 0)
  BEGIN 
    SET @index = CHARINDEX(@spiltor , @source) 
    IF (@index = 0) AND (LEN(@source) > 0) 
      BEGIN  
        INSERT INTO @result VALUES (@source)
          BREAK 
      END 
    IF (@index > 1) 
      BEGIN  
        INSERT INTO @result VALUES (LEFT(@source, @index - 1))  
        SET @source = RIGHT(@source, (LEN(@source) - @index)) 
      END 
    ELSE
      SET @source = RIGHT(@source, (LEN(@source) - @index))
    END
  RETURN
END

GO
```
## 使用 cte

```sql
create function dbo.fn_SplitString_cte
(
 @source varchar(max),
 @spiltor varchar(1)
)
returns table
as
return
(
 with splitlist(startposition, endposition)
 as
 (
  select 
   0 as startposition, 
   charindex(@spiltor, @source) as endposition
  union all
  select 
   convert(int, endposition) + 1, 
   charindex (@spiltor, @source, endposition + 1) 
  from splitlist 
  where endposition > 0
 )
 select substring(@source, startposition, coalesce(nullif(endposition, 0), len(@source) + 1) - startposition) as Id
 from splitlist
)
```
## 效能比較

1. 開啟執行時間統計資訊

    > `SET STATISTICS TIME ON`

2.  執行 script

    *   function return table

        ```sql
        select * from [dbo].[fn_SplitString] ('0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20',',')
        ```
    *   cte

        ```sql
        select * from [dbo].[fn_SplitString_cte] ('0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20',',')
        option (maxrecursion 0)
        ```

        *   需要注意的是 cte 預設僅會回傳 100 筆資料，需加上 `option (maxrecursion 0)` 解除限制


          *   錯誤訊息
              - 訊息內容

                    ```
                    The statement terminated. The maximum recursion 100 has been exhausted before statement completion.
                    ```
              - 錯誤截圖
                
                ![1cteerror](https://cloud.githubusercontent.com/assets/3851540/25290331/2b1ed22e-2700-11e7-824f-e94d3915dcdf.png)

3.  分別實際執行

    資料筆數|	使用 function return table	|使用 CTE
    :---|:---|:---
    200	|16 ms|	0 ms
    2000	|203 ms|	100 ms
    20000	|24,750 ms|	13,031 ms
    50000	|151,203 ms|	112,312 ms



## 心得

切字串功能用過好幾次，因為資料量不夠大，沒有遇到效能瓶頸，感謝同事 Colin 指導，針對日後可能產生的效能問題預做準備
