---
title: "使用 k6 來對 gRPC 做負載測試"
date: 2022-04-08T00:30:00+08:00
lastmod: 2022-04-08T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp","gRPC","Benchmark"]
slug: "grpc-load-test-k6"
---

## 使用 k6 來對 gRPC 做負載測試

第一次聽到 k6 是 twMVC 的活動宣傳 [讓我們用 k6 來進行壓測吧](https://mvc.tw/event/2022/2/19)，雖然後來時間因素沒有到場聽到實際應用的分享，但為了不要與技術潮流脫節還是自己找時間了解一下內容，其中第一眼的印象是使用 go 開發但以 js 做為測試腳本的語法，後來又看到 k6 支援 gRPC 的測試，當下就來興趣想做個簡單的體驗，順手紀錄一下用法

## 基本環境說明

1. macOS Monterey 12.3
2. k6 v0.37.0
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

1. 安裝 k6

    - k6 在多個 package manager 工具上都有上傳對應的 package

        - Debian/Ubuntu

            ```bash
            sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69 
            echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
            sudo apt-get update
            sudo apt-get install k6
            ```

        - Fedora/CentOS

            ```bash
            sudo dnf install https://dl.k6.io/rpm/repo.rpm
            sudo dnf install k6
            ```

        - Windows

            ```cmd
            choco install k6
            ```

            or

            ```cmd
            winget install k6
            ```

        - MacOS

            ```bash
            brew install k6
            ```

    - 可以直接下傳 os 對應的安裝檔

        - [GitHub : k6 Releases page](https://github.com/grafana/k6/releases)

## 使用語法與參數說明

1. 語法

    ```bash
    k6 [command] [flag]
    ```

2. 範例

    ```bash
    k6 
    ```

3. `command` 與 `flags` 說明

    - commad

        - `archive`：建立檔案

            > 建立一個可以獨立執行的完整測試檔案

            ```bash
            k6 archive [flags]
            ```

            - `-u`, `--vus int`：指定 virtual user 的數量 (預設為 `1`)
            - `-d`, `--duration duration`：load test 持續時間
            - `-i`, `--iterations int`：測試執行迭代次數 (以所有 virtual user 計算的總數)
            - `-s`, `--stage stage`：增加測試階段 (語法是 `[duration]:[target]`)
            - `--execution-segment string`：限定執行某個區段
            - `--execution-segment-sequence string`：指定執行區段的順序
            - `-p`, `--paused`：啟動 load test 但讓 load test 暫停
            - `--no-setup`：不要執行 `setup()`
            - `--no-teardown`：不要執行 `teardown()`
            - `--max-redirects int`：限定轉導次數 (預設 `10`)
            - `--batch int`：限定平行批量 request 數量 (預設 `20`)
            - `--batch-per-host int`：限定單一 host 平行批量 request 數量 (預設 `6`)
            - `--rps int`：限定每秒 request 數量
            - `--user-agent string`：指定 http request 的 user agent (預設 `k6/0.37.0 (https://k6.io/)`)
            - `--http-debug string[="headers"]`：紀錄所有 HTTP requests and responses. 預設排除 `body`可以透過 `--http-debug=full` 來加入 `body` 資訊
            - `--insecure-skip-tls-verify`：不檢查 TLS 憑證的有效性
            - `--no-connection-reuse`：不允許 `keep-alive` 的連線
            - `--no-vu-connection-reuse`：在不同的迭代間不重複使用連線
            - `--min-iteration-duration duration`：單一迭代最小的測試執行時間
            - `-w`, `--throw`：將 warning (http request 異常) 以 error 的方式拋出
            - `--blacklist-ip ip range`：將 ip range 加至 blacklist
            - `--block-hostnames pattern`： 阻斷特定的 hostname 呼叫，pattern 是不限大小寫且可有可無 wildcard 開頭的 hostname
            - `--summary-trend-stats stats`：指定想要顯示在概要的趨勢指標統計值
            - `--summary-time-unit string`：指定概要中的趨勢統計值顯示時間單位：`s`, `ms` 與 `us`
            - `--system-tags strings`：指定指標只包含的 tag (預設 `proto,subproto,status,method,url,name,group,check,error,error_code,tls_version,scenario,service,expected_response`)
            - `--tag tag`： 為所有 sample 加上 tag (格式為 `[name]=[value]`)
            - `--console-output string`：將 console log 資訊以檔案格式儲存
            - `--discard-response-bodies`：讀取但不處理或是儲存 HTTP response bodies
            - `--local-ips string`：指定每個 virtual user 可能發出 request 的 IP Ranges and/or CIDRs (e.g. '192.168.220.1,192.168.0.10-192.168.0.25', 'fd:1::0/120')
            - `--dns string`： DNS 解析設定.
                - ttl：`inf` 保存 cache；`0` 停用 cache (可以指定時間：`1s`, `1m`，如果沒有指定單位，會用 Milliseconds)
                - select：`first`, `random` or `roundRobin`.
                - policy：`preferIPv4`, `preferIPv6`, `onlyIPv4`, `onlyIPv6` or `any`(預設 `ttl=5m,select=random,policy=preferIPv4`)
            - `--include-system-env-vars`：是否將系統的 environment variables 傳給 k6 使用(預設 `true`)
            - `--compatibility-mode string`：JavaScript compiler 相容模式 `extended` or `base` (預設 `extended`)
                - `base`: pure goja - 支援 ES5.1+ 的 Golang JS VM
                - `extended`: base + 部份 ES2015 預設的 Babel (如果用到 base 不支援的語法， compile 速度對變慢)
            - `-e`, `--env VAR=value`：指定 environment variable
            - `--no-thresholds`：不要執行 thresholds
            - `--no-summary`：測試結束後不顯示概要
            - `--summary-export string`：將測試結果概要儲存至指定的 JSON 路徑
            - `-O, --archive-out string`：輸出檔案名稱(預設 `archive.tar`)
            - `-h`, `--help`：顯示 help 資訊

        - `cloud`： 在雲端執行測試

            > 將測試在 k6 的雲端服務上執行，使用 `"k6 login cloud` 來登入

            ```bash
            k6 cloud [flags]
            ```

            - `-u`, `--vus int`：指定 virtual user 的數量 (預設為 `1`)
            - `-d`, `--duration duration`：load test 持續時間
            - `-i`, `--iterations int`：測試執行迭代次數 (以所有 virtual user 計算的總數)
            - `-s`, `--stage stage`：增加測試階段 (語法是 `[duration]:[target]`)
            - `--execution-segment string`：限定執行某個區段
            - `--execution-segment-sequence string`：指定執行區段的順序
            - `-p`, `--paused`：啟動 load test 但讓 load test 暫停
            - `--no-setup`：不要執行 `setup()`
            - `--no-teardown`：不要執行 `teardown()`
            - `--max-redirects int`：限定轉導次數 (預設 `10`)
            - `--batch int`：限定平行批量 request 數量 (預設 `20`)
            - `--batch-per-host int`：限定單一 host 平行批量 request 數量 (預設 `6`)
            - `--rps int`：限定每秒 request 數量
            - `--user-agent string`：指定 http request 的 user agent (預設 `k6/0.37.0 (https://k6.io/)`)
            - `--http-debug string[="headers"]`：紀錄所有 HTTP requests and responses. 預設排除 `body`可以透過 `--http-debug=full` 來加入 `body` 資訊
            - `--insecure-skip-tls-verify`：不檢查 TLS 憑證的有效性
            - `--no-connection-reuse`：不允許 `keep-alive` 的連線
            - `--no-vu-connection-reuse`：在不同的迭代間不重複使用連線
            - `--min-iteration-duration duration`：單一迭代最小的測試執行時間
            - `-w`, `--throw`：將 warning (http request 異常) 以 error 的方式拋出
            - `--blacklist-ip ip range`：將 ip range 加至 blacklist
            - `--block-hostnames pattern`： 阻斷特定的 hostname 呼叫，pattern 是不限大小寫且可有可無 wildcard 開頭的 hostname
            - `--summary-trend-stats stats`：指定想要顯示在概要的趨勢指標統計值
            - `--summary-time-unit string`：指定概要中的趨勢統計值顯示時間單位：`s`, `ms` 與 `us`
            - `--system-tags strings`：指定指標只包含的 tag (預設 `proto,subproto,status,method,url,name,group,check,error,error_code,tls_version,scenario,service,expected_response`)
            - `--tag tag`： 為所有 sample 加上 tag (格式為 `[name]=[value]`)
            - `--console-output string`：將 console log 資訊以檔案格式儲存
            - `--discard-response-bodies`：讀取但不處理或是儲存 HTTP response bodies
            - `--local-ips string`：指定每個 virtual user 可能發出 request 的 IP Ranges and/or CIDRs (e.g. '192.168.220.1,192.168.0.10-192.168.0.25', 'fd:1::0/120')
            - `--dns string`： DNS 解析設定.
                - ttl：`inf` 保存 cache；`0` 停用 cache (可以指定時間：`1s`, `1m`，如果沒有指定單位，會用 Milliseconds)
                - select：`first`, `random` or `roundRobin`.
                - policy：`preferIPv4`, `preferIPv6`, `onlyIPv4`, `onlyIPv6` or `any`(預設 `ttl=5m,select=random,policy=preferIPv4`)
            - `--include-system-env-vars`：是否將系統的 environment variables 傳給 k6 使用(預設 `true`)
            - `--compatibility-mode string`：JavaScript compiler 相容模式 `extended` or `base` (預設 `extended`)
                - `base`: pure goja - 支援 ES5.1+ 的 Golang JS VM
                - `extended`: base + 部份 ES2015 預設的 Babel (如果用到 base 不支援的語法， compile 速度對變慢)
            - `-e`, `--env VAR=value`：指定 environment variable
            - `--no-thresholds`：不要執行 thresholds
            - `--no-summary`：測試結束後不顯示概要
            - `--summary-export string`：將測試結果概要儲存至指定的 JSON 路徑
            - `--exit-on-running`：當測試達到運行狀態時退出
            - `--show-logs`：在雲端測試時啟用顯示 log (預設 `true`)
            - `-h`, `--help`：顯示 help 資訊

        - `convert`：將 HAR 檔案轉成 k6 script

            > 將 HAR (HTTP Archive) 檔案轉成 k6 script

            ```bash
            k6 convert [flags]
            ```

            - `-O`, `--output string`：k6 script 輸出檔名 (預設 `stdout`)
            - `--options string`：將想要放進產出 k6 script 的參數以 JSON 檔案路徑提供
            - `--only strings`：僅包括來自預先提供 domain 的 request
            - `--skip strings`：忽略來自預先提供 domain 的 request
            - `--batch-threshold uint`：批量 request idle time threshold (預設 `500`)
            - `--no-batch`：不要產生批量呼叫
            - `--enable-status-code-checks`：為每個 HTTP response 加上狀態碼檢查
            - `--return-on-failed-check`：如果出現非預期的回應狀態碼在迭代中回傳
            - `--correlate`： 偵測在後續請求中回應的值並嘗試相應地調整腳本(目前僅限 redirects and JSON values)
            - `--min-sleep uint`：每次迭代後休眠的最小秒數 (預設 `20`)
            - `--max-sleep uint`：每次迭代後休眠的最大秒數 (預設 `40`)
            - `-h`, `--help`：顯示 help 資訊

        - `help`：顯示 Help

            > 提供程式中所有指令的 help

            ```bash
            k6 help [command] [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

        - `inspect`：檢查腳本或檔案

            ```bash
            k6 inspect [file] [flags]
            ```

            - `--system-tags strings`：指定指標只包含的 tag (預設 `proto,subproto,status,method,url,name,group,check,error,error_code,tls_version,scenario,service,expected_response`)
            - `--compatibility-mode string`：JavaScript compiler 相容模式 `extended` or `base` (預設 `extended`)
                - `base`: pure goja - 支援 ES5.1+ 的 Golang JS VM
                - `extended`: base + 部份 ES2015 預設的 Babel (如果用到 base 不支援的語法， compile 速度對變慢)
            - `-e`, `--env VAR=value`：指定 environment variable
            - `--no-thresholds`：不要執行 thresholds
            - `--no-summary`：測試結束後不顯示概要
            - `--summary-export string`：將測試結果概要儲存至指定的 JSON 路徑
            - `-t`, `--type type`：覆寫檔案格式，`js` or `archive`
            - `--execution-requirements`：包括測試執行要求的計算
            - `-h`, `--help`：顯示 help 資訊

        - `login`：服務的登入資訊

            > 當執行測試時使用 `-o [type=uri]` 但沒有傳參數時還是會修改到預設值，使用 login 可以覆寫原本儲存的資訊

            ```bash
            k6 login [flags]
            k6 login [command]
            ```

            - Commands
                - `cloud`：Load Impact 的認證資訊
                - `influxdb`：InfluxDB 的認證資訊
            - Flags
                - `-h`, `--help`：顯示 help 資訊

        - `pause`：暫停測試

            > 使用全域的 `--address` flag 來指定 api server url

            ```bash
            k6 pause [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

        - `resume`：恢復測試

            > 使用全域的 `--address` flag 來指定 api server url

            ```bash
            k6 resume [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

        - `run`: 執行 load test

            > 執行測試並且會啟動一個 REST API 來與測試互動，許多的 K6 子命令提供命令列介面來進行互動

            ```bash
            k6 run [flags]
            ```

            - `-u`, `--vus int`：指定 virtual user 的數量 (預設為 `1`)
            - `-d`, `--duration duration`：load test 持續時間
            - `-i`, `--iterations int`：測試執行迭代次數 (以所有 virtual user 計算的總數)
            - `-s`, `--stage stage`：增加測試階段 (語法是 `[duration]:[target]`)
            - `--execution-segment string`：限定執行某個區段
            - `--execution-segment-sequence string`：指定執行區段的順序
            - `-p`, `--paused`：啟動 load test 但讓 load test 暫停
            - `--no-setup`：不要執行 `setup()`
            - `--no-teardown`：不要執行 `teardown()`
            - `--max-redirects int`：限定轉導次數 (預設 `10`)
            - `--batch int`：限定平行批量 request 數量 (預設 `20`)
            - `--batch-per-host int`：限定單一 host 平行批量 request 數量 (預設 `6`)
            - `--rps int`：限定每秒 request 數量
            - `--user-agent string`：指定 http request 的 user agent (預設 `k6/0.37.0 (https://k6.io/)`)
            - `--http-debug string[="headers"]`：紀錄所有 HTTP requests and responses. 預設排除 `body`可以透過 `--http-debug=full` 來加入 `body` 資訊
            - `--insecure-skip-tls-verify`：不檢查 TLS 憑證的有效性
            - `--no-connection-reuse`：不允許 `keep-alive` 的連線
            - `--no-vu-connection-reuse`：在不同的迭代間不重複使用連線
            - `--min-iteration-duration duration`：單一迭代最小的測試執行時間
            - `-w`, `--throw`：將 warning (http request 異常) 以 error 的方式拋出
            - `--blacklist-ip ip range`：將 ip range 加至 blacklist
            - `--block-hostnames pattern`： 阻斷特定的 hostname 呼叫，pattern 是不限大小寫且可有可無 wildcard 開頭的 hostname
            - `--summary-trend-stats stats`：指定想要顯示在概要的趨勢指標統計值
            - `--summary-time-unit string`：指定概要中的趨勢統計值顯示時間單位：`s`, `ms` 與 `us`
            - `--system-tags strings`：指定指標只包含的 tag (預設 `proto,subproto,status,method,url,name,group,check,error,error_code,tls_version,scenario,service,expected_response`)
            - `--tag tag`： 為所有 sample 加上 tag (格式為 `[name]=[value]`)
            - `--console-output string`：將 console log 資訊以檔案格式儲存
            - `--discard-response-bodies`：讀取但不處理或是儲存 HTTP response bodies
            - `--local-ips string`：指定每個 virtual user 可能發出 request 的 IP Ranges and/or CIDRs (e.g. '192.168.220.1,192.168.0.10-192.168.0.25', 'fd:1::0/120')
            - `--dns string`： DNS 解析設定.
                - ttl：`inf` 保存 cache；`0` 停用 cache (可以指定時間：`1s`, `1m`，如果沒有指定單位，會用 Milliseconds)
                - select：`first`, `random` or `roundRobin`.
                - policy：`preferIPv4`, `preferIPv6`, `onlyIPv4`, `onlyIPv6` or `any`(預設 `ttl=5m,select=random,policy=preferIPv4`)
            - `--include-system-env-vars`：是否將系統的 environment variables 傳給 k6 使用(預設 `true`)
            - `--compatibility-mode string`：JavaScript compiler 相容模式 `extended` or `base` (預設 `extended`)
                - `base`: pure goja - 支援 ES5.1+ 的 Golang JS VM
                - `extended`: base + 部份 ES2015 預設的 Babel (如果用到 base 不支援的語法， compile 速度對變慢)
            - `-e`, `--env VAR=value`：指定 environment variable
            - `--no-thresholds`：不要執行 thresholds
            - `--no-summary`：測試結束後不顯示概要
            - `--summary-export string`：將測試結果概要儲存至指定的 JSON 路徑
            - `-o`, `--out uri`：外部指標儲存服務的 uri
            - `--no-usage-report`：不要使用匿名統計資訊
            - `-t`, `--type type`：覆寫檔案格式，`js` or `archive`
            - `-h`, `--help`：顯示 help 資訊

        - `scale`：擴展測試

            > 使用全域的 `--address` flag 來指定 api server url

            ```bash
            k6 scale [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊
            - `-u`, `--vus int`：指定 virtual user 的數量 (預設為 `1`)
            - `-m`, `--max int`：最大可用的 virtual user 的數量

        - `stats`：顯示測試指標

            > 使用全域的 `--address` flag 來指定 api server url

            ```bash
            k6 stats [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

        - `status`：顯示測試狀態

            > 使用全域的 `--address` flag 來指定 api server url

            ```bash
            k6 status [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

        - `version`：顯示程式版本

            > 顯示程式版本並退出

            ```bash
            k6 version [flags]
            ```

            - `-h`, `--help`：顯示 help 資訊

    - Global Flags:
        - `-a`, `--address string`：api server 的位址 (預設 `localhost:6565`)
        - `-c`, `--config string`：JSON config file (預設 `/Users/{username}/Library/Application Support/loadimpact/k6/config.json`)
        - `--log-output string`：修改 k6 log 的輸出，允許使用：`stderr`,`stdout`,`none`,`loki[=host:port]`,`file[=./path.fileformat]` (預設 `stderr`)
        - `--logformat string`    log output format
        - `--no-color`：停用輸出的顏色顯示
        - `-q`, `--quiet`：停用進度更新
        - `-v`, `--verbose`：啟用詳細日誌記錄

## 使用方式

1. Unary call

    - 測試腳本

        ```js
        // import grpc package
        import grpc from 'k6/net/grpc';
        import { check, sleep } from 'k6';
        // 建立 grpc client
        const client = new grpc.Client()
        //載入 proto 檔：第一個參數是 proto 檔的路徑，第二個參數是 proto 檔名
        client.load(['Protos'], 'greet.proto');
        
        export default () => {
            //連線至 grpc server endpoint
          client.connect('localhost:5143', {
            // 允許使用未加密的連線，預設是 `false`
            plaintext: true
          });
        
          const data = { name: 'Yowko' };
          // 呼叫 grpc 服務：{package name}.{service name}/{procedure name}
          const response = client.invoke('greet.Greeter/SayHello', data);
        
          // 檢查回應是否為 `grpc.StatusOK`
          check(response, {
            'status is OK': (r) => r && r.status === grpc.StatusOK,
          });
          // log 回應
          //console.log(JSON.stringify(response.message));
          // 關閉連線
          client.close();
          // sleep 一秒
          //sleep(1);
        };
        ```

    - 測試指令

        ```bash
        k6 run -i 20000 -u 20 k6.js
        ```

        ![1result](https://user-images.githubusercontent.com/3851540/162868719-332c0500-ed92-4146-a8fb-9d3a24174e8c.png)

2. client streaming

    > 暫不支援

3. server streaming

    > 暫不支援
  
4. bi-directional streaming

    > 暫不支援

## 心得

1. 使用 js 做為腳本語法

    > 我個人 js 底子差，寫起來特別不順手XD

2. 不支援 streaming 測試

    > [GitHub Issue: Support gRPC streaming](https://github.com/grafana/k6/issues/2020) 暫時還不支援 streaming 在使用上有不少限制

3. 測試結果一目瞭然

    > 這個是個人喜好，我覺得有上色，看起來賞心悅目許多XD

4. 測試效率較差？

    > 與 ghz 相比，一樣使用 20 concurrent 與 20000 total request 測試相同的 grpc service，ghz 可以壓出 9000 Requests/sec，但 k6 只有 4886.656447/s，不過也可能是寫法沒有優化的關係，只是兩者我都是參考官網建議寫的

5. 參數超多

    > 我在官網上沒有找到完整文件，cli 的 help 文件也有點亂：global flag、flag、subcommand....東西真的很多

## 參考資訊

1. [k6](https://k6.io/)
2. [GitHub : k6 Releases page](https://github.com/grafana/k6/releases)
3. [g6:Performance testing gRPC services](https://k6.io/blog/performance-testing-grpc-services/)
4. [k6 API>k6/net/grpc](https://k6.io/docs/javascript-api/k6-net-grpc/)
5. [GitHub Issue: Support gRPC streaming](https://github.com/grafana/k6/issues/2020)
