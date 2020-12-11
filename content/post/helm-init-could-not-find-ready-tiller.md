---
title: "解決 Helm init 時出現找不到 tiller pod 問題"
date: 2019-12-26T21:30:00+08:00
lastmod: 2020-12-11T21:30:31+08:00
draft: false
tags: ["Container","Helm","Kubernetes","macOS"]
slug: "helm-init-could-not-find-ready-tiller"
---

## 解決 Helm init 時出現找不到 tiller pod 問題

這是在嘗試 Docker for Mac 上的 Kubernetes 與 Helm 時遇到的問題，成功版的安裝過程可以參考之前筆記 [在Docker for Mac 啟用Kubernetes 後安裝Helm - Yowko's Notes](/docker-mac-kubernetes-helm/)，還是不太了解 Helm，筆記一下問題及解決待日後參考

## 基本環境說明

1. macOS Catalina 10.15.2
2. docker 19.03.5
3. Kubernetes v1.14.8
4. Helm v2.16.1

## 問題描述

1. 執行 Helm install 時

    ```bash
    helm install *.tgz
    ```

    - 錯誤訊息

        ```txt
        Error: could not find a ready tiller pod
        ```

    - 錯誤截圖

        ![1error](https://user-images.githubusercontent.com/3851540/73365824-e0865500-42e7-11ea-87c3-cea3c59a1d72.png)

2. 逐步檢查相關內容

    > 以結果來看都正常

    - kubectl get pods -n kube-system

        ```txt
        NAME                                     READY   STATUS    RESTARTS   AGE
        coredns-6dcc67dcbc-r6kmp                 1/1     Running   0          45h
        coredns-6dcc67dcbc-w576d                 1/1     Running   0          45h
        etcd-docker-desktop                      1/1     Running   0          45h
        kube-apiserver-docker-desktop            1/1     Running   0          45h
        kube-controller-manager-docker-desktop   1/1     Running   0          45h
        kube-proxy-985k2                         1/1     Running   0          45h
        kube-scheduler-docker-desktop            1/1     Running   0          45h
        tiller-deploy-dc4f6cccd-95dcr            1/1     Running   0          101s
        ```

        ![2getpod](https://user-images.githubusercontent.com/3851540/73365825-e11eeb80-42e7-11ea-9b59-4fe9cb9bef14.png)

    - kubectl get deploy -n kube-system

        ```txt
        NAME            READY   UP-TO-DATE   AVAILABLE   AGE
        coredns         2/2     2            2           45h
        tiller-deploy   1/1     1            1           45h
        ```

        ![3getdeploy](https://user-images.githubusercontent.com/3851540/73365827-e11eeb80-42e7-11ea-82d3-fc58e057d3eb.png)

    - ubectl describe pod tiller-deploy -n kube-system

        ```txt
        Name:               tiller-deploy-dc4f6cccd-95dcr
        Namespace:          kube-system
        Priority:           0
        PriorityClassName:  <none>
        Node:               docker-desktop/192.168.65.3
        Start Time:         Sun, 22 Dec 2019 15:53:20 +0800
        Labels:             app=helm
                            name=tiller
                            pod-template-hash=dc4f6cccd
        Annotations:        <none>
        Status:             Running
        IP:                 10.1.0.6
        Controlled By:      ReplicaSet/tiller-deploy-dc4f6cccd
        Containers:
          tiller:
            Container ID:   docker://       a62a5f0806da46f8aa933dd18d642f516787080acf09a5dc0af6ca782e91e2db
            Image:          gcr.io/kubernetes-helm/tiller:v2.16.1
            Image ID:       docker-pullable://gcr.io/kubernetes-helm/       tiller@sha256:3c70ee359d3ec305ca469395a2481b2375d569c6b4a928389ca07d829d12ec51
            Ports:          44134/TCP, 44135/TCP
            Host Ports:     0/TCP, 0/TCP
            State:          Running
              Started:      Sun, 22 Dec 2019 15:53:29 +0800
            Ready:          True
            Restart Count:  0
            Liveness:       http-get http://:44135/liveness delay=1s timeout=1s period=10s      #success=1 #failure=3
            Readiness:      http-get http://:44135/readiness delay=1s timeout=1s period=10s         #success=1 #failure=3
            Environment:
              TILLER_NAMESPACE:    kube-system
              TILLER_HISTORY_MAX:  0
            Mounts:
              /var/run/secrets/kubernetes.io/serviceaccount from default-token-cp7nb (ro)
        Conditions:
          Type              Status
          Initialized       True
          Ready             True
          ContainersReady   True
          PodScheduled      True
        Volumes:
          default-token-cp7nb:
            Type:        Secret (a volume populated by a Secret)
            SecretName:  default-token-cp7nb
            Optional:    false
        QoS Class:       BestEffort
        Node-Selectors:  <none>
        Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                         node.kubernetes.io/unreachable:NoExecute for 300s
        Events:
          Type    Reason     Age   From                     Message
          ----    ------     ----  ----                     -------
          Normal  Scheduled  119s  default-scheduler        Successfully assigned kube-system/      tiller-deploy-dc4f6cccd-95dcr to docker-desktop
          Normal  Pulling    117s  kubelet, docker-desktop  Pulling image "gcr.io/      kubernetes-helm/tiller:v2.16.1"
          Normal  Pulled     110s  kubelet, docker-desktop  Successfully pulled image "gcr.io/      kubernetes-helm/tiller:v2.16.1"
          Normal  Created    110s  kubelet, docker-desktop  Created container tiller
          Normal  Started    110s  kubelet, docker-desktop  Started container tiller
        ```

        ![4describepod](https://user-images.githubusercontent.com/3851540/73365828-e11eeb80-42e7-11ea-80cd-3e93dac9c251.png)

## 解決方式

> 既然 tiller 設定看起來 ok，乾脆只 config local 不另外安裝 tiller

```bash
helm init --client-only
```

此時再重新安裝即正常

![5deployed](https://user-images.githubusercontent.com/3851540/73365829-e11eeb80-42e7-11ea-83ab-320367d80318.png)

## 心得

雖然最後問題解決了，但關於發生原因本身並沒有找到相關資料，解決方式也是試出來的，暫時還不知道運作原理，不過沒關係，先記著待日後相關知識增長了再看也許就有想法了

## 參考資訊

1. [安装 FAQ](https://whmzsu.github.io/helm-doc-zh-cn/quickstart/install_faq-zh_cn.html)
