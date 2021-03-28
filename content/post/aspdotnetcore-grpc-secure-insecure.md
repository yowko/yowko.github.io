---
title: "ASP.NET Core gRPC 的 Secure 與 Insecure 不同做法"
date: 2021-03-07T09:30:00+08:00
lastmod: 2021-03-28T09:30:31+08:00
draft: false
tags: ["ASP.NET Core","gRPC","macOS"]
slug: "aspdotnetcore-grpc-secure-insecure"
---

## ASP.NET Core gRPC 的 Secure 與 Insecure 不同做法"

之前筆記 [ASP.NET Core gRPC 使用自發憑證時在 macOS 的特別處理](aspdotnetcore-grpc-self-signed-certificate-macos) 紀錄到如何在 macOS 與 container 間使用不同 port 與 protocol 來建立 gRPC service，當時查資料看到也許才是正確做法的設定方式，趁著假日空檔來快速紀錄一下

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

    > 今天使用的專案名稱為 `ConfigByOS`，請需要自行調整

    - Protos (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同，僅調整 project namespace)
        - greet.proto

            ```protobuf
            syntax = "proto3";

            option csharp_namespace = "ConfigByOS";
            
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
        - GreeterHealthChecker.cs (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同，僅調整 project namespace)

            ```cs
            using System.Threading;
            using System.Threading.Tasks;
            using Microsoft.Extensions.Diagnostics.HealthChecks;
            
            namespace ConfigByOS.Services
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

        - GreeterService.cs (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同，僅調整 project namespace)

            ```cs
            using System.Threading.Tasks;
            using Grpc.Core;
            using Microsoft.Extensions.Logging;
            
            namespace ConfigByOS.Services
            {
                public class GreeterService : Greeter.GreeterBase
                {
                    private readonly ILogger<GreeterService> _logger;
            
                    public GreeterService(ILogger<GreeterService> logger)
                    {
                        _logger = logger;
                    }
            
                    public override Task<HelloReply> SayHello(HelloRequest request,ServerCallContext context)
                    {
                        return Task.FromResult(new HelloReply
                        {
                            Message = "Hello " + request.Name
                        });
                    }
                }
            }
            ```

    - appsettings.json (預設值)

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
          "Kestrel": {
            "EndpointDefaults": {
              "Protocols": "Http2"
            }
          }
        }
        ```

    - appsettings.Development.json (預設值)

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

    - appsettings.linux.json (新增)

        ```json
        {
          "Kestrel": {
            "Endpoints": {
              "GrpcSecure": {
                "Url": "https://*:12345",
                "Protocols": "Http1AndHttp2"
              }
            }
          }
        }
        ```

    - appsettings.macos.json (新增)

        ```json
        {
          "Kestrel": {
            "Endpoints": {
              "Https": {
                "Url": "https://*:5001",
                "Protocols": "Http1"
              },
              "GrpcInsecure" : {
                "Url": "http://*:12345",
                "Protocols": "Http2"
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
        
        namespace ConfigByOS
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
                         .ConfigureAppConfiguration((hostingContext, config) =>
                         {
                             // 如果啟動環境是 macos 的話，即套用 appsettings.macos.json
                             if (RuntimeInformation.IsOSPlatform(OSPlatform.OSX))
                                 config.AddJsonFile("appsettings.macos.json", optional: true);
                             else // 若啟動環境不為 macos 就套用 appsettings.linux.json
                                 config.AddJsonFile("appsettings.linux.json", optional: true);
                         })
                         .ConfigureWebHostDefaults(webBuilder => { webBuilder.UseStartup<Startup>(); });
            }
        }
        ```

    - Startup.cs (有調整)

        ```cs
        using System;
        using System.Net.Http;
        using System.Runtime.InteropServices;
        using ConfigByOS.Services;
        using Microsoft.AspNetCore.Builder;
        using Microsoft.AspNetCore.Hosting;
        using Microsoft.AspNetCore.Http;
        using Microsoft.Extensions.Configuration;
        using Microsoft.Extensions.DependencyInjection;
        using Microsoft.Extensions.Hosting;
        
        namespace ConfigByOS
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
                // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
                public void ConfigureServices(IServiceCollection services)
                {
                    services.AddGrpc();
                    services.AddHealthChecks().AddCheck<GreeterHealthChecker>("example_health_check");
                    var _uri = _configuration.GetValue<string>("Kestrel:Endpoints:GrpcSecure:Url");
                    if (_isosx)
                    {
                        AppContext.SetSwitch("System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);
                        _uri = _configuration.GetValue<string>("Kestrel:Endpoints:GrpcInsecure:Url");
                    }
        
                    // for healthy check
                    services.AddGrpcClient<Greeter.GreeterClient>(o =>
                        {
                            //將 config 中 grpc url 的 host 改為 localhost
                            o.Address = new Uri(_uri.Replace("*","localhost"));
                        })
                        .ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler
                        {
                            ServerCertificateCustomValidationCallback = HttpClientHandler.        DangerousAcceptAnyServerCertificateValidator
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

3. dockerfile (與 [ASP.NET Core gRPC 使用自發憑證](/aspdotnetcore-grpc-self-signed-certificate) 相同，僅修改 project name)

    ```dockerfiles
    FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
    WORKDIR /app
    # 複製 sln csproj and restore nuget
    COPY *.sln .
    # 目標路徑資料夾要與來源路徑資料夾名稱相同(因 sln 已儲存專案路徑)
    COPY ConfigByOS/*.csproj ./ConfigByOS/
    #RUN dotnet dev-certs https --trust
    # nuget restore
    RUN dotnet restore
    # 複製其他檔案
    COPY ConfigByOS/. ./ConfigByOS/
    WORKDIR /app/ConfigByOS
    # 使用 Release 建置專案並輸出至 out
    RUN dotnet publish -c Release -o out
    FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
    WORKDIR /app
    # 將產出物複製至 app 下
    COPY --from=build /app/ConfigByOS/out ./
    # 啟動 TestDokcer
    ENTRYPOINT ["dotnet", "ConfigByOS.dll"]
    ```

4. build image (調整 image tag)

    ```cs
    docker build ./ -t yowko/configbyos:0.0.1
    ```

5. 啟動方式

    - docker run (使用新的 image tag)

        ```bash
        docker run --rm -it -p 12345:12345 -e ASPNETCORE_Kestrel__Certificates__Default__Password="pass.123" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/test.pfx -v ${HOME}/certs:/https/ yowko/configbyos:0.0.1
        ```

    - rider 直接啟動

## 心得

- 實際效果：

    - container

        1. 網頁 (透過 mvc controller 另外啟動 grpc client 來呼叫本身的 grpc service)

            ![1webpage](https://user-images.githubusercontent.com/3851540/109983959-4db3b780-7d3e-11eb-8cb2-9b4297cdb763.png)

        2. grpcurl

            > grpcurl 的用法可以參考之前筆記 [使用 grpcurl 呼叫 gRPC Service](/grpcurl)

            ```bash
            grpcurl -cacert ${HOME}/certs/test.crt  -d '{"name":"Yowko"}' -proto Protos/greet.proto localhost:12345 greet.Greeter/SayHello
            ```

            ![2grpcurl](https://user-images.githubusercontent.com/3851540/109983965-4f7d7b00-7d3e-11eb-956c-2feb3054e291.png)

        3. bloomrpc

            ![3bloomrpc](https://user-images.githubusercontent.com/3851540/109983971-50161180-7d3e-11eb-8290-2d9a3012b68c.png)

    - rider

        1. 網頁 (透過 mvc controller 另外啟動 grpc client 來呼叫本身的 grpc service)

            ![1webpage](https://user-images.githubusercontent.com/3851540/110076294-35d04800-7dbf-11eb-9ff9-0a2758270e2a.png)

        2. grpcurl

            > grpcurl 的用法可以參考之前筆記 [使用 grpcurl 呼叫 gRPC Service](/grpcurl)

            ```bash
            grpcurl -plaintext -d '{"name":"Yowko"}' -proto Protos/greet.proto localhost:12345 greet.Greeter/SayHello
            ```

            ![2grpcurl](https://user-images.githubusercontent.com/3851540/110076298-37017500-7dbf-11eb-9b24-4a28093bb16a.png)

        3. bloomrpc

            ![3bloomrpc](https://user-images.githubusercontent.com/3851540/110076301-3832a200-7dbf-11eb-96d2-b1c64cd5f61a.png)

個人覺得使用 ASP.NET Core 的預設處理機制比較好理解，程式碼也簡單，但還是有其他可以調整的空間

1. 套用不同 config 的方式
2. grpc client 使用的 grpc service 的 url 的處理方式

簡單紀錄一下，多個 solution 選擇參考

- 完整程式碼可以參考 [yowko/aspdotnetcore-grpc-self-signed-certificate-config](https://github.com/yowko/aspdotnetcore-grpc-self-signed-certificate-config)

## 參考資訊

1. [NetCoreTemplates/grpc/MyApp/appsettings.json](https://github.com/NetCoreTemplates/grpc/blob/769956a19f3b3f2cc4e9ea4aca363123865f8f20/MyApp/appsettings.json)
2. [How to use machine-specific configuration with ASP.NET Core](https://andrewlock.net/how-to-use-machine-specific-configuration-with-asp-net-core/)
3. [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
4. [Platform Conditional Compilation in .NET Core](https://blog.magnusmontin.net/2018/11/05/platform-conditional-compilation-in-net-core/)
5. [Use multiple environments in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
6. [yowko/aspdotnetcore-grpc-self-signed-certificate-config](https://github.com/yowko/aspdotnetcore-grpc-self-signed-certificate-config)
