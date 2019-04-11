---
title: "在 Mac 上安裝 Kafka-Manager"
date: 2019-03-09T21:30:00+08:00
lastmod: 2019-03-09T21:30:31+08:00
draft: false
tags: ["Kafka","Mac"]
slug: "kafka-manager"
---
# 在 Mac 上安裝 Kafka-Manager
最近的專案使用了 Kafka 當做中間層的訊息傳遞工具，功能上還不到遇到問題就先遭遇開發 debug 的種種狀況，其中最常發生但問題原因又不太一樣的就是 producer 有送出訊息但 consumer 沒有成功收到，究竟是 producer 送錯 topic、consumer 聽錯 topic、還是 zookeeper 有問題.....

當然直接連線至 kafka 查詢是最直接的也是最快速的，但永遠都是指令背不起來XD，常常是一開始開發或設定階段查完指令、問題解決後大概就沒再用過，使用率決定了記憶時間呀 @@"

## 基本環境說明
1. macOS Mojave 10.14.3
2. openjdk version "1.8.0_202"
3. OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_202-b08)
4. OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.202-b08, mixed mode)
3. kafka-manager-1.3.3.22

## 安裝步驟
1. 安裝 Java 1.8

    > 我自己用了 11 無法成功編譯，降至 8 才行

    ```bash
    brew cask install adoptopenjdk8
    ```
 
   - 可以透過 `/usr/libexec/java_home -V` 確認目前安裝的 java 版本
   - 透過 `export JAVA_HOME={上條指令查到的 java 位置}` 修改 java 版本

        ```bash
        export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
        ```

    - 使用 java 11 編譯時出現的錯誤資訊

        ```bash
        ./sbt: line 198: [[: .: syntax error: operand expected (error token is ".")
        Java HotSpot(TM) 64-Bit Server VM warning: Ignoring option MaxPermSize; support was removed in 8.0
        Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
        [info] Loading project definition from /Users/yowko.tsai/kafkatest/kafka-manager-1.3.3.22/project
        java.lang.NullPointerException
            at java.base/java.util.regex.Matcher.getTextLength(Matcher.java:1770)
            at java.base/java.util.regex.Matcher.reset(Matcher.java:416)
            at java.base/java.util.regex.Matcher.<init>(Matcher.java:253)
            at java.base/java.util.regex.Pattern.matcher(Pattern.java:1133)
            at java.base/java.util.regex.Pattern.split(Pattern.java:1261)
            at java.base/java.util.regex.Pattern.split(Pattern.java:1334)
            at sbt.IO$.pathSplit(IO.scala:744)
            at sbt.IO$.parseClasspath(IO.scala:859)
            at sbt.compiler.CompilerArguments.extClasspath(CompilerArguments.scala:62)
            at sbt.compiler.MixedAnalyzingCompiler$.withBootclasspath(MixedAnalyzingCompiler.scala:189)
            at sbt.compiler.MixedAnalyzingCompiler$.searchClasspathAndLookup(MixedAnalyzingCompiler.scala:167)
            at sbt.compiler.MixedAnalyzingCompiler$.apply(MixedAnalyzingCompiler.scala:177)
            at sbt.compiler.IC$.incrementalCompile(IncrementalCompiler.scala:138)
            at sbt.Compiler$.compile(Compiler.scala:128)
            at sbt.Compiler$.compile(Compiler.scala:114)
            at sbt.Defaults$.sbt$Defaults$$compileIncrementalTaskImpl(Defaults.scala:829)
            at sbt.Defaults$$anonfun$compileIncrementalTask$1.apply(Defaults.scala:820)
            at sbt.Defaults$$anonfun$compileIncrementalTask$1.apply(Defaults.scala:818)
            at scala.Function1$$anonfun$compose$1.apply(Function1.scala:47)
            at sbt.$tilde$greater$$anonfun$$u2219$1.apply(TypeFunctions.scala:40)
            at sbt.std.Transform$$anon$4.work(System.scala:63)
            at sbt.Execute$$anonfun$submit$1$$anonfun$apply$1.apply(Execute.scala:226)
            at sbt.Execute$$anonfun$submit$1$$anonfun$apply$1.apply(Execute.scala:226)
            at sbt.ErrorHandling$.wideConvert(ErrorHandling.scala:17)
            at sbt.Execute.work(Execute.scala:235)
            at sbt.Execute$$anonfun$submit$1.apply(Execute.scala:226)
            at sbt.Execute$$anonfun$submit$1.apply(Execute.scala:226)
            at sbt.ConcurrentRestrictions$$anon$4$$anonfun$1.apply(ConcurrentRestrictions.scala:159)
            at sbt.CompletionService$$anon$2.call(CompletionService.scala:28)
            at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
            at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
            at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
            at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
            at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
            at java.base/java.lang.Thread.run(Thread.java:834)
        [error] (compile:compileIncremental) java.lang.NullPointerException
        Project loading failed: (r)etry, (q)uit, (l)ast, or (i)gnore?
        ```

2. 至 [kafka-manager/releases](https://github.com/yahoo/kafka-manager/releases)下載檔案並解壓縮
3. 進到解壓後的路徑中

    ```bash
    cd kafka-manager-1.3.3.22
    ```

4. 使用 sbt 進行編譯

    > 將 kafka-manager 打包成可以進行部署的 zip 檔，儲存位置會在 `./target/univeral/kafka-manager-1.3.3.22.zip`

    ```bash
    ./sbt clean dist
    ```

    ![1createpackage](https://user-images.githubusercontent.com/3851540/54073296-76745400-42c0-11e9-83d3-27f1b7c3beef.png)

5. 解壓縮並進入目錄中

6. 修改設定 ： `./conf/application.conf`

    - kafka-manager.zkhosts：指定 zookeeper 位置
    - application.features : 功能開關
        - KMClusterManagerFeature - 允許 Kafka Manager 新增、修改、刪除 clsuter
        - KMTopicManagerFeature - 允許 Kafka Manager 新增、修改、刪除 topic
        - KMPreferredReplicaElectionFeature - 允許執行對 leader 進行重新 load balance
        - KMReassignPartitionsFeature - 允許產生 partition 分派及重新分派 partition
    - 啟用 jmx 的大型 cluster 可以考慮設定
        - kafka-manager.broker-view-thread-pool-size={3 * brokers 數量}
        - kafka-manager.broker-view-max-queue-size={3 *  所有 topic 的總 partition 數}
        - kafka-manager.broker-view-update-seconds={kafka-manager.broker-view-max-queue-size / (10 * brokers 數量) } 

7. 啟動 kafka-manager

    - 使用預設值啟動
    
        ```bash
        bin/kafka-manager
        ```
    
    - 指定 application.conf 位置並指定服務的 port 為 9090
        
        > 預設為 9000 

        ```bash
        bin/kafka-manager -Dconfig.file=/path/to/application.conf -Dhttp.port=9090
        ```

        ![2startservice](https://user-images.githubusercontent.com/3851540/54073297-76745400-42c0-11e9-815e-090117583938.png)

## 實際使用
1. 設定 cluster

    ![3addcluster](https://user-images.githubusercontent.com/3851540/54073298-76745400-42c0-11e9-8a4f-d68ba2dc524d.png)

2. 取得實際資料

    ![4listcluster](https://user-images.githubusercontent.com/3851540/54073299-76745400-42c0-11e9-84e9-0d79e4efb579.png)

    ![5data](https://user-images.githubusercontent.com/3851540/54073300-770cea80-42c0-11e9-8989-2e1f752f8618.png)

## 心得
kafka-manager 雖然宣稱使用 Java 8+ 但與同事都遇到非 Java 8 無法成功編譯的問題，依結果來看還是選用 Java 8 比較保險

扣除 Java 8 的問題，整體說來設定過程還算輕鬆寫意，加上如此一來就可以很容易地看出 Kafka 使用情況，安裝摸索的時間非常值得投資

# 參考資訊
1. [How do I install Java on Mac OSX allowing version switching?](https://stackoverflow.com/questions/52524112/how-do-i-install-java-on-mac-osx-allowing-version-switching) 
2. [yahoo/kafka-manager](https://github.com/yahoo/kafka-manager)