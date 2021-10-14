---
title: "右鍵選單 Create Unit Tests (建立單元測試) 選項不見？！"
date: 2017-05-30T16:46:00+08:00
lastmod: 2021-10-04T18:41:02+08:00
draft: false
tags: ["Debug","Unit Test","Visual Studio"]
slug: "missing-create-unit-tests"
aliases:
    - /2017/05/missing-create-unit-tests.html
---
## 右鍵選單 Create Unit Tests (建立單元測試) 選項不見？！

這是什麼情況？！ 是叫我不要寫 Unit Test 嗎XD，還是人品真的差成這樣，既然人品問題這麼嚴重，當然要紀錄一下，邀免下次人品再次遭受挑戰時，又要重新 debug，所幸最後不需要重灌就解決問題

## 實際情況

看不到 Create Unit Tests 選項

![1missing](https://cloud.githubusercontent.com/assets/3851540/26575124/7cf8a6be-4556-11e7-8857-e1f7f34f8f1f.png)

## Test Exporler 無法開啟 - 錯誤訊息

- 訊息內容

    ```txt
    An exception was encountered while constructing the content of this frame.  his information is also logged in C:\Users\YowkoTsai\AppData\Roaming\Microsoft\VisualStudio\15._c8ea91e2Exp\ActivityLog.xml".
        Exception details:
    System.ArgumentException: The parameter is incorrect. (Exception from RESULT: 0x80070057 (E_INVALIDARG))
    at System.Runtime.InteropServices.Marshal.ThrowExceptionForHRInternal(Int32 rrorCode, IntPtr errorInfo)
    at Microsoft.VisualStudio.Shell.Package.CreateToolWindow(Type toolWindowType,Int32 id, UInt32 flags)
    at Microsoft.VisualStudio.Shell.Package.CreateToolWindow(Type toolWindowType,Int32 id, ProvideToolWindowAttribute tool)
    at Microsoft.VisualStudio.Shell.Package.FindToolWindow(Type toolWindowType, nt32 id, Boolean create, ProvideToolWindowAttribute tool)
    at Microsoft.VisualStudio.Shell.Package.CreateToolWindow(Guid& oolWindowType, Int32 id)
    at icrosoft.VisualStudio.Shell.Package.Microsoft.VisualStudio.Shell.Interop.IVsoolWindowFactory.CreateToolWindow(Guid& toolWindowType, UInt32 id)
    at icrosoft.VisualStudio.Platform.WindowManagement.WindowFrame.ConstructContent)
    ```

- 錯誤截圖

    ![2error](https://cloud.githubusercontent.com/assets/3851540/26575123/7cf654a4-4556-11e7-8c8b-fc5660206cfe.png)

## 解決方式

- 在開啟 Visual Studio 時加上不同參數嘗試解決問題

    ![vswithparam](https://cloud.githubusercontent.com/assets/3851540/23978788/d500168c-0a30-11e7-9757-a8186fdb1ab3.png)

    ![vswithparam2](https://cloud.githubusercontent.com/assets/3851540/23978787/d4ff4306-0a30-11e7-8e6e-742b5cf86c6b.png)

1. 先確認是不是套件造成的問題

    - 使用 `乾淨模式` 開啟 Visual Studio

         在 Visual Studio 開啟時加上 `/safemode`，詳細資訊可參考 [Visual Studio 怪怪的？！ Visual Studio 好慢？！(Visual Studio 的無痕模式)](/visual-studio-safe-mode)

    - 如果是套件影響的，就需要逐一過濾(disable)是哪個套件造成問題

        ![3diable](https://cloud.githubusercontent.com/assets/3851540/26575261/0c2c539e-4557-11e7-817c-c1e5dea34d9b.png)

2. 如果不是套件造成的，請嘗試其他參數

    - `/resetskippkgs`
    - `/installvstemplates`
    - `/resetsettings`
    - `/resetuserdata`

        > 我一路試下來，做到這步才真的解決問題，只是個人設定都得要重來，提供給大家參考

## 參考資訊

1. [Visual Studio 怪怪的？！ Visual Studio 好慢？！(Visual Studio 的無痕模式)](//blog.yowko.com/2016/12/visual-studio-safe-mode.html)
2. [VS2013 periodically crashes and needs setup repair](https://social.msdn.microsoft.com/Forums/vstudio/en-US/1263cd5f-26f3-448f-80ec-a20c9d77ffd6/vs2013-periodically-crashes-and-needs-setup-repair?forum=vssetup)
