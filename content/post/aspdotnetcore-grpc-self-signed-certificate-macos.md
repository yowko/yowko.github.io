---
title: "ASP.NET Core gRPC 使用自發憑證時在 macOS 的特別處理"
date: 2021-03-05T09:30:00+08:00
lastmod: 2021-03-05T09:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","macOS"]
slug: "aspdotnetcore-grpc-self-signed-certificate-macos"
---

## ASP.NET Core gRPC 使用自發憑證時在 macOS 的特別處理

之前筆記 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 紀錄到使用自發憑證來為 ASP.NET Core gRPC service 加上 tls，透過新的做法看似可以正確在 container 內執行，只是寫法並不適用於 macOS 上的開發 (原因請參考官方說明 [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos))，尤其是開發新功能時 頻繁 build image 進行測試有些浪費時間，所以簡易調整一版先來試用看看，看後續狀況再來調整

## 基本環境說明

1. macOS Big Sur 11.2.2
2. .NET Core SDK 3.1.406
3. .NET Core gRPC Service 預設專案範本

## 設定方式

1. 建立憑證 (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

    > 這邊我是透過 [使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證](/cert-manager-pkcs12-pfx/) 建立憑證，再透過下列語法將憑證儲存至電腦的 `${HOME}/certs` 路徑備用

    ```bash
    kubectl get secrets/pkcs12 -o "jsonpath={.data['keystore\.p12']}" | base64 -D > test.pfx
    kubectl get secrets/pkcs12 -o "jsonpath={.data['tls\.crt']}" | base64 -D > test.crt
    kubectl get secrets/pkcs12 -o "jsonpath={.data['tls\.key']}" | base64 -D > test.key
    kubectl get secrets/pkcs12 -o "jsonpath={.data['ca\.crt']}" | base64 -D > ca.crt
    ```

2. ASP.NET Core 完整設定

    > 今天使用的專案名稱為 `ApplyTLS`，請需要自行調整

    - Protos (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)
        - greet.proto

            ```protobuf
            syntax = "proto3";

            option csharp_namespace = "ApplyTLS";
            
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

    - Services
        - GreeterHealthChecker.cs (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

            ```cs
            using System.Threading;
            using System.Threading.Tasks;
            using Microsoft.Extensions.Diagnostics.HealthChecks;
            
            namespace ApplyTLS.Services
            {
                public class GreeterHealthChecker: IHealthCheck
                {
                    private readonly Greeter.GreeterClient _client;
                    public GreeterHealthChecker(Greeter.GreeterClient client)
                    {
                        _client = client;
                    }
                    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext             context, CancellationToken cancellationToken = default(CancellationToken)            )
                    {
                        var reply = await _client.SayHelloAsync(
                            new HelloRequest { Name = "GreeterClient" });
                        if (reply.Message == "Hello GreeterClient")
                        {
                            return HealthCheckResult.Healthy("A healthy result.");
                        }
                        return HealthCheckResult.Unhealthy("An unhealthy result.");
                    }
                }
            }
            ```

        - GreeterService.cs (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

            ```cs
            using System.Threading.Tasks;
            using Grpc.Core;
            using Microsoft.Extensions.Logging;
            
            namespace ApplyTLS.Services
            {
                public class GreeterService : Greeter.GreeterBase
                {
                    private readonly ILogger<GreeterService> _logger;
            
                    public GreeterService(ILogger<GreeterService> logger)
                    {
                        _logger = logger;
                    }
            
                    public override Task<HelloReply> SayHello(HelloRequest request,             ServerCallContext context)
                    {
                        return Task.FromResult(new HelloReply
                        {
                            Message = "Hello " + request.Name
                        });
                    }
                }
            }
            ```

    - appsettings.json (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

        ```json
        {
          "Logging": {
            "LogLevel": {
              "Default": "Information",
              "Microsoft": "Warning",
              "Microsoft.Hosting.Lifetime": "Information"
            }
          },
          "AllowedHosts": "*",
          "GrpcPort": 12345,    
          "GrpcHost": "localhost"
        }
        ```

    - appsettings.Development.json (可調整)

        > 如果 rider 啟動也打算透過網頁來檢查 healthy check 的結果，可以設定 `ASPNETCORE_ENVIRONMENT=Development`，然後加上 web listen port

        ```json
        {
          "Logging": {
            "LogLevel": {
              "Default": "Debug",
              "System": "Information",
              "Grpc": "Information",
              "Microsoft": "Information"
            }
          },
          "Kestrel": {
            "Endpoints": {
              "Https": {
                "Url": "http://*:5001",
                "Protocols": "Http1"
              }
            }
          }
        }           
        ```

    - Program.cs (有調整)

        ```cs
        using System;
        using System.Runtime.InteropServices;
        using Microsoft.AspNetCore.Hosting;
        using Microsoft.AspNetCore.Server.Kestrel.Core;
        using Microsoft.Extensions.Configuration;
        using Microsoft.Extensions.Hosting;
        
        namespace ApplyTLS
        {
            public class Program
            {
                public static void Main(string[] args)
                {
                    CreateHostBuilder(args).Build().Run();
                }
        
                // Additional configuration is required to successfully run gRPC on         macOS.
                // For instructions on how to configure Kestrel and gRPC clients on         macOS, visit https://go.microsoft.com/fwlink/?linkid=2099682
                public static IHostBuilder CreateHostBuilder(string[] args) =>
                    Host.CreateDefaultBuilder(args)
                        .ConfigureWebHostDefaults(webBuilder =>
                        {
                            webBuilder.UseStartup<Startup>()
                                .ConfigureKestrel((context, kestrelOptions) =>
                                {
                                    // 從 appsetting 中取得 grpc server 的 host
                                    var port = context.Configuration.GetValue<int>("GrpcPort");
                                    // 判斷是否為 macOS 啟動
                                    var isosx= RuntimeInformation.IsOSPlatform(OSPlatform.OSX);
                                    // Kestrel 在 macOS 上不支援具有 TLS 的 HTTP/2：如果是 macOS 使用 Http2 protocol
                                    var protocol = isosx
                                        ? HttpProtocols.Http2
                                        : HttpProtocols.Http1AndHttp2;
        
                                    kestrelOptions.ListenAnyIP(port, listenOptions =>
                                    {
                                        listenOptions.Protocols = protocol;
                                        if (!isosx) //如果不是 macOS 就綁上憑證
                                        {
                                            // 使用過去慣用的環境變數來設定 grpc server 的憑證
                                            listenOptions.UseHttps(Environment.GetEnvironmentVariable("ASPNETCORE_Kestrel__Certificates__Default__Path"),Environment.GetEnvironmentVariable("ASPNETCORE_Kestrel__Certificates__Default__Password"));
                                        }
                                    });
                                });
                        });
            }
        }
        ```

    - Startup.cs (有調整)

        ```cs
        using System;
        using System.Net.Http;
        using System.Runtime.InteropServices;
        using ApplyTLS.Services;
        using Microsoft.AspNetCore.Builder;
        using Microsoft.AspNetCore.Hosting;
        using Microsoft.AspNetCore.Http;
        using Microsoft.Extensions.Configuration;
        using Microsoft.Extensions.DependencyInjection;
        using Microsoft.Extensions.Hosting;
        
        namespace ApplyTLS
        {
            public class Startup
            {
                private IConfiguration _configuration { get; }
        
                private bool _isosx { get; }

                public Startup(IConfiguration configuration)
                {
                    _configuration = configuration;
                    _isosx = RuntimeInformation.IsOSPlatform(OSPlatform.OSX);
                }
                // This method gets called by the runtime. Use this method to add services to the container.
                // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/S?LinkID=398940
                public void ConfigureServices(IServiceCollection services)
                {
                    services.AddGrpc();
                    services.AddHealthChecks().AddCheck<GreeterHealthChecker>("example_health_check");
        
                    if (_isosx)
                    {
                        // 取消 grpc client 做 healthycheck 的憑證驗證
                        AppContext.SetSwitch("System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);
                        protocol = "http";
                    }
        
                    // for healthy check
                    services.AddGrpcClient<Greeter.GreeterClient>(o =>
                        {
                            // 從 appsetting 中取得 grpc server 的 host 與 port
                            var host = _configuration.GetValue<string>("GrpcHost");
                            var port = _configuration.GetValue<int>("GrpcPort");
                            // 指定 grpc 完整 uri
                            o.Address = new Uri($"{protocol}://{host}:{port}");
                        })
                        .ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler
                        {
                            // 使用 httpclient 的方式來忽略憑證有效性
                            ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator
                            // 使用 grpc 方式忽略憑證有效性
                            //ServerCertificateCustomValidationCallback = (message, cert, chain, errors) => true
                        });
                }
        
                // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
                public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
                {
                    if (env.IsDevelopment())
                    {
                        app.UseDeveloperExceptionPage();
                    }
        
                    app.UseRouting();
        
                    app.UseEndpoints(endpoints =>
                    {
                        endpoints.MapHealthChecks("/health");
        
                        endpoints.MapGrpcService<GreeterService>();
        
                        endpoints.MapGet("/",
                            async context =>
                            {
                                await context.Response.WriteAsync(
                                    "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");
                            });
                    });
                }
            }
        }
        ```

    - .csproj (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

        > 修改 proto 編譯方式為 `Both`

        ```xml
        <Protobuf Include="Protos\greet.proto" GrpcServices="Both" />
        ```

3. dockerfile (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同)

    ```dockerfiles
    FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
    WORKDIR /app
    # 複製 sln csproj and restore nuget
    COPY *.sln .
    # 目標路徑資料夾要與來源路徑資料夾名稱相同(因 sln 已儲存專案路徑)
    COPY ApplyTLS/*.csproj ./ApplyTLS/
    #RUN dotnet dev-certs https --trust
    # nuget restore
    RUN dotnet restore
    # 複製其他檔案
    COPY ApplyTLS/. ./ApplyTLS/
    WORKDIR /app/ApplyTLS
    # 使用 Release 建置專案並輸出至 out
    RUN dotnet publish -c Release -o out
    FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
    WORKDIR /app
    # 將產出物複製至 app 下
    COPY --from=build /app/ApplyTLS/out ./
    # 啟動 TestDokcer
    ENTRYPOINT ["dotnet", "ApplyTLS.dll"]
    ```

4. build image (調整 image tag)

    ```cs
    docker build ./ -t yowko/applytls:0.0.2
    ```

5. 啟動方式

    - docker run (使用新的 image tag)

        ```bash
        docker run --rm -it -p 12345:12345 -e ASPNETCORE_Kestrel__Certificates__Default__Password="pass.123" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/test.pfx -v ${HOME}/certs:/https/ yowko/applytls:0.0.2
        ```

    - rider 直接啟動

## 心得

- 實際效果：

    > image 啟動 container 的效果與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同，以下只截 macOS 上使用 rider 啟動的實際狀況

    1. 網頁 (透過 mvc controller 另外啟動 grpc client 來呼叫本身的 grpc service)

        ![1webpage](https://user-images.githubusercontent.com/3851540/110076294-35d04800-7dbf-11eb-9ff9-0a2758270e2a.png)

    2. grpcurl

        > grpcurl 的用法可以參考之前筆記 [使用 grpcurl 呼叫 gRPC Service](/grpcurl)

        ```bash
        grpcurl -plaintext -d '{"name":"Yowko"}' -proto ApplyTLS/Protos/greet.proto localhost:12345 greet.Greeter/SayHello
        ```

        ![2grpcurl](https://user-images.githubusercontent.com/3851540/110076298-37017500-7dbf-11eb-9b24-4a28093bb16a.png)

    3. bloomrpc

        ![3bloomrpc](https://user-images.githubusercontent.com/3851540/110076301-3832a200-7dbf-11eb-96d2-b1c64cd5f61a.png)

- 完整程式碼可以參考 [aspdotnetcore-grpc-self-signed-certificate-sample](https://github.com/yowko/aspdotnetcore-grpc-self-signed-certificate)

## 參考資訊

1. [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate)
2. [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos)
3. [aspdotnetcore-grpc-self-signed-certificate-sample](https://github.com/yowko/aspdotnetcore-grpc-self-signed-certificate)
