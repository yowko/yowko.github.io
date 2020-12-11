---
title: "ASP.NET Core 中 Controller 與 ControllerBase 的差別"
date: 2019-06-09T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "aspdotnet-core-controller-controllerbase"
---

# ASP.NET Core 中 Controller 與 ControllerBase 的差別

之前筆記 [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore) 提到 `AddMvc()` 與 `AddMvcCore()` 的差別，今天剛好在整理如何從 Empty 專案加入 Web API 時聯想到似乎沒有很清楚實際差別，趁著自己查資料，順手筆記一下

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)

## ControllerBase

完整程式碼請參考 [AspNetCore/src/Mvc/Mvc.Core/src/ControllerBase.cs](https://github.com/aspnet/AspNetCore/blob/v2.2.5/src/Mvc/Mvc.Core/src/ControllerBase.cs)

- 部份程式碼

    ```cs
    /// <summary>
    /// A base class for an MVC controller without view support.
    /// </summary>
    [Controller]
    public abstract class ControllerBase
    {
        private ControllerContext _controllerContext;
        private IModelMetadataProvider _metadataProvider;
        private IModelBinderFactory _modelBinderFactory;
        private IObjectModelValidator _objectValidator;
        private IUrlHelper _url;

    ....
    ```

- 是 abstract class
- 非 View 相關的 MVC 核心操作皆定義於此



## Controller

完整程式碼請參考 [AspNetCore/src/Mvc/Mvc.ViewFeatures/src/Controller.cs](https://github.com/aspnet/AspNetCore/blob/v2.2.5/src/Mvc/Mvc.ViewFeatures/src/Controller.cs)

- 部份程式碼

    ```cs
    /// <summary>
    /// A base class for an MVC controller with view support.
    /// </summary>
    public abstract class Controller : ControllerBase, IActionFilter, IAsyncActionFilter, IDisposable
    {
        private ITempDataDictionary _tempData;
        private DynamicViewData _viewBag;
        private ViewDataDictionary _viewData;

    ....
    ```

- 是 abstract class 且繼承自 `ControllerBase`
- 比 `ControllerBase` 多了 View 相關支援



## 心得

1. `Controller` 繼承自 `ControllerBase`
2. `Controller` 比 `ControllerBase` 多了 View 相關的支援

    - View
    - PartialView
    - ViewComponent
    - Json (JsonResult)

3. `Controller` 比 `ControllerBase` 多了

    - TempData
    - ViewData
    - ViewBag

由上述內容可以發現，如果只是單純使用 Web API 不會使用到 View 相關功能，應該在建立 Controller 時就繼承自 `ControllerBase` 而非 `Controller`

## 參考資訊

1. [ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別](/aspdotnet-core-addmvc-addmvccore)
2. [AspNetCore/src/Mvc/Mvc.Core/src/ControllerBase.cs](https://github.com/aspnet/AspNetCore/blob/v2.2.5/src/Mvc/Mvc.Core/src/ControllerBase.cs)
3. [AspNetCore/src/Mvc/Mvc.ViewFeatures/src/Controller.cs](https://github.com/aspnet/AspNetCore/blob/v2.2.5/src/Mvc/Mvc.ViewFeatures/src/Controller.cs)
4. [Why derive from ControllerBase vs Controller for ASP.NET Core Web API?](https://stackoverflow.com/a/55239483/3600583)
