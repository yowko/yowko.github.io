---
title: "C# Socket 使用 proxy 連線"
date: 2021-08-23T12:30:00+08:00
lastmod: 2021-08-23T12:30:00+08:00
draft: false
tags: ["csharp","Network"]
slug: "csharp-socket-proxy"
---

## C# Socket 使用 proxy 連線

合作的 partner 在資料介接上提供 socket 的接口來確保資料更新的即時性，但為了有基本安全性所以只允許 whitelist server 可以連線，這在 production server 是很常見的限制，甚至在互相允許的測試環境也是合理的，但在開發階段這樣的安全性要求就顯得有些窒礙難行，所以打算透過 proxy server 來連線做開發，連同測試環境也透過 proxy 來跟 partner 做溝通

因為過去我沒有 socket 相關經驗，所以花了點時間做了簡單的 socket client/server 來協助測試開發，不過也擔心自己寫的程式無法反應出實際狀況，姑且走一步算一步，先求有再求好囉

最近團隊的 proxy 多數都使用 goproxy，今天也會以 goproxy 做範例，對於 goproxy 的詳細使用說明也可以參考之前筆記：[使用 goproxy](/goproxy)

## 基本環境說明

1. macOS Big Sur 11.5.1
2. .NET Core SDK 5.0.202
3. docker images

    - snail007/goproxy:v11.0

4. NuGet packages

    - ProxySocket 1.1.2

5. socket client/server

    > 詳細內容可以參考 [yowko/SocketProxyDemo](https://github.com/yowko/SocketProxyDemo)

    - server Program.cs

        ```cs
        using System;
        using System.Net;
        using System.Net.Sockets;
        using System.Text;
        using System.Threading;
        
        namespace SocketServer
        {
            public class ThreadWork
            {
                public static void Send(object obj)
                {
                    var client = ((Socket)obj);
                    while (true)
                    {
                        var input = Console.ReadLine();
                        if (input == "exit")
                        {
                            client.Close();
                            Environment.Exit(0);
                        }
        
                        Console.WriteLine("You: " + input);
        
                        client.Send(Encoding.UTF8.GetBytes(input));
                    }
                }
        
                public static void Receive(object obj)
                {
                    var client = ((Socket)obj);
                    while (true)
                    {
                        var data = new byte[1024];
        
                        var receiveData = client.Receive(data);
        
                        if (receiveData == 0)
                        {
                            Console.WriteLine("Disconnected from {0}", client.RemoteEndPoint);
        
                            break;
                        }
        
        
                        Console.WriteLine("Client: " + Encoding.UTF8.GetString(data, 0,         receiveData));
                    }
        
                    Environment.Exit(0);
                }
            }
        
            internal class Program
            {
                private static void Main(string[] args)
                {
                    Console.OutputEncoding = Encoding.UTF8;
        
                    var ipEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 9050);
        
                    var newSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream,         ProtocolType.Tcp);
        
                    newSocket.Bind(ipEndPoint);
        
                    newSocket.Listen(10);
        
        
                    AcceptClient(newSocket);
                }
        
                private static void AcceptClient(Socket newSocket)
                {
                    Console.WriteLine("Waiting for a client...");
        
                    var client = newSocket.Accept();
        
                    var clientEndpoint = (IPEndPoint)client.RemoteEndPoint;
        
                    Console.WriteLine("Connected with {0} at port {1}", clientEndpoint.        Address, clientEndpoint.Port);
        
                    const string welcome = "Welcome to my test server";
        
                    var data = Encoding.UTF8.GetBytes(welcome);
        
                    client.Send(data, data.Length, SocketFlags.None);
        
        
                    var sendThread = new Thread(ThreadWork.Send);
                    var receiveThread = new Thread(ThreadWork.Receive);
        
                    sendThread.Start(client);
                    receiveThread.Start(client);
                }
            }
        }
        ```

    - client Program.cs

        ```cs
        using System;
        using System.Net;
        using System.Net.Sockets;
        using System.Text;
        using System.Threading;
        
        namespace SocketClient
        {
            public class ThreadWork
            {
                public static void Send(object obj)
                {
                    while (true)
                    {
                        var input = Console.ReadLine();
                        var server = ((Socket)obj);
        
                        if (input == "exit")
                        {
                            Console.WriteLine("Disconnecting from server...");
        
                            server.Shutdown(SocketShutdown.Both);
        
                            server.Close();
        
                            Console.WriteLine("Disconnected!");
        
                            Console.ReadLine();
                            Environment.Exit(0);
                        }
        
        
                        Console.WriteLine("You: " + input);
        
                        server.Send(Encoding.UTF8.GetBytes(input));
                    }
                }
        
                public static void Receive(object server)
                {
                    while (true)
                    {
                        var data = new byte[1024];
        
                        var receiveData = ((Socket)server).Receive(data);
        
                        var stringData = Encoding.UTF8.GetString(data, 0, receiveData);
        
                        Console.WriteLine("Server: " + stringData);
                    }
                }
            }
        
            internal class Program
            {
                private static void Main(string[] args)
                {
                    Console.OutputEncoding = Encoding.UTF8;
        
                    var data = new byte[1024];
                    var server = new Socket(AddressFamily.InterNetwork, SocketType.Stream,         ProtocolType.Tcp);
        
                    var ipEndpoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 9050);
                    
                    try
                    {
                        server.Connect(ipEndpoint);
                    }
                    catch (SocketException e)
                    {
                        Console.WriteLine("Unable to connect to server.");
        
                        Console.WriteLine(e.ToString());
        
                        return;
                    }
        
                    var receiveData = server.Receive(data);
        
                    var stringData = Encoding.UTF8.GetString(data, 0, receiveData);
        
                    Console.WriteLine(stringData);
        
                    var sendThread = new Thread(ThreadWork.Send);
                    sendThread.Start(server);
        
                    var receiveThread = new Thread(ThreadWork.Receive);
                    receiveThread.Start(server);
                }
            }
        }
        ```

## 使用方式

- client 安裝 NuGet 套件： `ProxySocket`

    ```bash
    dotnet add package ProxySocket --version 1.1.2
    ```

1. 使用 http proxy

    - proxy 設定

        ```bash
        docker run --rm -d -p 33080:33080 snail007/goproxy http -p :33080
        ```

    - 程式調整

        ```cs
        //對於 server 的 socket 改用 ProxySocket 來建立
        var server = new ProxySocket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        //指定 proxy 的 endpoint
        var proxy = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 33080);
        //將上述的 proxy endpoint 設定給 ProxySocket
        server.ProxyEndPoint = proxy;
        //指定 ProxySocket 使用的 proxy 類型
        server.ProxyType = ProxyTypes.Https;
        ```

        ![3httpcode](https://user-images.githubusercontent.com/3851540/130390159-d6c54d22-947f-4e55-bb0e-d7bb36c46f8c.png)

    - 實際效果：正常連線

        - client

            ![1httpclient](https://user-images.githubusercontent.com/3851540/130390145-47455c72-a5ab-4c02-b6be-f96295dee693.png)

        - server

            ![2httpserver](https://user-images.githubusercontent.com/3851540/130390152-709c0ed1-cc40-4aca-b7b3-822cd7bf17df.png)

        - proxy

            ![11httpproxy](https://user-images.githubusercontent.com/3851540/130390182-805ffaf1-f8fd-4f7d-bd0d-56808641c4ba.png)

2. 使用 socks5 proxy

    - proxy 設定

        ```bash
        docker run --rm -d -p 33081:33080 snail007/goproxy socks -p :33080
        ```

    - 程式調整

        ```cs
        //對於 server 的 socket 改用 ProxySocket 來建立
        var server = new ProxySocket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        //指定 proxy 的 endpoint
        var proxy = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 33081);
        //將上述的 proxy endpoint 設定給 ProxySocket
        server.ProxyEndPoint = proxy;
        //指定 ProxySocket 使用的 proxy 類型
        server.ProxyType = ProxyTypes.Socks5;
        ```

        ![4socks5code](https://user-images.githubusercontent.com/3851540/130390161-11a9b879-2f15-4301-abba-25dc6306f467.png)

    - 實際效果：正常連線

        - client

            ![5socks5client](https://user-images.githubusercontent.com/3851540/130390162-237b35a6-6d11-4a14-86dc-87a4bb6ac89e.png)

        - server

            ![6socks5server](https://user-images.githubusercontent.com/3851540/130390163-8390c35f-134b-4df2-bd8f-6858af1afd67.png)

        - proxy

            ![10socksproxy](https://user-images.githubusercontent.com/3851540/130390176-4aab27fc-36a9-4013-81f2-8ddc6cf1d050.png)

3. 不使用 proxy

    - 程式調整

        ```cs
        //對於 server 的 socket 改用 ProxySocket 來建立
        var server = new ProxySocket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        //指定 proxy 的 endpoint
        var proxy = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 33081);
        //將上述的 proxy endpoint 設定給 ProxySocket
        server.ProxyEndPoint = proxy;
        //指定 ProxySocket 使用的 proxy 類型
        server.ProxyType = ProxyTypes.None;
        ```

        ![7nonecode](https://user-images.githubusercontent.com/3851540/130390165-f94bfaf4-1097-4964-87cd-1ce1b51c277c.png)

    - 實際效果：正常連線

        - client

            ![8noneclient](https://user-images.githubusercontent.com/3851540/130390169-7483fb07-009b-4a68-82e7-0fa0508055a8.png)

        - server

            ![9noneserver](https://user-images.githubusercontent.com/3851540/130390173-f276b2a0-e3ea-4aa6-8ba0-e7effa1e1054.png)

## 心得

[poma/ProxySocket](https://github.com/poma/ProxySocket) 是個非常好用的套件，個人覺得可以將 socket 連線都換成這個套件，如果不需要 proxy 時就將 `ProxyType` 設定成 `ProxyTypes.None` 即可停用 proxy 了

另外需要特別留意的是我用了簡單的方式建立 socket client server，可能沒辦法完整模擬實際情境，在實際使用上還需要另外測試

詳細程式碼跟歷程，請參考 [yowko/SocketProxyDemo](https://github.com/yowko/SocketProxyDemo)

## 參考資訊

1. [使用 goproxy](/goproxy)
2. [yowko/SocketProxyDemo](https://github.com/yowko/SocketProxyDemo)
3. [poma/ProxySocket](https://github.com/poma/ProxySocket)
