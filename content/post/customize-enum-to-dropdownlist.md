---
title: "有 EnumDropDownListFor 為什麼還要客製 Enum to Dropdownlist ？！"
date: 2017-02-02T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["ASP.NET MVC"]
slug: "customize-enum-to-dropdownlist"
aliases:
    - /2017/02/customize-asp-dot-net-mvc-enum-to-dropdownlist.html
---
## 有 EnumDropDownListFor 為什麼還要客製 Enum to Dropdownlist ？

enum 可以讓資料儲存更有效率(string --> int), 擺脫字串比對時的不便(e.g. 手誤..)，也可以有效降低無用資料進入資料庫的機會，為了讓開發人員擁有更大的開發效率 ASP.NET MVC 5 也提供了 `EnumDropDownListFor` 的 HTML helper，那為什麼我還想要客製呢？ 就讓我細說分明

## EnumDropDownListFor 不足之處

1. 預設使用 Value 當做下拉選單選項文字
    - enum 的值無法有空白字元，移除空白字元後意思就不同了

2. 無法自訂 html name
    - name 是 model binding 的基礎，在需要 post list 時，無法自訂 name 會造成程式複雜度提高

## 如何解決

>大家可能都有不同想法，以下是我自己常用的方式，提供給大家參考，如果大家有更好的方式請教我幾招

- 原始 enum

    ```cs
    public enum RegionEnum
    {
        Default,
        ROC,
        SouthKorea,
        USA,
        UK,
        Japan
    }
    ```

0. 前置作業
    - 為 enum 加上正確顯示名稱

         > 以下兩種方法擇一即可
  
        - Description("showName")
            - using System.ComponentModel;

                ```cs
                public enum RegionEnum
                {
                    Default,
                    [Description("Description:R.O.C")]
                    ROC,
                    [Description("Description:South Korea")]
                    SouthKorea,
                    [Description("Description:U.S.A")]
                    USA,
                    [Description("Description:U.K.")]
                    UK,
                    Japan
                }
                ```

        - Display(Name="showName")
            - using System.ComponentModel.DataAnnotations;

                ```cs
                public enum RegionEnum
                {
                    Default,
                    [Display(Name= "DisplayName:R.O.C")]
                    ROC,
                    [Display(Name = "DisplayName:South Korea")]
                    SouthKorea,
                    [Display(Name = "DisplayName:U.S.A")]
                    USA,
                    [Display(Name = "DisplayName:U.K.")]
                    UK,
                    Japan
                }
                ```

    - 加上取得正確名稱的 helper
        - `Description("showName")` 使用 `GetEnumDescription`
        - `Display(Name="showName")` 則使用 `GetEnumDisplayName`

            ```cs
            public static class CustomerEnumHelper
            {
                /// <summary>
                /// 取得 Enum 的 Description
                /// </summary>
                /// <param name="value">Enum</param>
                /// <returns>Enum 的 Description</returns>
                public static string GetEnumDescription(System.Enum value)
                {
                    FieldInfo field = value.GetType().GetField(value.ToString());
        
                    DescriptionAttribute customAttribute = field.GetCustomAttribute<DescriptionAttribute>(false);
        
                    if (customAttribute != null)
                    {
                        string name = string.IsNullOrWhiteSpace(customAttribute.Description) ? string.Empty : customAttribute.Description;
                        if (!string.IsNullOrEmpty(name))
                            return name;
                    }
                    return value.ToString();
                }
                /// <summary>
                /// 取得 Enum 的 DisplayName
                /// </summary>
                /// <param name="value">Enum</param>
                /// <returns>Enum 的 DisplayName</returns>
                public static string GetEnumDisplayName(System.Enum value)
                {
                    FieldInfo field = value.GetType().GetField(value.ToString());
        
                    DisplayAttribute customAttribute = field.GetCustomAttribute<DisplayAttribute>(false);
                    if (customAttribute != null)
                    {
                        string name = string.IsNullOrWhiteSpace(customAttribute.GetName()) ? string.Empty : customAttribute.GetName();
                        if (!string.IsNullOrEmpty(name))
                            return name;
                    }
                    return value.ToString();
                }
            }
            ```

1. 自行組裝 SelectList
    - Controller(請記得選用正確方法)

        ```cs
        IList<SelectListItem> list = Enum.GetValues(typeof(RegionEnum))
                                .Cast<RegionEnum>()
                                .Select(x => new SelectListItem
                                {
                                    Text = CustomerEnumHelper.GetEnumDisplayName(x),//CustomerEnumHelper.GetEnumDescription(x) ,
                                    Value = ((int)x).ToString()
                                })
                                .ToList();
            ViewBag.list = list;
        ```

    - View

        ```cs
         @{
            var list= ViewBag.list as List<SelectListItem>;
          }
        @Html.DropDownList("a", list)
        ```

    - 效果

        ![1selectlist](https://cloud.githubusercontent.com/assets/3851540/22261722/0bc14024-e2a9-11e6-865a-4df0716668fb.png)

2. 前端 html 組裝
    - <span style="color:red">需注意用量，避免影響效能問題</span>
    - View (請記得選用正確方法)

        ```cs
         <select name="a">
        @foreach (RegionEnum enumItem in Enum.GetValues(typeof(RegionEnum)))
        {
            <option @((enumItem == RegionEnum.Default) ? "selected='selected'" : String.Empty) value="@(enumItem)">@CustomerEnumHelper.GetEnumDisplayName(enumItem)</option>
        }
        </select>
       ```

    - 效果

        ![2view](https://cloud.githubusercontent.com/assets/3851540/22261723/0bece5e4-e2a9-11e6-8a02-3258aa5ce0d1.png)

3. 擴充 html helper
    - 程式碼是參考 `Html.EnumDropDownListFor` 原始碼來的，出處在 [ASP-NET-MVC/aspnetwebstack/src/System.Web.Mvc/Html/SelectExtensions.cs](https://github.com/ASP-NET-MVC/aspnetwebstack/blob/master/src/System.Web.Mvc/Html/SelectExtensions.cs)
    - 擴充 helper (請記得選用正確方法)

        ```cs
        public static MvcHtmlString EnumDropDownList(this HtmlHelper htmlHelper, string name, Type enumType, string defaultValue = "")
        {
            IList<SelectListItem> selectList = GetSelectList(enumType, defaultValue);
            return SelectExtensions.DropDownList(htmlHelper, name, selectList, defaultValue);

        }

        public static IList<SelectListItem> GetSelectList(Type type, string defaultValue)
        {
            IList<SelectListItem> selectList = GetSelectList(type);
            if (selectList.Count != 0 && string.IsNullOrEmpty(defaultValue))
            {
                selectList[0].Selected = true;
            }
            else
            {
                for (int index = selectList.Count - 1; index >= 0; --index)
                {
                    SelectListItem selectListItem = selectList[index];
                    selectListItem.Selected = selectListItem.Text == defaultValue;
                }
            }
            return selectList;
        }

        public static IList<SelectListItem> GetSelectList(Type type)
        {
            IList<SelectListItem> selectListItemList = (IList<SelectListItem>)new List<SelectListItem>();
            Type type1 = Nullable.GetUnderlyingType(type);
            if ((object)type1 == null)
                type1 = type;
            Type type2 = type1;
            if (type2 != type)
                selectListItemList.Add(new SelectListItem()
                {
                    Text = string.Empty,
                    Value = string.Empty
                });
            foreach (FieldInfo field in type2.GetFields(BindingFlags.DeclaredOnly | BindingFlags.Static | BindingFlags.Public | BindingFlags.GetField))
            {
                object rawConstantValue = field.GetRawConstantValue();
                Enum enumValue = (Enum)(Enum.Parse(type,field.Name));
                selectListItemList.Add(new SelectListItem()
                {
                    Text = GetEnumDisplayName(enumValue),//GetEnumDescription(enumValue)
                    Value = rawConstantValue.ToString()
                });
            }
            return selectListItemList;
        }
        ```

    - View

        ```cs
        @Html.EnumDropDownList("a", typeof(RegionEnum), " - Choice - ")
        ```

    - 效果

        ![2view](https://cloud.githubusercontent.com/assets/3851540/22261723/0bece5e4-e2a9-11e6-8a02-3258aa5ce0d1.png)

## 參考資料

1. [SelectExtensions.EnumDropDownListFor 方法](https://msdn.microsoft.com/zh-tw/library/system.web.mvc.html.selectextensions.enumdropdownlistfor.aspx)
2. [ASP-NET-MVC/aspnetwebstack/src/System.Web.Mvc/Html/SelectExtensions.cs](https://github.com/ASP-NET-MVC/aspnetwebstack/blob/master/src/System.Web.Mvc/Html/SelectExtensions.cs)
