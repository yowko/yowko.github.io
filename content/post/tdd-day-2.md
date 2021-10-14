---
title: "自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 2"
date: 2017-06-10T01:00:00+08:00
lastmod: 2021-10-04T22:23:26+08:00
draft: false
tags: ["TDD","Unit Test"]
slug: "tdd-day-2"
aliases:
    - /2017/06/tdd-day-2.html
---
## 自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 2

TDD 來到了第二天，邀請我參加這次 TDD 課程的前同事 - Dino 問了我個問題：這樣參加完課程，如果沒有馬上應用或是應用後有實務面的問題，我怎麼看待這件事。我自認對學習這件事有一定的熱誠，我常常參加各個單位舉辦的技術訓練課程，對 Dino 的問題有很深的體悟，在此分享一下個人想法。

剛開始學程式設計時，我的啟蒙老師 - 青輔會職訓中心 高新煌 老師，一直強調一個觀念：`師父引入門 修行在個人`，在程式設計這條路上更是明顯，你也不會期待同個寫法可以放諸四海都適用，老師能給你的只是基礎觀念或是基本介紹，想走得多遠、走得多深都是個人選擇，課程這件事也是如此，相信大家也發現了相對於學校的老師，像 保哥 或是 91 哥這樣有業界實戰經驗的講師才能滿足現階段我們學習上的需求，也才能給予我們不只是觀念上實質幫助。

當然隨著時間的流逝，課程的記憶一定漸漸變得模糊，原本你覺得簡單的操作流程或是工具，可能再也想不起該如何開始，這個問題超困擾我的XD，所以我常說不得不服老，我的解決方式 - 將學習過程留下紀錄，而且千萬不只是個人筆記，一定要整理成文章，在整理的過程為了要讓其他人看得懂，一定會加上不少描述跟相關說明，雖然很花時間，但有一天你真的都忘光的時候，你馬上就能體會這麼做的好處：你的文章救了你自己呀

如果是課程結束後才遇到的實務問題或是過了一段時間才能實作怎麼辦呢？我的想法是這樣：無論老師的表達或是教學功力再怎樣的高深，你的吸收能力再怎麼強大，還是都會有損耗，所以我秉持著有需要就再上一次，一定有更多的收獲，第一次沒辦法靈活運用是很正常的，只要可以在需要用到時可以找到資源或是知道該怎麼找到可用的資訊就很夠了

## 第一天課程回顧

> 第一天課程詳細內容可以參考 [自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 1](//blog.yowko.com/2017/05/tdd-day-1.html)，以下內容只是個人理解後用來加強記憶用

1. legacy code 如何測試

    * 單一職責 class 拆出
    * 使用 interface
    * inject --> constructor/ public field

2. internal visible to 測試專案

    > 不該 public 就不要 public，所以保留使用 internal

3. 非 public 需要測試嗎？

    > 模擬使用者使用方法時，就會涵蓋非 public

    * private --> internal

        > * constructor 可用，可能會違反物件導向原則，可以新增一個測試用的 constructor
        > * 可以參考 單元測試的藝術 --> ch3 --> 6小節

4. 單元測試應該從重要或是常修改的程式碼先加起，效益最大

    > git 有工具可以統計出修改頻率，工具名稱待我確定後再補上

5. code review 時從測試先看

    > 可以知道明確需求

6. 使用擴充方法可以比 utility 有更好的命名

    > * 因為擴充方法是針對特定型別，不需像 utility 需要考慮泛型
    > * 擴充方法在團隊中需要被規範，不該什麼都寫成擴充方法

7. 意圖導向程式設計 (Programming-by-Intention)

    > 寫上想要的，其他的讓 ide 自動產生，這樣有助於 api 的簡化及 api 使用的便利性

## 課程心得 - Web Testing

* unit test 不足之處

    1. 避免單元測試皆正常，整合上卻出現非預期的狀況
    2. 除了 unit test 還需要其他保障

* Test 粒度

    1. Unit Test

        > isolated，將外部相依全部隔絕

    2. Intgration Test

        > * 不模擬外部相依，讓測試可以依 production 實際情境執行
        > * 真實使用的相依皆實際使用 不隔離，還是在測試 class 或是 assembly

    3. 驗收測試(Acceptance Test 或是稱 End-to-End Test)

        > 操作方式與使用者相同，intgration Test 是以程式觸發，一般還是由測試專案發動，驗收測試則是由使用者從外部觸發，可能是 UI 或是 App 驅動

    * 粒度愈大愈擬真，愈接近使用者，也愈好寫

        > 只管 input 與 output ，其他都黑箱

* 測試金字塔

    ![5測試金字塔](https://user-images.githubusercontent.com/3851540/26967247-1a12caca-4d30-11e7-8dd1-62ef4101dec2.png)

    1. 粒度愈小 比例愈高
    2. 粒度愈小 測試成本愈低
    3. 粒度愈高 人力測試比例愈高

* Selenium IDE

    > 基本操作方式請參考 [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html)

    1. 可以用來降低開發人員進入 web testing 的門檻
    2. 可以將錄製完的結果轉換為測試腳本程式碼

        > 原生支援 nunit ，可以透過自訂 formatter 產生 MSTest 及 xUnit，有興趣的請參考 [製作 Selenium IDE 的 xUnit.net 2.0 版 Formatter](//blog.yowko.com/2017/06/selenium-ide-xunitnet-20-formatter.html)

    3. clickboard 指定取得不同語言的 element 寫法

        * 先選擇需要的語言

            ![6clcikboard](https://user-images.githubusercontent.com/3851540/26967246-1a0e08c8-4d30-11e7-9345-9ffab8fe54ff.png)

        * 在需要的 Command 上按下 `Ctrl+c` 即可取得指定語言的指令

    4. 透過錄製網站操作會轉為 command

        * commnad --> 支援 css selector 及 xpath

            ![1command1](https://user-images.githubusercontent.com/3851540/26967241-19caeff2-4d30-11e7-937e-3eb51cedf160.png)

            ![2command2](https://user-images.githubusercontent.com/3851540/26967243-19f01944-4d30-11e7-935a-32fa293e1d4b.png)

    5. target 可以用選的

        ![4targetselect](https://user-images.githubusercontent.com/3851540/26967245-1a0d5c70-4d30-11e7-9c8f-04c80ae7a229.png)

        * 按 find 會反白

            ![3find](https://user-images.githubusercontent.com/3851540/26967244-1a07bb12-4d30-11e7-9a4f-2d5ed8f11869.png)

    6. 特性：SAFE
        * S：Simple
        * A：Auto
        * F：Fast
        * E：Export

    7. 缺點：Selenium IDE 無法處理檔案，使用 Selenium API 可以

* Selenium 的問題

    * Selenium api 太像程式碼

        1. 測試案例用來描述需求
        2. 測試程式用來驗證需求是否與結果相符

* Fluent Automation

    > 讓測試程式更加語意化

    1. 支援：selenium 及 watin
    2. 詳細介紹與使用教學可以參考 [使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式](//blog.yowko.com/2017/06/fluent-automation-selenium-web-ui-test.html)

* PageObject (Page Object Pattern)

    1. 讓測試案例與實際 UI 互動抽象化
        * test case 專注於描述及說明需求
        * 實際 UI 可能會經常異動，不該讓 test case 直接相依

    2. DRY

        > 讓測試程式更符合物件導向程式設計，不是一直重複寫相同的程式碼

    * 詳細介紹與使用教學可以參考 [使用 PageObject(Page Object Pattern) 建立物件導向的 Web UI 測試程式](//blog.yowko.com/2017/06/pageobject-web-ui-test.html)


## 課程心得 - 重構(Refactoring)

> 這個部份不會提及實際程式碼，只會有觀念：一來是要尊重智慧財產權，二來是大家遇到問題千變萬化，絕對不是一招半式就可以解決，首重心法呀

* 如何讓程式碼品質變好

    > 從現在開始不要再繼續疊床架屋

    1. 建立 web testing
    2. 從測試有案驅動 web testing
    3. 重構 加上註解 並透過 viewmodel 讓 logic 與 ui 分離

        > 拆離 viewmodel 與 ui control 的關係

    4. 重構、擷取方法
    5. 重構 職責分離
    6. 建立單元測試
    7. 重構  擷取介面
    8. 重構 獨立生成物件職責

* 如何在 legacy code 加上測試

    > 詳細做法可以參考 [C# Test Legacy Code（1）Isolated by Inheritance and Override](http://www.codedata.com.tw/social-coding/csharp-legacy-code-test-1-isolated-by-inheritance-override/)

    1. extract and overwrite

        > static or sealed

    2. 重構前有測試保護

        > 建立粒度更大的測試來保護

    3. Microsoft fakes

        > 可以針對幾乎所有物件生成假物件

* 測試程式的基本要求

    1. Readable

        > 可讀性，才知道到底測試做了哪些事

    2. Maintainable

        > 測試程式如果難以維護，就會讓測試日漸荒廢

    3. Reliable

        > 測試如果信任，就不用浪費時間寫測試

    4. test case 應該包含所有程式可能路徑

* 實際重構步驟

    1. RTFLC

        > reading the fucking legacy code，看一下該如何下刀

    2. comments + viewmodel
    3. extract method
    4. move to class
    5. create unit test
    6. extract interface

        > 抽出可共用部份

    7. 獨立生成物件職責

        * factory method
        * simple factory
        * abstract factory

* 重構成 design pattern

    1. strategy pattern (step 6)

        > 依據合適情境來挑選適合的工作模式或是方法

    2. 工廠模式
    3. 重構結果與 context diagram 不同

        * 你的重構還有改善空間
        * 你的重構比 context diagram 建議的更適合你

* interface 與 class 一對一問題

    > 首先避免惡化，這是 over design，針對沒有被外部參考的 class 開始移除使用 interface

* 單一 interface 太過龐大，包含各式各樣功能

    1. 將其中的職責拆分成不同 interface
    2. 將原interface 繼承新拆分的 interface

* interface 對應的實體需要數量不一的參數

    1. 參數陣列(一個物件)
    2. 多個參數
    3. 沒有參數
        * 不同 class 在各自的 constructor 給定
        * 工廠方法
                * 關閉 public constructor ，依實際需要丟不同參數，使用不同 static 方法回應不同實作

* 重構的好書推薦

    1. refactor to patterns
    2. working enffiency with legacy code
    3. refactoring

## 其他補充資訊

* 好用工具

    1. codemaid
        * VS 套件
        * 使用程式碼執行路徑來計算程式碼循環複雜度

    2. ghostdoc

        * 程式的 XML 註解更容易產生，詳細資訊可以參考 [使用 GhostDoc 自動產出符合語意的註解](https://dotblogs.com.tw/wasichris/2016/01/21/172429)

* 好的設計準則

    1. 好的抽象看 interface
    2. 好的重用看 abstract

* 團隊程式碼規範建議

    1. 針對 3 layer 程式架構

        > service unit test first，針對 service 建立 unit test 是最省成本也是最有效率的作法

    2. 不給 new ,DI design --->浮現式設計

        > 除了 dto ，強制無法直接相依 (只能用 di 或是 工廠)

    3. INTENSION - DRIVEN

        > 意圖導向

    4. 繼承深度

        > * 自己寫的只能兩層，production issue 可以 3 層
        > * 因為不可逆

        * 使用工具

            * vs

    5. 複雜程式 <= 10

        > 方法複雜度

        * 使用工具

            * sourceMonitor
            * vs

    6. code clone < 25 行

        > 檢查是否為重複程式碼

        * 使用工具
            * vs
            * simian

    7. block depth <=5

        > 就是 Visual Studio 縮排的階層

        * 使用工具

            * sourceMonitor

    8. SonarQube

        > 靜態程式碼分析工具，~~NCrunch 也是~~ 經 Cash 大大提醒 NCrunch 不是分析工具喔

## 心得

Day 2 內容很豐富，精彩度自然不在話下，只是以一整天的時間來看，個人感受覺得太趕，暫且不看整段跳過的內容(ex. MVC 的單元測試、Web API MessageHandler 的單元測試...)，就當天有講到的部份，不少內容很快就帶過，很可能是個人能力不足才造成跟不上，所以個人覺得吸收效果稍差

## 參考資訊

1. [自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 1](//blog.yowko.com/2017/05/tdd-day-1.html)
2. [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html)
3. [製作 Selenium IDE 的 xUnit.net 2.0 版 Formatter](//blog.yowko.com/2017/06/selenium-ide-xunitnet-20-formatter.html)
4. [使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式](//blog.yowko.com/2017/06/fluent-automation-selenium-web-ui-test.html)
5. [使用 PageObject(Page Object Pattern) 建立物件導向的 Web UI 測試程式](//blog.yowko.com/2017/06/pageobject-web-ui-test.html)
6. [C# Test Legacy Code（1）Isolated by Inheritance and Override](http://www.codedata.com.tw/social-coding/csharp-legacy-code-test-1-isolated-by-inheritance-override/)
7. [使用 GhostDoc 自動產出符合語意的註解](https://dotblogs.com.tw/wasichris/2016/01/21/172429)
