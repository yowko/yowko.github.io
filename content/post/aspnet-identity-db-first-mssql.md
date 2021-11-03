---
title: "ASP.NET Identity 搭配 DataBase first 與 SQL Server"
date: 2017-11-08T23:58:00+08:00
lastmod: 2021-11-03T23:58:20+08:00
draft: false
tags: ["ASP.NET Identity","SQL Server"]
slug: "aspnet-identity-db-first-mssql"
aliases:
    - /2017/11/aspnet-identity-db-first-mssql.html
---
## ASP.NET Identity 搭配 DataBase first 與 SQL Server

之前曾經輕描淡寫地介紹過 ASP.NET Identity 預設使用 Code First 機制。

在 MVC 預設專案範本中，會在 Regiser 時建立 ASP.NET Identity 的相關 db table 及 code first 需要用到的 __MigrationHistory table，曾經在多個專案中使用都未曾發生問題，相當穩定，只是在較具規模的團隊中，DBA 不會允許 code first 這樣的行為出現，畢竟可以執行 DDL 的帳號權限太大，一不小心就可能造成系統重大問題

還記得曾經有個專案因為啟用 code first，而團隊成員因為不熟悉設定，改了 model 造成 db table 整個 drop 重建，氣得客戶七竅生煙呀

回到主題，今天就來紀錄該如何在無法使用 code first 的情況下，一樣可以享用 ASP.Net Identity 的便利功能

## 建立 ASP.NET Identity 相關表格

* 預設使用的 table 名稱

    > 可以自行更換，程式那邊再指定 mapping 即可

  * AspNetRoles
  * AspNetUserClaims
  * AspNetUserLogins
  * AspNetUserRoles
  * AspNetUsers

* 上述 table 的 DDL

    > 以下將使用自訂 table 來示範

  * AspNetRoles --> CustomRole

        ```sql
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE TABLE [dbo].[CustomRoles](
        [Id] [nvarchar](128) NOT NULL,
        [Name] [nvarchar](256) NOT NULL,
        CONSTRAINT [PK_dbo.CustomRoles] PRIMARY KEY CLUSTERED 
        (
        [Id] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        ```

  * AspNetUserClaims --> CustomUserClaims

        ```sql
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE TABLE [dbo].[CustomUserClaims](
        [Id] [int] IDENTITY(1,1) NOT NULL,
        [UserId] [nvarchar](128) NOT NULL,
        [ClaimType] [nvarchar](max) NULL,
        [ClaimValue] [nvarchar](max) NULL,
        CONSTRAINT [PK_dbo.CustomUserClaims] PRIMARY KEY CLUSTERED 
        (
        [Id] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
        GO
        ```

  * AspNetUserLogins --> CustomUserLogins

        ```sql
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE TABLE [dbo].[CustomUserLogins](
        [LoginProvider] [nvarchar](128) NOT NULL,
        [ProviderKey] [nvarchar](128) NOT NULL,
        [UserId] [nvarchar](128) NOT NULL,
        CONSTRAINT [PK_dbo.CustomUserLogins] PRIMARY KEY CLUSTERED 
        (
        [LoginProvider] ASC,
        [ProviderKey] ASC,
        [UserId] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        ```

  * AspNetUserRoles --> CustomUserRoles

        ```sql
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE TABLE [dbo].[CustomUserRoles](
        [UserId] [nvarchar](128) NOT NULL,
        [RoleId] [nvarchar](128) NOT NULL,
        CONSTRAINT [PK_dbo.CustomUserRoles] PRIMARY KEY CLUSTERED 
        (
        [UserId] ASC,
        [RoleId] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY]
        GO
        ```

  * AspNetUsers --> CustomUsers

        ```sql
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE TABLE [dbo].[CustomUsers](
        [Id] [nvarchar](128) NOT NULL,
        [Email] [nvarchar](256) NULL,
        [EmailConfirmed] [bit] NOT NULL,
        [PasswordHash] [nvarchar](max) NULL,
        [SecurityStamp] [nvarchar](max) NULL,
        [PhoneNumber] [nvarchar](max) NULL,
        [PhoneNumberConfirmed] [bit] NOT NULL,
        [TwoFactorEnabled] [bit] NOT NULL,
        [LockoutEndDateUtc] [datetime] NULL,
        [LockoutEnabled] [bit] NOT NULL,
        [AccessFailedCount] [int] NOT NULL,
        [UserName] [nvarchar](256) NOT NULL,
        CONSTRAINT [PK_dbo.CustomUsers] PRIMARY KEY CLUSTERED 
        (
        [Id] ASC
        )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
        ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
        GO
        ```

  * 其他相關的 constraint

        ```
        ALTER TABLE [dbo].[CustomUserClaims]  WITH CHECK ADD  CONSTRAINT [FK_dbo.CustomUserClaims_dbo.CustomUsers_UserId] FOREIGN KEY([UserId])
        REFERENCES [dbo].[CustomUsers] ([Id])
        ON DELETE CASCADE
        GO
        ALTER TABLE [dbo].[CustomUserClaims] CHECK CONSTRAINT [FK_dbo.CustomUserClaims_dbo.CustomUsers_UserId]
        GO
        ALTER TABLE [dbo].[CustomUserLogins]  WITH CHECK ADD  CONSTRAINT [FK_dbo.CustomUserLogins_dbo.CustomUsers_UserId] FOREIGN KEY([UserId])
        REFERENCES [dbo].[CustomUsers] ([Id])
        ON DELETE CASCADE
        GO
        ALTER TABLE [dbo].[CustomUserLogins] CHECK CONSTRAINT [FK_dbo.CustomUserLogins_dbo.CustomUsers_UserId]
        GO
        ALTER TABLE [dbo].[CustomUserRoles]  WITH CHECK ADD  CONSTRAINT [FK_dbo.CustomUserRoles_dbo.CustomRoles_RoleId] FOREIGN KEY([RoleId])
        REFERENCES [dbo].[CustomRoles] ([Id])
        ON DELETE CASCADE
        GO
        ALTER TABLE [dbo].[CustomUserRoles] CHECK CONSTRAINT [FK_dbo.CustomUserRoles_dbo.CustomRoles_RoleId]
        GO
        ALTER TABLE [dbo].[CustomUserRoles]  WITH CHECK ADD  CONSTRAINT [FK_dbo.CustomUserRoles_dbo.CustomUsers_UserId] FOREIGN KEY([UserId])
        REFERENCES [dbo].[CustomUsers] ([Id])
        ON DELETE CASCADE
        GO
        ALTER TABLE [dbo].[CustomUserRoles] CHECK CONSTRAINT [FK_dbo.CustomUserRoles_dbo.CustomUsers_UserId]
        GO
        ```

## ASP.NET Identity 設定

1. 確認專案已正確加入 ASP.NET Identity

    * 使用 Empty MVC 專案範本

        > 之前筆記 [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/add-aspnet-identity-empty-project) 有介紹該如何從無至有加入 ASP.NET Identity

    * 使用預設 MVC 專案範本

        * 建立專案時選 MVC 專案範本

            ![1defaulttemplate](https://user-images.githubusercontent.com/3851540/32558085-ef178384-c4de-11e7-8b59-2799ca4cc943.png)

        * Authentication 選用 `Indivisual User Accounts`

            ![2identity](https://user-images.githubusercontent.com/3851540/32558088-ef84bd96-c4de-11e7-9de7-20682d098b8d.png)

2. 設定 table mapping

    > 前面已經將預設 `AspNet` 開頭的 table 名稱換成 `Custom` 開頭，所以需要自行指定 mapping，不然還是會執行 code first 建立 table

    * 修改 `Models` 中 `IdentityModels.cs` 檔案的 `ApplicationDbContext`

        > 如果不是使用預設檔案結構，請直接修改 `ApplicationDbContext` 內容

    * override `OnModelCreating` 方法

        ```cs
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            //這必需在第一行
            base.OnModelCreating(modelBuilder);
            //以下依實際情境來調整 table name
            modelBuilder.Entity<ApplicationUser>().ToTable("CustomUsers");
            modelBuilder.Entity<IdentityRole>().ToTable("CustomRoles");
            modelBuilder.Entity<IdentityUserRole>().ToTable("CustomUserRoles");
            modelBuilder.Entity<IdentityUserClaim>().ToTable("CustomUserClaims");
            modelBuilder.Entity<IdentityUserLogin>().ToTable("CustomUserLogins");
        }
        ```

    * `ApplicationDbContext` 完整程式碼

        ```cs
        public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
        {
            public ApplicationDbContext() : base("DefaultConnection", throwIfV1Schema: false)
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
                //以下依實際情境來調整 table name
                modelBuilder.Entity<ApplicationUser>().ToTable("CustomUsers");
                modelBuilder.Entity<IdentityRole>().ToTable("CustomRoles");
                modelBuilder.Entity<IdentityUserRole>().ToTable("CustomUserRoles");
                modelBuilder.Entity<IdentityUserClaim>().ToTable("CustomUserClaims");
                modelBuilder.Entity<IdentityUserLogin>().ToTable("CustomUserLogins");
            }
        }
        ```

3. 調整 Connection String

    * 預設連線為 localdb 且直接使用 attach 檔案方式掛載資料庫

        ```xml
        <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-Identity-20171108101321.mdf;Initial Catalog=aspnet-Identity-20171108101321;Integrated Security=True" providerName="System.Data.SqlClient" />
        ```

    * 調整為已建立 Identity 相關 table 的 db

        ```xml
        <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;Initial Catalog=IdentityDBFirst;Integrated Security=True" providerName="System.Data.SqlClient" />
        ```

4. 重新啟動網站並執行 Register

    > 可以發現並沒有透過 code first 建立 table，並已正確將資料寫入設定的 table 中

    ![3success](https://user-images.githubusercontent.com/3851540/32558089-efba0924-c4de-11e7-9f20-700a1acab2f2.png)

## 新增自訂欄位做法

> 以下使用 user 示範

1. 先在 db 中建立欄位

2. 在 ApplicationUser class 中加入 property

    ```cs
    public class ApplicationUser : IdentityUser
    {
        public async Task<ClaimsIdentity> GenerateUserIdentityAsync(UserManager<ApplicationUser> manager)
        {
            // Note the authenticationType must match the one defined in CookieAuthenticationOptions.AuthenticationType
            var userIdentity = await manager.CreateIdentityAsync(this, DefaultAuthenticationTypes.ApplicationCookie);
            // Add custom user claims here
            return userIdentity;
        }
        
        public string Test { get; set; }
    }
    ```

* 只在 db table 加欄位不會出錯，只是拿不到資料
* 只在 class 中加 property，則會出現 table column 找不到的問題

    ![4invalidcolumn](https://user-images.githubusercontent.com/3851540/32558091-efebf312-c4de-11e7-86c4-9995da757e2a.png)

## 心得

從 ASP.NET Identiy 問世以來，一直覺得是滿棒的功能，雖然無法滿足各式各樣千變萬化的需求，但比起前一代的 merbership 已經是天壤之別，不過我個人覺得門檻還是稍高，有不少東西都被封裝起來，沒有去翻 source code 根本就不知道背後做了什麼，再來像是 role 的管理預設被拿掉或是 userclaim table 的用途....etc 諸如此類的細節，在文件的介紹上相對比較不足的，是稍稍美中不足的部份

## 參考資訊

1. [Using Asp.Net Identity DataBase first approach](https://stackoverflow.com/questions/20668328/using-asp-net-identity-database-first-approach)
2. [將 ASP.NET Identity 加至 ASP.NET MVC Empty 專案中](/add-aspnet-identity-empty-project)
