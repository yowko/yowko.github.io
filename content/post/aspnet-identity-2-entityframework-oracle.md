---
title: "ASP.NET Identity 2 使用 EntityFramework 搭配 Oracle"
date: 2017-09-30T01:01:00+08:00
lastmod: 2018-09-26T01:01:23+08:00
draft: false
tags: ["ASP.NET Identity","EntityFramework","Oracle"]
slug: "aspnet-identity-2-entityframework-oracle"
aliases:
    - /2017/09/aspnet-identity-2-entityframework-oracle.html
---
# ASP.NET Identity 2 使用 EntityFramework 搭配 Oracle
相信開發 ASP.NET 比較久的朋友都曾經聽過甚至是用過 ASP.NET Membership - ASP.NET 2.0 時期的預設使用者權限管理系統，但也相信不少人對它深惡痛絕，不可否認以現在的觀點來看它確實有許多缺點，不過它的的確確曾經替不少工程師解決了問題，而在收集許多 ASP.NET Membership 開發者意見後打造出全新的使用者權限系統就是 - ASP.NET Identity

ASP.NET Identity 從 ASP.NET MVC 5 成為預設專案範本後陸陸續續用過幾次，只是都用的不深。最近剛好有個新專案，需求與 ASP.NET Identity 的幾個功能剛好搭得上，所以趁這個機會紀錄一下

首先因為公司主力 DB 使用 Oracle，所以就先紀錄如何讓 ASP.NET Identity 使用 EntityFramework Oracle 開始吧

## 建立 ASP.NET Identity 所需的資料庫物件

因為 ASP.NET Identity 預設使用 code first 模式，在實際存取 DB 時才直接建立相關資料庫 table,index,trigger，而這樣的方式大部份都會直接被 DBA 拒絕，為了與 production 一致，就從開發時期就使用 database first 來開發，因此我們必需先手動建立資料庫相關物件(建議分段執行，尤其是 `TRIGGER` 那段)

```sql
CREATE TABLE "AspNetRoles" ( 
  "Id" NVARCHAR2(128) NOT NULL,
  "Name" NVARCHAR2(256) NOT NULL,
  PRIMARY KEY ("Id")
);


CREATE TABLE "AspNetUserRoles" ( 
  "UserId" NVARCHAR2(128) NOT NULL,
  "RoleId" NVARCHAR2(128) NOT NULL,
  PRIMARY KEY ("UserId", "RoleId")
);


CREATE TABLE "AspNetUsers" ( 
  "Id" NVARCHAR2(128) NOT NULL,
  "Email" NVARCHAR2(256) NULL,
  "EmailConfirmed" NUMBER(1) NOT NULL,
  "PasswordHash" NVARCHAR2(256) NULL,
  "SecurityStamp" NVARCHAR2(256) NULL,
  "PhoneNumber" NVARCHAR2(256) NULL,
  "PhoneNumberConfirmed" NUMBER(1) NOT NULL,
  "TwoFactorEnabled" NUMBER(1) NOT NULL,
  "LockoutEndDateUtc" TIMESTAMP(7) NULL,
  "LockoutEnabled" NUMBER(1) NOT NULL,
  "AccessFailedCount" NUMBER(10) NOT NULL,
  "UserName" NVARCHAR2(256) NOT NULL,
  PRIMARY KEY ("Id")
);


CREATE TABLE "AspNetUserClaims" ( 
  "Id" NUMBER(10) NOT NULL,
  "UserId" NVARCHAR2(128) NOT NULL,
  "ClaimType" NVARCHAR2(256) NULL,
  "ClaimValue" NVARCHAR2(256) NULL,
  PRIMARY KEY ("Id")
);


CREATE SEQUENCE "AspNetUserClaims_SEQ";


CREATE OR REPLACE TRIGGER "AspNetUserClaims_INS_TRG"
  BEFORE INSERT ON "AspNetUserClaims"
  FOR EACH ROW
BEGIN
  SELECT "AspNetUserClaims_SEQ".NEXTVAL INTO :NEW."Id" FROM DUAL;
END;


CREATE TABLE "AspNetUserLogins" ( 
  "LoginProvider" NVARCHAR2(128) NOT NULL,
  "ProviderKey" NVARCHAR2(128) NOT NULL,
  "UserId" NVARCHAR2(128) NOT NULL,
  PRIMARY KEY ("LoginProvider", "ProviderKey", "UserId")
);


CREATE UNIQUE INDEX "RoleNameIndex" ON "AspNetRoles" ("Name");

CREATE INDEX "IX_AspNetUserRoles_UserId" ON "AspNetUserRoles" ("UserId");


CREATE INDEX "IX_AspNetUserRoles_RoleId" ON "AspNetUserRoles" ("RoleId");


CREATE UNIQUE INDEX "UserNameIndex" ON "AspNetUsers" ("UserName");


CREATE INDEX "IX_AspNetUserClaims_UserId" ON "AspNetUserClaims" ("UserId");


CREATE INDEX "IX_AspNetUserLogins_UserId" ON "AspNetUserLogins" ("UserId");


ALTER TABLE "AspNetUserRoles"
  ADD CONSTRAINT "FK_UserRoles_Roles" FOREIGN KEY ("RoleId") REFERENCES "AspNetRoles" ("Id")
  ON DELETE CASCADE;

ALTER TABLE "AspNetUserRoles"
  ADD CONSTRAINT "FK_UserRoles_Users" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id")
  ON DELETE CASCADE;

ALTER TABLE "AspNetUserClaims"
  ADD CONSTRAINT "FK_UserClaims_Users" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id")
  ON DELETE CASCADE;

ALTER TABLE "AspNetUserLogins"
  ADD CONSTRAINT "FK_UserLogins_Users" FOREIGN KEY ("UserId") REFERENCES "AspNetUsers" ("Id")
  ON DELETE CASCADE;
```

## 安裝 ASP.NET Identity

透過安裝 `Microsoft.AspNet.Identity.EntityFramework` 會將 `EntityFramework` 及 `Microsoft.AspNet.Identity.Core` 一併安裝進來

![1identityEF](https://user-images.githubusercontent.com/3851540/31026759-2eac5878-a57a-11e7-9ca9-659594e79cbb.png)

![2identity3](https://user-images.githubusercontent.com/3851540/31026760-2eae2e8c-a57a-11e7-8ea0-be7f71247a03.png)

## 安裝 Oracle provider

透過安裝 `Oracle.ManagedDataAccess.EntityFramework` 連帶安裝 `Oracle.ManagedDataAccess`

![3oracle1](https://user-images.githubusercontent.com/3851540/31026761-2eae7892-a57a-11e7-9816-afdc9561ee6d.png)

![4oracle2](https://user-images.githubusercontent.com/3851540/31026762-2eaf6392-a57a-11e7-9784-d3168b2751b1.png)

## 修改 web.config 的 ConnectionString

1.  Oracle server 的 ip 及帳密
2.  providerName 為 `Oracle.ManagedDataAccess.Client`


  *   範例如下

    ```xml
    <add name="DefaultConnection" providerName="Oracle.ManagedDataAccess.Client" connectionString="User Id=TestIdentity;Password=password;Data Source=localhost:1521/XE"/>
    ```

## 將 db table 與程式 class 綁定

修改 `ApplicationDbContext`，加入 override `OnModelCreating` 方法

    ```sql
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //這必需在第一行
        base.OnModelCreating(modelBuilder);
        //schema 名稱，如果建立時沒有刻意指定小寫，預設就是大寫
        modelBuilder.HasDefaultSchema("TESTIDENTITY"); 
        //以下依實際情境來調整 table name
        modelBuilder.Entity<ApplicationUser>().ToTable("AspNetUsers");
        modelBuilder.Entity<IdentityRole>().ToTable("AspNetRoles");
        modelBuilder.Entity<IdentityUserRole>().ToTable("AspNetUserRoles");
        modelBuilder.Entity<IdentityUserClaim>().ToTable("AspNetUserClaims");
        modelBuilder.Entity<IdentityUserLogin>().ToTable("AspNetUserLogins");
    }
    ```

*   如果出現 `ORA-01918: user 'xx' does not exist` 請先檢查 HasDefaultSchema 設定
    
    ![5ora-01918](https://user-images.githubusercontent.com/3851540/31026763-2eb4ad84-a57a-11e7-917e-13c71fc4db82.png)

*   完整程式碼

    ```cs
    public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
    {
        public ApplicationDbContext()
            : base("DefaultConnection", throwIfV1Schema: false)
        {
        }
                    public static ApplicationDbContext Create()
        {
            return new ApplicationDbContext();
        }
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            //這必需在第一行
            base.OnModelCreating(modelBuilder);
            //schema 名稱，如果建立時沒有刻意指定小寫，預設就是大寫
            modelBuilder.HasDefaultSchema("TESTIDENTITY"); 
            //以下依實際情境來調整 table name
            modelBuilder.Entity<ApplicationUser>().ToTable("AspNetUsers");
            modelBuilder.Entity<IdentityRole>().ToTable("AspNetRoles");
            modelBuilder.Entity<IdentityUserRole>().ToTable("AspNetUserRoles");
            modelBuilder.Entity<IdentityUserClaim>().ToTable("AspNetUserClaims");
            modelBuilder.Entity<IdentityUserLogin>().ToTable("AspNetUserLogins");
        }
    }
    ```

## 設定完成

1.  直接 Register

    ![6register](https://user-images.githubusercontent.com/3851540/31026764-2ebe6720-a57a-11e7-9593-59c31a856e63.png)

2.  順利登入也成功將資料寫至 Oracle

    ![6result](https://user-images.githubusercontent.com/3851540/31026765-2ed73f7a-a57a-11e7-8f30-3472f481d4fe.png)

## 心得

之前工作開發主要搭配的 database 都是 MS-SQL，使用的習慣跟工具的熟悉度都很高，加上與微軟其他開發工具充份整合，讓開發可以 focus 在核心功能的打造，從 MS-SQL 改用 Oracle 難免還是比較不熟悉，很多工具、語法、設定都要重新適應，有時候卡了很久才發現是一個小小的動作，的確會讓人比較沮喪些，但終究是搞定了第一步，接著才是更大的挑戰呀

# 參考資訊

1.  [ASP.NET MVC5 - Keeping Users in Oracle Database](https://stackoverflow.com/questions/28878718/asp-net-mvc5-keeping-users-in-oracle-database)
2.  [Introduction to ASP.NET Identity](https://docs.microsoft.com/en-us/aspnet/identity/overview/getting-started/introduction-to-aspnet-identity)
3.  [Oracle 用戶、對象權限、系統權限](http://fatlinuxguy.blogspot.tw/2012/01/oracle.html)
4.  [在 Windows 下安裝 Oracle 11g XE (Express Edition)](https://www.oschina.net/question/12_27650)
