---
title: "將 ConfigMap 做為 Helm volume mount 的來源"
date: 2020-01-29T21:30:00+08:00
lastmod: 2020-01-29T21:30:31+08:00
draft: false
tags: ["Helm","Kubernetes"]
slug: "configmap-helm-volume-mount"
---

## 將 ConfigMap 做為 Helm volume mount 的來源

有用過 container 的朋友相信對於 volume mount 有一定的了解，如果有用過 db 類型的 container (mysql、mongodb...) 對於透過 `docker-entrypoint-initdb.d` 來建立初始資料應該也不陌生，但相同機制在 Helm 中該如何實現呢？！

因為 Helm 只是用來管理 Kubernetes service 的工具，我才疏學淺不知道該如何正確將初始資料用的檔案隨著 Helm 部署，後來查到可以將設定內容存至 ConfigMap 中，再 mount 至 container 中，趁著印憶猶新趕緊紀錄一下，不然本來就沒有很了解前因後果，一定很快就忘了

## 基本環境說明

1. macOS Mojave 10.15.2
2. docker desktop community 2.2.0.0(42247)
3. Docker Engine 19.03.5
4. Kubernetes v1.15.5
5. Helm v2.16.1
6. 原始 helm (以 mysql 為例)

    - templates/deployment.yaml

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: {{ include "yowkochart.fullname" . }}
          labels:
        {{ include "yowkochart.labels" . | indent 4 }}
        spec:
          replicas: {{ .Values.replicaCount }}
          selector:
            matchLabels:
              app.kubernetes.io/name: {{ include "yowkochart.name" . }}
              app.kubernetes.io/instance: {{ .Release.Name }}
          template:
            metadata:
              labels:
                app.kubernetes.io/name: {{ include "yowkochart.name" . }}
                app.kubernetes.io/instance: {{ .Release.Name }}
            spec:
              containers:
                - name: {{ .Chart.Name }}
                  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
                  imagePullPolicy: {{ .Values.image.pullPolicy }}
                  ports:
                    - name: http
                      containerPort: 3306
                      protocol: TCP
                    env:
                    - name: MYSQL_ROOT_PASSWORD
                      value: {{.Values.mysql.password}}

        ```

    - templates/service.yaml

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: {{ include "yowkochart.fullname" . }}
          labels:
        {{ include "yowkochart.labels" . | indent 4 }}
        spec:
          type: {{ .Values.service.type }}
          ports:
            - port: {{ .Values.service.port }}
              targetPort: http
              protocol: TCP
              name: mysql
          selector:
            app.kubernetes.io/name: {{ include "yowkochart.name" . }}
            app.kubernetes.io/instance: {{ .Release.Name }}
        ```

    - values.yaml

        ```yaml
        replicaCount: 1

        image:
          repository: mysql
          tag: 5.7.28
          pullPolicy: IfNotPresent

        service:
          type: ClusterIP
          port: 3306

        mysql:
          password: pass.123
        ```

    - Chart.yaml

        ```yaml
        apiVersion: v1
        appVersion: "1.0"
        description: A Helm chart for Kubernetes
        name: yowkochart
        version: 0.1.0
        ```

    > 僅有預設資料庫

    ![1default](https://user-images.githubusercontent.com/3851540/73425397-795eb400-436c-11ea-8e11-3ca226d19f43.png)

## 設定方式

1. 建立 `ConfigMap` - `templates/initial-configmap.yaml`

    > sql script 內容請依實際需求自行調整

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: "mysql-init-configmap"
    data:
      init.sql: |-
        create database if not exists yowkodb;

        USE yowkodb;

        CREATE TABLE IF NOT EXISTS Users
        (
            Id int auto_increment
                primary key,
            Name varchar(20) not null default '',
            Gender bool default 1 not null,
            DateUpdated datetime default current_timestamp() not null,
            DateCreated datetime default current_timestamp() not null
        );

        ALTER TABLE Users AUTO_INCREMENT = 1001;

        INSERT INTO yowkodb.Users (Name, Gender) VALUES ('yowko', 1);
        INSERT INTO yowkodb.Users (Name, Gender) VALUES ('ann', 0);
    ```

2. 修改 `Deployment` - `templates/deployment.yaml`

    > 以下是我多方嘗試得到的結果，如果有錯敬請指教

    - `containers.volumeMounts.name` 需要與 `volume.name` 相同
    - `containers.volumeMounts.mountPath` 需要指定到 file name
    - `containers.volumeMounts.subPath` 與 `containers.volumeMounts.mountPath` 的 file name 相同
    - `volumes.configMap.name` 與 `templates/initial-configmap.yaml` 的 `metadata.name` 相同

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ include "yowkochart.fullname" . }}
      labels:
    {{ include "yowkochart.labels" . | indent 4 }}
    spec:
      replicas: {{ .Values.replicaCount }}
      selector:
        matchLabels:
          app.kubernetes.io/name: {{ include "yowkochart.name" . }}
          app.kubernetes.io/instance: {{ .Release.Name }}
      template:
        metadata:
          labels:
            app.kubernetes.io/name: {{ include "yowkochart.name" . }}
            app.kubernetes.io/instance: {{ .Release.Name }}
        spec:
          containers:
            - name: {{ .Chart.Name }}
              image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
              imagePullPolicy: {{ .Values.image.pullPolicy }}
              ports:
                - name: http
                  containerPort: 3306
                  protocol: TCP
              env:
              - name: MYSQL_ROOT_PASSWORD
                value: {{.Values.mysql.password}}
              volumeMounts:
              - name: init-scripts
                mountPath: /docker-entrypoint-initdb.d/init.sql
                subPath: init.sql
          volumes:
          - name: init-scripts
            configMap:
              name: mysql-init-configmap
    ```

3. 安裝測試

    > kubectl 指令中的 pod name 請依實際狀況調整

    - databases

        - 語法

            ```bash
            kubectl exec -it {pod name} -- mysql -h {mysql ip} -P {mysql port} -u root -p{密碼} -e "show databases"
            ```

        - 範例

            ```bash
            kubectl exec -it pod/mysql-yowkochart-5474d9dd57-hg2qg -- mysql -h 127.0.0.1 -P 3306 -u root -ppass.123 -e "show databases"
            ```

        ![2databases](https://user-images.githubusercontent.com/3851540/73425398-795eb400-436c-11ea-9858-71c08bf0e363.png)

    - data

        - 語法

            ```bash
            kubectl exec -it {pod name} -- mysql -h {mysql ip} -P {mysql port} -u root -p{密碼} -e "select * from {db name}.{table name}"
            ```

        - 範例

            ```bash
            kubectl exec -it pod/mysql-yowkochart-5474d9dd57-hg2qg -- mysql -h 127.0.0.1 -P 3306 -u root -ppass.123 -e "select * from yowkodb.Users"
            ```

        ![3data](https://user-images.githubusercontent.com/3851540/73425399-795eb400-436c-11ea-9f79-f6af23094c95.png)

## 心得

雖然東拼西湊有達成想要的目標，但還是有些做法搞不清來龍去脈：

1. 為什麼 `mountPath` 要加上 filename
2. 為什麼需要 `subPath`

另外我也不確定這是不是個好的方式，畢竟需要調整 initial script 的內容就變成要改 Helm Chart，我個人覺得 Helm Chart 只是簡化管理的工具，不適合把頻繁異動的內容往裡頭塞，如果後續異動頻率較高可能就得考慮其他做法了

## 參考資源

1. [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
2. [configmap file mount path results in command not found error](https://github.com/kubernetes/kubernetes/issues/44815#issuecomment-297077509)
