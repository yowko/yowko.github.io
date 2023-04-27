---
title: "再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能 (Streaming)"
date: 2023-04-27T00:30:00+08:00
lastmod: 2023-04-27T00:30:31+08:00
draft: false
tags: ["csharp","grpc","dotnet","aspdotnetcore"]
slug: "grpc-aspdotnetcore7-json-streaming"
---

## 再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能 (Streaming)

之前筆記 [再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json-post) 紀錄到如何使用 ASP.NET Core 7 加入的 JSON 轉碼功能讓 gRPC service 與 rest api 有接近的使用體驗：使用 POST 方法搭配 JSON 來發送 request (取代原本使用 GET 方法並透過 URL parameter 來傳遞參數)

今天就來額外測試 streaming 功能，官方文件 [gRPC JSON transcoding in ASP.NET Core gRPC apps](https://learn.microsoft.com/en-us/aspnet/core/grpc/json-transcoding?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594) 有提到 gRPC 支援 client streaming,server streaming 跟 bidirectional streaming 三種 streaming，但 ASP.NET Core 7 的 JSON 轉碼功能僅支援 server streaming

## 基本環境說明

1. macOS Ventura 13.2
2. .NET SDK 7.0.203
3. JetBrains Rider 2023.1

    > 使用 gRPC service 預設專案範本

4. NuGet packages
    - Microsoft.AspNetCore.Grpc.JsonTranscoding 7.0.5

## 設定方式

1. 新增 streaming 用的 proto：`example.proto`

    ```proto
    syntax = "proto3";
    option csharp_namespace = "NET7GrpcService";
    
    package greet;
    
    service ExampleService {
      // Unary
      rpc UnaryCall (ExampleRequest) returns (ExampleResponse);
    
      // Server streaming
      rpc StreamingFromServer (ExampleRequest) returns;
    
      // Client streaming
      rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);
    
      // Bi-directional streaming
      rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
    }
    
    message ExampleRequest {
      int32 pageIndex = 1;
      int32 pageSize = 2;
      bool isDescending = 3;
    }
    
    message ExampleResponse {
      string text = 1;
    }
    ```

2. 新增 server streaming 實作

    ```cs
    using Grpc.Core;

    namespace NET7GrpcService.Services;
    
    public class ExampleService:NET7GrpcService.ExampleService.ExampleServiceBase
    {
        public override async Task StreamingFromServer(ExampleRequest request, IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
        {
            for (var i = 1; i <= 3; i++)
            {
                await responseStream.WriteAsync(new ExampleResponse { Text = $"Message {i}" });
                await Task.Delay(TimeSpan.FromSeconds(1));
            }
        }
    }
    ```

3. 註冊 streaming service

    - 前

        ```cs
        app.MapGrpcService<GreeterService>();
        app.MapGet("/",
            () =>
                "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

        ```

    - 後

        ```cs
        app.MapGrpcService<GreeterService>();
        //加入下面這一行
        app.MapGrpcService<ExampleService>();
        app.MapGet("/",
            () =>
                "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

        ```

4. 將 NuGet package `Microsoft.AspNetCore.Grpc.JsonTranscoding` 加入 project
5. 註冊轉碼功能：修改 `Program.cs`

    - 前

        ```cs
        builder.Services.AddGrpc();
        ```

    - 後

        ```cs
        builder.Services.AddGrpc().AddJsonTranscoding();
        ```

6. 新增 `google/api/http.proto` 與 `google/api/annotations.proto`

    > 引用額外的資料結構，以預設專案的例子可以在 `Protos` 資料下新增 `google/api` 並放入以下兩個檔案 (檔案位置會影響實際引用的設定，另個做法是與 .proj 放在同一層，引用填 .proj 的相對位置即可)

    - [http.proto](https://github.com/dotnet/aspnetcore/blob/main/src/Grpc/JsonTranscoding/test/testassets/Sandbox/google/api/http.proto)

        <details>

        <summary style="color: LightGreen;">展開摺疊區塊</summary>

        ```proto
        // Copyright 2019 Google LLC.
        //
        // Licensed under the Apache License, Version 2.0 (the "License");
        // you may not use this file except in compliance with the License.
        // You may obtain a copy of the License at
        //
        //     http://www.apache.org/licenses/LICENSE-2.0
        //
        // Unless required by applicable law or agreed to in writing, software
        // distributed under the License is distributed on an "AS IS" BASIS,
        // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        // See the License for the specific language governing permissions and
        // limitations under the License.
        //

        syntax = "proto3";

        package google.api;

        option cc_enable_arenas = true;
        option go_package = "google.golang.org/genproto/googleapis/api/annotations;annotations";
        option java_multiple_files = true;
        option java_outer_classname = "HttpProto";
        option java_package = "com.google.api";
        option objc_class_prefix = "GAPI";
        
        // Defines the HTTP configuration for an API service. It contains a list of
        // [HttpRule][google.api.HttpRule], each specifying the mapping of an RPC method
        // to one or more HTTP REST API methods.
        message Http {
          // A list of HTTP configuration rules that apply to individual API methods.
          //
          // **NOTE:** All service configuration rules follow "last one wins" order.
          repeated HttpRule rules = 1;
        
          // When set to true, URL path parameters will be fully URI-decoded except in
          // cases of single segment matches in reserved expansion, where "%2F" will be
          // left encoded.
          //
          // The default behavior is to not decode RFC 6570 reserved characters in multi
          // segment matches.
          bool fully_decode_reserved_expansion = 2;
        }
        
        // # gRPC Transcoding
        //
        // gRPC Transcoding is a feature for mapping between a gRPC method and one or
        // more HTTP REST endpoints. It allows developers to build a single API service
        // that supports both gRPC APIs and REST APIs. Many systems, including [Google
        // APIs](https://github.com/googleapis/googleapis),
        // [Cloud Endpoints](https://cloud.google.com/endpoints), [gRPC
        // Gateway](https://github.com/grpc-ecosystem/grpc-gateway),
        // and [Envoy](https://github.com/envoyproxy/envoy) proxy support this feature
        // and use it for large scale production services.
        //
        // `HttpRule` defines the schema of the gRPC/REST mapping. The mapping specifies
        // how different portions of the gRPC request message are mapped to the URL
        // path, URL query parameters, and HTTP request body. It also controls how the
        // gRPC response message is mapped to the HTTP response body. `HttpRule` is
        // typically specified as an `google.api.http` annotation on the gRPC method.
        //
        // Each mapping specifies a URL path template and an HTTP method. The path
        // template may refer to one or more fields in the gRPC request message, as long
        // as each field is a non-repeated field with a primitive (non-message) type.
        // The path template controls how fields of the request message are mapped to
        // the URL path.
        //
        // Example:
        //
        //     service Messaging {
        //       rpc GetMessage(GetMessageRequest) returns (Message) {
        //         option (google.api.http) = {
        //             get: "/v1/{name=messages/*}"
        //         };
        //       }
        //     }
        //     message GetMessageRequest {
        //       string name = 1; // Mapped to URL path.
        //     }
        //     message Message {
        //       string text = 1; // The resource content.
        //     }
        //
        // This enables an HTTP REST to gRPC mapping as below:
        //
        // HTTP | gRPC
        // -----|-----
        // `GET /v1/messages/123456`  | `GetMessage(name: "messages/123456")`
        //
        // Any fields in the request message which are not bound by the path template
        // automatically become HTTP query parameters if there is no HTTP request body.
        // For example:
        //
        //     service Messaging {
        //       rpc GetMessage(GetMessageRequest) returns (Message) {
        //         option (google.api.http) = {
        //             get:"/v1/messages/{message_id}"
        //         };
        //       }
        //     }
        //     message GetMessageRequest {
        //       message SubMessage {
        //         string subfield = 1;
        //       }
        //       string message_id = 1; // Mapped to URL path.
        //       int64 revision = 2;    // Mapped to URL query parameter `revision`.
        //       SubMessage sub = 3;    // Mapped to URL query parameter `sub.subfield`.
        //     }
        //
        // This enables a HTTP JSON to RPC mapping as below:
        //
        // HTTP | gRPC
        // -----|-----
        // `GET /v1/messages/123456?revision=2&sub.subfield=foo` |
        // `GetMessage(message_id: "123456" revision: 2 sub: SubMessage(subfield:
        // "foo"))`
        //
        // Note that fields which are mapped to URL query parameters must have a
        // primitive type or a repeated primitive type or a non-repeated message type.
        // In the case of a repeated type, the parameter can be repeated in the URL
        // as `...?param=A&param=B`. In the case of a message type, each field of the
        // message is mapped to a separate parameter, such as
        // `...?foo.a=A&foo.b=B&foo.c=C`.
        //
        // For HTTP methods that allow a request body, the `body` field
        // specifies the mapping. Consider a REST update method on the
        // message resource collection:
        //
        //     service Messaging {
        //       rpc UpdateMessage(UpdateMessageRequest) returns (Message) {
        //         option (google.api.http) = {
        //           patch: "/v1/messages/{message_id}"
        //           body: "message"
        //         };
        //       }
        //     }
        //     message UpdateMessageRequest {
        //       string message_id = 1; // mapped to the URL
        //       Message message = 2;   // mapped to the body
        //     }
        //
        // The following HTTP JSON to RPC mapping is enabled, where the
        // representation of the JSON in the request body is determined by
        // protos JSON encoding:
        //
        // HTTP | gRPC
        // -----|-----
        // `PATCH /v1/messages/123456 { "text": "Hi!" }` | `UpdateMessage(message_id:
        // "123456" message { text: "Hi!" })`
        //
        // The special name `*` can be used in the body mapping to define that
        // every field not bound by the path template should be mapped to the
        // request body.  This enables the following alternative definition of
        // the update method:
        //
        //     service Messaging {
        //       rpc UpdateMessage(Message) returns (Message) {
        //         option (google.api.http) = {
        //           patch: "/v1/messages/{message_id}"
        //           body: "*"
        //         };
        //       }
        //     }
        //     message Message {
        //       string message_id = 1;
        //       string text = 2;
        //     }
        //
        //
        // The following HTTP JSON to RPC mapping is enabled:
        //
        // HTTP | gRPC
        // -----|-----
        // `PATCH /v1/messages/123456 { "text": "Hi!" }` | `UpdateMessage(message_id:
        // "123456" text: "Hi!")`
        //
        // Note that when using `*` in the body mapping, it is not possible to
        // have HTTP parameters, as all fields not bound by the path end in
        // the body. This makes this option more rarely used in practice when
        // defining REST APIs. The common usage of `*` is in custom methods
        // which don't use the URL at all for transferring data.
        //
        // It is possible to define multiple HTTP methods for one RPC by using
        // the `additional_bindings` option. Example:
        //
        //     service Messaging {
        //       rpc GetMessage(GetMessageRequest) returns (Message) {
        //         option (google.api.http) = {
        //           get: "/v1/messages/{message_id}"
        //           additional_bindings {
        //             get: "/v1/users/{user_id}/messages/{message_id}"
        //           }
        //         };
        //       }
        //     }
        //     message GetMessageRequest {
        //       string message_id = 1;
        //       string user_id = 2;
        //     }
        //
        // This enables the following two alternative HTTP JSON to RPC mappings:
        //
        // HTTP | gRPC
        // -----|-----
        // `GET /v1/messages/123456` | `GetMessage(message_id: "123456")`
        // `GET /v1/users/me/messages/123456` | `GetMessage(user_id: "me" message_id:
        // "123456")`
        //
        // ## Rules for HTTP mapping
        //
        // 1. Leaf request fields (recursive expansion nested messages in the request
        //    message) are classified into three categories:
        //    - Fields referred by the path template. They are passed via the URL path.
        //    - Fields referred by the [HttpRule.body][google.api.HttpRule.body]. They are passed via the HTTP
        //      request body.
        //    - All other fields are passed via the URL query parameters, and the
        //      parameter name is the field path in the request message. A repeated
        //      field can be represented as multiple query parameters under the same
        //      name.
        //  2. If [HttpRule.body][google.api.HttpRule.body] is "*", there is no URL query parameter,all fields
        //     are passed via URL path and HTTP request body.
        //  3. If [HttpRule.body][google.api.HttpRule.body] is omitted, there is no HTTP request body,all
        //     fields are passed via URL path and URL query parameters.
        //
        // ### Path template syntax
        //
        //     Template = "/" Segments [ Verb ] ;
        //     Segments = Segment { "/" Segment } ;
        //     Segment  = "*" | "**" | LITERAL | Variable ;
        //     Variable = "{" FieldPath [ "=" Segments ] "}" ;
        //     FieldPath = IDENT { "." IDENT } ;
        //     Verb     = ":" LITERAL ;
        //
        // The syntax `*` matches a single URL path segment. The syntax `**` matches
        // zero or more URL path segments, which must be the last part of the URL path
        // except the `Verb`.
        //
        // The syntax `Variable` matches part of the URL path as specified by its
        // template. A variable template must not contain other variables. If a variable
        // matches a single path segment, its template may be omitted, e.g. `{var}`
        // is equivalent to `{var=*}`.
        //
        // The syntax `LITERAL` matches literal text in the URL path. If the `LITERAL`
        // contains any reserved character, such characters should be percent-encoded
        // before the matching.
        //
        // If a variable contains exactly one path segment, such as `"{var}"` or
        // `"{var=*}"`, when such a variable is expanded into a URL path on the client
        // side, all characters except `[-_.~0-9a-zA-Z]` are percent-encoded. The
        // server side does the reverse decoding. Such variables show up in the
        // [Discovery
        // Document](https://developers.google.com/discovery/v1/reference/apis) as
        // `{var}`.
        //
        // If a variable contains multiple path segments, such as `"{var=foo/*}"`
        // or `"{var=**}"`, when such a variable is expanded into a URL path on the
        // client side, all characters except `[-_.~/0-9a-zA-Z]` are percent-encoded.
        // The server side does the reverse decoding, except "%2F" and "%2f" are left
        // unchanged. Such variables show up in the
        // [Discovery
        // Document](https://developers.google.com/discovery/v1/reference/apis) as
        // `{+var}`.
        //
        // ## Using gRPC API Service Configuration
        //
        // gRPC API Service Configuration (service config) is a configuration language
        // for configuring a gRPC service to become a user-facing product. The
        // service config is simply the YAML representation of the `google.api.Service`
        // proto message.
        //
        // As an alternative to annotating your proto file, you can configure gRPC
        // transcoding in your service config YAML files. You do this by specifying a
        // `HttpRule` that maps the gRPC method to a REST endpoint, achieving the same
        // effect as the proto annotation. This can be particularly useful if you
        // have a proto that is reused in multiple services. Note that any transcoding
        // specified in the service config will override any matching transcoding
        // configuration in the proto.
        //
        // Example:
        //
        //     http:
        //       rules:
        //         # Selects a gRPC method and applies HttpRule to it.
        //         - selector: example.v1.Messaging.GetMessage
        //           get: /v1/messages/{message_id}/{sub.subfield}
        //
        // ## Special notes
        //
        // When gRPC Transcoding is used to map a gRPC to JSON REST endpoints, the
        // proto to JSON conversion must follow the [proto3
        // specification](https://developers.google.com/protocol-buffers/docs/proto3#json).
        //
        // While the single segment variable follows the semantics of
        // [RFC 6570](https://tools.ietf.org/html/rfc6570) Section 3.2.2 Simple String
        // Expansion, the multi segment variable **does not** follow RFC 6570 Section
        // 3.2.3 Reserved Expansion. The reason is that the Reserved Expansion
        // does not expand special characters like `?` and `#`, which would lead
        // to invalid URLs. As the result, gRPC Transcoding uses a custom encoding
        // for multi segment variables.
        //
        // The path variables **must not** refer to any repeated or mapped field,
        // because client libraries are not capable of handling such variable expansion.
        //
        // The path variables **must not** capture the leading "/" character. The reason
        // is that the most common use case "{var}" does not capture the leading "/"
        // character. For consistency, all path variables must share the same behavior.
        //
        // Repeated message fields must not be mapped to URL query parameters, because
        // no client library can support such complicated mapping.
        //
        // If an API needs to use a JSON array for request or response body, it can map
        // the request or response body to a repeated field. However, some gRPC
        // Transcoding implementations may not support this feature.
        message HttpRule {
          // Selects a method to which this rule applies.
          //
          // Refer to [selector][google.api.DocumentationRule.selector] for syntax details.
          string selector = 1;
        
          // Determines the URL pattern is matched by this rules. This pattern can be
          // used with any of the {get|put|post|delete|patch} methods. A custom method
          // can be defined using the 'custom' field.
          oneof pattern {
            // Maps to HTTP GET. Used for listing and getting information about
            // resources.
            string get = 2;
        
            // Maps to HTTP PUT. Used for replacing a resource.
            string put = 3;
        
            // Maps to HTTP POST. Used for creating a resource or performing an action.
            string post = 4;
        
            // Maps to HTTP DELETE. Used for deleting a resource.
            string delete = 5;
        
            // Maps to HTTP PATCH. Used for updating a resource.
            string patch = 6;
        
            // The custom pattern is used for specifying an HTTP method that is not
            // included in the `pattern` field, such as HEAD, or "*" to leave the
            // HTTP method unspecified for this rule. The wild-card rule is useful
            // for services that provide content to Web (HTML) clients.
            CustomHttpPattern custom = 8;
          }
        
          // The name of the request field whose value is mapped to the HTTP request
          // body, or `*` for mapping all request fields not captured by the path
          // pattern to the HTTP body, or omitted for not having any HTTP request body.
          //
          // NOTE: the referred field must be present at the top-level of the request
          // message type.
          string body = 7;
        
          // Optional. The name of the response field whose value is mapped to the HTTP
          // response body. When omitted, the entire response message will be used
          // as the HTTP response body.
          //
          // NOTE: The referred field must be present at the top-level of the response
          // message type.
          string response_body = 12;
        
          // Additional HTTP bindings for the selector. Nested bindings must
          // not contain an `additional_bindings` field themselves (that is,
          // the nesting may only be one level deep).
          repeated HttpRule additional_bindings = 11;
        }
        
        // A custom pattern is used for defining custom HTTP verb.
        message CustomHttpPattern {
          // The name of this custom HTTP verb.
          string kind = 1;
        
          // The path matched by this custom verb.
          string path = 2;
        }
        ```

        </details>

    - [annotations.proto](https://github.com/dotnet/aspnetcore/blob/main/src/Grpc/JsonTranscoding/test/testassets/Sandbox/google/api/annotations.proto)

        <details>

        <summary style="color:LightGreen">展開摺疊區塊</summary>

        ```proto
        // Copyright (c) 2015, Google Inc.
        //
        // Licensed under the Apache License, Version 2.0 (the "License");
        // you may not use this file except in compliance with the License.
        // You may obtain a copy of the License at
        //
        //     http://www.apache.org/licenses/LICENSE-2.0
        //
        // Unless required by applicable law or agreed to in writing, software
        // distributed under the License is distributed on an "AS IS" BASIS,
        // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        // See the License for the specific language governing permissions and
        // limitations under the License.
        
        syntax = "proto3";
        
        package google.api;
        
        import "Protos/google/api/http.proto";
        import "google/protobuf/descriptor.proto";
        
        option go_package = "google.golang.org/genproto/googleapis/api/annotations;annotations";
        option java_multiple_files = true;
        option java_outer_classname = "AnnotationsProto";
        option java_package = "com.google.api";
        option objc_class_prefix = "GAPI";
        
        extend google.protobuf.MethodOptions {
          // See `HttpRule`.
          HttpRule http = 72295728;
        }
        ```

        </details>

7. 透過 HTTP binding 與 route 來標注 gRPC service

    > 修改 `example.proto` 檔案

    - 前

        ```proto
        syntax = "proto3";
        option csharp_namespace = "NET7GrpcService";
        
        package greet;
        
        service ExampleService {
          // Unary
          rpc UnaryCall (ExampleRequest) returns (ExampleResponse);
        
          // Server streaming
          rpc StreamingFromServer (ExampleRequest) returns (stream ExampleResponse);
        
          // Client streaming
          rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);
        
          // Bi-directional streaming
          rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
        }
        
        message ExampleRequest {
          int32 pageIndex = 1;
          int32 pageSize = 2;
          bool isDescending = 3;
        }
        
        message ExampleResponse {
          string text = 1;
        }
        ```

    - 後

        ```proto
        syntax = "proto3";
        //以實際放的位置為主
        import "Protos/google/api/annotations.proto";
        option csharp_namespace = "NET7GrpcService";
        
        package greet;

        service ExampleService {
          // Unary
          rpc UnaryCall (ExampleRequest) returns (ExampleResponse);
        
          // Server streaming
          rpc StreamingFromServer (ExampleRequest) returns (stream ExampleResponse){
            option (google.api.http) = {
              post: "/ExampleService/StreamingFromServer",
              body: "*"
            };
          };
        
          // Client streaming
          rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);
        
          // Bi-directional streaming
          rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
        }

        message ExampleRequest {
          int32 pageIndex = 1;
          int32 pageSize = 2;
          bool isDescending = 3;
        }
        
        message ExampleResponse {
          string text = 1;
        }
        ```

8. (<span style="color: red;">非必要</span>，可在 macOS 上的啟用 insecure grpc) 設定 gRPC 與 rest api 走不同 port

    > 修改 appsettings.json

    - 前

        ```json
        {
          "Logging": {
            "LogLevel": {
              "Default": "Information",
              "Microsoft.AspNetCore": "Warning"
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

    - 後

        ```json
        {
          "Logging": {
            "LogLevel": {
              "Default": "Information",
              "Microsoft.AspNetCore": "Warning"
            }
          },
          "AllowedHosts": "*",
          "Kestrel": {
            "Endpoints": {
              "Https": {
                "Url": "https://*:7019",
                "Protocols": "Http1"
              },
              "GrpcInsecure" : {
                "Url": "http://*:5187",
                "Protocols": "Http2"
              }
            }
          }
        }
        ```

9. 實際效果

    > 兩者 port 不同，是因為 macOS 的源故

    - gRPC

        ![1grpc](https://user-images.githubusercontent.com/3851540/234550882-73986f03-05e9-4170-b61a-2dfd94bf9357.gif)

    - rest api

        ![2rest](https://user-images.githubusercontent.com/3851540/234550908-4e5d15f1-401f-41d9-b684-dedfac220fbf.gif)

## 心得

雖然 ASP.NET Core 7 的 JSON 轉碼功能僅支援 server streaming，讓實際使用上有些受限，過還是多了個應用設計時不同的選項可供選擇，我想到的就是不用再自行實作 long polling 的機制與相關的 exception handling

完整程式碼： [yowko/grpc-aspdotnetcore7-json-streaming](https://github.com/yowko/grpc-aspdotnetcore7-json-streaming)

## 參考資訊

1. [.NET 7 的新功能](https://learn.microsoft.com/zh-tw/dotnet/core/whats-new/dotnet-7?WT.mc_id=DOP-MVP-5002594)
2. [ASP.NET Core 7.0 的新功能](https://learn.microsoft.com/zh-tw/aspnet/core/release-notes/aspnetcore-7.0?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
3. [ASP.NET Core gRPC 應用程式中的 gRPC JSON 轉碼](https://learn.microsoft.com/en-us/aspnet/core/grpc/json-transcoding?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
4. [gRPC JSON 轉碼與 Swagger/OpenAPI](https://learn.microsoft.com/en-us/aspnet/core/grpc/json-transcoding-openapi?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594)
5. [Build High Performance Services using gRPC and .NET7](https://medium.com/geekculture/build-high-performance-services-using-grpc-and-net7-7c0c434abbb0)
6. [針對 .NET Core 上的 gRPC 進行疑難排解](https://learn.microsoft.com/en-us/aspnet/core/grpc/troubleshoot?view=aspnetcore-7.0&WT.mc_id=DOP-MVP-5002594#unable-to-start-aspnet-core-grpc-app-on-macos)
7. [yowko/grpc-aspdotnetcore7-json-streaming](https://github.com/yowko/grpc-aspdotnetcore7-json-streaming)
8. [ASP.NET Core gRPC 的 Secure 與 Insecure 不同做法](/aspdotnetcore-grpc-secure-insecure/)
9. [gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json)
10. [gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能 (macOS)](/grpc-aspdotnetcore7-json-macos)
11. [再探 gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json-post)
