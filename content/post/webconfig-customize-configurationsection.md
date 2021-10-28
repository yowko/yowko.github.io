---
title: "使用 ConfigurationSection 自訂 ASP.NET config (web.config) 區段"
date: 2017-02-19T01:42:34+08:00
lastmod: 2021-10-08T00:42:34+08:00
draft: false
tags: ["csharp","web.config"]
slug: "webconfig-customize-configurationsection"
aliases:
    - /2017/02/use-configurationsection-customize-aspnet-config.html
---
## 使用 ConfigurationSection 自訂 ASP.NET config (web.config) 區段

不得不服老呀，一樣的功能二、三年前寫的時候還相當流暢，想不到這二、三年的光景過去，就搞得像是沒寫過一樣XD 乾脆當做沒寫過，好好地留個紀錄 避免下次遇到又要重來...

透過 ConfigurationSection 的做法可以讓我們使用 .NET Framework 內建的 ConfigurationManager api 來讀取 config 檔案內容，而不用自行處理檔案讀寫與解析，但也因為要使用 .NET Framewrok 功能，開發及使用上就需要符合相關規範，接著就一步步來看看開發上有什麼需要留意的眉角

## 1. 自訂解析的 handler

- 1-1. 新增 `public` class 並繼承 `System.Configuration.ConfigurationSection`

    ```cs
    public class YowkoSettingConfiguration : ConfigurationSection
    {
    }
    ```

- 1-2. 定義需要的屬性(property)

    > 關於 ConfigurationProperty 設定請參考 [ConfigurationPropertyAttribute 類別](https://msdn.microsoft.com/zh-tw/library/system.configuration.configurationpropertyattribute.aspx)

  - 基本型別的 property 用於表示 attibute

      ![1property](https://cloud.githubusercontent.com/assets/3851540/23091842/582df16e-f5fa-11e6-86d9-ba5018f2867d.png)

    - 套用 `ConfigurationProperty` 指定對應 config 檔內容的屬性名稱

    - `getter` ：用 xml 的屬性名稱 return

        ```cs
        get
        {
            return this["{propertyName}"] as string;
        }
        ```

    - `setter` ：用 xml 的屬性名稱設定值

        ```cs
        set
        {
            this["{propertyName}"] = value;
        }
        ```

  - 自訂型別的 perperty 用於表示 child element

      ![2element](https://cloud.githubusercontent.com/assets/3851540/23091905/320e13f0-f5fb-11e6-856d-a83341cc77b6.png)
    - 套用 `ConfigurationProperty` 指定對應 config 檔內容的元名稱
    - 自訂型別 `需` 繼承 `ConfigurationElement`
    - 只有 `getter`
    - 元素中的屬性規則 follow 基本型別的 property 的規則

  - 範例 1 及 範例 2 可用的 handler 完整程式碼

      ```cs
      public class YowkoConfiguration : ConfigurationSection
      {
          [ConfigurationProperty("name")]
          public string Attr_Name
          {
              get
              {
                  return this["name"] as string;
              }
              set
              {
                  this["name"] = value;
              }
          }
  
          [ConfigurationProperty("tel")]
          public string Attr_Tel
          {
              get
              {
                  return this["tel"] as string;
              }
              set
              {
                  this["tel"] = value;
              }
          }
  
          [ConfigurationProperty("yowkoSetting")]
          public YowkoSettingElement Elem_YowkoSetting => this["yowkoSetting"] as YowkoSettingElement;
      }
      public class YowkoSettingElement : ConfigurationElement
      {
          [ConfigurationProperty("site", DefaultValue = "", IsRequired = true)]
          public string Site
          {
              get
              {
                  return this["site"] as string;
              }
              set
              {
                  this["site"] = value;
              }
          }
  
          [ConfigurationProperty("userName", IsRequired = true)]
          public string UserName
          {
              get
              {
                  return this["userName"] as string;
              }
              set
              {
                  this["userName"] = value;
              }
  
          }
  
          [ConfigurationProperty("password", IsRequired = true)]
          public string Password
          {
              get
              {
                  return this["password"] as string;
              }
              set
              {
                  this["password"] = value;
              }
          }
      }
      ```

## 2. web.config 中 configSections 加入自訂 section 宣告

- MSDN 建議使用 `sectionGroup` 把將 section 內容包起來，使資料結構性更好
- 關於 section
  - type 設定必需符合組件資訊清單 (Assembly Manifest)
  - type ="{Namespace + ClassName},[dllName]",如果configSections 定義在 web.config 中可忽略不填，如是置於外部檔還是需要填寫
  - 參考的組件 assembly - dll 必需與 web.config 在相同的應用程式目錄中
- 範例 1 (官方建議寫法) - 需搭配下方 `範例 1` 使用

    ```cs
    <configSections>
        <sectionGroup name="yowkoSettingGroup">
          <section name="yowkoConfig" type="TestConfig.Handler.YowkoConfiguration,TestConfig" />
        </sectionGroup>
    </configSections>
    ```

- 範例 2 (簡潔寫法) - 需搭配下方 `範例 2` 使用

    ```cs
    <configSections>
          <section name="yowkoConfig" type="TestConfig.Handler.YowkoConfiguration,TestConfig" />
    </configSections>
    ```

## 3. web.config 加上 自定 section 內容

- 名稱需與 configSection 宣告名稱相同
- 如有 group ，階層也必需相同(下圖 config 為了清楚標示，有多幾個空白)

    ![3webconfig](https://cloud.githubusercontent.com/assets/3851540/23091844/58701c74-f5fa-11e6-9eb6-846747592e67.png)
- 範例 1 (官方建議寫法) - 需搭配上方及下方 `範例 1` 使用

    ```cs
    <yowkoSettingGroup>
        <yowkoConfig name="Yowko Tsai" tel="0123456789">
          <yowkoSetting site="blog.yowko.com" userName="yowko" password="yowkoPass"></yowkoSetting>
        </yowkoConfig>
    </yowkoSettingGroup>
    ```

- 範例 2 (簡潔寫法) - 需搭配上方及下方 `範例 2` 使用

    ```cs
    <yowkoConfig name="Yowko Tsai" tel="0123456789">
      <yowkoSetting site="blog.yowko.com" userName="yowko" password="yowkoPass"></yowkoSetting>
    </yowkoConfig>
    ```

## 4. 使用方式

- 使用 `ConfigurationManager` 指定區段並轉型為一開始自訂的 configuration handler
- 如有 group 包覆，`GetSection` 必需使用正確階層
- 範例 1 (官方建議寫法)
  - 需搭配上方 `範例 1` 使用

      ```cs
      YowkoConfiguration clientConfiguration = ConfigurationManager.GetSection("yowkoSettingGroup/yowkoConfig") as YowkoConfiguration;
      ```

- 範例 2 (簡潔寫法)
  - 需搭配上方 `範例 2` 使用

      ```cs
      YowkoConfiguration clientConfiguration = ConfigurationManager.GetSection("yowkoConfig") as YowkoConfiguration;
      ```

## 5. 需要多組相同設定格式的情境時

- 5-1. handler
  - 新增自訂型別屬性
  - 套用 `ConfigurationProperty` 指定對應 config 檔內容的元素名稱
  - 套用 `ConfigurationCollection` attribute 並指定`需要重複格式的型別`以及`元素名稱`

    ![4multiple](https://cloud.githubusercontent.com/assets/3851540/23091845/58708efc-f5fa-11e6-9847-bb6e144bcc6e.png)
  - 自訂型別 `必須`要繼承 `ConfigurationElementCollection`  並覆寫兩個方法
      1. CreateNewElement
            - 指定建立新元素的方式

                ```cs
                protected override ConfigurationElement CreateNewElement()
                {
                    return new YowkoSettingElement();
                }
                ```

      2. GetElementKey
            - 指定一個 unique key

                ```cs
                protected override object GetElementKey(ConfigurationElement element)
                {
                    return (element as YowkoSettingElement).Site;
                }
                ```

  - 完整範例
      ![5configrelation](https://cloud.githubusercontent.com/assets/3851540/23091846/5872cf82-f5fa-11e6-98a1-3955ccdba93f.png)

      ```cs
      public class YowkoConfiguration : ConfigurationSection
      {
          [ConfigurationProperty("name")]
          public string Attr_Name
          {
              get
              {
                  return this["name"] as string;
              }
              set
              {
                  this["name"] = value;
              }
          }
    
          [ConfigurationProperty("tel")]
          public string Attr_Tel
          {
              get
              {
                  return this["tel"] as string;
              }
              set
              {
                  this["tel"] = value;
              }
          }
          
          [ConfigurationProperty("yowkoSettings")]
          [ConfigurationCollection(typeof(YowkoSettingElement), AddItemName = "yowkoSetting")]
          public YowkoSettingElementCollection YowkoSettings => this["yowkoSettings"] as YowkoSettingElementCollection;
      
      }
    
      public class YowkoSettingElementCollection :ConfigurationElementCollection
      {
          protected override ConfigurationElement CreateNewElement()
          {
              return new YowkoSettingElement();
          }
    
          protected override object GetElementKey(ConfigurationElement element)
          {
              return (element as YowkoSettingElement).Site;
          }
      }
    
    
      public class YowkoSettingElement :ConfigurationElement
      {
          [ConfigurationProperty("site", DefaultValue = "", IsRequired = true)]
          public string Site
          {
              get
              {
                  return this["site"] as string;
              }
              set
              {
                  this["site"] = value;
              }
          }
    
          [ConfigurationProperty("userName", IsRequired = true)]
          public string UserName
          {
              get
              {
                  return this["userName"] as string;
              }
              set
              {
                  this["userName"] = value;
              }
    
          }
    
          [ConfigurationProperty("password", IsRequired = true)]
          public string Password
          {
              get
              {
                  return this["password"] as string;
              }
              set
              {
                  this["password"] = value;
              }
          }
      }
      ```

- 5-2. web.config 設定
  - web.config 中 configSections 加入自訂 section 宣告

      ```xml
      <configSections>
          <sectionGroup name="yowkoSettingGroup">
              <section name="yowkoConfig" type="TestConfig.Handler.YowkoConfiguration,TestConfig" />
          </sectionGroup>
      </configSections>
      ```

  - web.config 加上 自定 section 內容

      ```xml
      <yowkoSettingGroup>
          <yowkoConfig name="Yowko Tsai" tel="0123456789">
              <yowkoSettings>
                  <yowkoSetting site="blog.yowko.com" userName="yowko" password="yowkoPass"></yowkoSetting>
                  <yowkoSetting site="www.yowko.com" userName="yowko2" password="yowkoPass2"></yowkoSetting>
              </yowkoSettings>
          </yowkoConfig>
      </yowkoSettingGroup>
      ```

- 5-3. 使用方式
  - 透過 `ConfigurationManager.GetSection` 取得設定(需注意階層)
  - 範例

      ```cs
      var section = ConfigurationManager.GetSection("yowkoSettingGroup/yowkoConfig");
      ```

## 6. 如果 config 是獨立檔案的情境

1. handler 皆與上述各個流程情境一致
2. config
    - 設定結構與上述各流程相同
    - 需留意 config 檔案格式需與 web.config 相同 (e.g.XML,由 configuration 元素為根元素)
3. 使用方式
    - 指定 config 位置

        ```cs
        var configFile = Path.Combine(Server.MapPath("~/App_Data"), "YowkoSetting.config");
        ```

    - 使用 config 位置來取得 ConfigurationFileMap

        ```cs
        ConfigurationFileMap fileMap = new ConfigurationFileMap(configFile);
        ```

    - 將 ConfigurationFileMap 餵給 ConfigurationManager.OpenMappedMachineConfiguration 並使用 GetSection 取出資料 (注意階層)

        ```cs
        YowkoConfiguration managerConfiguration =ConfigurationManager.OpenMappedMachineConfiguration(fileMap).GetSection("yowkoSettingGroup/yowkoConfig") as YowkoConfiguration;
        ```

    - 完整程式碼

        ```cs
        var configFile = Path.Combine(Server.MapPath("~/App_Data"), "YowkoSetting.config");
        ConfigurationFileMap fileMap = new ConfigurationFileMap(configFile);
        YowkoConfiguration managerConfiguration =ConfigurationManager.OpenMappedMachineConfiguration(fileMap).GetSection("yowkoSettingGroup/yowkoConfig") as YowkoConfiguration;
        ```

4. 如果想要取出設定時可以支援 IEnumerable 功能，記得在自訂的 ConfigurationElementCollection 一併繼承 IEnumerable 並實作 GetEnumerator 方法
    > 因為 ConfigurationElementCollection 有許多基礎類別繼承而來的屬性，我們是用不到的，除了自己一個一個對應外，使用 lambda 來處理相對容易些

    - 繼承 IEnumerable

        ```cs
        public class YowkoSettingElementCollection : ConfigurationElementCollection,IEnumerable<YowkoSettingElement>
        ```

    - 實作 GetEnumerator

        ```cs
        IEnumerator<YowkoSettingElement> IEnumerable<YowkoSettingElement>.GetEnumerator()
        {
            foreach (YowkoSettingElement element in Enumerable.Range(0, base.Count).Select(base.BaseGet))
                yield return element;
        }
        ```

    - 完整程式碼

        ```cs
        public class YowkoSettingElementCollection : ConfigurationElementCollection,IEnumerable<YowkoSettingElement>
        {
            protected override ConfigurationElement CreateNewElement()
            {
                return new YowkoSettingElement();
            }
    
            protected override object GetElementKey(ConfigurationElement element)
            {
                return (element as YowkoSettingElement).Site;
            }
            IEnumerator<YowkoSettingElement> IEnumerable<YowkoSettingElement>.GetEnumerator()
            {
                foreach (YowkoSettingElement element in Enumerable.Range(0, base.Count).Select(base.BaseGet))
                    yield return element;
            }
        }
        ```

    - 就可以使用 lambda 來取得需要屬性並轉換成 list

        ```cs
        var test = managerConfiguration.YowkoSettings.Select(a => new YowkoSettingElement {  Site= a.Site, UserName = a.UserName, Password = a.Password}).ToList();
        ```

## 心得

我怎麼記得以前好像沒這麼多需要注意的，不知道是不是因為以文字紀錄的關係，總覺得可以再詳細點再多點說明，不過經過這次整理我相信下次再次遇到時一定可以更快回憶起來的

## 參考資料

1. [組態區段結構描述](https://msdn.microsoft.com/zh-tw/library/0hyxd0xc.aspx)
2. [configSections 的 section 項目 (一般設定結構描述)](https://msdn.microsoft.com/zh-tw/library/ms228245.aspx)
3. [ASP.NET 組態檔階層架構和繼承](https://msdn.microsoft.com/zh-tw/library/ms178685.aspx)
4. [ASP.NET 組態檔結構 (區段和區段處理常式)](https://msdn.microsoft.com/zh-tw/library/w7w4sb0w.aspx) - Visual Studio 2010
5. [HOW TO：使用 ConfigurationSection 建立自訂組態區段](https://msdn.microsoft.com/zh-tw/library/2tw134k3.aspx) - Visual Studio 2010
6. [C# - App.config自訂section程式碼架構Part I(基本用法)](http://limitedcode.blogspot.tw/2016/10/c-appconfigsectionpart-i.html)
7. [ConfigurationPropertyAttribute 類別](https://msdn.microsoft.com/zh-tw/library/system.configuration.configurationpropertyattribute.aspx)
