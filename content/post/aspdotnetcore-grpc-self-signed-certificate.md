---
title: "ASP.NET Core gRPC 使用自發憑證"
date: 2021-03-04T09:30:00+08:00
lastmod: 2021-03-04T09:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC"]
slug: "aspdotnetcore-grpc-self-signed-certificate"
---

## ASP.NET Core gRPC 使用自發憑證

距之前筆記 [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate/) 也不過幾個月時間，最近想要搭配 [Kubernetes 發行憑證給 ASP.NET Core 使用](kubernetes-certificate-aspdotnet-core) 透過 Kubernetes 的 cert-manager 使用自簽憑證，想不到一直不通，從一開始懷疑憑證格式不相容，後來嘗試調整程式語法也不行，也試了使用之前筆記的 image 也無法成功，最後推測可能的原因有兩個：

1. 之前筆記 [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate/) 本來就有問題，只是剛好當下沒測出來
2. 新版的 .NET SDK 或是 grpc.net 做了調整

以結果來看比較像是之前筆記 [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate/) 本來就有問題，因為使用當時的 image 也無法成功使用XD

無論如何，現在再做一次紀錄加深印象，把上次可能漏掉的設定檔一併補上!!!

## 基本環境說明

1. macOS Big Sur 11.2.2
2. .NET Core SDK 3.1.406
3. .NET Core gRPC Service 預設專案範本

## 設定方式

1. 建立憑證

    > 這邊我是透過 [使用 cert-manager 建立 PKCS12 格式 (.pfx) 憑證](/cert-manager-pkcs12-pfx/) 建立憑證，再透過下列語法將憑證儲存至電腦的 `${HOME}/certs` 路徑備用

    ```bash
    kubectl get secrets/pkcs12 -o "jsonpath={.data['keystore\.p12']}" | base64 -D > test.pfx
    kubectl get secrets/pkcs12 -o "jsonpath={.data['tls\.crt']}" | base64 -D > test.crt
    kubectl get secrets/pkcs12 -o "jsonpath={.data['tls\.key']}" | base64 -D > test.key
    kubectl get secrets/pkcs12 -o "jsonpath={.data['ca\.crt']}" | base64 -D > ca.crt
    ```

2. ASP.NET Core 完整設定

    > 今天使用的專案名稱為 `ApplyTLS`，請需要自行調整

    - Protos
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
        - GreeterHealthChecker.cs

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

        - GreeterService.cs

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

    - appsettings.json

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

    - appsettings.Development.json

        ```json
        {
          "Logging": {
            "LogLevel": {
              "Default": "Debug",
              "System": "Information",
              "Grpc": "Information",
              "Microsoft": "Information"
            }
          }
        }           
        ```

    - Program.cs

        ```cs
        using System;
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
        
                                    kestrelOptions.ListenAnyIP(port, listenOptions =>
                                    {
                                        // 將 grpc 的 protocal 同時註冊 http1 與 http2
                                        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
                                        // 使用過去慣用的環境變數來設定 grpc server 的憑證
                                        listenOptions.UseHttps(Environment.GetEnvironmentVariable("ASPNETCORE_Kestrel__Certificates__Default__Path"),Environment.GetEnvironmentVariable("ASPNETCORE_Kestrel__Certificates__Default__Password"));
                                    });
                                });
                        });
            }
        }
        ```

    - Startup.cs

        ```cs
        using System;
        using System.Net.Http;
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
        
        
                public Startup(IConfiguration configuration)
                {
                    _configuration = configuration;
                }
                // This method gets called by the runtime. Use this method to add services to the container.
                // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/S?LinkID=398940
                public void ConfigureServices(IServiceCollection services)
                {
                    services.AddGrpc();
                    services.AddHealthChecks().AddCheck<GreeterHealthChecker>("example_health_check");
        
                    //指定 grpc server 連線的 protocal
                    const string protocol = "https";
        
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

    - csproj

        > 修改 proto 編譯方式為 `Both`

        ```xml
        <Protobuf Include="Protos\greet.proto" GrpcServices="Both" />
        ```

3. dockerfile

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

4. build image

    ```cs
    docker build ./ -t yowko/applytls:0.0.1 
    ```

5. 啟動方式

    ```bash
    docker run --rm -it -p 12345:12345 -e ASPNETCORE_Kestrel__Certificates__Default__Password="pass.123" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/test.pfx -v ${HOME}/certs:/https/ yowko/applytls:0.0.1
    ```

## 心得

- 實際效果：

    1. 網頁 (透過 mvc controller 另外啟動 grpc client 來呼叫本身的 grpc service)

        ![1webpage](https://user-images.githubusercontent.com/3851540/109983959-4db3b780-7d3e-11eb-8cb2-9b4297cdb763.png)

    2. grpcurl

        > grpcurl 的用法可以參考之前筆記 [使用 grpcurl 呼叫 gRPC Service](/grpcurl)

        ```bash
        grpcurl -cacert ${HOME}/certs/test.crt  -d '{"name":"Yowko"}' -proto ApplyTLS/Protos/greet.proto localhost:12345 greet.Greeter/SayHello
        ```

        ![2grpcurl](https://user-images.githubusercontent.com/3851540/109983965-4f7d7b00-7d3e-11eb-956c-2feb3054e291.png)

    3. bloomrpc

        ![3bloomrpc](https://user-images.githubusercontent.com/3851540/109983971-50161180-7d3e-11eb-8290-2d9a3012b68c.png)

- 今天紀錄的內容，不支援 macOS 直接執行 (原因請參考官方說明 [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos))，預計會另外筆記處理方式

## 參考資訊

1. [讓 container 中的 ASP.NET Core 也有憑證](/aspdotnet-core-container-certificate/)
2. [Kubernetes 發行憑證給 ASP.NET Core 使用](kubernetes-certificate-aspdotnet-core)
3. [使用 grpcurl 呼叫 gRPC Service](/grpcurl)
4. [無法在 macOS 上啟動 ASP.NET Core gRPC 應用程式](https://docs.microsoft.com/zh-tw/aspnet/core/grpc/troubleshoot?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos)
