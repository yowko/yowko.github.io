---
title: "自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 3"
date: 2017-07-16T18:14:00+08:00
lastmod: 2018-09-24T18:14:54+08:00
draft: false
tags: ["TDD"]
slug: "tdd-day-3"
aliases:
    - /2017/07/tdd-day-3.html
---
## 自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 3
自 TDD 課程第三天課程結束至今約莫一個月，期間持續針對 Day 3 內容斷斷續續做些紀錄，但大多是工具操作，課程核心內容較少，可以想見的隨著時間的流逝，對於課程內容的相關記憶也跟著迅速消失，雖然早就預知會有這種狀況，原本打算在課程剛結束立馬就要儘早紀錄，但無奈課程中使用不少工具都是原本不熟悉的，所以就先動手紀錄工具使用，加上自己莫名其妙的堅持想要自己動手做過，筆記才不會只是流於紀錄

剛好公司專案正在如火如塗地進行中，也瓜分了不少時間在紀錄專案相關技術上，以致於拖到現在才能好好地來進行 Day 3 的紀錄，不過以上就是找個藉口只是為了合理化自己沒有好好規劃分配時間與說服自己接受既定的事實 @@"

## Day2 課程回顧與作業回顧

1.  重構(Refactoring)

    > 這個部份請參考 [自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 2](https://blog.yowko.com/2017/06/tdd-day-2.html)，有較詳細的說明

    *   RTFLC
    *   comments + viewmodel
    *   extract method
    *   move to class
    *   create unit test
    *   extract interface
    *   獨立生成物件職責

2.  API 設計應該包含但不超出當前已知的所有需求

    *   包含當前已知所有需求

        > 已知訂單可以接受多筆，不用刻意一開始將購物車設計成只接受單筆訂單，等到需要多筆訂單測試時才修改為允許多筆訂單

    *   不超出當前已知需求

        > 需求要求可以綁定 FB 註冊及登錄，不該將 goole 或是 Microsoft Live ID 登錄一併完成

3.  先寫測試

    > 練習用測試來描述需求，藉此釐清需求

4.  練習一次只寫一個測試案例，並且只寫剛好讓測試通過的程式碼

    > 練習化整為零的思考

    *   將需求視為一整包

        > 容易出現多個開發項目互有相依，而造成交付交付周期過長，屆時如果需要分段上線，想要完整開發的想法就會造成需求可能無法切分而沒辦法達到分階段上線。

    *   將需求拆分為多個小需求

        > 總開發時程會因此拉長，但交付周期較短，面對變動市場有較靈活的反應能力

5.  針對程式碼執行效能也該被考慮

    > 需要針對常用 function 或是 loading 較重的 function 訂下效能標準，並加入測試案例

6.  重構習慣
    *   先簡單再彈性
    *   即時重構

        > 即針對新增的最少 production code 來重構 而不是一次針對所有 production code，不僅是針對程式碼，也該為 API 整體設計進行重構

7.  Moq 可以直接 mock protected method ，詳細內容可以參考 [使用 Moq 來 Mock protected Method](https://blog.yowko.com/2017/07/moq-mock-protected-method.html)

    > 延伸閱讀：[如何 Mock Private Method 的回傳值 - 使用 JuskMock](https://blog.yowko.com/2017/07/mock-private-method-juskmock.html)

8.  經典語錄：連開會都無法準時結束   你怎麼能期待專案能準時上線


## Unit test v.s. TDD v.s. BDD

1.  目的

    *   Unit test

        > 確保程式執行結果跟你想的一樣

    *   TDD

        > 確保你先想清楚才動手寫程式

    *   BDD

        > 確保你想的是使用者要的

2.  Cost v.s. Length of Feedback Cycle

    ![Cost v.s. Length of Feedback Cycle](://www.ambysoft.com/artwork/comparingTechniques.jpg)圖片擷取自 [Why Agile Software Development Techniques Work: Improved Feedback](://www.ambysoft.com/essays/whyAgileWorksFeedback.html)

    
    > 指出傳統軟體開發流程與 Unit test v.s. TDD 間的回饋周期與成本對應比較

## TDD

*   TDD 一次只關注一條路徑

    > TDD 很重要的是 test case 順序的選擇，最短路徑優先

*   [RIP TDD](https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847)

    1.  Over-engineering

        > 過度開發 - 不該多寫任何一行不在需求內的程式

    2.  API feedback

        > API 回饋 -透過 TDD 的過程來調整 API 讓 API 更符合使用者需求，以 改善 API 的設計與可用性

    3.  Logic errors

        > 邏輯謬誤 - 透過測試來確認需求，再利用測試來驗證結果與預期相符，避免想的跟寫的不一樣 寫的跟需求不一樣)

    4.  Documentation

        > 說明文件的問題 - 工程師最討厭的兩件事：寫文件 跟 別人不寫文件，透過測試來描述需求，就可得知 API 使用方式也能避免寫文件，也不用擔心隨著需求變化文件疏於修改而出現文件與程式行為不一致的狀況

    5.  Feeling overwhelmed

        > 舉足無措 - TDD 可以協助開始邁出第一步。在最後 solution 還沒有被確認甚至 solution 還沒發展出來前，仍可以從測試開始做，為未知專案找到切入點

    6.  Separate interface from implementaion thinking

        > 抽象設計 - 透過 interface 來隔離實作思維與 API 設計，避免 API 實作被汙染

    7.  Agreement

        > 協議 - 透過 TDD 來確保問題確實得到解決

    8.  Anxiety

        > 擔心受怕 - 害怕改東壞西

*   TDD 並不是一種規範而是一個習慣

    > 讓你可以一次只專注在一件事 並且讓程式碼有較佳的架構及可讀性

*   TDD 三大原則

    > 內容請參考 [The Three Laws of TDD.](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd)

    1.  You are not allowed to write any production code unless it is to make a failing unit test pass.

        *   必需先用一個紅燈來描述要做什麼，否則不可以修改 production code
        *   新增 test scenario 前，不能改 code - 除非需求變更，否則不該改 code(重構不在此限)
        *   異動 production code 之前，先用一個紅燈描述目的或需求

    2.  You are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures.

        *   得到測試紅燈(增加 test scenario)後，增加新功能的 production code 讓紅燈變綠
        *   一次只新增一個單元測試
        *   不能編譯不算成功得到紅燈
        *   使用編譯成功，並且得到一個 failed test (紅燈)

    3.  You are not allowed to write any more production code than is sufficient to pass the one failing unit test.

        *   不該多寫任何讓測試通過無關的程式碼
        *   目標就是通過測試，與紅燈變綠燈無關的一行 production code 都不多寫

*   練習寫語意化程式

    > 讓程式可以自我解釋，也能擁有較佳的可閱讀性及較好的 API 設定

*   測試案例與測試程式是同個東西

    > 不同用語只是用來讓人區別指的是需求或是程式

*   simple Design

    > 可以參考 [Xp Simplicity Rules](http://wiki.c2.com/?XpSimplicityRules)

    1.  可讀性高 容易理解意圖
    2.  抽象層次一致
    3.  測試保護

*  kent beck xp programming

    > 可以參考 [BeckDesignRules by Martin Fowler](https://martinfowler.com/bliki/BeckDesignRules.html) 重要性依順序不同

    1.  Passes the tests

        > 綠燈 且滿足需求

    2.  Reveals intention

        > 易讀

    3.  No duplication

        > 可維護性

    4.  Fewest elements

        > 滿足上面的條件後 最少的 class 及 interface ，剛好 愈少愈好

## BDD

*   為什麼 TDD 不夠

    > 因為目的不同

    *   Unit test

        > 確保程式執行結果跟你想的一樣

    *   TDD

        > 確保你先想清楚才動手寫程式

    *   BDD

        > 確保你想的是使用者要的

*   BDD behavior driven development - domain speficic languange

    *   從行為面用人話描述系統功能
    *   從人話自動繫結結式執行流程
    *   user story 與測試程式的橋樑



## SpecFlow

透過 SpecFlow 來將需求說明與程式程式分離，讓需求說明使用更貼近人類語言的方式來描述，說明需求的效果更一目了然，讓非開發人員也可以看得懂、可以修改甚至寫得出來一份合格的需求文件

*   如何使用 SpecFlow 請參考 [使用 SpecFlow 建立人語化的單元測試](https://blog.yowko.com/2017/06/specflow.html)

*   SpecFlow 也支援像是 NUnit 中 TestCase 、TestCaseSource，使用資料集進行批量測試的做法，請參考 [餵資料集給 SpecFlow 來執行測試及驗證](https://blog.yowko.com/2017/06/specflowoutline.html)

## Living Document

工程師不喜歡寫文件應該是大家的共識，加上文件很容易隨著時間而與程式碼出現落差，畢竟會隨著需求異動的只有程式碼，為了讓文件可以與程式碼對得上，最好的方式就是透過程式碼來產生說明文件

*   實際做法請參考 [使用 Pickles 搭配 SpecFlow 產生即時更新文件(living documentation)](https://blog.yowko.com/2017/06/pickles-specflow.html)


    *   內容使用到 MSTest 指令來進行測試，如果對於 MSTest 指令可以參考 [使用 MSTest.exe 指令來進行測試](https://blog.yowko.com/2017/06/mstest-exe.html)
    *   內容有使用到 Visual Studio 中的外部工具(External Tools)，對於 Visual Studio 中的外部工具(External Tools) 不熟悉的請參考 [關於 Visual Studio 中的外部工具(External Tools)](https://blog.yowko.com/2017/06/visual-studio-external-tools.html)



## 心得

拖了一個月，終於完成了 Day 3 的筆記 ~~~~ 呼  

雖說是完成，但實際內容果然跟預期一樣很多都忘記了，有些自己當時留下的文字也不記得詳細內容，但還好之前針對部份工具先紀錄，不是全忘光

這次課程我是參加台中班，課程期間台北也有開班，後來有幸拜讀台北班學員 -James 的筆記，筆記詳細與完整程度令人讚嘆不已，這也讓我一直在考慮是不是還需要繼續 Day 3 筆記，畢竟我相信就算在我記憶深刻時，我也無法像 James 那樣鉅細糜遺地紀錄(看 James 的筆記可能效果比看自己的筆記還好)。但後來我還是決定完成它，畢竟自己的文字還是比較有親切感 XD 加上自己寫東西的過程可以加深印象。不過排除這些原因，想要更完整地了解 Day 3 內容，還是建議參考 James 的 [【TDD】課堂心得與筆記 - Day 3](https://dotblogs.com.tw/jameswang/2017/06/26/184019)

透過整整三天的 TDD 訓練，加上車程往返，還有回家作業跟其他相關工具使用的紀錄，真的花了不少時間跟精力，當然也學到很多東西。雖然我盡力地紀錄著相關內容，但重點不是這些筆記所紀錄的內容跟工具的使用方式，最重要的是 `思維的改變`，而這也是最難透過文字傳達的部份。如果你沒有接觸過測試，我很推薦寫程式的朋友去上 91 的 TDD；如果想提升程式碼品質或是即將導入測試，就一定得去上，千萬不要用錯誤的方式來測試。

在第一次上課時對測試有了基本的概念跟認識，實際使用上遇到問題，第二次重新修正錯誤的方式跟釐清觀念。雖然付出了很多，不管時間還是金錢，但我有信心地說這對我的工程師生涯覺得是值得的

# 參考資訊

1.  [自動測試與 TDD 實務開發 (使用 C# ) 心得 - Day 2](https://blog.yowko.com/2017/06/tdd-day-2.html)
2.  [使用 Moq 來 Mock protected Method](https://blog.yowko.com/2017/07/moq-mock-protected-method.html)
3.  [如何 Mock Private Method 的回傳值 - 使用 JuskMock](https://blog.yowko.com/2017/07/mock-private-method-juskmock.html)
4.  [Why Agile Software Development Techniques Work: Improved Feedback](http://www.ambysoft.com/essays/whyAgileWorksFeedback.html)
5.  [RIP TDD](https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847)
6.  [The Three Laws of TDD.](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd)
7.  [Xp Simplicity Rules](http://wiki.c2.com/?XpSimplicityRules)
8.  [BeckDesignRules by Martin Fowler](https://martinfowler.com/bliki/BeckDesignRules.html)
9.  [使用 SpecFlow 建立人語化的單元測試](https://blog.yowko.com/2017/06/specflow.html)
10.  [餵資料集給 SpecFlow 來執行測試及驗證](https://blog.yowko.com/2017/06/specflowoutline.html)
11.  [使用 Pickles 搭配 SpecFlow 產生即時更新文件(living documentation)](https://blog.yowko.com/2017/06/pickles-specflow.html)
12.  [使用 MSTest.exe 指令來進行測試](https://blog.yowko.com/2017/06/mstest-exe.html)
13.  [關於 Visual Studio 中的外部工具(External Tools)](https://blog.yowko.com/2017/06/visual-studio-external-tools.html)
14.  [【TDD】課堂心得與筆記 - Day 3](https://dotblogs.com.tw/jameswang/2017/06/26/184019)
