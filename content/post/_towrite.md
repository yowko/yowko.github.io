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

https://blog.csdn.net/panshiqu/article/details/53788067


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






## mongodb migrate
https://github.com/golang-migrate/migrate/tree/master/cmd/migrate
https://github.com/golang-migrate/migrate/blob/master/MIGRATIONS.md
https://github.com/golang-migrate/migrate
https://docs.mongodb.com/manual/reference/command/#user-commands


## docker-compose not stop
https://stackoverflow.com/questions/38546755/docker-compose-keep-container-running

## SCD deploy

## mongo backup and restore

## mysqldump with gzip

## change helm version via kubespray

## redis cluster for lan


## nexus enable helm ,docker,nuget

## nexus push helm

plugin

## redis muti-master

dynomite
https://blog.csdn.net/qq_36666651/article/details/82900439
https://medium.com/@yenthanh/master-master-replication-high-availability-and-multithread-for-redisdb-using-dynomite-665e7111bd34

## how to exit when docker run -it
https://noob.tw/docker-basic/


## docker build pass parameter to dockerfile

## docker-compose parameter

## docker-compose environment

## conductor


## nanoserver/iis

https://blogs.iis.net/davidso/iisnano

## versioning gRPC

https://docs.microsoft.com/en-us/aspnet/core/grpc/versioning?view=aspnetcore-3.1&fbclid=IwAR3ij6H2eb0DlZly1-r_6kz-hKFLjbfcUe6vkKRqgsVyv0eSYZzUa2QXCNA

https://developers.google.com/protocol-buffers/docs/proto3#reserved

https://developers.google.com/protocol-buffers/docs/proto3#unknowns


## bash prepare mongo command
jq -c '.[] | { "q":{"displaykey":.displaykey, "langcode":.langcode},"u":{"$set":{"displayvalue":.displayvalue}},"upsert":true}' test.json


cat test.json|jq 'map({ "q":{"displaykey":.displaykey, "langcode":.langcode},"u":{"$set":{"displayvalue":.displayvalue}},"upsert":true},)'

jq 'map({ "q":{"displaykey":.displaykey, "langcode":.langcode},"u":{"$set":{"displayvalue":.displayvalue}},"upsert":true})'| \
        jq -c '. | [{"update":"localization","updates":.}]' > ${Migrate_MongoFolder}/$(date +%s)_${DATE}.up.json


jq 'map(db.localization.update( {"displaykey":.displaykey, "langcode":.langcode} } , { $set : {"displayvalue":.displayvalue} },true,true );'

db.localization.update( {"displaykey":.displaykey, "langcode":.langcode} } , { $set : {"displayvalue":.displayvalue} },true,true );




cat source.json|jq 'map("db.localization.update"({"displaykey":.displaykey, "langcode":.langcode},{"$set":{"displayvalue":.displayvalue}},true,true)'> test.json


cat source.json| jq 'map({"displaykey":.displaykey, "langcode":.langcode},{"$set":{"displayvalue":.displayvalue}})'



cat source.json|jq -c 'map({"displaykey":.displaykey, "langcode":.langcode},{"$set":{"displayvalue":.displayvalue}})' | jq 'map("db.localization.update(" + .+",true,true);")'> test.json

cat source.json|jq -c 'map({"displaykey":.displaykey, "langcode":.langcode},{"$set":{"displayvalue":.displayvalue}})' | jq -c '.[]'> test.json


cat source.json|jq -c '.[] | {"displaykey":.displaykey, "langcode":.langcode},{ $set : {"displayvalue":.displayvalue} }' 



cat source.json|jq -c '.[] | "db.localization.update(\(.displaykey)\(.langcode)" '


cat source.json|jq -c '.[] | "db.localization.update({""displaykey"":\(.displaykey), ""langcode"":\(.langcode)\" '


db.localization.update"({"displaykey":.displaykey, "langcode":.langcode},{"$set":{"displayvalue":.displayvalue}},true,true)


## bash
filename and extension 
https://tecadmin.net/how-to-extract-filename-extension-in-shell-script/



# bash shell string.join
zookeepers=$(IFS=, ; echo "${array[*]/%/2181}")

https://stackoverflow.com/questions/6426142/how-to-append-a-string-to-each-element-of-a-bash-array

sbcruser@skylineww.com / Skyline19!@#$
# bash shell index

## bash loop check curl status code
https://gist.github.com/rgl/f90ff293d56dbb0a1e0f7e7e89a81f42


## change default kafka's replicationfactor and partition
https://github.com/yahoo/kafka-manager/issues/315
https://docs.cloudera.com/documentation/kafka/latest/topics/kafka_command_line.html

## dynamic change topic's replicationfactor
https://stackoverflow.com/questions/37960767/how-to-change-the-number-of-replicas-of-a-kafka-topic
https://blog.csdn.net/yanshu2012/article/details/53761284
https://blog.csdn.net/russle/article/details/83421904
https://blog.csdn.net/lzufeng/article/details/81743521

## mongo replica set failover
https://www.reddit.com/r/mongodb/comments/4pbnh8/how_to_test_my_replicaset/

https://docs.atlas.mongodb.com/tutorial/test-failover/

## bash find by name
https://blog.camel2243.com/2016/09/21/linux-find-%E6%8C%87%E4%BB%A4%EF%BC%8C%E6%90%9C%E5%B0%8B%E6%AA%94%E6%A1%88%E8%B3%87%E6%96%99%E5%A4%BE%E5%90%8D%E7%A8%B1%E8%88%87%E5%85%A8%E6%96%87%E6%90%9C%E5%B0%8B/

## bash 參數

## bash string

## bash default value

## custom config source

- from consul
- from vault


## linux list size

## change permission all files in folder
https://stackoverflow.com/questions/3740152/how-do-i-change-permissions-for-a-folder-and-all-of-its-subfolders-and-files-in

https://unix.stackexchange.com/questions/401207/linux-how-to-give-only-specific-user-to-read-the-file
until [$(curl -o /dev/null -s -w "%{http_code}\n" 204 http://localhost:18086/ping) -ne 204]; do
printf '.'
sleep 1
done


## Using mkdir -m -p and chown together correctly
https://stackoverflow.com/questions/25873479/using-mkdir-m-p-and-chown-together-correctly


## kibana version
http://staging.env.sb.skylinetw.com:32109/api/status
https://discuss.elastic.co/t/version-check/162811



## rdm install
http://docs.redisdesktop.com/en/latest/install/


https://blog.csdn.net/weixin_42044508/article/details/89439881

https://www.qt.io/offline-installers

https://www.elfgirl.top/system/mac/932.html

https://blog.csdn.net/daaikuaichuan/article/details/90446247

## chunk 反解


## mongo retention policy
https://medium.com/@Baresse/auto-expire-documents-with-mongodb-772fee84cc23


## jq custom function
https://github.com/stedolan/jq/wiki/Cookbook
https://shapeshed.com/jq-json/#how-to-use-pipes-with-jq

cat source.json|jq -c '.[] | "db.localization.update({#displaykey#:#\(.displaykey)#,#langcode#:#\(.langcode)#},{ $set : {#displayvalue#:#\(.displayvalue)#} },true,true );" '|sed "s/\"//g;s/#/\"/g"> test
.json



## mongo wiredtiger

https://stackoverflow.com/questions/6861184/is-there-any-option-to-limit-mongodb-memory-usage

https://www.bmc.com/blogs/mongodb-memory-usage-and-management/


## helm values by env

https://github.com/helm/helm/issues/2620#issuecomment-314719592

https://blog.gmem.cc/gotpl


## ~~redis delete key~~

https://rdbtools.com/blog/redis-delete-keys-matching-pattern-using-scan/

redis-cli -a pass.123 -h 192.168.1.151 -p 6389 rexbet_keys "accounts*"| xargs redis-cli -a pass.123 X-h 192.168.1.151 -p 6389 unlink

redis-cli -a pass.123 -h 192.168.1.151 -p 6379 --scan --pattern accounts:* | xargs redis-cli unlink

redis-cli -a pass.123 -h 192.168.1.152 -p 6391 rexbet_keys "accounts:*" | xargs redis-cli -a pass.123 -h 192.168.1.152 -p 6391 unlink

## linux 

- linux kernal log - quick valid result
- quick apply setting - ansible
- EFK dashboard sync (trello)
- resource 

## yaml math

https://github.com/Masterminds/sprig/blob/bf29da0d74f74aeb5c3e3e7207eab76c28ac4049/functions.go#L182


https://whmzsu.github.io/helm-doc-zh-cn/chart_template_guide/accessing_files-zh_cn.html#Glob%20%E6%A8%A1%E5%BC%8F


http://masterminds.github.io/sprig/math.html


https://www.skyarch.net/blog/?p=16691

## ansible for big architecture


https://ithelp.ithome.com.tw/articles/10186729


https://unix.stackexchange.com/questions/208819/list-all-ansible-variables-for-a-host-or-group-with-an-ad-hoc-command


## regular expresssionyowkotest
http://notepad.yehyeh.net/Content/Program/RegularExpression/9.php


## mongo manual failover

https://docs.mongodb.com/manual/reference/method/rs.stepDown/

## mac touchbat missing esc
https://apple.stackexchange.com/questions/379627/esc-button-from-touchbar-has-disappeared


## redis manual failover


## set ip to a variable

https://apple.stackexchange.com/questions/20547/how-do-i-find-my-ip-address-from-the-command-line

https://stackoverflow.com/questions/6829605/putting-ip-address-into-bash-variable-is-there-a-better-way


## fluentd performance

fluentd 1.10 worker 4

```
<buffer>
  flush_at_shutdown true
  flush_mode immediate
  flush_thread_count 8
  #flush_thread_interval 1
  #flush_thread_burst_interval 1
</buffer>
```

2020-05-15 06:25:50  2020-05-15 06:26:08

All-20200514_012.log 24109
May 16 00:22:13 2020
2020-05-15 16:25:51.885089200

"May 16 14:59:47 2020"
"May 16 15:03:28 2020"

48218
"May 16 20:57:22 2020"
"May 16 20:52:47 2020"

48218
"May 16 22:27:25 2020" 
"May 16 22:21:31 2020"

24109
"May 16 23:43:42 2020" 
"May 16 23:38:40 2020"

24109
"May 16 23:57:29 2020" 
"May 16 23:53:00 2020"

flush_interval 1s
    <buffer>
      @type memory
      #flush_mode immediate
      flush_thread_count 8
      flush_interval 1s
      #retry_forever
      #retry_max_interval 30
      chunk_limit_size 16m
      queue_limit_length 64
      overflow_action block
    </buffer>

24109
"May 17 00:08:35 2020" 
"May 17 00:05:26 2020"
flush_interval 5s


## mongo wiredTiger

db.adminCommand( { "setParameter": 1, "wiredTigerEngineRuntimeConfig": "cache_size=5G"})

## git delete old commit

https://gist.github.com/stephenhardy/5470814

du -chd 1 | sort -h

4.0K	./.dependabot
4.0K	./.github
4.0K	./tools
 80K	./Documentation
104K	./samples
976K	./temp-deployment
9.8M	./.git
 16M	./release-notes
 27M	.
 27M	total


## delay start
https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/

## delete remote branch expect master
https://www.hacksparrow.com/git/delete-all-remote-branches-except-master.html

## delete local branch expect master
https://www.hacksparrow.com/git/delete-all-branches-except-master.html

## linux yum update
https://www.thegeekdiary.com/yum-command-examples-to-install-remove-and-upgrade-packages/

https://electrictoolbox.com/yum-list-installed-packages/

## kafka one down ha fail

https://muxx.me/2018/06/12/kafka%E9%AB%98%E5%8F%AF%E7%94%A8%E5%A4%B1%E8%B4%A5%E9%97%AE%E9%A2%98-3broker-%E5%8D%95%E6%9D%80%E4%B8%80%E4%B8%AAbroker%E5%B0%B1%E4%B8%8D%E8%83%BD%E6%B6%88%E8%B4%B9%E7%9A%84%E9%97%AE%E9%A2%98%E6%8E%A2%E8%AE%A8/


/data/ssd/kafka/bin/kafka-topics.sh --bootstrap-server 10.0.0.13:9092,10.0.0.4:9092,10.0.0.12:9092  --describe --topic test

/data/ssd/kafka/bin/kafka-console-producer.sh --broker-list 10.0.0.13:9092,10.0.0.4:9092,10.0.0.12:9092 --topic test

/data/ssd/kafka/bin/kafka-console-consumer.sh  --bootstrap-server 10.0.0.13:9092,10.0.0.4:9092,10.0.0.12:9092 --topic test

https://www.cnblogs.com/wangzhuxing/p/10111831.html

## grpc consul discover ?


## kafka cluster setting

https://stackoverflow.com/questions/47483016/recommended-settings-for-kafka-internal-topics-after-upgrade-to-1-0

https://www.itread01.com/content/1550457744.html

https://my.oschina.net/feinik/blog/1806488


## docker-compose up  error

Starting backofficefluentd_fluentd-uat_1 ... error

ERROR: for backofficefluentd_fluentd-uat_1  Cannot start service fluentd-uat: error creating overlay mount to /var/lib/docker/overlay2/6972b20001d7b95f642db6a7ba5d25ce4803c89029c4a3029fbe55b7b377513c/merged: invalid argument

ERROR: for fluentd-uat  Cannot start service fluentd-uat: error creating overlay mount to /var/lib/docker/overlay2/6972b20001d7b95f642db6a7ba5d25ce4803c89029c4a3029fbe55b7b377513c/merged: invalid argument
ERROR: Encountered errors while bringing up the project.


https://github.com/docker/for-linux/issues/711#issuecomment-517927071


## fluentd parser by varaible


## iptable 
https://gigenchang.wordpress.com/2014/04/19/10%E5%88%86%E9%90%98%E5%AD%B8%E6%9C%83iptables/

https://www.cnblogs.com/whych/p/9147900.html

## mitmproxy transparent https






helm redis_cluster
https://github.com/inspur-iop/charts-src/tree/master/incubator/redis-cluster



## go template check nil

http://masterminds.github.io/sprig/reflection.html
https://github.com/Masterminds/sprig/issues/53#issuecomment-483414063


## .net core linux service
http://www.chrischiaro.com/2016/12/hosting-dotnet-core-web-api-on-ubuntu--getting-it-to-run.html

https://docs.microsoft.com/zh-tw/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-3.1

https://swimburger.net/blog/dotnet/how-to-run-a-dotnet-core-console-app-as-a-service-using-systemd-on-linux


## ansible daemon-reload

## ansible enable reboot

## dotnet-monitor

https://devblogs.microsoft.com/dotnet/introducing-dotnet-monitor/?fbclid=IwAR1N644hwhUBjDxsTHwlcMtqvDrvuCeBNa0qv0YtCDhTLBu5xDK-TS_s5ow

## ansible only run handler
https://serverfault.com/questions/617548/always-trigger-handler-execution-in-ansible
https://stackoverflow.com/questions/40139757/ansible-playbook-directly-run-handler

## .net core config issue
https://rimdev.io/avoiding-aspnet-core-configuration-pitfalls-with-array-values/

## ansible run only once 
https://stackoverflow.com/questions/22070232/how-to-get-an-ansible-check-to-run-only-once-in-a-playbook

## grep multi
https://unix.stackexchange.com/questions/37313/how-do-i-grep-for-multiple-patterns-with-pattern-having-a-pipe-character


## yum specific version 
https://www.golinuxcloud.com/yum-install-specific-version/


## kafka reassign
https://wzktravel.github.io/2015/12/31/kafka-reassign/

## docker run overwrite entrypoint
https://medium.com/@oprearocks/how-to-properly-override-the-entrypoint-using-docker-run-2e081e5feb9d 27017


## pass varaible docker-compose 
https://serversforhackers.com/c/div-variables-in-docker-compose

# docker-compose pass env
https://stackoverflow.com/questions/49293967/how-to-pass-environment-variable-to-docker-compose-up

https://vsupalov.com/docker-build-pass-environment-variables/

https://blog.scottchayaa.com/post/2018/11/04/docker-arg-env-variable/
https://serversforhackers.com/c/div-variables-in-docker-compose
https://medium.com/better-programming/using-variables-in-docker-compose-265a604c2006
https://stackoverflow.com/questions/35093256/how-do-i-pass-an-argument-along-with-docker-compose-up
https://www.baeldung.com/ops/docker-container-environment-variables
https://medium.com/better-programming/using-variables-in-docker-compose-265a604c2006

## pod nodeselector
https://kubernetes.io/docs/concepts/workloads/controllers/job/


## kafka cluster 
https://github.com/zoidbergwill/docker-compose-kafka

## reassign
https://jaceklaskowski.gitbooks.io/apache-kafka/kafka-admin-ReassignPartitionsCommand.html


## kafka mistake

https://blog.softwaremill.com/7-mistakes-when-using-apache-kafka-44358cd9cd6

## mongodb index process
https://stackoverflow.com/questions/22309268/mongodb-status-of-index-creation-job

## mongodb currentop
https://docs.mongodb.com/manual/reference/method/db.currentOp/
https://docs.mongodb.com/manual/tutorial/terminate-running-operations/


## drbd

cat /proc/drbd

https://blog.51cto.com/wangzhijian/1711284

https://www.cnblogs.com/f-ck-need-u/p/8684648.html

dd if=/dev/zero of={{ nfs_disk }} bs=1M count=100


## deload kube node
目前node7上都沒有任何Application服務, 因此不影響目前跑的測試, 如果要調整為一致, 只需要將node7設定為維護模式 腳本如下
kubectl drain node7 --delete-local-data --force --ignore-daemonsets
重新開機請MIS設定好
重新將node7 恢復工作模式 腳本如下
kubectl uncordon node7

## grpc server no response

## grpc library upgrade

## remove library



## nats
https://zhuanlan.zhihu.com/p/48530602

## mongodb objectid
ObjectId = 時間戳 + 機器 + PID + 計數器


## dotnet dump
https://github.com/6opuc/lldb-netcore-use-cases/blob/master/memory.md



docker run --name poc -p 8111:80 -e ASPNETCORE_URLS="https://+:443;http://+:80" -e ASPNETCORE_Kestrel__Certificates__Default__Password="pass.123" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ healthcheck:latest 


## istio
- ServiceEntry > external service
- Gateway > output redsocks

https://istio.io/latest/zh/blog/2018/egress-mongo/


## wrk vs jmeter process and threads
https://juejin.im/entry/6844904094054744072

https://testerhome.com/topics/17068

## grpc with polly
1. global intrceptor
2. every client


## dnf with module
https://serverfault.com/questions/991801/ansible-centos-8-dnf-module-module-switch-missing

/yum-specific-version/
https://blog.csdn.net/coco3848/article/details/107605687


## older 
http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-core-5.7.12-1.el8.elrepo.x86_64.rpm


http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-modules-5.7.12-1.el8.elrepo.x86_64.rpm


http://repos.ord.lax-noc.com/elrepo/archive/kernel/el8/x86_64/RPMS/kernel-ml-5.7.12-1.el8.elrepo.x86_64.rpm


## ansible dnf redis6 replication



## ansible role by condition
https://stackoverflow.com/questions/60826471/ansible-playbook-how-to-run-play-based-on-when-condition

## ansible flush handler

## loki in .net core

## ansible debug template
https://stackoverflow.com/questions/35407822/how-can-i-test-jinja2-templates-in-ansible

## mariadb backup user

https://stackoverflow.com/questions/46554804/mysqldump-not-dumping-stored-procedures
https://qastack.cn/server/912162/
https://serverfault.com/questions/912162/mysqldump-throws-unknown-table-column-statistics-in-information-schema-1109
https://devopspoints.com/mysql-mariadb-user-accounts-and-privileges.html


CREATE USER 'backupuser' IDENTIFIED BY 'pass.123';
GRANT SELECT,INSERT, LOCK TABLES, CREATE,
CREATE TEMPORARY TABLES, INDEX, ALTER on *.* TO 'backupuser'@'localhost';

-- backup

CREATE USER 'backupuser' IDENTIFIED BY 'pass.123';
GRANT SELECT on mysql.proc TO 'backupuser';
GRANT SELECT, LOCK TABLES on gaming.* TO 'backupuser';

-- restore
GRANT SUPER on *.* to 'backupuser';
GRANT ALTER ROUTINE,CREATE ROUTINE,SELECT, INSERT, LOCK TABLES, CREATE, Drop, CREATE TEMPORARY TABLES, INDEX, ALTER on gaming.* TO 'backupuser';




GRANT FILE, INSERT, LOCK TABLES, CREATE,
CREATE TEMPORARY TABLES, INDEX, ALTER on gaming.* TO 'backupuser';


mysqldump -u backupuser -ppass.123 --column-statistics=0  --routines --compress gaming  > mysqldump-backup.sql

## docker build multiple images one dockerfile
https://stackoverflow.com/questions/49754286/multiple-images-one-dockerfile



## nysql client and mongo client


## WT.mc_id=DOP-MVP-5002594