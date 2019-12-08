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




## IsAssignableFrom vs getinterfaces.contains
http://bbs.bugcode.cn/t/2989
https://www.hanselman.com/blog/DoesATypeImplementAnInterface.aspx
https://theburningmonk.com/2011/03/type-issubclssof-and-type-isassignablefrom/

* gitlab can't merge before build success

* dapper get subclass list


* get all assembly class
* https://stackoverflow.com/questions/30763207/get-private-property-of-a-private-property-using-reflection
* https://social.msdn.microsoft.com/Forums/vstudio/en-US/71a75f62-61e5-4f3b-87e5-f126bf5c80d2/get-list-of-all-assemblies-in-application-in-a-core-20-application?forum=csharpgeneral
* https://stackoverflow.com/questions/42524704/asp-net-core-find-all-class-types-in-all-assemblies



# mac install kafka
https://medium.com/@Ankitthakur/apache-kafka-installation-on-mac-using-homebrew-a367cdefd273

# generate poco from cassandra

# C# cassadra


https://edi.wang/post/2018/11/25/netcore-webutility-urlencode-httputility-urlencode




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


# protainer vs simple dockerui


# .net core env
https://stackoverflow.com/questions/32548948/how-to-get-the-development-staging-production-hosting-environment-in-configurese



# mac link extensio to applicaion
https://www.laptopmag.com/articles/how-to-change-default-applications-mac




# actionfilter DI
http://www.dalbll.com/Group/Topic/ASP.NET/6333

# middleware httpclientfactory


# async actionfilter
https://www.c-sharpcorner.com/article/working-with-filters-in-asp-net-core-mvc/


# actionfilter test


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

# docker-compose pass env
https://stackoverflow.com/questions/49293967/how-to-pass-environment-variable-to-docker-compose-up



# grpc load test



# service discovery
https://columns.chicken-house.net/2017/12/31/microservice9-servicediscovery/#service-discovery-concepts
http://ljchen.net/2019/01/04/consul%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/



## automapper use specific profile
https://stackoverflow.com/questions/14266108/create-two-automapper-maps-between-the-same-two-object-types


# FODY


~~# unit test for asp.net core  middleware~~

http://anthonygiretti.com/2018/09/04/asp-net-core-2-1-middlewares-part2-unit-test-a-custom-middleware/


~~# asp.net core deploy to docker~~
˜

export POD_NAME=$(kubectl get pods eyewitness-seagull-jaeger-operator-746cb84b56-xpj44 -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 16686



# kubernetes with helm
https://godleon.github.io/blog/Kubernetes/k8s-Helm-Introduction/

https://whmzsu.github.io/helm-doc-zh-cn/quickstart/quickstart-zh_cn.html

http://www.iceyao.com.cn/2017/08/16/%E5%BF%AB%E9%80%9F%E4%BD%93%E9%AA%8Ckubernetes/


*** http://www.iceyao.com.cn/2017/08/16/%E5%BF%AB%E9%80%9F%E4%BD%93%E9%AA%8Ckubernetes/


# helm
yum install wget -y


wget -c https://kubernetes-helm.storage.googleapis.com/helm-v2.13.1-linux-amd64.tar.gz
tar xf helm-v2.13.1-linux-amd64.tar.gz
cp linux-amd64/helm  /usr/bin/


# kubernetes

https://itnext.io/install-kubernetes-on-bare-metal-centos7-fba40e9bb3de

https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux

https://www.psychz.net/client/question/en/turn-off-firewall-centos-7.html

https://www.itzgeek.com/how-tos/linux/centos-how-tos/ssh-passwordless-login-centos-7-rhel-7.html




# sonorqube
https://rules.sonarsource.com/csharp/tag/async-await/RSPEC-4457
https://atomaras.wordpress.com/2012/10/21/a-word-of-caution-on-argument-validation-in-async-methods/


# Asp.net core modelbinding not work
https://stackoverflow.com/questions/51689153/model-validation-net-core-2-0-web-api-not-working

# asp.net core modelbinding
value type required

# asp.net core default error json validate fail


# The difference between GetService() and GetRequiredService() in ASP.NET Core
https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/


# empty project add asp.net core webapi or mvc
https://www.iambacon.co.uk/blog/create-a-minimal-asp-net-core-2-0-mvc-web-application


# generic host grpc

# nexus docker registry

# nexus-cli


# .net core di with string
https://dev.to/justinjstark/injecting-configuration-variables-into-components-4knh

# factory DI
https://stackoverflow.com/questions/31950362/factory-method-with-di-and-ioc
https://long2know.com/2018/02/net-core-open-generics-with-factory-pattern/

Console.WriteLine($"Delivered '{dr.Value}' to '{dr.TopicPartitionOffset}'");


# private registry on gcp


# asp.net core route
https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/routing?view=aspnetcore-2.2


# test not same instance
https://github.com/aspnet/Extensions/issues/994

## testhost

## influxdb insert performance

## extension method unittest
https://social.technet.microsoft.com/wiki/contents/articles/51706.c-unit-testing-extension-methods-and-validation.aspx

## asp.net core vault
https://medium.com/getamis/vault-dynamic-credentials-fd651d6c28a9


## should i new valuetask when return?
https://dotnetcodr.com/2018/01/17/using-the-valuetask-of-t-object-in-c-7-0/
https://rubikscode.net/2018/06/11/asynchronous-programming-in-net-benefits-and-tradeoffs-of-using-valuetask/
https://stackoverflow.com/questions/43000520/why-would-one-use-taskt-over-valuetaskt-in-c
https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/


# bash 參數

# 雙向 chunk

# protobuf vs messagepack
https://github.com/dchenk/messagepack-vs-protobuf

# nuget package scan

# DELETE pv hangs
https://medium.com/@miyurz/kubernetes-deleting-resource-like-pv-with-force-and-grace-period-0-still-keeps-pvs-in-3f4ad8710e51

https://stackoverflow.com/questions/51358856/kubernetes-cant-delete-persistentvolumeclaim-pvc

# kubernetes ha
https://cpp.la/234.html

	
ansible-playbook -i inventory/k8s/inventory.ini cluster.yml -b -v -k


## how to make server time is sync
https://www.tecmint.com/execute-commands-on-multiple-linux-servers-using-pssh/

## same command to multipleserver
https://www.tecmint.com/run-commands-on-multiple-linux-servers/


## kubernetes add azure data disk / azure file

## kafka load balance message

## nexus 設定
docker run -d -p 8081:8081 -p 9082:8082 --name nexus sonatype/nexus3

docker exec -it nexus more /nexus-data/admin.password

https://help.sonatype.com/repomanager3/formats/docker-registry/hosted-repository-for-docker-%28private-registry-for-docker%29


docker login -u admin -p pass.123 localhost:8082


CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go


## helm verify

helm dry-run

## mac kestral


# kafka
https://github.com/edenhill/kafkacat

kafkacat -L -b kafka-sb-skylinetw-com:9092 -d broker 

# kubernetes
service gateway - vistio

> ingress

https://github.com/nmnellis/vistio

> deployment

- enovy

    > 

- pilot
- citadel
- mixer

# REST VS GRPC

https://dev.to/thangchung/performance-benchmark-grpc-vs-rest-in-net-core-3-preview-8-45ak




 curl -is -u "$AUTH" "$REPO_URL" --upload-file "$CHART_PACKAGE" | indent


 # grpc runtime exception catch 不到？


 "MODULE_TYPE": "BackOffice"

"MODULE_TYPE": "OperatorConsole"

https://vsupalov.com/docker-build-pass-environment-variables/

https://blog.scottchayaa.com/post/2018/11/04/docker-arg-env-variable/


2019-09-17 07:50:54.247 [1][INF][Skyline.Sports.ProductReportAggregator.Core.Services.LedgerConsumerService]Ledger Push into Reporting done
arguments:betMessageType:PlaceBet|wagerNo:6234281852534824961 

curl -i -u admin  http://192.168.1.109:8081/repository/helm-hosted/ --upload-file fluentd-forwarder-stable-0.1.1.tgz

 kubectl edit persistentvolume/local-pv6


 rm -rf /home/happy/baidu/*

  5.0.3-v1


  kubectl get pvc -n demo --no-headers -o custom-columns=":metadata.name" | xargs -n 1 kubectl -n demo delete pvc

  kubectl -n sport-betting-dev get pod --no-headers -o custom-columns=":metadata.name" | grep betting-ds-sb | xargs -I % kubectl -n sport-betting-dev exec -it % --  date -Ins


  p-ssh
  https://www.tecmint.com/run-commands-on-multiple-linux-servers/

  https://man.linuxde.net/xargs


  rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet
rm -rf /var/lib/etcd
rm -rf /usr/local/bin/kubectl
rm -rf /etc/systemd/system/calico-node.service
rm -rf /etc/systemd/system/kubelet.service
systemctl stop etcd.service
systemctl disable etcd.service
systemctl stop calico-node.service
systemctl disable calico-node.service
docker stop $(docker ps -q)
docker rm $(docker ps -a -q)
service docker restart



# redis lua -- cmsgpack /cjson


# kafka access from external
https://medium.com/@tsuyoshiushio/configuring-kafka-on-kubernetes-makes-available-from-an-external-client-with-helm-96e9308ee9f4


# .net core 3 mvc option
https://github.com/aspnet/AspNetCore/issues/9542


# kafka and zookeeper security
https://docs.confluent.io/2.0.0/kafka/zookeeper-authentication.html


# redis lua dictionary | check table | default value

# helm push

matchDisplayKey  awayDisplayKey homeDisplayKey


helm package fluentd-forwarder-dev version 0.1.2 

helm nexus-push local domain-service-reporting-repository-0.2.0.tgz --username admin

helm repo update
helm upgrade --install --debug --dry-run
helm upgrade application-product-report-aggregator skyline-nexus/application-product-report-aggregator  --install --debug --dry-run

helm upgrade reporting-repository skyline-nexus/domain-service-reporting-repository --install --debug --dry-run

helm upgrade reporting-mongo-setup skyline-nexus/domain-service-reporting-mongo-setup --install --debug --dry-run

public class
            MessageDictionaryConverter : ITypeConverter<DictionaryWithStringDecimalPair, Dictionary<string, decimal>>
        {
            public Dictionary<string, decimal> Convert(DictionaryWithStringDecimalPair source,
                Dictionary<string, decimal> destination, ResolutionContext context)
            {
                return source.Dictionary.ToDictionary(item => item.Key, item => System.Convert.ToDecimal(item.Value));
            }
        }



# shell string.join
zookeepers=$(IFS=, ; echo "${array[*]/%/2181}")

https://stackoverflow.com/questions/6426142/how-to-append-a-string-to-each-element-of-a-bash-array

sbcruser@skylineww.com / Skyline19!@#$
# shell index


# mongodb migrate
https://github.com/golang-migrate/migrate/tree/master/cmd/migrate
https://github.com/golang-migrate/migrate/blob/master/MIGRATIONS.md
https://github.com/golang-migrate/migrate
https://docs.mongodb.com/manual/reference/command/#user-commands


# docker-compose not stop
https://stackoverflow.com/questions/38546755/docker-compose-keep-container-running


# dictionary to object