---
title: "安裝 Gatling"
date: 2022-04-12T00:30:00+08:00
lastmod: 2022-04-12T00:30:31+08:00
draft: false
tags: ["Benchmark","Tools"]
slug: "gatling-install"
---

## 安裝 Gatling

原本想要使用 Gatling 來進行 gRPC 的 load test，但光安裝 Gatling 就不是很理解，所以先了解並紀錄一下 Gatling 的基本安裝，我認為應該是我平常熟悉的開發語言跟工具不是 `Java`, `Kotlin` and `Scala` 在語法或是設定上都看不懂官網文件XD

關於 Gatling

- 先決條件：安裝 64bits OpenJDK LTS：`8`,`11`,`17`
    > 其他 JVM 像是 `JDK 12`, `client JVMs`, `32bits systems` or `OpenJ9` 都不支援
- Gatling 3.7 開始支援 `Java`, `Kotlin` and `Scala` 來撰寫測試腳本
    > 舊版本只能使用 `Scala`
- 不要使用 maven central 上的 `M` 版本
    > 僅供 內部 與 Gatling Enterprise 使用

## 基本環境說明

1. macOS Monterey 12.3
2. openjdk 17.0.2 2022-01-18 LTS
3. Apache Maven 3.8.5
4. Gradle 7.4.2
5. Sbt 1.6.2
6. Scala 2.12.15
7. Gatling 3.7.6
8. io.gatling:gatling-maven-plugin 4.1.5
9. io.gatling.gradle 3.7.6.2
10. io.gatling:gatling-sbt 4.1.5

## 安裝方式

- 先決條件：安裝 64bits OpenJDK LTS：`8`,`11`,`17`

    > 其他 JVM 像是 `JDK 12`, `client JVMs`, `32bits systems` or `OpenJ9` 都不支援

1. 至 [Open Source page](https://gatling.io/open-source/) 下載整合壓縮檔

    - 整合壓縮檔只支援 `Java` 跟 `Scala`
    - 整合包內容如下：
        - `bin`: `Gatling` 跟 `Recorder` 的啟動腳本
        - `conf`: `Gatling`, `Akka` 跟 `Logback` 的設定檔
        - `lib`: Gatling 跟相依套件
        - `user-files`:
            - `simulations`: 用來放測試情境程式的位置
            - `resources`: 非測試程式的其他資源
        - `results`: 測試結果

2. 使用 build tool plugin

    > 透過 build tool 搭配 gatling plugin 來執行測試

    - Maven：支援 `Java`, `Kotlin` and `Scala`

        - macOS 安排 Maven

            ```bash
            brew install maven
            ```

        - 詳細資訊請參考 [Gatling 官方文件:Maven Plugin](https://gatling.io/docs/gatling/reference/current/extensions/maven_plugin/)
        - 設定方式：在專案的 `pom.xml` 新增以下設定

            ```xml
            <dependencies>
              <dependency>
                <groupId>io.gatling.highcharts</groupId>
                <artifactId>gatling-charts-highcharts</artifactId>
                <version>MANUALLY_REPLACE_WITH_LATEST_VERSION</version>
                <scope>test</scope>
              </dependency>
            </dependencies>
            
            <plugin>
              <groupId>io.gatling</groupId>
              <artifactId>gatling-maven-plugin</artifactId>
              <version>MANUALLY_REPLACE_WITH_LATEST_VERSION</version>
            </plugin>
            ```

        - `gatling-charts-highcharts` 與 `gatling-maven-plugin` 版本可以到 [Maven Central](https://search.maven.org/search?q=g:io.gatling%20AND%20a:gatling-maven-plugin&core=gav) 查詢
        - 各語言的範例

            - Kotlin：[Gatling plugin for Maven - Kotlin demo project](https://github.com/gatling/gatling-maven-plugin-demo-kotlin)
            - Java：[Gatling plugin for Maven - Java demo project](https://github.com/gatling/gatling-maven-plugin-demo-java)
            - Scala：[Gatling plugin for Maven - Scala demo project](https://github.com/gatling/gatling-maven-plugin-demo-scala)

    - Gradle：支援 `Java`, `Kotlin` and `Scala`

        - macOS 安裝 Gradle

            ```bash
            brew install gradle
            ```

        - 詳細資訊請參考 [Gatling 官方文件:Gradle Plugin](https://gatling.io/docs/gatling/reference/current/extensions/gradle_plugin/)
        - 設定方式：在專案的 `build.gradle` 新增以下設定

            ```grandle
            plugins {
              id 'io.gatling.gradle' version "MANUALLY_REPLACE_WITH_LATEST_VERSION"
            }
            ```

        - `io.gatling.gradle` 的版本可以至 [Gradle Plugins Portal](https://plugins.gradle.org/plugin/io.gatling.gradle) 查詢
        - 各語言範例

            - Kotlin：[Gatling plugin for Gradle - Kotlin demo project](https://github.com/gatling/gatling-gradle-plugin-demo-kotlin)
            - Java：[Gatling plugin for Gradle - Java demo project](https://github.com/gatling/gatling-gradle-plugin-demo-java)
            - Scala：[Gatling plugin for Gradle - Scala demo project](https://github.com/gatling/gatling-gradle-plugin-demo-scala)

    - Sbt：僅支援 `Scala`

        - 詳細資訊請參考 [Gatling 官方文件:SBT Plugin](https://gatling.io/docs/gatling/reference/current/extensions/sbt_plugin/)
        - 設定方式：

            - macOS 安裝 sbt

                ```bash
                brew install sbt
                ```

            - 在專案的 `project/plugins.sbt` 新增相依套件

                ```sbt
                addSbtPlugin("io.gatling" % "gatling-sbt" % "MANUALLY_REPLACE_WITH_LATEST_VERSION")
                ```

            - 在專案的 `build.sbt` 新增相依套件並啟用

                ```sbt
                enablePlugins(GatlingPlugin)
                libraryDependencies += "io.gatling.highcharts" % "gatling-charts-highcharts" % "MANUALLY_REPLACE_WITH_LATEST_VERSION" % "test"
                libraryDependencies += "io.gatling"            % "gatling-test-framework"    % "MANUALLY_REPLACE_WITH_LATEST_VERSION" % "test"
                ```

            - `gatling-charts-highcharts`,`gatling-test-framework`,`gatling-sbt` 版本可以至 [Maven Central](https://search.maven.org/artifact/io.gatling/gatling-sbt) 查詢

            - 使用範例：[Gatling plugin for SBT - Scala demo project](https://github.com/gatling/gatling-sbt-plugin-demo)

3. 使用 IDE

    - IntelliJ IDEA

        > 原生就已支援 `Java`, `Kotlin`, `maven` and `gradle`，只有 `Scala` 需要額外安裝套件

    - Visual Studio Code

        > 安裝語言的相應套件可以正確 build 專案，但無法如同官網說明的單執行 `Engine` 會出現 build error

        ![1document](https://user-images.githubusercontent.com/3851540/163741940-63ef3fcb-8397-49ea-98a0-f6adf1e43d46.png)

        ![2builderror](https://user-images.githubusercontent.com/3851540/163741947-36f5e647-8744-48b6-a2b5-0ccb6e469aea.png)

## 心得

Gatling 提供的安裝方式好幾個，不過對於非原生 `Java`, `Kotlin` and `Scala` 的使用者並不友善，像是 build tool 的整合，以我一個其他語言的開發人員只想做 load test 還得要去了解 `Maven` `Gradle` `Sbt` 可以想見一定是障礙重重；以 IDE 的整合而言，IntelliJ IDEA 使用上沒問題，但要非 `Java`, `Kotlin` 開發人員只為了 load test 安裝，我個人覺得太材小用了，而常見的 Visual Studio Code 卻沒有更完整的設定說明要重新了解整個開發跟工具的生態圈，進入門檻過高，實在可惜

## 參考資訊

1. [Gatling:Installation](https://gatling.io/docs/gatling/tutorials/installation/)
2. [How to install Maven on macOS](https://mkyong.com/maven/install-maven-on-mac-osx/)
3. [Gatling:Maven Plugin](https://gatling.io/docs/gatling/reference/current/extensions/maven_plugin/)
4. [Grandle:Install](https://gradle.org/install/)
5. [Gatling:Gradle Plugin](https://gatling.io/docs/gatling/reference/current/extensions/gradle_plugin/)
6. [sbt:Install](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Mac.html)
7. [Gatling:SBT Plugin](https://gatling.io/docs/gatling/reference/current/extensions/sbt_plugin/)
