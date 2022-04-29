---
title: "使用 Gatling 來對 gRPC 做負載測試"
date: 2022-04-28T00:30:00+08:00
lastmod: 2022-04-28T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","gRPC","Benchmark"]
slug: "grpc-load-test-gatling"
---

## 使用 Gatling 來對 gRPC 做負載測試

關於 Gatling

- 先決條件：安裝 64bits OpenJDK LTS：`8`,`11`,`17`
    > 其他 JVM 像是 `JDK 12`, `client JVMs`, `32bits systems` or `OpenJ9` 都不支援
- Gatling 3.7 開始支援 `Java`, `Kotlin` and `Scala` 來撰寫測試腳本
    > 舊版本只能使用 `Scala`
- 不要使用 maven central 上的 `M` 版本
    > 僅供 內部 與 Gatling Enterprise 使用

## 基本環境說明

1. macOS Monterey 12.3
2. ~~openjdk 17.0.2 2022-01-18 LTS~~ openjdk 11.0.14.1 2022-02-08

    ```txt
    java.lang.UnsupportedOperationException: The Security Manager is deprecated and will be removed in a future release
        at java.base/java.lang.System.setSecurityManager(System.java:416)
        at sbt.TrapExit$.installManager(TrapExit.scala:54)
        at sbt.StandardMain$.runManaged(Main.scala:184)
        at sbt.xMain$.$anonfun$run$6(Main.scala:100)
        at scala.util.DynamicVariable.withValue(DynamicVariable.scala:62)
        at scala.Console$.withIn(Console.scala:230)
        at sbt.internal.util.Terminal$.withIn(Terminal.scala:553)
        at sbt.internal.util.Terminal$.$anonfun$withStreams$1(Terminal.scala:343)
        at scala.util.DynamicVariable.withValue(DynamicVariable.scala:62)
        at scala.Console$.withOut(Console.scala:167)
        at sbt.internal.util.Terminal$.$anonfun$withOut$2(Terminal.scala:543)
        at scala.util.DynamicVariable.withValue(DynamicVariable.scala:62)
        at scala.Console$.withErr(Console.scala:196)
        at sbt.internal.util.Terminal$.withOut(Terminal.scala:543)
        at sbt.internal.util.Terminal$.withStreams(Terminal.scala:343)
        at sbt.xMain$.run(Main.scala:83)
        at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:104)
        at java.base/java.lang.reflect.Method.invoke(Method.java:577)
        at sbt.internal.XMainConfiguration.run(XMainConfiguration.scala:83)
        at sbt.xMain.run(Main.scala:46)
        at xsbt.boot.Launch$.$anonfun$run$1(Launch.scala:149)
        at xsbt.boot.Launch$.withContextLoader(Launch.scala:176)
        at xsbt.boot.Launch$.run(Launch.scala:149)
        at xsbt.boot.Launch$.$anonfun$apply$1(Launch.scala:44)
        at xsbt.boot.Launch$.launch(Launch.scala:159)
        at xsbt.boot.Launch$.apply(Launch.scala:44)
        at xsbt.boot.Launch$.apply(Launch.scala:21)
        at xsbt.boot.Boot$.runImpl(Boot.scala:78)
        at xsbt.boot.Boot$.run(Boot.scala:73)
        at xsbt.boot.Boot$.main(Boot.scala:21)
        at xsbt.boot.Boot.main(Boot.scala)
    [error] [launcher] error during sbt launcher: java.lang.UnsupportedOperationException: The Security Manager is deprecated and will be removed in a future release
    ```

3. Sbt 1.6.2

    > 實際上還會看專案設定

4. Scala 2.12.15
5. Gatling 3.7.6
6. gatling-sbt
7. ASP.NET Core gRPC server default project template

    - greet.proto

        ```proto
        syntax = "proto3";
        option csharp_namespace = "grpc6";
        
        package greet;
        
        // The greeting service definition.
        service Greeter {
          // Sends a greeting
          rpc SayHello (HelloRequest) returns (HelloReply);
        }
        
        // The request message containing the user's name.
        message HelloRequest {
          string name = 1;
        }
        
        // The response message containing the greetings.
        message HelloReply {
          string message = 1;
        }
        
        ```

8. streaming 部份參考之前筆記 [C# 搭配 gRPC 中使用 stream RPC](/csharp-grpc-stream/)

    - greet.proto

        ```proto
        service CandidateService {
          rpc DownloadCv (DownloadByName) returns (stream Candidate);
          rpc CreateCv (stream Candidate) returns (CreateCvResponse);
          rpc CreateDownloadCv (stream Candidate) returns (stream Candidates);
        }
        
        message Candidates {
            repeated Candidate Candidates = 2;
        }
        message Candidate {
            string Name = 1;
            repeated Job Jobs = 2;
        }
        message Job {
            string Title = 1;
            int32 Salary = 2;
            string JobDescription = 3;
        }
        message DownloadByName {
            string Name = 1;
        }
        message CreateCvResponse {
            bool IsSuccess = 1;
        }
        ```

    - CandidateService

        ```cs
        public class CandidateService:grpc6.CandidateService.CandidateServiceBase
        {
            public override async Task<CreateCvResponse> CreateCv(IAsyncStreamReader<Candidate> requestStream, ServerCallContext context)
            {
                var result = new CreateCvResponse
                {
                    IsSuccess = false
                };
                // stream 讀取
                while (await requestStream.MoveNext())
                {
                    var candidate = requestStream.Current;
                    // 實際處理
                    Console.WriteLine(candidate.Name);
                }
                return result;
            }
        
        
            public override async Task DownloadCv(DownloadByName request, IServerStreamWriter<Candidate> responseStream, ServerCallContext context)
            {
                var fakeJobs = new Faker<Job>()
                    .RuleFor(a => a.Title, (f, u) => f.Company.Bs())
                    .RuleFor(a => a.Salary, (f, u) => f.Commerce.Random.Int(1000, 2000))
                    .RuleFor(a => a.JobDescription, (f, u) => f.Lorem.Text());
                var createRequests = new Faker<Candidate>()
                    .RuleFor(a => a.Name, (f, u) => f.Name.FullName())
                    .RuleFor(a => a.Jobs, (f, u) =>
                    {
                        u.Jobs.AddRange(fakeJobs.GenerateBetween(3, 5));
                        return u.Jobs;
                    }).Generate();
                // 將每筆資料逐一透過 WriteAsync 輸出
                await responseStream.WriteAsync(createRequests);
            }
        
            public override async Task CreateDownloadCv(IAsyncStreamReader<Candidate> requestStream, IServerStreamWriter<Candidates> responseStream, ServerCallContext context)
            {
                var candidates = new Candidates();
                // 將收到的資料逐一取出
                while (await requestStream.MoveNext())
                {
                    var candidate = requestStream.Current;
                    candidates.Candidates_.Add(candidate);
                    // 將處理後的資料回傳
                    await responseStream.WriteAsync(candidates);
                }
            }
        }
        ```

## 下載與安裝

1. 安裝 64bits OpenJDK LTS：`8` or `11` or `17`

    > - 其他 JVM 像是 `JDK 12`, `client JVMs`, `32bits systems` or `OpenJ9` 都不支援
    > - 我在使用 `OpenJDK 17` 參考幾個網路上的範例都會出現 `error during sbt launcher: java.lang.UnsupportedOperationException: The Security Manager is deprecated and will be removed in a future release` 我個人是沒有能力解決，所以我選擇退到 `OpenJDK 11`

2. 安裝 sbt

    ```bash
    brew install sbt
    ```

3. 專案設定

    > 這個部份我卡很久，嘗試一陣子搞不定後就 fork 其他專案來修改

    我參考的是 Gatling-gRPC 的作者所建立的 example project [GitHub:phiSgr/gatling-grpc-gradle-demo](https://github.com/phiSgr/gatling-grpc-gradle-demo)，他作者有寫篇文章在 [A Demo of Gatling-gRPC](https://medium.com/@georgeleung_7777/a-demo-of-gatling-grpc-bc92158ca808)，另外部份內容參考 [Load testing for gRPC - the case](https://alexromanov.github.io/2021/08/23/gatling-grpc/)，專案範本在 [GitHub:alexromanov/gatling-grpc-tests-sample](https://github.com/alexromanov/gatling-grpc-tests-sample)

4. 調整 `.proto`

    > proto 檔案位置在 `src/main/protobuf`

    需要加上下面這行，原來是 scala 會這個設定來做為 import service 與參數型態的 package 名稱

    ```proto
    option java_package = "demo.yowko.helloworld.grpc";
    ```

## 實際使用

> 需要留意 import 的 proto service 與參數定義格式：`[java_package].[proto_file_name].[{需要的內容} or _]`

1. Unary call

    ```scala
    package load

    import com.github.phisgr.gatling.grpc.Predef._
    import io.gatling.core.Predef._
    import demo.yowko.helloworld.grpc.greet.{GreeterGrpc,HelloRequest}
    import io.grpc.Status
    import scala.concurrent.duration._

    class UnarySimulation extends Simulation {

      val HOST: String = "127.0.0.1"
      val PORT = 5143
      val grpcConf = grpc(managedChannelBuilder(name = HOST, port = PORT).usePlaintext())
      
      val request = grpc("request_1")
        .rpc(GreeterGrpc.METHOD_SAY_HELLO)
        .payload(HelloRequest("Gatling Load Test"))
        .extract(_.message.some)(_ saveAs "message")
        .check(statusCode is Status.Code.OK)

      val scn = scenario("Scenario Name")
        .exec(request)
        .pause(7.seconds)
        .exec(request)

      setUp(scn.inject(rampUsersPerSec(1) to (2) during (20 seconds)).protocols(grpcConf.shareChannel))
    }
    ```

2. client streaming

    > 這個我覺得有缺陷，執行測試完成後不會停止專案，而是等待，造成卡住後面的測試，我不知道怎麼解決

    ```scala
    package load
    
    import com.github.phisgr.gatling.grpc.Predef._
    import io.gatling.core.Predef._
    import demo.yowko.helloworld.grpc.greet._ //{GreeterGrpc,Candidate}
    import io.grpc.{CallOptions, Status}
    import scala.concurrent.duration._

    class ClientStreamingSimulation extends Simulation {

      val HOST: String = "127.0.0.1"
      val PORT = 5143
      val grpcConf = grpc(managedChannelBuilder(name = HOST, port = PORT).usePlaintext())
    
      val clientStream = grpc("client").clientStream("client")

      var z = new Array[Job](2)
      z(0) = new Job("it",1,"test")
      z(1)= new Job("hr",2,"test hr")
    
      var candidate = new Candidate("yowko",z)
      val speaker = scenario("Client streaming")
        .exec(
          clientStream.connect(CandidateServiceGrpc.METHOD_CREATE_CV)
          .check(statusCode is Status.Code.OK)
        )
        .exec(clientStream.send(candidate))
        .exec(clientStream.copy(requestName = "Wrong Type").send("wrong type"))
        .exec(clientStream.completeAndWait)
        .exec(
          clientStream.connect(CandidateServiceGrpc.METHOD_CREATE_CV)
            .callOptions(CallOptions.DEFAULT.withDeadlineAfter(500, MILLISECONDS))
            .check(statusCode is Status.Code.CANCELLED)
        )

      setUp(speaker.inject(rampUsersPerSec(1) to (2) during (20 seconds)))
        .protocols(grpcConf)
    }
    ```

3. server streaming

    ```scala
    package load
    
    import com.github.phisgr.gatling.generic.SessionCombiner
    import com.github.phisgr.gatling.grpc.Predef._
    import demo.yowko.helloworld.grpc.greet.{CandidateServiceGrpc,DownloadByName}
    import io.gatling.core.Predef.{stringToExpression => _, _}
    import io.grpc.Status
    import scala.concurrent.duration.DurationInt
    import scala.language.postfixOps
    
    class ServerStreamingSimulation extends Simulation {
    
    val HOST: String = "127.0.0.1"
      val PORT = 5143
      val grpcConf = grpc(managedChannelBuilder(name = HOST, port = PORT).usePlaintext())
      val serverCall = grpc("Replying").serverStream("replier")
    
      val scn = scenario("Server Streaming Flow")
        .exec(serverCall.start(CandidateServiceGrpc.METHOD_DOWNLOAD_CV)
        (DownloadByName("yowkotest"))
          .extract(_.name.some)(_ saveAs "ServerReply")
          .sessionCombiner(SessionCombiner.pick("ServerReply"))
          .endCheck(statusCode is Status.Code.OK)
        )
    
      setUp(scn.inject(rampUsersPerSec(1) to (2) during (20 seconds)).protocols(grpcConf.shareChannel))
    }
    ```
  
4. bi-directional streaming

    ```scala
    package load

    import com.github.phisgr.gatling.grpc.Predef._
    import demo.yowko.helloworld.grpc.greet._
    import io.gatling.core.Predef.{stringToExpression => _, _}
    import io.grpc.Status
    import scala.concurrent.duration.DurationInt
    import scala.language.postfixOps

    class BiDiStreamingSimulation extends Simulation {

      val HOST: String = "127.0.0.1"
      val PORT = 5143
      val grpcConf = grpc(managedChannelBuilder(name = HOST, port = PORT).usePlaintext())
    
      var z = new Array[Job](2)
      z(0) = new Job("it",1,"test")
      z(1)= new Job("hr",2,"test hr")

      var candidate = new Candidate("yowko",z)


      val bidiCall = grpc("BiDi call").bidiStream("bidi")

      val scn = scenario("BiDi streaming")
        .exec(bidiCall.connect(CandidateServiceGrpc.METHOD_CREATE_DOWNLOAD_CV)
        .endCheck(statusCode is Status.Code.OK))
        .exec(bidiCall.send(candidate))
        .exec(bidiCall.complete)

      setUp(scn.inject(rampUsersPerSec(1) to (2) during (20 seconds)).protocols(grpcConf.shareChannel))
    }
    ```

5. 執行測試

    > 第一次執行或許需要先執行一次 `sbt compile` (我不確定)

    ```bash
    sbt gatling-it:test
    ```

## 心得

1. scala 開發經驗不足

   > 因為沒有使用 scala 開發，更沒有 sbt 設定經驗，所以在套件安裝、 project 規劃、測試腳本的語法 甚至連參考資源的 import 都一直卡關

2. 套件相依很侷限

    > 原本想要用新版套件 (e.g. OpebJDK、sbt...)，但一改版本就會出現各式各樣的錯誤而無法執行，所以就依照 example project [GitHub:phiSgr/gatling-grpc-gradle-demo](https://github.com/phiSgr/gatling-grpc-gradle-demo) 內容來調整成自己的 service

3. IDE 熟悉度不差

    > 不確定是不是因為我沒有 scala 或是 sbt 的經驗，造成我在使用 Visual Studio Code 或是 IntelliJ 時都無法提供有用的 debug 訊息，當然也沒有方法與參數的提示

4. 文件對新手不友善

    > 文件寫得很簡略，也許對 scala 開發人員已經足夠，但對新手來說是無法照著文件完成設定的

結論是以我一個非 scala 開發者的角色是不會再考慮使用 Gatling 來對 gRPC service 做負載測試，不僅要花太多時間來設定與撰寫測試腳本，還不確定測試的結果是否符合預期，如果是 scala 開發者也許會順手許多 像是 [Load testing for gRPC - the case](https://alexromanov.github.io/2021/08/23/gatling-grpc/)

## 參考資訊

1. [Gatling](https://gatling.io/)
2. [A Demo of Gatling-gRPC](https://medium.com/@georgeleung_7777/a-demo-of-gatling-grpc-bc92158ca808)
3. [GitHub:phiSgr/gatling-grpc-gradle-demo](https://github.com/phiSgr/gatling-grpc-gradle-demo)
4. [Load testing for gRPC - the case](https://alexromanov.github.io/2021/08/23/gatling-grpc/)
5. [GitHub:alexromanov/gatling-grpc-tests-sample](https://github.com/alexromanov/gatling-grpc-tests-sample)
