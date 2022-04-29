---
title: "使用 ghz 來對 gRPC 做負載測試"
date: 2022-04-04T00:30:00+08:00
lastmod: 2022-04-04T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","gRPC","Benchmark"]
slug: "grpc-load-test-ghz"
---

## 使用 ghz 來對 gRPC 做負載測試

最近興起想要比較幾個 gRPC load test 工具的使用心得，這才發現過去在建立 gRPC service 時因為專案時間壓力並沒有特別紀錄 ghz 的用法，後來工作也慢慢以 DevOps 為主，剛好趁這個機會順便複習一下

ghz 是一套用 `go` 撰寫的 command line 工具，主要是用於進行 load test 與 benchmark gRPC 服務

## 基本環境說明

1. macOS Monterey 12.3.1
2. ghz v0.106.1
3. ASP.NET Core gRPC server default project template

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

4. streaming 部份參考之前筆記 [C# 搭配 gRPC 中使用 stream RPC](/csharp-grpc-stream/)

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

> 擇一即可

1. 下載

    可以至 [ghz:GitHub releases page](https://github.com/bojand/ghz/releases) 下載所需環境 os 的壓縮檔後解壓縮使用
2. mac 可以使用 hoembrew 安裝

    ```bash
    brew install ghz
    ```

## 使用方式

1. 語法

    ```BASH
    ghz [<flags>] [<host>]
    ```

2. 範例

    ```bash
    ghz --insecure --proto ./grpc6/Protos/greet.proto --call greet.Greeter.SayHello -d '{"name":"Yowko"}' 0.0.0.0:5143
    ```

3. `<flags>` 參數說明

    - `--config`

        > - 指定 `JSON` 或是 `TOML` 格式的檔案位置
        > - config 檔案的設定值會與 cli 參數整併且 cli 參數優先權較高 (cli 參數會覆寫 config 檔案設定值)

    - `--proto`

        > - 指定定義 service 的 `.proto` 檔案位置
        > - 若未指定 `--proto` 與 `--protoset` 則會嘗試透過 server reflection 來取得可用 service

    - `--protoset`

        > - 指定編譯過的 `.protoset` 檔案位置
        > - 若未指定 `--proto` 與 `--protoset` 則會嘗試透過 server reflection 來取得可用 service

    - `--call` ?

        > - 呼叫完全符合規則的方法：格式為 `package.Service/Method` or `package.Service.Method`

    - `-i`, `--import-paths`

        > - 用來指定 import proto 的路徑
        > - 可以使用 `,` 來指定多組路徑
        > - 目前所在資料夾與使用 `--proto` 指定的 proto 檔所在資料夾都會自動加入 import 清單中

    - `--cacert`

        > - 指定可用來驗證 server 的可信任根憑證位置
        > - ghz 預設會使用系統的預設根憑證來建立安全連線
        > - 加上 `--skipTLS` 可以略過 TLS 檢查

    - `--cert`

        > - 指定 client 憑證 (公鑰) 的位置
        > - 必需同時使用 `--key`

    - `--key`

        > - 指定 client 私鑰的位置
        > - 必需同時使用 `--cert`

    - `--cname`

        > - 驗證 TLS 憑證時覆寫 server 名稱

    - `--skipTLS`

        > - 略過 server 憑證鏈與主機名稱的 TLS client 驗證

    - `--insecure`

        > - 使用明碼與不安全連線

    - `--authority`

        > - 會將值做為 `:authority` pseudo-header
        > - 只在使用 `--insecure` 有用

    - `--async`

        > - 儘快使 request 做非同步

    - `-r`, `--rps`

        > - 所有 RPS (requests per second) 的速率限制
        > - 預設值為 `0` (無限制)
        > - 所有 RPS 會依照指定的 concurrency 選項分送給 worker

    - `--load-schedule`

        > - 指定 load test 排程模式
        > - 允許設定值：`const`、`step` 與 `line`
        > - 預設值為：`const`
        > - 使用 `const` 時以 `q` 選項來指定 RPS (我看說明文件沒有關係 `q` option 的內容)
        > - 使用 `step` 時以 `load-start`, `load-step`, `load-end`, `load-step-duration`, 與 `load-max-duration` 來決定階段性增減的細節
        > - 使用 `line` 時以 `load-start`, `load-step`, `load-end`, `load-step-duration`, 與 `load-max-duration` 來線性增減，但 `line` 就是線性增減，原則上就是透過 `load-step` 指定斜率且 `load-step-duration` 就是 `1s`

    - `--load-start`

        > - 指定 load test 開始的 RPS
        > - 適用於 `step` 或 `line` 模式

    - `--load-step`

        > - 在 `step` 模式下指定 load test 遞增的 RPS 值
        > - 在 `line` 模式下指定 load test 斜率

    - `--load-end`

        > - 指定 load test 終止的 RPS (可不提供)
        > - 適用於 `step` 或 `line` 模式
        > - `load-end` 與 `load-max-duration` 以先達成的優先

    - `--load-max-duration`

        > - 指定 load test 最長持續時間(可不提供)
        > - 以 `--load-end` 的 RPS 數量執行這個時間設定的 load test
        > - `load-end` 與 `load-max-duration` 以先達成的優先

    - `-c`, `--concurrency`

        > - 指定同時執行的 worker 數
        > - 適用於 `const` 並發排程模式

    - `--concurrency-schedule`

        > - 指定 load test 並發排程模式
        > - 允許設定值：`const`、`step` 與 `line`
        > - 預設值為：`const`

    - `--concurrency-start`

        > - 指定 load test 開始的並發 RPS
        > - 適用於 `step` 或 `line` 並發模式

    - `--concurrency-end`

        > - 指定 load test 終止的並發 RPS
        > - 適用於 `step` 或 `line` 並發模式

    - `--concurrency-step=1`

        > - 在 `step` 並發模式下指定 load test 遞增的 RPS 值
        > - 在 `line` 並發模式下指定 load test 斜率

    - `--concurrency-step-duration`

        > - 指定 load test 並發 step 持續時間
        > - 適用於 `step` 並發模式

    - `--concurrency-max-duration`

        > - 指定 load test 並發最長持續時間
        > - 適用於 `step` 或 `line` 並發模式

    - `-n`, `--total`

        > - 指定總測試的 request 數量
        > - 預設為 `200`
        > - benchmark 主要就是 `-n` 挑配 `-c` 取得的

    - `-t`, `--timeout`

        > - 指定每個 request timeout 的時間
        > - 預設為 `20s`
        > - `0` 表示永不 timeout

    - `-z`, `--duration`

        > - 指定發出 request 的持續時間
        > - 如果設定這個值，`-n` 的設定就會被忽略

    - `-x`, `--max-duration`

        > - 不忽略 `-n` 設定的最長執行時間
        > - 如果在 `-n` (最大測試總量) 完成前，達成這個設求，程式依然會停止

    - `--duration-stop`

        > - 指定 `-z` (duration) 已滿足的情況下，如何處理已發送但仍未完成的 request
        > - 允許設定值：`close`、`wait` 與 `ignore`
        > - `close`：立即關閉連線，可能會出現錯誤：`transport is closing`
        > - `wait`：會等待 request 完成，但仍需遵守 `-t` 設定
        > - `ignore`：會立即關閉連線，但報告不會將這些 request 計入

    - `-d`, `--data`

        > - 以 stringified JSON 做為呼叫的資料
        > - 如果 value 中有 `@` 則以 `stdin` 讀取內容
        > - 一元請求(unary requests) 允許 `一個` (重複用) 或是 `陣列` (循環用) 訊息
        > - client streaming or bi-directional streaming 則僅允許 json 陣列 (就算只提供一個 json 也會轉為只有一個元素的陣列)

    - `-D`, `--data-file`

        > - 提供呼叫資料的 json 路徑

    - `-b`, `--binary`

        > - 呼叫資料可以從 `stdin` 讀取並序列化為 protocol buffer 訊息

    - `-B`, `--binary-file`

        > - 指定序列化過的呼叫資料檔案路徑

    - `-m`, `--metadata`

        > - stringified JSON 的 request metadat

    - `-M`, `--metadata-file`

        > - metadata 的檔案路徑

    - `--stream-interval`

        > - streaming 間隔時間
        > - 僅適用於 `client streaming` or `bi-directional streaming`

    - `--stream-call-duration`

        > - 最大的 stream 呼叫持續時間
        > - `client streaming` or `bi-directional streaming` 會持續發送至時間到
        > - `server streaming` 會持續接受到時間到 (會出現 canceled error)

    - `--stream-call-count`

        > - 設定 streaming 最大的發送訊息數量 (`client streaming` or `bi-directional streaming`)或接收訊數量 (`server streaming`)
        > - 若 call data 大於指定數量，只會取部份使用；若資料小於指定數量則會重複使用

    - `--stream-dynamic-messages`

        > - 允許重態建立 streaming 訊息內容

    - `--reflect-metadata`

        > - 將 metadata 反射為 stringified JSON
        > - 僅用於 reflection 的 request

    - `--max-recv-message-size`

        > - 指定最大允許 client 接收的訊息大小
        > - 可以使用 `bytes` 設定，或是 `42 MB` 這類 [human readable value](https://pkg.go.dev/github.com/dustin/go-humanize#ParseBytes)

    - `--max-send-message-size`

        > - 指定最大允許 client 傳送的訊息大小
        > - 可以使用 `bytes` 設定，或是 `42 MB` 這類 [human readable value](https://pkg.go.dev/github.com/dustin/go-humanize#ParseBytes)

    - `-o`, `--output`

        > - 指定輸出的檔案位置
        > - 預設是 `none`，即透過 `stdout` 輸出

    - `-O`, `--format`

        > - 設定輸出格式，如果未設定只會顯示概要
        > - `csv` : 將指標以逗號分隔呈現
        > - `json` : 將指標以 json 格式呈現
        > - `pretty` : 將指標以利於閱讀的 json 格式呈現
        > - `html` : 將指標以 htlm 呈現
        > - `influx-summary` : 將指標概要以 InfluxDB line protocol 呈現
        > - `influx-details`  : 將指標細節以 InfluxDB line protocol 呈現
        > - `prometheus` : 將指標概要以  Prometheus exposition 呈現.

    - `--skipFirst`

        > - 設定忽略頭幾個 response

    - `--connections`

        > - 設定 gRPC 連線數 (預設只有建立一條 gRPC 連線)
        > - 這個設定值不能大於 `-c` 數量
        > - 連線數會分圴分配給 worker 使用

    - `--connect-timeout`

        > - 設定初始化連線 timeout 時間
        > - 預設為 `20s`

    - `--keepalive`

        > - 設定 keepalive 時間
        > - 只在明確設定且數值大於 `0` 會套用

    - `--name`

        > - 自訂 load test 名稱

    - `--tags`

        > - 自訂的 tag
        > - 以 `json` string 方式設定

    - `--cpus`

        > - 指定用於執行 load test 的 cpu 數量
        > - 預設是本機的所有邏輯 cpu 總數

    - `--debug`

        > - 指定 debug log file 的路徑
        > - debug log 是 `json` 格式

    - `-e`, `--enable-compression`

        > - 在 request 上啟用 gzip 壓縮

    - `--count-errors`

        > - 啟用統計 error
        > - 預設 `最快`,`最慢`, `平均`, `直方圖`, `延遲分佈` 都只計算正確回應

    - `-v`, `--version`

        > - 顯示版本

    - `-h`, `--help`

        > - 顯示 help 文件
        > - 也可以使用 `--help-long` 與 `--help-man`

## 實際使用方式

1. Unary call

    ```bash
    ghz -c 20 -n 20000 --insecure --proto ./grpc6/Protos/greet.proto --call greet.Greeter.SayHello -d '{"name":"Yowko"}' 0.0.0.0:5143
    ```

    ![1unarycall](https://user-images.githubusercontent.com/3851540/161880801-a240df0f-6534-4711-8004-f56ce7b1d9be.png)

2. client streaming

    ```bash
    ghz -c 20 -n 20000 --insecure --proto ./grpc6/Protos/greet.proto --call greet.CandidateService.CreateCv -d '[{ "Name": "Yowko", "Jobs": [{"Title": "SRE","Salary": 10,"JobDescription": "DevOps"},{"Title": "Programmer","Salary": 5,"JobDescription": "C#"}] }, { "Name": "Test", "Jobs": [{"Title": "SRE","Salary": 8,"JobDescription": "DevOps"},{"Title": "Programmer","Salary": 7,"JobDescription": "golang"}] }]' 0.0.0.0:5143
    ```

    ![2clientstream](https://user-images.githubusercontent.com/3851540/161880810-d32f1038-9723-4ed1-b201-b2c1d7cebd85.png)

3. server streaming

    ```bash
    ghz -c 20 -n 20000 --insecure --proto ./grpc6/Protos/greet.proto --call greet.CandidateService.DownloadCv -d '{ "Name": "Yowko" }' 0.0.0.0:5143
    ```

    ![3serverstream](https://user-images.githubusercontent.com/3851540/161880815-e8f2e545-58a5-4218-8dd0-fbb676684746.png)

4. bi-directional streaming

    ```bash
    ghz -c 20 -n 20000 --insecure --proto ./grpc6/Protos/greet.proto --call greet.CandidateService.CreateDownloadCv -d '[{ "Name": "Yowko", "Jobs": [{"Title": "SRE","Salary": 10,"JobDescription": "DevOps"},{"Title": "Programmer","Salary": 5,"JobDescription": "C#"}] }, { "Name": "Test", "Jobs": [{"Title": "SRE","Salary": 8,"JobDescription": "DevOps"},{"Title": "Programmer","Salary": 7,"JobDescription": "golang"}] }]' 0.0.0.0:5143
    ```

    ![4bidirection](https://user-images.githubusercontent.com/3851540/161880816-a01334f4-259c-440f-879a-7d63e110cde9.png)

## 心得

ghz 是我最早接觸的 gRPC load test 工具，一直以來都是用它來為 gRPC service 做基本的效能評估，以我的立場來看，優點是功能多可以滿足多數情境，缺點是功能多導致參數也多，官網的 sample 又偏向簡單，不少參數不好理解實際應用情境，另外 grpc service 的呼叫格式與其他工具(grpcurl、ghz)略有不同 (`{package name}.{service name}.{procedure name}` vs `{package name}.{service name}/{procedure name}`)

## 參考資訊

1. [ghz:gRPC benchmarking and load testing tool](https://ghz.sh/)
2. [ghz:GitHub releases page](https://github.com/bojand/ghz/releases)
3. [ghz: Options Reference](https://ghz.sh/docs/options)
