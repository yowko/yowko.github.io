---
title: "使用 Play-With-Kubernetes 服務建立 Kubernetes Cluster"
date: 2019-04-26T21:30:00+08:00
lastmod: 2021-10-29T21:30:31+08:00
draft: false
tags: ["Kubernetes"]
slug: "play-with-kubernetes-cluster"
---
## 使用 Play-With-Kubernetes 服務建立 Kubernetes Cluster

之前都是透過 Azure VM 來建立 Kubernetes 環境，使用上沒什麼問題，唯一缺點就是勞民傷財，想測個功能才來架整個 Kubernetes cluster 有點緩不濟急，但常態保留 Kubernetes Cluster 所使用的 VM 就算 VM 沒有啟動也因為佔用資源而需要計價，加上有時是要測試底層的服務 (e.g 網路套件...) 重新架設 clsuer 本來就是需要的，這時候 [Play with Kubernetes](http://labs.play-with-k8s.com/) 就非常實用，[Play with Kubernetes](http://labs.play-with-k8s.com/) 提供每次四個小時、最多 五 個 node 的環境可以用來建立 Kubernetes，每次四個小時結束後需要重新建立，適合短暫驗證功能測試

## 前期準備

1. 開啟 [Play with Kubernetes](http://labs.play-with-k8s.com/)

    ![1playwithkubernetes](https://user-images.githubusercontent.com/3851540/56843634-a231b480-68d5-11e9-85da-1bf57e44f7ca.png)

2. 登入 (可以使用 `GitHub` or `docker` 帳號)

    ![2login](https://user-images.githubusercontent.com/3851540/56843635-a231b480-68d5-11e9-8420-053b02535984.png)

3. 成功登入後會出現 `Start` 按鈕

    ![3start](https://user-images.githubusercontent.com/3851540/56843636-a231b480-68d5-11e9-84ce-b613280744f2.png)

    ![4startsession](https://user-images.githubusercontent.com/3851540/56843637-a231b480-68d5-11e9-9a33-fc01fe457ce9.png)

## 建立 instance

`ADD NEW INSTANCE` 按鈕每次可以產生一個 node

![5createinstance](https://user-images.githubusercontent.com/3851540/56843638-a2ca4b00-68d5-11e9-9b00-0421633bc7ca.png)

## 建立 master

點選任一個 node 以開啟 terminal，並依序執行下列指令

![6master](https://user-images.githubusercontent.com/3851540/56843639-a2ca4b00-68d5-11e9-8034-1fd576c5b5e8.png)

1. 初始化 master node

    ```bash
    kubeadm init --apiserver-advertise-address $(hostname -i)
    ```

    > 初始化完成後，會顯示該如何將其他 node 加入該 cluster 的語法，需要先保存下來

    ![7joincluster](https://user-images.githubusercontent.com/3851540/56843640-a2ca4b00-68d5-11e9-822c-2e0e442f9150.png)

2. 初始化 cluster 網路 (使用 weave)

    ```bash
    kubectl apply -n kube-system -f \
    "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"
    ```

## 將 worker 加入 master

token 每次建立 session 與 cluster 都會不同

```bash
kubeadm join 192.168.0.6:6443 --token 38nfgj.n9tol1xutiwg9qg2 --discovery-token-ca-cert-hash sha256:72d9b5130a41b5c5906523c03aec90c96dfc308ae761ad3204a0ee2c705ac332
```

![8joined](https://user-images.githubusercontent.com/3851540/56843641-a362e180-68d5-11e9-8efd-c6b1a24056d5.png)

## 確認各個 node 狀態

需在 master 上執行，各個 node 皆顯示 `Ready` 即成功完成 cluster 建立

```bash
kubectl get node
```

![9getnode](https://user-images.githubusercontent.com/3851540/56843642-a362e180-68d5-11e9-8da9-0ee00db4da4d.png)

## 心得

我自己頻繁使用了 [Play with Kubernetes](http://labs.play-with-k8s.com/) 幾天，佛心完全不收費的服務實在沒什麼好要求的，這邊就列一下有幾個需要留意的點：

1. 台北時間下午四、五點服務穩定性會下降

    > 有時候會無法建立 session，有時按下 create instance 沒有反應，有時 kubectl 指令執行時間會變很長 接著出現 timeout

2. 測試情境需要大於 4 小時的操作

    > 儘管可以重新建立，但時間是寶貴的

3. 複雜的情境

    > 至多 5 個 node，每個 node 有 3.9 GB 左右 ram

4. 最新功能測試

    > 2019/4/27 Kubernetes 版本為 `v1.11.3`, Docker 版本為 `18.06.1-ce`

## 參考資訊

1. [Play with Kubernetes](http://labs.play-with-k8s.com/)
2. [快速體驗kubernetes](http://www.iceyao.com.cn/2017/08/16/%E5%BF%AB%E9%80%9F%E4%BD%93%E9%AA%8Ckubernetes/)
