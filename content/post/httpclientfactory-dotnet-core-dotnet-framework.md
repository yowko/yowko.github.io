---
title: "在 .NET Core 與 .NET Framework 上使用 HttpClientFactory"
date: 2019-01-16T23:45:00+08:00
lastmod: 2020-12-11T23:44:30+08:00
draft: false
tags: ["C#","dotnet core","dotnet"]
slug: "httpclientfactory-dotnet-core-dotnet-framework"
---
# 在 .NET Core 與 .NET Framework 上使用 HttpClientFactory
之前筆記 [探討 HttpClient 可能的問題](/httpclient-issue/) 與 [HttpClient 無法反應 DNS 異動的解決方式](/httpclient-not-respect-dns-change) 的出現是因為工作任務需要將一些重要訊息傳送至 Slack 而留意到 .NET Core 使用的 HttpClientFactory 是改善過去 HttpClient 存在的一些問題，為了可以更完整理解 HttpClient 缺憾做得的一些紀錄

既然對於 HttpClient 過去的問題有些認識後，當然還是得來搞清楚 HttpClientFactory 的內容囉

## 關於 HttpClientFactory
- 設計理念：
	- 提供一個集中位置來命名和設定 HttpClient instance
	- 透過委派 DelegatingHandlers 來實現 outgoing 的 middleware
	- 統一管理 HttpClientMessageHandler (HttpClientHandler 的基底類別) 的生命周期與連線
- 從 .NET Core 2.1 開始加入，用來建立及管理 HttpClient instance
- HttpClient 底層使用的 HttpClientHandler 的生命周期及 DNS 過期問題會統一由 HttpClientFactory  管理
- 透過重複使用 HttpClient 底層 connection 來避免 socket 耗盡

	> 過去在 .NET Framework 上可以透過 singleton 或是 static instance 來避免問題發生

- 透過增加 `PooledConnectionLifetime` 屬性來處理 `ServicePointManager.ConnectionLeaseTimeout`

	> 過去在 .NET Framework 上可以透過指定 `DefaultRequestHeaders.ConnectionClose` 設定為 `true` 或是修改 `ServicePointManager.ConnectionLeaseTimeout` 解決，詳細內容可以參考 [HttpClient 無法反應 DNS 異動的解決方式](/httpclient-not-respect-dns-change)

- 每次都取得新的 HttpClient instance 但成本最高的底層 HttpClientHandler 與 connection 則依生命周期決定由 pool 中或是建立新的 instance
    - HttpClientHandler 預設的存活時間為 2 分鐘
		-  HttpClientHandler 到期時不會立即被 dispose 而是移至過期的 pool 避免再被新建立的 HttpClient 取用並等待到使用該 HttpClientHandler 中的  HttpClient 結束工作後，自然被 gc 掉

	- 每個具名 HttpClient 可以擁有單獨設定 SetHandlerLifetime 可以指定 Handler 過期時間，這也是 DNS 異動被採用的時間，不得低於 1 秒



## 前提設定
1. Visual Studio 2017 15.9.4
2. .NET Core 2.2.101

	- Program.cs

		```cs
		public class Program
		{
			public static void Main(string[] args)
			{
				CreateWebHostBuilder(args).Build().Run();
			}

			public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
				WebHost.CreateDefaultBuilder(args)
					.UseStartup<Startup>();
		}
		``` 
	- Startup.cs
		
		```cs
		public class Startup
		{
			// This method gets called by the runtime. Use this method to add services to the container.
			// For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
			public void ConfigureServices(IServiceCollection services)
			{
			}

			// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
			public void Configure(IApplicationBuilder app, IHostingEnvironment env)
			{
				if (env.IsDevelopment())
				{
					app.UseDeveloperExceptionPage();
				}

				app.Run(async (context) =>
				{
					await context.Response.WriteAsync("Hello World!");
				});
			}
		}
		```
3. .NET Framework 4.7.2

	> 詳細使用方法請參考 [在 ASP.NET MVC 5 中使用 ASP.NET Core Dependency Injection 與 HttpClientFactory](/aspnet-core-di-httpclientfactory-in-aspnet-mvc-5)

	- Startup.cs

		```cs
		public partial class Startup
		{
			public void Configuration(IAppBuilder app)
			{
				var services = new ServiceCollection();
				ConfigureAuth(app);
				ConfigureServices(services);
				var resolver = new DefaultDependencyResolver(services.BuildServiceProvider());
				DependencyResolver.SetResolver(resolver);
			}
			public void ConfigureServices(IServiceCollection services)
			{
				
				services.AddControllersAsServices(typeof(Startup).Assembly.GetExportedTypes()
				.Where(t => !t.IsAbstract && !t.IsGenericTypeDefinition)
				.Where(t => typeof(IController).IsAssignableFrom(t) || t.Name.EndsWith("Controller", StringComparison.OrdinalIgnoreCase)));
			}
		}
		```
	- DefaultDependencyResolver.cs

		```cs
		public class DefaultDependencyResolver : IDependencyResolver
		{
			protected IServiceProvider serviceProvider;

			public DefaultDependencyResolver(IServiceProvider serviceProvider)
			{
				this.serviceProvider = serviceProvider;
			}

			public object GetService(Type serviceType)
			{
				return this.serviceProvider.GetService(serviceType);
			}

			public IEnumerable<object> GetServices(Type serviceType)
			{
				return this.serviceProvider.GetServices(serviceType);
			}
		}
		``` 
	- ServiceProviderExtensions.cs
    	
		```cs
		public static class ServiceProviderExtensions
		{
			public static IServiceCollection AddControllersAsServices(this IServiceCollection services,
			IEnumerable<Type> controllerTypes)
			{
				foreach (var type in controllerTypes)
				{
					services.AddTransient(type);
				}

				return services;
			}
		}
		``` 


## 使用方式

* .NET Core 因已預設參考 `Microsoft.AspNetCore.App` 的 metapackage，已內建 `Microsoft.Extensions.Http` plugin，故無須額外安裝套件
* .NET Framework 需另外安裝 `Microsoft.Extensions.Http` NuGet 套件

	> 這邊使用 `Microsoft.Extensions.Http 2.2.0`

	- Package Manger

		```
		Install-Package Microsoft.Extensions.Http
		``` 
	- .NET CLI

		```
		dotnet add package Microsoft.Extensions.Http 
		```

1. 基本用法

	> 最適合用來重構既有系統，有相同的 HttpClient 使用方式，只有將建立 HttpClient instance 使用 `CreateClient` 取代即可

	- 在 `Startup.cs` 的 `ConfigureServices` 方法中透過 `IServiceCollection` 呼叫擴充方法：`AddHttpClient` 來進行註冊

		```cs
		public void ConfigureServices(IServiceCollection services)
		{
			services.AddHttpClient();
		}
		```
	- 在需要使用 HttpClient 的 class 中透過 .NET Core 的建構式注入

		```cs
		private readonly IHttpClientFactory _clientFactory;

		public HomeController(IHttpClientFactory clientFactory)
		{
			_clientFactory = clientFactory;
		}
		```
	- 需要使用 HttpClient 直接透過 `CreateClient` 方法取得即可

		```cs
		var client = _clientFactory.CreateClient();
		```


2. 使用具名 HttpClient
   
	> 透過在 `Startup.ConfigureServices` 註冊時指定 client 的名稱及基本設定

	- 在 `Startup.cs` 註冊 HttpClientFactory 時指定名稱及預做基本設定
        	
		```cs
		services.AddHttpClient("yowkoblog", c =>
		{
			c.BaseAddress = new Uri("/");
		});
		``` 
    - 在需要使用 HttpClient 的 class 中透過 .NET Core 的建構式注入 (與 `基本用法` 相同)

		```cs
		private readonly IHttpClientFactory _clientFactory;

		public HomeController(IHttpClientFactory clientFactory)
		{
			_clientFactory = clientFactory;
		}
		```
	- 需要使用 HttpClient 直接透過 `CreateClient` 方法並指定名稱 ('yowkoblog') 即可  (與 `基本用法` 接近)

		```cs
		//名稱與 services.AddHttpClient 註冊時相同
		var client = _clientFactory.CreateClient("yowkoblog");
		``` 

3. 使用具型別的 HttpClient

	> 將 client 的基本設定、使用方法與處理邏輯封裝在特定類別中
	
	* 方法一：基本設定寫在建構式中
    	- 建立類別

    		```cs
    		public class YowkoBlogService
    		{

    			public HttpClient Client { get; }

    			public YowkoBlogService(HttpClient client)
    			{
    				client.BaseAddress = new Uri("");
    				
    				Client = client;
    			}

    			public async Task<string> GetPosts()
    			{
    				var response = await Client.GetAsync("/");

    				response.EnsureSuccessStatusCode();

    				var result = await response.Content.ReadAsStringAsync();

    				return result;
    			}
    		}
    		```
    	- 在 `Startup.cs` 使用自訂類別來註冊 HttpClientFactory
        	
    		```cs
    		services.AddHttpClient<YowkoBlogService>();
    		``` 
        - 在需要使用 HttpClient 的 class 中透過 .NET Core 的建構式注入 (與 `基本用法` 相同)

    		```cs
    		private readonly YowkoBlogService _yowkoBlogService;

    		public HomeController(YowkoBlogService yowkoBlogService)
    		{
    			_yowkoBlogService = yowkoBlogService;
    		}
    		```
    	- 直接呼叫自訂類別中的自訂方法即可

    		```cs
    		var result = await _yowkoBlogService.GetPosts();
    		```  
	- 方法二：基本設定寫在 `Startup.cs` 的註冊中

		- 自訂類別

			```cs
			public class YowkoBlogService
			{

				public HttpClient Client { get; }

				public YowkoBlogService(HttpClient client)
				{
					Client = client;
				}

				public async Task<string> GetPosts()
				{
					var response = await Client.GetAsync("/");

					response.EnsureSuccessStatusCode();

					var result = await response.Content.ReadAsStringAsync();

					return result;
				}
			}
			```
		- 在 `Startup.cs` 使用自訂類別來註冊 HttpClientFactory
		
			```cs
			services.AddHttpClient<YowkoBlogService>(c =>
			{
				c.BaseAddress = new Uri("");
			});
			```
		- 其他動作與 `方法一` 相同
   - 方法三：將 HttpClient 完全封裝不對外公開
		- 自訂類別

			```cs
			public class YowkoBlogService
			{

				private readonly HttpClient Client;

				public YowkoBlogService(HttpClient client)
				{
					Client = client;
				}

				public async Task<string> GetPosts()
				{
					var response = await Client.GetAsync("/");

					response.EnsureSuccessStatusCode();

					var result = await response.Content.ReadAsStringAsync();

					return result;
				}
			}
			```
		- 在 `Startup.cs` 使用自訂類別來註冊 HttpClientFactory
		
			```cs
			services.AddHttpClient<YowkoBlogService>(c =>
			{
				c.BaseAddress = new Uri("");
			});
			```
		- 其他動作與 `方法一` 相同
4. 使用 refit 套件來產生 HttpClient

	> 可以將 REST API 透過 interface 來呈現

	- 安裝 refit

		- Package Manager

			```
			Install-Package refit
			```
		- .NET CLI

			```
			dotnet add package refit
			```
	- 建立 interface

		```cs
		public interface IJsonbinClient
		{
			[Get("/_/me")]
			Task<User> GetMeAsync();
		}
		``` 
	- 取得結果 class
    	
		```cs
		public class AboutMe
		{
			public string email { get; set; }
			public string githubId { get; set; }
			public string username { get; set; }
			public Requests requests { get; set; }
			public Accounttype accountType { get; set; }
			public DateTime updated { get; set; }
			public DateTime created { get; set; }
			public string[] _public { get; set; }
			public string apikey { get; set; }
			public string publicId { get; set; }
		}

		public class Requests
		{
			public int PATCH { get; set; }
			public int POST { get; set; }
			public int GET { get; set; }
			public int PUT { get; set; }
		}

		public class Accounttype
		{
			public string name { get; set; }
		} 
		```
	- 在 `Startup.cs` 註冊並透過 refit 動態產生 IJsonbinClient 實作

		```cs
		services.AddHttpClient("JSonbin", c =>
            {
                c.BaseAddress = new Uri("https://jsonbin.org/");
				c.DefaultRequestHeaders.Add("authorization", "token {api token}");
            })
            .AddTypedClient(c => Refit.RestService.For<IJsonbinClient>(c));       

		```
	- 在需要使用 HttpClient 的 class 中透過 .NET Core 的建構式注入 (與 `基本用法` 相同)

    	```cs
    	private readonly IJsonbinClient _jsonbinClient;

    	public HomeController(IJsonbinClient jsonbinClient)
        {
           _jsonbinClient = jsonbinClient;
        }
    	```
    - 直接呼叫自訂類別中的自訂方法即可

    	```cs
    	var result =await _yowkoBlogService.GetPosts();
    	```  

## Outgoing Request middleware

middleware 在 .NET Core 中佔有舉足輕重的地位，許多設計都有 middleware 的影子，HttpClientFactory 便是其一，與 ASP.NET Core middleware 相似  outgoing Request middleware 可以在具名 HttpClient 上註冊與套用多個 handler 並用來管理 cache、error handling 、serialization 與 logging

1. 新增自訂 OuterHandler 、InnerHandler 皆繼承自 `DelegatingHandler`

	- OuterHandler.cs

		```cs
		public class OuterHandler : DelegatingHandler
		{
			private readonly ILogger<OuterHandler> _logger;

			public OuterHandler(ILogger<OuterHandler> logger)
			{
				_logger = logger;
			}

			protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request,
				CancellationToken cancellationToken)
			{
				_logger.LogInformation($"OuterHandler Call Start");
				var response = await base.SendAsync(request, cancellationToken);
				_logger.LogInformation($"OuterHandler Call End");
				return response;
			}
		}
		``` 
	- InnerHandler.cs

		```cs
		public class InnerHandler : DelegatingHandler
		{
			private readonly ILogger<InnerHandler> _logger;

			public InnerHandler(ILogger<InnerHandler> logger)
			{
				_logger = logger;
			}

			protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request,
				CancellationToken cancellationToken)
			{
				_logger.LogInformation($"InnerHandler Call Start");
				var response = await base.SendAsync(request, cancellationToken);
				_logger.LogInformation($"InnerHandler Call End");
				return response;
			}
		}
		```

2. 在 `Startup.cs` 的 `ConfigureServices` 方法中註冊自訂 Handler

	```cs
	services.AddTransient<OuterHandler>();
    services.AddTransient<InnerHandler>();

    services.AddHttpClient("yowkoblog", c =>
    {
        c.BaseAddress = new Uri("");
        
    })
	// 註冊的順序會影響執行的順序
    .AddHttpMessageHandler<OuterHandler>() //發出 request 時第一個執行，取回 response 時最後一個執行，對 HttpClientHandler 是較外層
    .AddHttpMessageHandler<InnerHandler>(); //發出 request 時最後執行，取回 response 時第一個執行，對 HttpClientHandler 是較內層
            
	```
3. 實際結果

	![1result](https://user-images.githubusercontent.com/3851540/51261569-76a45180-19eb-11e9-9572-cc01fccf6081.png)

* 完整流程
	
	![httpclient](https://user-images.githubusercontent.com/3851540/51324487-ce9f8e80-1aa5-11e9-9523-38b9440d5938.png)

## 重試策略：使用 Polly
過去在使用 HttpClient 取得外部資料時，最難控制的大概就是網路問題及遠端資源的狀態，一般都是自行實做重試及錯誤處理，而這樣的問題在 IHttpClientFactory 透過整合 [Polly](https://github.com/App-vNext/Polly) 後而變得不同了

Polly 是綜合彈性和暫時故障處理的套件，允許開發人員使用流暢及 thread-safe 的方式來達成 Retry、鎔斷、Timeout、隔離、降級退回等策略

1. 安裝 `Microsoft.Extensions.Http.Polly` NuGet 套件

	> Install-Package Microsoft.Extensions.Http.Polly

	- Package Manager
	
		```
		Install-Package Microsoft.Extensions.Http.Polly
		```

	- .NET CLI

		```
		dotnet add package Microsoft.Extensions.Http.Polly
		``` 
2. 建立 Polly 重試策略

	- 暫時性錯誤：透過 `HandleTransientHttpError`
	
		```cs
		static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
		{
			return HttpPolicyExtensions
				.HandleTransientHttpError()//遇到 HTTP 5xx
				.OrResult(msg => msg.StatusCode == System.Net.HttpStatusCode.NotFound)//或是得到 404 NoFound
				.WaitAndRetryAsync(6, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2,retryAttempt)));//重試六次，間隔秒數為 2 的 {重試次數} 次方:重試第一次間隔 2 的 1 次方、重試第三次間隔為 2 的 3 次方
		}
		```

	- 預先定義一般策略

		```cs
		var timeout = Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(10));
		var longTimeout = Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(30));
		```

3. 在 Startup.cs 的 ConfigureServices 方法中透過上述的 Polly 重試策略來註冊 HttpClient

	- 透過靜態方法來註冊
	
		```cs
		services.AddHttpClient()
			.AddPolicyHandler(GetRetryPolicy());
		```
	- 動態註冊策略

		```cs
		services.AddHttpClient()
			.AddPolicyHandler(request => 
				request.Method == HttpMethod.Get ? timeout : longTimeout);
		```

	- 同時使用多種策略

		```cs
		services.AddHttpClient()
		.AddTransientHttpErrorPolicy(p => p.RetryAsync(3))//至多重試三次
		.AddTransientHttpErrorPolicy(p => p.CircuitBreakerAsync(5, TimeSpan.FromSeconds(30)));//若失敗五次，就暫停嘗試 30秒
		```
	- 具名註冊

		> 為策略命名且定義，可以依不同的具名 client 指定不同策略

		```cs
		var registry = services.AddPolicyRegistry();

		registry.Add("regular", timeout);
		registry.Add("long", longTimeout);

		services.AddHttpClient()
			.AddPolicyHandlerFromRegistry("regular");
		```

4. 實際效果

	> 原本無法成功取得，重試第三次後正常

	![2pollyretry](https://user-images.githubusercontent.com/3851540/51261570-76a45180-19eb-11e9-88c1-ddbb0e167662.png)


* 實作 jitter 策略

	> jitter 是讓重試間隔加入隨機的時間，避免大量重試行為同時發生

	```cs
	Random jitterer = new Random(); 
	Policy
	.Handle<HttpResponseException>()
	//預設重試間隔為 2 的 {重試次數} 次方秒，再加入一個隨機時間以錯開重試動作
	.WaitAndRetry(5,  retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))  + TimeSpan.FromMilliseconds(jitterer.Next(0, 100)));
	```

## 心得
從 .NET Core 與 ASP.NET Core 問世以來，我不時會被官方文件搞混，尤其是這次的主角 HttpClientFactory 我特別有感覺：就是文件常常會有兩者混用的問題，以 HttpClientFactory 為例，到底是 .NET Core 2.1 加入還是 ASP.NET Core 2.1 加入，目標到底是 .NET Core 還是 ASP.NET Core 

- .NET Core

	![3dotnetcore](https://user-images.githubusercontent.com/3851540/51261574-773ce800-19eb-11e9-948e-bd48791cb1fa.png)

- ASP.NET Core

	![4aspnetcore](https://user-images.githubusercontent.com/3851540/51261576-773ce800-19eb-11e9-835f-e9591e522c62.png)

	![5aspnet](https://user-images.githubusercontent.com/3851540/51261564-760bbb00-19eb-11e9-8426-f603c0c04e8b.png)


這篇筆記關於 HttpClient 的部份雖然還不完整 (缺了 HttpMessageHandler、logging、lifetime 管理)，但因為已經拖快兩個月了 最後決定先完成基本常用功能的介紹其他功能留在之後再補，不然我怕筆記可能永遠只會是草稿了 XD


# 參考資料
1. [HttpClientFactory in ASP.NET Core 2.1 (Part 1)](https://www.stevejgordon.co.uk/introduction-to-httpclientfactory-aspnetcore)
2. [The Outgoing Request Middleware Pipeline with Handlers](https://www.stevejgordon.co.uk/httpclientfactory-aspnetcore-outgoing-request-middleware-pipeline-delegatinghandlers)
3. [Initiate HTTP requests](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-requests?view=aspnetcore-2.1&WT.mc_id=DOP-MVP-5002594)
4. [Use HttpClientFactory from .NET 4.6.2](https://stackoverflow.com/questions/51498896/use-httpclientfactory-from-net-4-6-2)
5. [Use HttpClientFactory to implement resilient HTTP requests](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests?WT.mc_id=DOP-MVP-5002594)
6. [Singleton HttpClient doesn't respect DNS changes](https://github.com/dotnet/corefx/issues/11224)
7. [Implement HTTP call retries with exponential backoff with HttpClientFactory and Polly policies](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly?WT.mc_id=DOP-MVP-5002594)
8. [探討 HttpClient 可能的問題](/httpclient-issue/)
9. [HttpClient 無法反應 DNS 異動的解決方式](/httpclient-not-respect-dns-change) 
10. [在 ASP.NET MVC 5 中使用 ASP.NET Core Dependency Injection 與 HttpClientFactory](/aspnet-core-di-httpclientfactory-in-aspnet-mvc-5)
