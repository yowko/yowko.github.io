---
title: "towrite"
date: 2017-06-14T21:30:00+08:00
lastmod: 2018-12-15T21:30:31+08:00
draft: true
tags: ["C#"]
slug: "towrite"
---
- GetAwaiter
    https://www.cnblogs.com/cmt/p/configure_await_false.html
- ConfigureAwait

    https://github.com/dotnet/corefx/issues/31017?fbclid=IwAR2eI7g6q-V-D6TDflzUxqXet6ad-OLdj_pIqD-hJy_9qxXS9h5AmJUSHrc

    https://msdn.microsoft.com/en-us/magazine/jj991977.aspx
- avoid return await --> why?
    - not in using or try/catch block


# .net framework use azure signalr service
https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom


# signalr custom connection id

# IIS 部署錯誤
https://scottsauber.com/2017/04/10/how-to-troubleshoot-an-error-occurred-while-starting-the-application-in-asp-net-core-on-iis/

# user secret deploy
https://www.humankode.com/asp-net-core/asp-net-core-configuration-best-practices-for-keeping-secrets-out-of-source-control



* azure function - unsafe dll load exception

* azure function - app settings


* azure function - aad protect
https://peteskelly.com/secure-functions-aad-2/


* 修改定序失敗

```
Msg 5030, Level 16, State 5, Line 4
The database could not be exclusively locked to perform the operation.
Msg 5072, Level 16, State 1, Line 4
ALTER DATABASE failed. The default collation of database 'Test' cannot be set to Chinese_Taiwan_Stroke_CI_AS.
```
http://www.voidcn.com/article/p-gcqaxlku-pp.html


* 中文亂碼


## C# GRPC


## IsAssignableFrom vs getinterfaces.contains
http://bbs.bugcode.cn/t/2989
https://www.hanselman.com/blog/DoesATypeImplementAnInterface.aspx
https://theburningmonk.com/2011/03/type-issubclssof-and-type-isassignablefrom/

* gitlab can't merge before build success

* dapper get subclass list

~~* build redis desktop manager on mac~~
* C# grpc



* get all assembly class
* https://stackoverflow.com/questions/30763207/get-private-property-of-a-private-property-using-reflection
* https://social.msdn.microsoft.com/Forums/vstudio/en-US/71a75f62-61e5-4f3b-87e5-f126bf5c80d2/get-list-of-all-assemblies-in-application-in-a-core-20-application?forum=csharpgeneral
* https://stackoverflow.com/questions/42524704/asp-net-core-find-all-class-types-in-all-assemblies


~~* httpclientfactory~~
https://twitter.com/davidfowl/status/866929855740264448


# mac install kafka
https://medium.com/@Ankitthakur/apache-kafka-installation-on-mac-using-homebrew-a367cdefd273

# generate poco from cassandra

# C# cassadra


https://edi.wang/post/2018/11/25/netcore-webutility-urlencode-httputility-urlencode



~~httpclient proxy https~~
https://dzone.com/articles/udts-in-cassandra-simplified-1



# redis di
https://stackoverflow.com/questions/46368234/using-stackexchange-redis-in-a-asp-net-core-controller


# redis lua di 



# gRPC repeated 

# gRPC timestamp mapping


# chrome extension:Simple Docker UI


# datetime to timespan
https://stackoverflow.com/questions/17959440/convert-datetime-to-timespan


# get current test from setup and teardown
https://groups.google.com/forum/#!topic/nunit-discuss/JylMVpiFHDw

# nsubstitute internal can't create proxy



# no space left

CT Tsai - YoungOG [2:51 PM]
docker system prune -a

清除所有
- stopped容器
- unused的volumes
- unused的networks
- unused 的images (edited) 


# .gitignore

Install gitignore command line tool for terminal
Mac OS

Bash

$ echo "function gi() { curl -L -s https://www.gitignore.io/api/\$@ ;}" >> ~/.bashrc && source ~/.bashrc

ZSH

$ echo "function gi() { curl -L -s https://www.gitignore.io/api/\$@ ;}" >> ~/.zshrc && source ~/.zshrc

產生 .gitignore
執行
gi cake,visualstudio,jetbrains+all,visualstudiocode > .gitignore


# polly log
https://www.stevejgordon.co.uk/polly-using-context-to-obtain-retry-count-diagnostics


# return task
            return await Task.Run(() => true);
                        return   Task.FromResult(false);


~~# single node mongodb transaction~~

~~# kafka-manager~~

~~# test race condition~~
~~TestContext.CurrentContext.Test.Name~~


# protainer vs simple dockerui


~~# asp.net core startup kestrel log~~

# .net core env
https://stackoverflow.com/questions/32548948/how-to-get-the-development-staging-production-hosting-environment-in-configurese



# gRPC any

~~# gRPC timestamp~~

# mac link extensio to applicaion
https://www.laptopmag.com/articles/how-to-change-default-applications-mac




# gRPC jaeger
https://github.com/opentracing-contrib/csharp-grpc/blob/master/getting_started/README.md

~~# gRPC zipkin~~
https://github.com/grpc/grpc/pull/12613

~~# gRPC opentracing~~

~~# gRPC skywalking~~



# actionfilter DI
http://www.dalbll.com/Group/Topic/ASP.NET/6333

# middleware httpclientfactory


# async actionfilter
https://www.c-sharpcorner.com/article/working-with-filters-in-asp-net-core-mvc/


# actionfilter test

# middleware test

# actionfilter
https://www.one-tab.com/page/qhPIkdL9RGKOnFqR1V-ZsA


# postman ssl 
https://blog.csdn.net/u011466469/article/details/78377859

# jaeger asp.net core
https://www.cnblogs.com/catcher1994/p/10662999.html

# Consul-Template
https://yq.aliyun.com/articles/691311


https://www.ibm.com/developerworks/cn/cloud/library/cl-plug-and-play-service-discovery-with-consul-and-docker-bluemix/index.html

https://juejin.im/entry/59be80975188257e7a42819a

http://devopstarter.info/service-discovery-with-docker-and-consul-part-1/

https://www.jianshu.com/p/0e14ec06797f


# asp.net core configure cann't use configure[""]

# asp.net core set config
https://stackoverflow.com/a/52405277
https://adrientorris.github.io/aspnet-core/manage-base64-encoding.html

# json.net set jobject to a jobject
https://stackoverflow.com/a/48171460


# launchsetting ? 用途


# remove sunmodule
https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule
https://www.jianshu.com/p/ed0cb6c75e25
https://blog.csdn.net/pcjustin/article/details/79350987


# jaeger span trace
https://www.one-tab.com/page/WKmyH1KiRt6oH1o59MQcnw


# 程式關閉 .net core 
http://www.dalbll.com/Group/Topic/ASP.NET/7315
https://thinkrethink.net/2017/03/09/application-shutdown-in-asp-net-core/


# docker-compose pass env
https://stackoverflow.com/questions/49293967/how-to-pass-environment-variable-to-docker-compose-up