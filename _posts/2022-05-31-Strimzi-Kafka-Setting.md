---
title: "Strimzi Kafka Setting"
categories:
  - Kafka
tags:
  - kafka
  - python
  - kafkacat
  - strimzi
toc: true
---

# SA Kafka Cluster Setting

> 새로운 클러스터 추가시 필요한 SA kafka 셋팅 작업 설명



# 1. 사전설정

## 1.1 namespace 생성
strimzi operator 와 kafka cluster 를 kafka namespace 에 설치해야 한다.  worker node 를 준비한후 kafka namespace 를 생성하자.
```
oc create ns kafka
```

## 1.2 권한설정

openshift에서는 zookeeper 와 kafka 를 실행할때 anyuid 도 실행될 수 있도록 권한을 부여해야 한다.
권한을 주지 않으면 pod 생성시 operation not permitted 에러 발생한다.


ㅇ kafka SA 로 네임스페이스내 권한 부여

```bash
# 권한부여시
oc adm policy add-scc-to-group    anyuid  system:serviceaccounts:kafka -n kafka-system

# 권한삭제시
oc adm policy remove-scc-from-group anyuid  system:serviceaccounts:kafka -n kafka-system
```

<-- 권한 주지 않아도 잘 된다. 



## 1.3 필요한 image

필요한 container image 들은 Cluster 에서 접근가능한 registry 에 push 해 놓아야 한다.

- 필요한 image

| Container image       | Namespace/Repository                                         | Description                                                  |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Kafka                 | quay.io/strimzi/kafka:0.28.0-kafka-3.0.0quay.io/strimzi/kafka:0.28.0-kafka-3.1.0 | Strimzi image for running Kafka, including:Kafka BrokerKafka ConnectKafka MirrorMakerZooKeeperTLS Sidecars |
| Operator              | quay.io/strimzi/operator:0.28.0                              | Strimzi image for running the operators:Cluster OperatorTopic OperatorUser OperatorKafka Initializer |
| Kafka Bridge          | quay.io/strimzi/kafka-bridge:0.21.4                          | Strimzi image for running the Strimzi kafka Bridge           |
| Strimzi Drain Cleaner | quay.io/strimzi/drain-cleaner:0.3.0                          | Strimzi image for running the Strimzi Drain Cleaner          |
| JmxTrans              | quay.io/strimzi/jmxtrans:0.28.0                              | Strimzi image for running the Strimzi JmxTrans               |



## 1.4 image pull push

```


## pull
docker pull quay.io/strimzi/kafka:0.28.0-kafka-3.0.0
docker pull quay.io/strimzi/kafka:0.28.0-kafka-3.1.0
docker pull quay.io/strimzi/operator:0.28.0
docker pull quay.io/strimzi/kafka-bridge:0.21.4
docker pull quay.io/strimzi/jmxtrans:0.28.0
docker pull quay.io/strimzi/kaniko-executor:0.28.0
docker pull quay.io/strimzi/maven-builder:0.28.0



## tag
docker tag quay.io/strimzi/kafka:0.28.0-kafka-3.1.0 ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.28.0-kafka-3.1.0
docker tag quay.io/strimzi/operator:0.28.0          ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.28.0 
docker tag quay.io/strimzi/kafka-bridge:0.21.4      ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka-bridge:0.21.4 
docker tag quay.io/strimzi/jmxtrans:0.28.0          ktis-bastion01.container.ipc.kt.com:5000/strimzi/jmxtrans:0.28.0  
docker tag quay.io/strimzi/kaniko-executor:0.28.0   ktis-bastion01.container.ipc.kt.com:5000/strimzi/kaniko-executor:0.28.0 
docker tag quay.io/strimzi/maven-builder:0.28.0     ktis-bastion01.container.ipc.kt.com:5000/strimzi/maven-builder:0.28.0


## push
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.28.0-kafka-3.1.0
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.28.0 
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka-bridge:0.21.4 
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/jmxtrans:0.28.0
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/kaniko-executor:0.28.0
docker push ktis-bastion01.container.ipc.kt.com:5000/strimzi/maven-builder:0.28.0


```





# 2. Strimzi Cluster Operator Install

srimzi  operator 를 install 한다.



## 2.1 관련 file download

- 해당 사이트(https://strimzi.io/downloads/) 에서 해당 버젼을 다운로드 받는다.

## 2.2 single name 모드 namespace 설정

- single name 모드로 설치진행
  - strimzi operator 는 다양한 namespace 에서 kafka cluster 를 쉽게 생성할 수 있는 구조로 운영이 가능하다.  이때 STRIMZI_NAMESPACE 를 설정하여 특정 namespace 만으로 cluster 를 제한 할 수 있다.  ICIS-TR SA의 경우는 kafka-system 라는 namespace 에서만  kafka cluster 를 구성할 수 있도록 설정한다. 그러므로 아래 중 Single namespace 설정에 해당한다.

```sh
$ cd ~/song/kafka/strimzi/strimzi-0.28.0

$ sed -i 's/namespace: .*/namespace: kafka/' install/cluster-operator/*RoleBinding*.yaml

```



## 2.3 image 주소 설정

```sh
$ cd ~/song/kafka/strimzi/strimzi-0.28.0

$ sed -i 's/quay.io/ktis-bastion01.container.ipc.kt.com:5000/' install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml

```



## 2.4 deploy

```sh
$ cd ~/song/kafka/strimzi/strimzi-0.28.0

$ oc -n kafka create -f install/cluster-operator
```



## 2.5 clean up

```sh
$ cd ~/song/kafka/strimzi/strimzi-0.28.0

$ oc -n kafka delete -f install/cluster-operator
```









```bash

vi 050-Deployment-strimzi-cluster-operator.yaml
		...
      containers:
      - name: strimzi-cluster-operator
        image: ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.17.0
        args:
        - /opt/strimzi/bin/cluster_operator_run.sh
        env:
        - name: STRIMZI_NAMESPACE
          value: kafka
        - name: STRIMZI_FULL_RECONCILIATION_INTERVAL_MS
          value: "120000"
        - name: STRIMZI_OPERATION_TIMEOUT_MS
          value: "300000"
        - name: STRIMZI_DEFAULT_TLS_SIDECAR_ENTITY_OPERATOR_IMAGE
          value: ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_DEFAULT_TLS_SIDECAR_KAFKA_IMAGE
          value: ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_DEFAULT_TLS_SIDECAR_ZOOKEEPER_IMAGE
          value: ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_DEFAULT_KAFKA_EXPORTER_IMAGE
          value: ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_KAFKA_IMAGES
          value: |
            2.3.1=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.3.1
            2.4.0=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_KAFKA_CONNECT_IMAGES
          value: |
            2.3.1=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.3.1
            2.4.0=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_KAFKA_CONNECT_S2I_IMAGES
          value: |
            2.3.1=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.3.1
            2.4.0=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_KAFKA_MIRROR_MAKER_IMAGES
          value: |
            2.3.1=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.3.1
            2.4.0=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_KAFKA_MIRROR_MAKER_2_IMAGES
          value: |
            2.4.0=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.17.0-kafka-2.4.0
        - name: STRIMZI_DEFAULT_TOPIC_OPERATOR_IMAGE
          value: "ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.17.0"
        - name: STRIMZI_DEFAULT_USER_OPERATOR_IMAGE
          value: "ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.17.0"
        - name: STRIMZI_DEFAULT_KAFKA_INIT_IMAGE
          value: "ktis-bastion01.container.ipc.kt.com:5000/strimzi/operator:0.17.0"
        - name: STRIMZI_DEFAULT_KAFKA_BRIDGE_IMAGE
          value: "ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka-bridge:0.15.2"
        - name: STRIMZI_DEFAULT_JMXTRANS_IMAGE
          value: "ktis-bastion01.container.ipc.kt.com:5000/strimzi/jmxtrans:0.17.0"
        - name: STRIMZI_LOG_LEVEL
          value: "INFO"
        ...
          
#3) 실행

$ cd ~/song/kafka/strimzi/strimzi-0.17.0/install/cluster-operator

$ oc -n kafka create -f .
```





# 3. Kafka Cluster 생성



## 3.1 ephemeral sample

### 1) kafka cluster 생성(no 인증)

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.1.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.1"
    storage:
      type: ephemeral
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
  entityOperator:
    topicOperator: {}
    userOperator: {} 
```





### 2) kafka cluster 생성(인증)

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.1.0
    replicas: 3
    authorization:
      type: simple
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
        authentication:
          type: scram-sha-512
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.1"
    storage:
      type: ephemeral
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
  entityOperator:
    topicOperator: {}
    userOperator: {} 
```

- 인증메커니즘

  - SASL 은 인증 멫 보안 서비스를 제공하는 프레임워크이다.
  - 위 yaml 파일의 인증방식은 scram-sha-512  방식인데 이는 SASL 이 지원하는 메커니즘 중 하나이며 Broker 를 SASL 구성로 구성한다.

- Client 에서 접속하는 두가지 방법

  - sasl.jaas.config 설정

  - JAAS Configuration

    - 애플리케이션 구동시 아규먼트 이용

    ```
    -Djava.security.auth.login.config={JAAS File} 
    ```

  - Kafka client 0.10.2.X 이상 버전 일때는 JAAS Config 방식을 사용하지 않고 클라이언트의 특성에서 직접 SASL 인증을 구성

- security.protocol 설정

  - tls 방식은 아니므로 _SASL_PLAINTEXT_로 설정한다.

- cluster 생성

```
$ oc -n kafka create -f 11.kafka-sa-cluster.yaml
```

- deleteClaim
  - kafka cluster 가 undeployed 되었을때 데이터를 삭제할지를 묻는다.





## 3.2 pv / pvc 생성

Cluster 를 생성하기전에 pv/pvc 를 사전에 준비해야 한다.



### 1)  pv / pvc 관계


|pv                  |pvc                                 |
|--------------------|------------------------------------|
|arsenal-kafka-0     |data-arsenal-cluster-kafka-0        |
|arsenal-kafka-1     |data-arsenal-cluster-kafka-1        |
|arsenal-kafka-2     |data-arsenal-cluster-kafka-2        |
|arsenal-zookeeper-0 |data-arsenal-cluster-zookeeper-0    |
|arsenal-zookeeper-1 |data-arsenal-cluster-zookeeper-1    |
|arsenal-zookeeper-2 |data-arsenal-cluster-zookeeper-2    |



### 2) pv 생성시 주의사항

- pv 생성시에 명시하는 path는 반드시 해당 경로가 실제로 존재해야 한다.
- 존재하지 않으면 pv/pvc bound 시에는 문제가 없지만 해당 pvc 를 kafka 에서 사용시 error 발생한다.  
- 접근권한이 존재해야 한다. 

```
...
nfs:
  server: 10.220.190.108
  path: /p_tcpmp_pk1_nas/tpcmp-pv/kafka/arsenal-kafka-0   <-- nfs 에 해당 경로가 반드시 존재해야 한다.
...
```




### 3) pvc 

아래와 같이 6개의 pvc 를 생성한다.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-kafka-0
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-kafka-0
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-kafka-1
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-kafka-1
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-kafka-2
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-kafka-2
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-zookeeper-0
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-zookeeper-0
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-zookeeper-1
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-zookeeper-1
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-arsenal-cluster-zookeeper-2
  namespace: kafka
  labels:
    app.kubernetes.io/instance: arsenal-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: strimzi
    strimzi.io/cluster: arsenal-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: arsenal-cluster-kafka
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: arsenal-zookeeper-2
---
```





### 4) cluster 반영



```
$ cat 11.kafka-sa-cluster.yaml
---
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: arsenal-cluster
  namespace: kafka
spec:
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafka:
    authorization:     # <-- listener 에서 authentication 를 사용하려면 반드시 필요하다.
      type: simple
    config:
      log.message.format.version: '2.4'
      offsets.topic.replication.factor: 1
      transaction.state.log.min.isr: 1
      transaction.state.log.replication.factor: 1
    listeners:
#      plain: {}
      plain:
        authentication:
          type: scram-sha-512
      tls: {}
    replicas: 3
    storage:
      deleteClaim: false   
      size: 10Gi
      type: persistent-claim
  zookeeper:
    replicas: 3
    storage:
      deleteClaim: false
      size: 10Gi
      type: persistent-claim
```

##### [Resizing persistent volumes](#proc-resizing-persistent-volumes-deployment-configuration-kafka)

You can provision increased storage capacity by increasing the size of the persistent volumes used by an existing Strimzi cluster. Resizing persistent volumes is supported in clusters that use either a single persistent volume or multiple persistent volumes in a JBOD storage configuration.



## 3.3 metric - 작성중

모니터링이 필요할 경우 아래와 같이 exporter 를 설치후 promtheus 로 metric 데이터를 보낼 수 있다.



### 1) sa-cluter 에 exporter 생성

```
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka-system
  ...
  kafkaExporter:
    groupRegex: .*
    topicRegex: .*
  ...    
    
```

- exporter 전용이미지가 반드시 존재해야한다.   

  - strimzi/kafka:0.28.0-kafka-3.1.0    <-- strimzi-cluster-operator 에서 기본 이미지 설정가능
- kafka cluster 에서 위 yaml 파일만 수정후 apply 해도 추가된다.(kafka cluster 를 재설치 하지 않아도 된다.)
- prometheus / grafana 는 일반적인 내용과 동일하다.
- grafana 에서는 strimzi dashboard 를 찾아서 import 한다.
- arsenal-cluster-kafka-exporter:9404 로 접근가능



- exporter pod 생성 여부 확인

```

$ oc get pod
NAME                                          READY   STATUS    RESTARTS   AGE
kafka-cat                                     1/1     Running   0          5d8h
sa-cluster-entity-operator-7fb45fd595-586sg   3/3     Running   0          71m
sa-cluster-kafka-0                            1/1     Running   0          65m
sa-cluster-kafka-1                            1/1     Running   0          66m
sa-cluster-kafka-2                            1/1     Running   0          65m
sa-cluster-kafka-exporter-6d98f5cd86-l6tgf    1/1     Running   0          8m16s
sa-cluster-zookeeper-0                        1/1     Running   0          83m
sa-cluster-zookeeper-1                        1/1     Running   0          83m
sa-cluster-zookeeper-2                        1/1     Running   0          83m

```



- exporter pod 에서 metric 정상 수집 여부 확인

```

sh-4.4$ curl localhost:9404/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.00037912
go_gc_duration_seconds{quantile="0.25"} 0.00037912
go_gc_duration_seconds{quantile="0.5"} 0.00037912
go_gc_duration_seconds{quantile="0.75"} 0.00037912
go_gc_duration_seconds{quantile="1"} 0.00037912
go_gc_duration_seconds_sum 0.00037912
go_gc_duration_seconds_count 1
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 19
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.1"} 1

```





### 9) Cruise Control 을 위한 sa-cluster 셋팅

크루즈 컨트롤메트릭을 노출하려면 Prometheus 설정을 이용한다.

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
  ...
  kafka:
    ...
      metricsConfig:
        type: jmxPrometheusExporter
        valueFrom:
          configMapKeyRef:
            name: kafka-metrics
            key: kafka-metrics-config.yml  
  zookeeper:
    ...
      metricsConfig:
        type: jmxPrometheusExporter
        valueFrom:
          configMapKeyRef:
            name: kafka-metrics
            key: kafka-metrics-config.yml  
  kafkaExporter:
    topicRegex: ".*"
    groupRegex: ".*"  
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kafka-metrics
  labels:
    app: strimzi
data:
  kafka-metrics-config.yml: |
    # See https://github.com/prometheus/jmx_exporter for more info about JMX Prometheus Exporter metrics
    lowercaseOutputName: true
    rules:
    # Special cases and very specific rules
    - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), topic=(.+), partition=(.*)><>Value
      name: kafka_server_$1_$2
      type: GAUGE
      labels:
       clientId: "$3"
       topic: "$4"
       partition: "$5"
    - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), brokerHost=(.+), brokerPort=(.+)><>Value
      name: kafka_server_$1_$2
      type: GAUGE
      labels:
       clientId: "$3"
       broker: "$4:$5"
    - pattern: kafka.server<type=(.+), cipher=(.+), protocol=(.+), listener=(.+), networkProcessor=(.+)><>connections
      name: kafka_server_$1_connections_tls_info
      type: GAUGE
      labels:
        cipher: "$2"
        protocol: "$3"
        listener: "$4"
        networkProcessor: "$5"
    - pattern: kafka.server<type=(.+), clientSoftwareName=(.+), clientSoftwareVersion=(.+), listener=(.+), networkProcessor=(.+)><>connections
      ...
      ...
      ...
      ...
        
  
```

- 
  





### 9) route

prometheus 에서 접근하기 위해서 라우터가 필요하다.

- route 생성

````

````





## 3.4 최종 yaml

### 1) 최종 kafka cluster yaml

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.1.0
    replicas: 3
    #authorization:
    #  type: simple         <-- scram-sha-512 방식 사용시 필수
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
        #authentication:
        #  type: scram-sha-512
      - name: tls
        port: 9093
        type: internal
        tls: true
      - name: route               <-- OpenShift routes
        port: 9094
        type: route
        tls: false
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.1"
    storage:
      type: ephemeral
    #storage:
    #  deleteClaim: false
    #  size: 64Gi
    #  type: persistent-claim
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
    #storage:
    #  deleteClaim: false
    #  size: 64Gi
    #  type: persistent-claim
  entityOperator:
    topicOperator: {}
    userOperator: {} 
    
    
    
---
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: arsenal-cluster
  namespace: kafka
spec:
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafka:
    authorization:
      type: simple
    config:
      log.message.format.version: '2.4'
      offsets.topic.replication.factor: 1
      transaction.state.log.min.isr: 1
      transaction.state.log.replication.factor: 1
    listeners:
      plain:
        authentication:
          type: scram-sha-512
      tls: {}
    replicas: 3
    storage:
      type: ephemeral
    #storage:
    #  deleteClaim: false
    #  size: 64Gi
    #  type: persistent-claim
  kafkaExporter:
    enableSaramaLogging: true
    groupRegex: .*
    livenessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    logging: debug
    readinessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    resources:
      limits:
        cpu: 500m
        memory: 128Mi
      requests:
        cpu: 200m
        memory: 64Mi
    template:
      pod:
        metadata:
          labels:
            app: arsenal-kafka
        terminationGracePeriodSeconds: 120
    topicRegex: .*
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
    #storage:
    #  deleteClaim: false
    #  size: 64Gi
    #  type: persistent-claim

```





### 2) yaml - 2022.04.16

#### 원본

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"kafka.strimzi.io/v1beta2","kind":"Kafka","metadata":{"annotations":{},"name":"sa-cluster","namespace":"kafka-system"},"spec":{"entityOperator":{"topicOperator":{},"userOperator":{}},"kafka":{"authorization":{"type":"simple"},"config":{"default.replication.factor":3,"inter.broker.protocol.version":"3.1","min.insync.replicas":2,"offsets.topic.replication.factor":3,"transaction.state.log.min.isr":2,"transaction.state.log.replication.factor":3},"listeners":[{"authentication":{"type":"scram-sha-512"},"name":"plain","port":9092,"tls":false,"type":"internal"},{"name":"tls","port":9093,"tls":true,"type":"internal"},{"authentication":{"type":"scram-sha-512"},"name":"route","port":9094,"tls":true,"type":"route"}],"replicas":3,"storage":{"type":"ephemeral"},"version":"3.1.0"},"zookeeper":{"replicas":3,"storage":{"type":"ephemeral"}}}}
  selfLink: /apis/kafka.strimzi.io/v1beta2/namespaces/kafka-system/kafkas/sa-cluster
  resourceVersion: '217668164'
  name: sa-cluster
  uid: e5949593-b9f9-4252-ba1b-6d20d68db8d7
  creationTimestamp: '2022-03-28T07:46:18Z'
  generation: 3
  namespace: kafka-system
spec:
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafka:
    authorization:
      type: simple
    config:
      default.replication.factor: 3
      inter.broker.protocol.version: '3.1'
      min.insync.replicas: 2
      offsets.topic.replication.factor: 3
      transaction.state.log.min.isr: 2
      transaction.state.log.replication.factor: 3
    listeners:
      - authentication:
          type: scram-sha-512
        name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
      - authentication:
          type: scram-sha-512
        name: route
        port: 9094
        tls: true
        type: route
    replicas: 3
    storage:
      type: ephemeral
    version: 3.1.0
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
status:
  clusterId: S-tI3sFhR6mx2BU3uYPIEw
  conditions:
    - lastTransitionTime: '2022-04-16T12:41:36.233Z'
      status: 'True'
      type: Ready
  listeners:
    - addresses:
        - host: sa-cluster-kafka-bootstrap.kafka-system-system.svc
          port: 9092
          ......

## 아래는 비정상일때
status:
  conditions:
    - lastTransitionTime: '2022-04-08T15:58:23.727Z'
      message: >-
        Operation: [get]  for kind: [Kafka]  with name: [sa-cluster]  in
        namespace: [kafka-system]  failed.
      reason: KubernetesClientException
      status: 'True'
      type: NotReady               <---
  observedGeneration: 3

```





#### 수정본

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka-system
spec:
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafka:
    authorization:
      type: simple
    config:
      default.replication.factor: 3
      inter.broker.protocol.version: '3.1'
      min.insync.replicas: 2
      offsets.topic.replication.factor: 3
      transaction.state.log.min.isr: 2
      transaction.state.log.replication.factor: 3
    listeners:
      - authentication:
          type: scram-sha-512
        name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
      - authentication:
          type: scram-sha-512
        name: route
        port: 9094
        tls: true
        type: route
    replicas: 3
    storage:
      type: ephemeral
    version: 3.1.0
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral

```





# 4.  KafkaUser

- KafkaUser 를 생성하면 secret 에 Opaque 가 생성되며 향후 인증 password 로 사용됨
- 어떤 topic 에 접근 가능할지를 명시할 수 있다.
- 그러므로 특정 user로 namespace별 topic 간 경계설정이 가능하다.



## 4.1. User 정책

- user 정책

```
[Part명]-user
[Part명]-[서비스명]-user
[Part명]-[서비스명]-[서브도메인]-user
```



- sample user 별 설명

```
ㅇ order-user
order로 시작하는 모든 topic을 처리할 수 있음
order로 시작하는 모든 group을 Consume 가능

ㅇ order-user-readonly
order로 시작하는 모든 topic을 읽을 수 있음
order로 시작하는 모든 group을 Consume 가능

ㅇ order-intl-user
order-intl로 시작하는 모든 topic을 처리할 수 있음
order-intl로 시작하는 모든 group을 Consume 가능

ㅇ order-intl-user-readonly
order-intl로 시작하는 모든 topic을 읽을 수 있음
order-intl로 시작하는 모든 group을 Consume 가능

ㅇ rater-user
rater로 시작하는 모든 topic을 처리할 수 있음
rater로 시작하는 모든 group을 Consume 가능
```



## 4.2 user 생성

```
cd /home/mobaxterm/song/kafka/strimzi/kafkaUser 
cat > 11.KafkaUser-my-bridge-user-my-topic.yaml
---
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: order-user
  labels:
    strimzi.io/cluster: arsenal-cluster
  namespace: kafka
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
    acls:
      - operation: All
        resource:
          type: topic
          name: order
          patternType: prefix     # 1)
      - operation: All
        resource:
          name: order            # 2)
          patternType: prefix
          type: group
      - operation: All
        resource:
          name: console           # 3)
          patternType: prefix
          type: group
---
```

- 1) order로 시작하는 topic 을 모두 처리가능
  - ex) order-intl-board-create,  order-intl-board-update
- 2) consumer group 은 항상 동일하다. 
  - ex) order-intl-board-group
- 3) default consumer-group 을 위해서 생성한다.



```sh
$ oc -n kafka create -f 11.KafkaUser-order-user-my-topic.yaml
$ oc -n kafka get secret order-user
NAME             TYPE      DATA      AGE
order-user   Opaque    1         26d

$ kubectl -n kafka get secret my-user


# user/pass 
  order-user / ppHG312DQ5fC
  my-user / RUqfDsDzvZdL
  
```





## 4.3 참고 yaml 

### 1) 참고 - my-bridge-user

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: my-bridge-user
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
    acls:
      - resource:
          type: topic
          name: rater       <--- rater.my-topic
          patternType: literal
          #patternType: prefix
        operation: All
        #operation: Read
      - operation: All
        resource:
          name: my-topic
          patternType: literal
          type: topic
      - operation: All
        resource:
          name: bridge-consumer-group
          patternType: literal
          type: group
      - operation: All
        resource:
          name: console
          patternType: prefix
          type: group
---
```



### 2) 참고 - my-user

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: my-user
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
    acls:
      - operation: All
        resource:
          name: my
          patternType: prefix
          type: topic
      - operation: All
        resource:
          name: my
          patternType: prefix
          type: group

```







### 3) 참고 - order-user

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: order-user
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  authentication:
    type: scram-sha-512
  authorization:
    acls:
      - operation: All
        resource:
          name: order
          patternType: prefix
          type: topic
      - operation: All
        resource:
          name: order
          patternType: prefix
          type: group
    type: simple

## 
```





# 5. KafkaTopic



## 5.1. Topic 정책 

- topic 정책

```
[Part명]-[서비스명]-[서브도메인]-[사용자정의]
```



- sample topic 

```
order-intl-board-create
order-intl-board-update
order-intl-board-delete

bill-intl-board-create
bill-intl-board-update
bill-intl-board-delete

rater-intl-board-create
rater-intl-board-update
rater-intl-board-delete
```





## 5.2 Topic 생성

### 1) Topic 생성

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-my-topic1
  labels:
    strimzi.io/cluster: sa-cluster
  namespace: kafka
spec:
  partitions: 3
  replicas: 1
  config:
    #retention.ms: 7200000      # 2시간
    retention.ms: 86400000      # 24시간
    segment.bytes: 1073741824   # 1GB

```

- partitions 1이면 producer 수행시 아래 메세지 발생할 수 있음.
  -  LEADER_NOT_AVAILABLE



## 5.3. 참고 yaml



### 1) 참고 yaml

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-intl-board-create
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  partitions: 3
  replicas: 3
  topicName: order-intl-board-create
  config:
    #retention.ms: 7200000      # 2시간
    retention.ms: 86400000      # 24시간
    segment.bytes: 1073741824   # 1GB
  
```



### 2) 참고 yaml

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-intl-board-create
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  partitions: 3
  replicas: 3
  topicName: order-intl-board-create
  config:
    #retention.ms: 7200000      # 2시간
    retention.ms: 86400000      # 24시간
    segment.bytes: 1073741824   # 1GB
 
 
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-intl-board-update
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  partitions: 3
  replicas: 3
  topicName: order-intl-board-update
  config:
    #retention.ms: 7200000      # 2시간
    retention.ms: 86400000      # 24시간
    segment.bytes: 1073741824   # 1GB
  
  
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-intl-board-delete
  namespace: kafka-system
  labels:
    strimzi.io/cluster: sa-cluster
spec:
  partitions: 3
  replicas: 3
  topicName: order-intl-board-delete
  config:
    #retention.ms: 7200000      # 2시간
    retention.ms: 86400000      # 24시간
    segment.bytes: 1073741824   # 1GB
  
```









# 6. internal access

- 일반적으로는 인증(user/pass)없이 테스트가 됨.    
- cluster 생성시 authentication.type이 scram-sha-512 일 경우 반드시 인증(user/pass) 가 있어야 함.
- topic 관련 sh 은 zookeeper 에 연결되므로 sha-512인증필요 없이 바로 접속 가능
- producer/consumer 는 brocker 에 접속되며느로 인증이 필요함.

```
$ bin/kafka-topics.sh --create --topic users.registrations --replication-factor 1 \
  --partitions 2  --zookeeper localhost:2181
  
$ bin/kafka-topics.sh --create --topic users.verfications --replication-factor 1 \
  --partitions 2  --zookeeper localhost:2181



kubectl -n kafka run kafka-producer -ti --image=ktis-bastion01.container.ipc.kt.com:5000/strimzi/kafka:0.28.0-kafka-3.1.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list sa-cluster-kafka-bootstrap:9092 --topic my-topic

# producer
bin/kafka-console-producer.sh --broker-list sa-cluster-kafka-bootstrap:9092 --topic my-topic

# consumer
bin/kafka-console-consumer.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

```





## 6.1 kafka  client test

<-- zookeeper 를 인식하지 못하는 현상 존재함

```sh
# sa-cluster-kafka-2 pod 내에서 실행

# 1) topic list
$ cd /opt/kafka
bin/kafka-topics.sh --zookeeper localhost:2181 --list
bin/kafka-topics.sh --zookeeper sa-cluster-zookeeper-client:2181 --list
bin/kafka-topics.sh --zookeeper sa-cluster-zookeeper-client:2181 --list    


Exception in thread "main" joptsimple.UnrecognizedOptionException: zookeeper is not a recognized option
        at joptsimple.OptionException.unrecognizedOption(OptionException.java:108)
        at joptsimple.OptionParser.handleLongOptionToken(OptionParser.java:510)
        at joptsimple.OptionParserState$2.handleArgument(OptionParserState.java:56)
        at joptsimple.OptionParser.parse(OptionParser.java:396)
        at kafka.admin.TopicCommand$TopicCommandOptions.<init>(TopicCommand.scala:567)
        at kafka.admin.TopicCommand$.main(TopicCommand.scala:47)
        at kafka.admin.TopicCommand.main(TopicCommand.scala)
        
<-- zookeeper 를 인식하지 못함.  왜 안될까...


# describe topic
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic order-my-topic1
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic order-my-topic3
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic order-my-topic
<-- 잘 안됨.



<-- 정상 리턴 샘플
Topic: order-my-topic1 PartitionCount: 3       ReplicationFactor: 1    Configs: segment.bytes=1073741824,retention.ms=7200000
        Topic: order-my-topic1 Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: order-my-topic1 Partition: 1    Leader: 2       Replicas: 2     Isr: 2
        Topic: order-my-topic1 Partition: 2    Leader: 1       Replicas: 1     Isr: 1

# delete topic
bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic order.my-topic2
-- 바로 삭제 되지 않고 makred for deletion 처리된다.

# create topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic order-my-topic2

```





## 6.2 consumer/producer

- no 인증시

```sh
# sa-cluster-kafka-2 pod 내에서 실행

# 1) consumer
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic2
bin/kafka-console-consumer.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --topic my-topic
bin/kafka-console-consumer.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning



# 2) producer
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic2
bin/kafka-console-producer.sh --broker-list sa-cluster-kafka-bootstrap:9092 --topic my-topic
bin/kafka-console-producer.sh --broker-list sa-cluster-kafka-bootstrap.kafka-system.svc:9092 --topic my-topic
bin/kafka-console-producer.sh --broker-list sa-cluster-kafka-bootstrap.kafka-system.svc.cluster.local:9092 --topic my-topic


# 3) kafka-consumer-groups
# 3.1) CG목록
bin/kafka-consumer-groups.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --list
console-consumer-67956
order-consumer-group

# 3.2) CG desc
bin/kafka-consumer-groups.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --describe --group order-consumer-group2

bin/kafka-consumer-groups.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --describe --group console-consumer-51928


GROUP                 TOPIC            PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                                           HOST            CLIENT-ID
order-consumer-group order-my-topic2 0          30              30              0               consumer-order-consumer-group-1-473b0d30-c2e7-4af3-9bea-592362d46a32 /10.1.2.36      consumer-order-consumer-group-1
order-consumer-group order-my-topic2 1          13              13              0               consumer-order-consumer-group-1-473b0d30-c2e7-4af3-9bea-592362d46a32 /10.1.2.36      consumer-order-consumer-group-1
order-consumer-group order-my-topic2 2          7               7               0               consumer-order-consumer-group-1-473b0d30-c2e7-4af3-9bea-592362d46a32 /10.1.2.36      consumer-order-consumer-group-1
order-consumer-group test20180604     0          0               0               0               -                                                                     -              -



# 3.3) CG추가

# 3.3) CG삭제
bin/kafka-consumer-groups.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --delete --group order-consumer-group2

# 3.3) 구독취소


```

- 인증시

```
인증파일 이용
```





## 6.3 kafkacat



### 1) kafkacat설치
```

$ kubectl -n kafka create deploy kafkacat \
    --image=confluentinc/cp-kafkacat:latest \
    -- sleep 365d

$ kubectl -n kafka exec -it deploy/kafkacat -- bash

## delete deploy
$ kubectl -n kafka delete deploy kafkacat 


```





### 2) no 인증시

```sh


# 테스트
$ export BROKERS=sa-cluster-kafka-bootstrap:9092
$ export TOPIC=my-topic
 
// topic 리스트
$ kafkacat -b $BROKERS -L

Metadata for all topics (from broker -1: sa-cluster-kafka-bootstrap:9092/bootstrap):
 3 brokers:
  broker 0 at sa-cluster-kafka-0.sa-cluster-kafka-brokers.kafka-system.svc:9092
  broker 2 at sa-cluster-kafka-2.sa-cluster-kafka-brokers.kafka-system.svc:9092 (controller)
  broker 1 at sa-cluster-kafka-1.sa-cluster-kafka-brokers.kafka-system.svc:9092
 11 topics:
  topic "my-topic2" with 1 partitions:
    partition 0, leader 2, replicas: 2, isrs: 2
  topic "my-board-delete-topic-dlq" with 1 partitions:
    partition 0, leader 1, replicas: 1,0,2, isrs: 1,2,0
  topic "__consumer_offsets" with 50 partitions:
    partition 0, leader 2, replicas: 2,1,0, isrs: 1,2,0
    partition 1, leader 1, replicas: 1,0,2, isrs: 1,2,0
    partition 2, leader 0, replicas: 0,2,1, isrs: 1,2,0
    partition 3, leader 2, replicas: 2,0,1, isrs: 1,2,0
    partition 4, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 5, leader 0, replicas: 0,1,2, isrs: 1,2,0
    partition 6, leader 2, replicas: 2,1,0, isrs: 1,2,0
    partition 7, leader 1, replicas: 1,0,2, isrs: 1,2,0
    partition 8, leader 0, replicas: 0,2,1, isrs: 1,2,0
    partition 9, leader 2, replicas: 2,0,1, isrs: 1,2,0
    partition 10, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 11, leader 0, replicas: 0,1,2, isrs: 1,2,0
    partition 12, leader 2, replicas: 2,1,0, isrs: 1,2,0
    partition 13, leader 1, replicas: 1,0,2, isrs: 1,2,0
    partition 14, leader 0, replicas: 0,2,1, isrs: 1,2,0
    partition 15, leader 2, replicas: 2,0,1, isrs: 1,2,0
    partition 16, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 17, leader 0, replicas: 0,1,2, isrs: 1,2,0
    partition 18, leader 2, replicas: 2,1,0, isrs: 1,2,0
    partition 19, leader 1, replicas: 1,0,2, isrs: 1,2,0
    partition 20, leader 0, replicas: 0,2,1, isrs: 1,2,0
    partition 21, leader 2, replicas: 2,0,1, isrs: 1,2,0
    partition 22, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 23, leader 0, replicas: 0,1,2, isrs: 1,2,0

    
// producer
$ kafkacat -b $BROKERS \
  -t my-topic -P -X acks=1
 
//consumer
$ kafkacat -b $BROKERS \
  -t my-topic -C



```



### 3) 인증시

id/pass 가 필요

```sh
$ kubectl -n kafka exec -it deploy/kafkacat -- bash



export BROKERS=my-cluster-kafka-bootstrap:9092
export KAFKAUSER=my-user
export PASSWORD=RUqfDsDzvZdL
export TOPIC=my-topic
 
## topic 리스트
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD -L

Metadata for all topics (from broker -1: sasl_plaintext://my-cluster-kafka-bootstrap:9092/bootstrap):
 3 brokers:
  broker 0 at my-cluster-kafka-0.my-cluster-kafka-brokers.kafka.svc:9092
  broker 2 at my-cluster-kafka-2.my-cluster-kafka-brokers.kafka.svc:9092
  broker 1 at my-cluster-kafka-1.my-cluster-kafka-brokers.kafka.svc:9092 (controller)
 1 topics:
  topic "my-topic" with 3 partitions:
    partition 0, leader 0, replicas: 0, isrs: 0
    partition 1, leader 1, replicas: 1, isrs: 1
    partition 2, leader 2, replicas: 2, isrs: 2

    
    
## consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -C -o -5

<-- OK 

## consumer group
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -C \
  -X group.id=order-intl-board-group

<-- OK 



## consumer group
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -C \
  -X group.id=order-intl-board-group -o -5

<-- OK 




## producer : 입력모드
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -P -X acks=1
 
<-- OK



## 대량 발송 모드
$ cat > msg.txt
---
{"eventName":"a","num":1,"title":"a", "writeId":"", "writeName": "", "writeDate":"" }
---

## producer : file mode
kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -P ./msg.txt

<-- OK

## producer : while
while true; do kafkacat -b $BROKERS \
  -X security.protocol=SASL_PLAINTEXT \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -t $TOPIC -P ./msg.txt; done;

<-- OK



```









## 6.3 Node Port - 실패



### 1) nodeport service 이용하는 방법 - 실패

- 인프라에서 보내준 node port 사용 권고사항

```

두 개 이상의 노드에서 실행되도록 설정하신 경우에는
 1) worker node n대에 대한 host port 오픈 또는
 2) service를 통한 HA가 가능하도록 node port 를 사용하실 수 있습니다. 
  - infra ip(10.217.166.60, 10.217.166.64) 에 사용하시는 node port 오픈 하시면 됩니다. 

```



- nodeport service 생성

```
kind: Service
apiVersion: v1
metadata:
  name: sa-cluster-kafka-bootstrap-nodeport
  namespace: kafka-system
  labels:
    app.kubernetes.io/instance: sa-cluster
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: kafka
    app.kubernetes.io/part-of: strimzi-sa-cluster
    strimzi.io/cluster: sa-cluster
    strimzi.io/discovery: 'true'
    strimzi.io/kind: Kafka
    strimzi.io/name: sa-cluster-kafka
spec:
  ports:
    - name: tcp-clients
      protocol: TCP
      port: 9092
      targetPort: 9092
  selector:
    strimzi.io/cluster: sa-cluster
    strimzi.io/kind: Kafka
    strimzi.io/name: sa-cluster-kafka
  type: NodePort
---

  ports:
    - name: tcp-clients
      protocol: TCP
      port: 9092
      targetPort: 9092
      nodePort: 32592            <-- 생성된 nodeport
      
```



```
# sa-cluster-kafka-2 pod 내에서 실행

# 1) consumer
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic

# 2) producer
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic

## node port 이용한 producer
bin/kafka-console-producer.sh --broker-list 10.217.166.60:32592 --topic my-topic

sh-4.4$ bin/kafka-console-producer.sh --broker-list 10.217.166.60:32592 --topic my-topic
>[2022-04-16 12:09:17,144] WARN [Producer clientId=console-producer] Connection to node -1 (/10.217.166.60:32592) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,144] WARN [Producer clientId=console-producer] Bootstrap broker 10.217.166.60:32592 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,241] WARN [Producer clientId=console-producer] Connection to node -1 (/10.217.166.60:32592) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,241] WARN [Producer clientId=console-producer] Bootstrap broker 10.217.166.60:32592 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,394] WARN [Producer clientId=console-producer] Connection to node -1 (/10.217.166.60:32592) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,394] WARN [Producer clientId=console-producer] Bootstrap broker 10.217.166.60:32592 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,597] WARN [Producer clientId=console-producer] Connection to node -1 (/10.217.166.60:32592) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
[2022-04-16 12:09:17,597] WARN [Producer clientId=console-producer] Bootstrap broker 10.217.166.60:32592 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)


<-- 테스트 실패
안된다.   단순 nodeport type 의 서비스 생성이 아닌 nodeport type 의 클러스터가 필요하다.

```



```
안녕하세요?

kafka도 nodeport 로 열면 편할것 같아서 몇몇 테스트 해봤습니다.
그냥 단순히 nodeport service 하나를 sa-cluster-kafka-bootstrap 서비스와 비슷하게 선언하고 테스트했는데 연결이 안되네요.
좀더 자세히 찾아보니 아래와 같이 type을 nodeport 로 선언하고 클러스터를 재 생성 해야하네요.

listeners:
  # ...
  - name: external
    port: 9094
    type: nodeport
    tls: false    

이는 우리가 route 로 설정해 놓은 것과 택일해야 하는 것처럼 보입니다.

listeners:
  # ...
  - name: route   
    port: 9094
    tls: true
    type: route
    
    
Kafka 는 nodeport 이용대신 route 이용하는 방식으로 가야 할듯 합니다.
참조링크: https://strimzi.io/blog/2019/04/23/accessing-kafka-part-2/

송양종드림.
```





### 2) nodeport Cluster 이용

- sa-cluster 생성 yaml

```
listeners:
  # ...
  - name: external
    port: 9094
    type: nodeport
    tls: false    
```

이 방법을 사용하려면 route 를 사용하지 못하는듯...



# 7. external access

openshift route 기능을 이용하여 외부에서 접근가능하다.  route 를 통환 접근은  tls 접속만 허용되므로 인증서를 이용한 jks 파일이 필요하다.



## 7.1 openshift route - no 인증

참조링크: https://strimzi.io/blog/2019/04/30/accessing-kafka-part-3/

### 1) rout 등록

```yaml

## route 등록을 위한 listener 등록후 apply 수행
---
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
  ...
spec:
  ...
  kafka:
    ...
    listeners:
      - name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
      - name: route     <-- route 등록을 위한 listener 등록
        port: 9094
        tls: true
        type: route
---


## 확인
$ oc -n kafka get route
NAME                               HOST/PORT                                                                               PATH   SERVICES                           PORT   TERMINATION   WILDCARD
sa-cluster-kafka-route-0           sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr                  sa-cluster-kafka-route-0           9094   passthrough   None
sa-cluster-kafka-route-1           sa-cluster-kafka-route-1-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr                  sa-cluster-kafka-route-1           9094   passthrough   None
sa-cluster-kafka-route-2           sa-cluster-kafka-route-2-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr                  sa-cluster-kafka-route-2           9094   passthrough   None
sa-cluster-kafka-route-bootstrap   sa-cluster-kafka-route-bootstrap-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr          sa-cluster-kafka-route-bootstrap   9094   passthrough   None


	
https://sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr


## 접근가능한 route 4개 생성완료

```

 

### 2) [Client test] 인증서 export

client 에서 사용할 인증서 획득

이 인증서는 1년짜리 인정서이다. 향후 연장할 수 있는 방안을 마련해야 한다.

```sh
## openshift 에서 인증서 획득
$ oc -n kafka-system extract secret/sa-cluster-cluster-ca-cert --keys=ca.crt  --to=- > ca.crt

$ cat  ca.crt
-----BEGIN CERTIFICATE-----
MIIFLTCCAxWgAwIBAgIUNPtIMUxZaOcFbXxzjONkswsM7U8wDQYJKoZIhvcNAQEN
BQAwLTETMBEGA1UECgwKaW8uc3RyaW16aTEWMBQGA1UEAwwNY2x1c3Rlci1jYSB2
MDAeFw0yMjA0MTYxMTM2MzdaFw0yMzA0MTYxMTM2MzdaMC0xEzARBgNVBAoMCmlv
LnN0cmltemkxFjAUBgNVBAMMDWNsdXN0ZXItY2EgdjAwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQDIHzpUqQjWDIjxnktUY6eR8RJdntWsRYdzekCFQfZH
p+u/BhhrMlIvcvlEzScNpxKJlJPzqzGKKhI5v7O1pMBIAGrkmCGvWV5KiJ95rvKt
WbUwfCdnvOrUXRPdJR4gSRAxj1ft/C+i60wkVyBVAULXzDyqUveg7ZE6EjE2XxvG
+W4KrPEUOoAQhljGUzAXt+kQlfuo2TCKxtLG/LkErCaNAH70DCb35NtEMqpQcAJN
wD5/mRZbgR+pjmqKUPEOB1mMWdV+dDcSJ5g/8H7f6OuDq9Rb9fheCD6sAOH5Hv+N
bHsn+6697zrQzZLENZVCAbvUTavhHWsz83ub6cpatLql18KRvLdTyLmxw9ivZ6nd
6dqBaaYHYTBZHgUjhzqAuwDJJU33Ag0dl3Xsm8J/4SzhIdfd0MOGTYIurkY9bBaJ
/zwhNUVmcKwiB8AhouMu6L7tuwQnoqpa7czdYTcQiDWTL2cZU/EkxE9W5dUDKFjD
zvy7r4n0HaPMhoILyHgTViFPt8D2WCXGjl8wY+2tnPczWglq+s8Bl+NIo2K2rGld
MKAO8UjK4lovdstOfNnjWg+HyeNYDspyzSFYv+0N4wblP61oQJkvEQTPOeYoafOC
0P/qNDmEmnfhx/E3NedhKM3vop2b5HiswuZ+S0uewws/thIQAlgtkdluBY9tR1b/
lwIDAQABo0UwQzAdBgNVHQ4EFgQUACQloeODbESTtimFbq2kGHv6+sowEgYDVR0T
AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAQYwDQYJKoZIhvcNAQENBQADggIB
AMMa6+o1OBNQmJV6d116YtXxkudy2eUe+kg4eDBDN48YKfFVihtNK1zr42hX68O5
1x5u3DNm1YfTn1hBTvVFnfF8wXZwnI2qNDQecD3iSjR5PR6vs15IDGoXH3pGdikX
4utD8oNEjBqwyeMUHUuYWs0ibRfUtsGIlLHzgwwiecnBVWwwqGReUC3r2qd41fTe
epD/gTrLqWIPZWVrg6DQdTO83mPfJJpBXLEnneqlGEcYfYWwBah22PEB3q4nJhSU
8F4651MiRoYABJIJQd+YuexTFLbSYKl9Uw6JJTqso8rOgDek3ycjzWjpxpW6qdGc
iZlCkei/pWoZRcDSxK8nPhuYyDl7GG5/5e6D2eECtXmK3VbB/53Yo4VCE1slGked
17oCtcaVOtchajfcUo0Yb1eHHJYqHIiP9JM9KCIuejIaBgGLvCWvN6W8Qiu4EiVt
q95eOMnSU/1kjR3Yawq9HhZioSSMQL/uhjr5o4KcD1CgCoVYUsgh7KM3IWqA5WAy
zLfz0VNuMwSi4kqUBhprBUZUil4AX9c7C/GK+WVuCcBebMZEm1Zwy/oS9BhZoyCT
MScAvOl+saYvrPFJmgO0IYb+1Dy/Q1EfM5ZJ7dnThjfgEj7oNYjMo16zfTX2biJR
RDvdsS3PzkM0Fisr9YP7tMgwQxkUlgqx2MVB3e58jllB
-----END CERTIFICATE-----
```





### 3) kafka client 실행 (docker 이용)

```sh
$ docker run --name kafka-client -d --rm --user root bitnami/kafka:latest sleep 365d

```





### 4) client 접근을 위한 jks 생성

```sh

## 1. jks 파일 생성
$ cd /opt/bitnami/kafka

$ cat > ca.crt
## -- 인증서 붙여넣기 --


## 2. jks 파일 생성
$ keytool -import -trustcacerts \
    -alias root \
    -file ca.crt \
    -keystore truststore.jks \
    -storepass new1234 \
    -noprompt


## 3. 확인
$ keytool -list -keystore truststore.jks
Enter keystore password:
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

root, Mar 11, 2022, trustedCertEntry,
Certificate fingerprint (SHA-256): 2A:F1:53:D1:B0:E4:00:54:67:E0:2F:E3:D2:E6:83:0D:69:10:27:C5:F7:B3:05:0D:8B:86:29:02:D2:75:30:96


```



### 5) hosts 파일등록

container 내부 에서 cluster kafka 에 접근하기 위해서는 host 별 IP(L4) 매핑이 필요하다.  

```sh

## /etc/hosts 파일에 아래 등록
$ vi /etc/hosts
...
10.217.166.67  sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-0-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-1-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-2-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr


```



### 6) producer / consumer test 

- consumer 
  - prducere 확인을 위해서 consumer 를 하나 실행 해 놓자.

```sh
## sa-cluter-kafka-0 pod 내에서
$ bin/kafka-console-consumer.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --topic my-topic
```



- producer

```sh


export BROKERS=sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
export TOPIC=my-topic
export KAFKAUSER=my-user
export PASSWORD=StzbQLCa1XHH


## openshift connect
$ cd /opt/bitnami/kafka

## producer
$ bin/kafka-console-producer.sh \
  --broker-list $BROKERS \
  --producer-property security.protocol=SSL \
  --producer-property ssl.truststore.password=new1234 \
  --producer-property ssl.truststore.location=./truststore.jks \
  --topic my-topic

<-- 테스트 성공  2022.04.16


<-- cluster 내부 DNS 가 문제가 있는지 되다 안되다 하는 현상이 존재함
<-- 오류내용
java.net.UnknownHostException: sa-cluster-kafka-route-2-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
        at java.base/java.net.InetAddress$CachedAddresses.get(InetAddress.java:797)
        at java.base/java.net.InetAddress.getAllByName0(InetAddress.java:1509)


<-- 정상작동하지 않음.... 이상하다. 
[2022-04-16 10:11:39,855] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 10:11:40,337] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 10:11:40,856] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
-- kafka operator 이상있는 경우이다. 재기동 하자.





## 특정 노드로 접근하는 route 로 해보자.
$ bin/kafka-console-producer.sh \
  --broker-list sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 \
  --producer-property security.protocol=SSL \
  --producer-property ssl.truststore.password=new1234 \
  --producer-property ssl.truststore.location=./truststore.jks \
  --topic my-topic

<-- 정상 작동 확인함

```



### 7) kafka cat 이용

```sh

export BROKERS=sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
export TOPIC=my-topic

  
## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X ssl.ca.location=./ca.crt -L


## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1


## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C

 
## consumer 처음부터 읽어 확인할 경우
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C -o beginning


```



## 7.2 openshift route - 인증

### 1) rout 등록

```yaml
## route 등록을 위한 listener 등록후 apply 수행
---
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
  ...
spec:
  ...
  kafka:
    authorization:
      type: simple  <-- 인증
    ...
    listeners:
      - name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
      - authentication:
          type: scram-sha-512  <-- 인증
        name: route            <-- route 등록을 위한 listener 등록
        port: 9094
        tls: true
        type: route
---

```

 





### 2) producer / consumer test 

- consumer 
  - prducere 확인을 위해서 consumer 를 하나 실행 해 놓자.

```sh
## sa-cluter-kafka-0 pod 내에서
$ bin/kafka-console-consumer.sh --bootstrap-server sa-cluster-kafka-bootstrap:9092 --topic my-topic
```



- producer

```sh

export BROKERS=sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
export TOPIC=my-topic
export KAFKAUSER=my-user
export PASSWORD=StzbQLCa1XHH


## openshift connect
$ cd /opt/bitnami/kafka

## producer
$ bin/kafka-console-producer.sh \
  --broker-list $BROKERS \
  --producer-property security.protocol=SSL \
  --producer-property ssl.truststore.password=new1234 \
  --producer-property ssl.truststore.location=./truststore.jks \
  --topic my-topic

<-- 테스트 성공  2022.04.16


<-- cluster 내부 DNS 가 문제가 있는지 되다 안되다 하는 현상이 존재함
<-- 오류내용
java.net.UnknownHostException: sa-cluster-kafka-route-2-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
        at java.base/java.net.InetAddress$CachedAddresses.get(InetAddress.java:797)
        at java.base/java.net.InetAddress.getAllByName0(InetAddress.java:1509)


<-- 정상작동하지 않음.... 이상하다. 
[2022-04-16 10:11:39,855] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 10:11:40,337] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2022-04-16 10:11:40,856] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
-- kafka operator 이상있는 경우이다. 재기동 하자.

[2022-04-21 14:38:04,545] WARN [Producer clientId=console-producer] Bootstrap broker sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)





## 특정 노드로 접근하는 route 로 해보자.
$ bin/kafka-console-producer.sh \
  --broker-list sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443 \
  --producer-property security.protocol=SSL \
  --producer-property ssl.truststore.password=new1234 \
  --producer-property ssl.truststore.location=./truststore.jks \
  --topic my-topic

<-- 정상 작동 확인함

```



### 3) kafka cat 이용

```sh
export BROKERS=sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
export TOPIC=my-topic

  
## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X ssl.ca.location=./ca.crt -L


## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1


## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C

 
## consumer 처음부터 읽어 확인할 경우
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C -o beginning


```

테스트는 아래   kafkacat  이용





## 7.3 Ingress - 인증

### 1) Ingress 등록

외부에서는 안된다. ㅠㅠ  -- 2022.05.29

```yaml
## route 등록을 위한 listener 등록후 apply 수행
---
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
  ...
spec:
  ...
  kafka:
    authorization:
      type: simple  <-- 인증
    ...
    listeners:
      - name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
      - authentication:
          type: scram-sha-512
        name: external
        port: 9094
        type: ingress
        tls: true
        configuration:
          bootstrap:
            host: bootstrap.kafka.ktcloud.211.254.212.105.nip.io
          brokers:
          - broker: 0
            host: broker-0.kafka.ktcloud.211.254.212.105.nip.io
          - broker: 1
            host: broker-1.kafka.ktcloud.211.254.212.105.nip.io
          - broker: 2
            host: broker-2.kafka.ktcloud.211.254.212.105.nip.io
---

```



### 2) 인증서 획득



```sh

$ kubectl -n kafka get secret my-cluster-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 -d

-----BEGIN CERTIFICATE-----
MIIFLTCCAxWgAwIBAgIUKBNsdw/sbFA0OR5O94le00UiSmcwDQYJKoZIhvcNAQEN
BQAwLTETMBEGA1UECgwKaW8uc3RyaW16aTEWMBQGA1UEAwwNY2x1c3Rlci1jYSB2
MDAeFw0yMjA1MjkwMjQ5NDdaFw0yMzA1MjkwMjQ5NDdaMC0xEzARBgNVBAoMCmlv
LnN0cmltemkxFjAUBgNVBAMMDWNsdXN0ZXItY2EgdjAwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQDBJ/+j7VdEZ3lnuvZwh169J+ODqFyEcHZodPU332Yt
q3n2kDrpm67d4ZTDNJAxItjO6Bo67AHpgEwLydi116ukJR9YOFYSBDqtTr+Lm6Gh
yB6vlmPR0Zg9vtGGbYMz0rsPoj7r85YpAKuMygXzXunjpLznWyVIPTkOP77gJmT2
q79KBEPZeOwVm/zrBLLC+98jCPfT52UShkNMwagPnhTz/VszAGNYXpBUqDygPRZD
4vc/x+8hZMvPEe2vEXt9rd3Pmcl9PaD9EgH9yN5u6kcnxd/M2vadoLjoq4lci2Ti
jtX+9j2o6QIDRFcHRtOF3y4/H3wM4F8eYEI2NQVdyRHFgLVR+nwp5lFRScpv00xM
M4pxaJoQds42xwSZ0DaBKewV28yQG/6GhFWMYVXbils24xdSylZwi5Knr57NSr4H
WeWvuuAYlWnwtcnsGCRRIQEJR/7B9L6q0Q31jECtT6WGbZwUX9Xqhw1gEbdKu/hT
+Zyo3Q6e6JrYyCes2eDYpVzF3MDyfv1wO05gNvxlsFSMhoAGwXbiS4EVqpw8QtxR
/WsHMTpyDnz29LJynE/NYf0PQ17PJdzLsFNP2zYNgE5X9KRmFVd6CxgNhdpB59lw
gpeBvPZlJ6LfYTU6ljGptA7T5+dQ3og/RfEU1I5E+Y8kocCbeweiVNyTQMLklh7o
lwIDAQABo0UwQzAdBgNVHQ4EFgQUh0lZfVAJhhJWMcDB1uKvDk1s750wEgYDVR0T
AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAQYwDQYJKoZIhvcNAQENBQADggIB
AA/b/E+9ZncsHiyuvo+6YGtjwgCSjtMN2ypiNdRW5PdZy6vdeZWLpLra+1/Dl8tb
+mEqjh5c25oMW8VO4hMJGwmD4TA9h7UzHZQBSKj5m7KsOQJYOmJChBl6WMZBhTaA
NXainBZzFZU0v3LKJR0yUmn1EylL/UiEMnmLmjqgB69Q3xe2sidmH5qji6mmZ6Kr
Lpfd6TggO4M435PJ1unjw/XuZQ1tji3O9sf5xTfGRRgZKz48hEXZqjfQZmjn2FEL
OEgCSinBwO+MtO3BRQ22Q/+YkUjbr/aS0iAb9+KynJhE8vHGCxWYpOO5z96m2s4G
ntoiY9iu0rjHrg5AEY/JZ1DIcqoEVbs04MCeBDYnY0GXm5QA/CuWjL76Tt4Ralzf
kro7L+5yv09pD0cHtiNCUM6Wbi5yMU1/allKa5GOHByhbD+9whUNUXuBWVKBprxb
nEd8rmMEo63dNAQY4M5jYCtl6d8/SgnBqO4xzj7RDeL6f4bmgxO52Y9arGc/ehXl
6Yft7f/1grnWSEOLJnNc7DuClJ+cb/FJkJZYaSAWvtWveWa7yqQsV39wkBy0VUyF
PYS4QVnN6N5cD90FElxDZM0wuYP/kpw5fiHvBY8/ifcVJSw+TrjBw6m/1O/5Hwe2
3t1uC5xezMXJ+jW2O06+bQzBwuGwjr0rJx+ez9H/dOXz
-----END CERTIFICATE-----



$ cat > ca.crt



```



### 3) kafkacat

```sh

            host: bootstrap.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-0.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-1.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-2.kafka.ktcloud.211.254.212.105.nip.io


export BROKERS=bootstrap.kafka.ktcloud.211.254.212.105.nip.io:443
export BROKERS="bootstrap.kafka.ktcloud.211.254.212.105.nip.io:443,broker-0.kafka.ktcloud.211.254.212.105.nip.io:443,broker-1.kafka.ktcloud.211.254.212.105.nip.io:443,broker-2.kafka.ktcloud.211.254.212.105.nip.io:443"
export KAFKAUSER=my-user
export PASSWORD=RUqfDsDzvZdL
export TOPIC=my-topic
export GROUP=my-topic-group
    
      

## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X sasl.username=$KAFKAUSER \
   -X sasl.password=$PASSWORD \
   -X ssl.ca.location=./ca.crt -L

<-- 성공
% ERROR: Failed to acquire metadata: Local: Broker transport failure   <-- 주소인식이 안된경우



## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공


## 파일의 내용을 보내기
cat > msg.txt
abcdefg
---

## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1 msg.txt
<-- 성공



## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C
<-- 성공
 
 
## consumer 처음부터 읽어 확인할 경우
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C -o beginning
<-- 성공



## Consumer Group-id
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -G $GROUP  -C
<-- 성공
    
    




```





### 4) 임시 kafka-client CLI 

```sh
$ kubectl -n kafka create deploy kafka-client \
    --image=quay.io/strimzi/kafka:0.29.0-kafka-3.2.0 \
    -- sleep 365d


$ kubectl -n kafka exec -it deploy/kafka-client -- bash

## delete deploy
$ kubectl -n kafka delete deploy kafka-client

```







```sh

$ kubectl -n kafka exec -it deploy/kafka-client -- bash

cat > ca.crt
...


keytool -import -trustcacerts -alias root -file ca.crt -keystore truststore.jks -storepass new1234 -noprompt


            host: bootstrap.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-0.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-1.kafka.ktcloud.211.254.212.105.nip.io
            host: broker-2.kafka.ktcloud.211.254.212.105.nip.io
            
bin/kafka-console-producer.sh --broker-list bootstrap.kafka.ktcloud.211.254.212.105.nip.io:443 --producer-property security.protocol=SSL --producer-property ssl.truststore.password=password --producer-property ssl.truststore.location=./truststore.jks --topic my-topic


bin/kafka-console-producer.sh --broker-list bootstrap.192.168.64.46.nip.io:443 --producer-property security.protocol=SSL --producer-property ssl.truststore.password=password --producer-property ssl.truststore.location=./truststore.jks --topic <your-topic>


```







## 7.4 NodePort - 인증

### 1) NodePort  등록

외부에서는 안된다. ㅠㅠ  -- 2022.05.29

```yaml
## route 등록을 위한 listener 등록후 apply 수행
---
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: sa-cluster
  namespace: kafka
  ...
spec:
  ...
  kafka:
    authorization:
      type: simple  <-- 인증
    ...
    listeners:
      - name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal        
      - authentication:
          type: scram-sha-512
        name: external
        port: 9094
        type: nodeport
        tls: true        
        configuration:
          bootstrap:
            nodePort: 32100
          brokers:
          - broker: 0
            nodePort: 32000
          - broker: 1
            nodePort: 32001
          - broker: 2
            nodePort: 32002
---

```





### 3) kafkacat - internal

```sh


export BROKERS="172.27.0.2:32100"
export BROKERS="172.27.0.2:32100,172.27.0.2:32000,172.27.0.2:32001,172.27.0.2:32002"
export KAFKAUSER=my-user
export PASSWORD=RUqfDsDzvZdL
export TOPIC=my-topic
export GROUP=my-topic-group
    
      

## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X sasl.username=$KAFKAUSER \
   -X sasl.password=$PASSWORD \
   -X ssl.ca.location=./ca.crt -L

<-- 성공


## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공


## 파일의 내용을 보내기
cat > msg.txt
abcdefg
---

## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1 msg.txt
<-- 성공



## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C
<-- 성공
 
```







### 4) kafkacat - external

외부에서는 안된다. ㅠㅠ  -- 2022.05.29

```sh

export BROKERS="211.254.212.105:32100"
export BROKERS="211.254.212.105:32100,211.254.212.105:32000,211.254.212.105:32001,211.254.212.105:32002"
export KAFKAUSER=my-user
export PASSWORD=RUqfDsDzvZdL
export TOPIC=my-topic
export GROUP=my-topic-group
    
      

## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X sasl.username=$KAFKAUSER \
   -X sasl.password=$PASSWORD \
   -X ssl.ca.location=./ca.crt -L

<-- 성공


## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공


## 파일의 내용을 보내기
cat > msg.txt
abcdefg
---

## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1 msg.txt
<-- 성공



## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C
<-- 성공
 
```











## 7.9 [확인] kafkacat 이용



### 1) docker run

```

docker login nexus.dspace.kt.co.kr -u icistr-cmmn-readonly -p icis1234

docker pull nexus.dspace.kt.co.kr/confluentinc/cp-kafkacat:latest

docker run --name kafkacat -d --user root nexus.dspace.kt.co.kr/confluentinc/cp-kafkacat:latest sleep 365d

docker run --name kafkacat -d --user root confluentinc/cp-kafkacat:latest sleep 365d

```



### 2) ca 인증서



```
## openshift 에서 인증서 획득
$ oc -n kafka-system extract secret/sa-cluster-cluster-ca-cert --keys=ca.crt  --to=- > ca.crt

$ cat  ca.crt
-----BEGIN CERTIFICATE-----
MIIFLTCCAxWgAwIBAgIUKBNsdw/sbFA0OR5O94le00UiSmcwDQYJKoZIhvcNAQEN
BQAwLTETMBEGA1UECgwKaW8uc3RyaW16aTEWMBQGA1UEAwwNY2x1c3Rlci1jYSB2
MDAeFw0yMjA1MjkwMjQ5NDdaFw0yMzA1MjkwMjQ5NDdaMC0xEzARBgNVBAoMCmlv
LnN0cmltemkxFjAUBgNVBAMMDWNsdXN0ZXItY2EgdjAwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQDBJ/+j7VdEZ3lnuvZwh169J+ODqFyEcHZodPU332Yt
q3n2kDrpm67d4ZTDNJAxItjO6Bo67AHpgEwLydi116ukJR9YOFYSBDqtTr+Lm6Gh
yB6vlmPR0Zg9vtGGbYMz0rsPoj7r85YpAKuMygXzXunjpLznWyVIPTkOP77gJmT2
q79KBEPZeOwVm/zrBLLC+98jCPfT52UShkNMwagPnhTz/VszAGNYXpBUqDygPRZD
4vc/x+8hZMvPEe2vEXt9rd3Pmcl9PaD9EgH9yN5u6kcnxd/M2vadoLjoq4lci2Ti
jtX+9j2o6QIDRFcHRtOF3y4/H3wM4F8eYEI2NQVdyRHFgLVR+nwp5lFRScpv00xM
M4pxaJoQds42xwSZ0DaBKewV28yQG/6GhFWMYVXbils24xdSylZwi5Knr57NSr4H
WeWvuuAYlWnwtcnsGCRRIQEJR/7B9L6q0Q31jECtT6WGbZwUX9Xqhw1gEbdKu/hT
+Zyo3Q6e6JrYyCes2eDYpVzF3MDyfv1wO05gNvxlsFSMhoAGwXbiS4EVqpw8QtxR
/WsHMTpyDnz29LJynE/NYf0PQ17PJdzLsFNP2zYNgE5X9KRmFVd6CxgNhdpB59lw
gpeBvPZlJ6LfYTU6ljGptA7T5+dQ3og/RfEU1I5E+Y8kocCbeweiVNyTQMLklh7o
lwIDAQABo0UwQzAdBgNVHQ4EFgQUh0lZfVAJhhJWMcDB1uKvDk1s750wEgYDVR0T
AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAQYwDQYJKoZIhvcNAQENBQADggIB
AA/b/E+9ZncsHiyuvo+6YGtjwgCSjtMN2ypiNdRW5PdZy6vdeZWLpLra+1/Dl8tb
+mEqjh5c25oMW8VO4hMJGwmD4TA9h7UzHZQBSKj5m7KsOQJYOmJChBl6WMZBhTaA
NXainBZzFZU0v3LKJR0yUmn1EylL/UiEMnmLmjqgB69Q3xe2sidmH5qji6mmZ6Kr
Lpfd6TggO4M435PJ1unjw/XuZQ1tji3O9sf5xTfGRRgZKz48hEXZqjfQZmjn2FEL
OEgCSinBwO+MtO3BRQ22Q/+YkUjbr/aS0iAb9+KynJhE8vHGCxWYpOO5z96m2s4G
ntoiY9iu0rjHrg5AEY/JZ1DIcqoEVbs04MCeBDYnY0GXm5QA/CuWjL76Tt4Ralzf
kro7L+5yv09pD0cHtiNCUM6Wbi5yMU1/allKa5GOHByhbD+9whUNUXuBWVKBprxb
nEd8rmMEo63dNAQY4M5jYCtl6d8/SgnBqO4xzj7RDeL6f4bmgxO52Y9arGc/ehXl
6Yft7f/1grnWSEOLJnNc7DuClJ+cb/FJkJZYaSAWvtWveWa7yqQsV39wkBy0VUyF
PYS4QVnN6N5cD90FElxDZM0wuYP/kpw5fiHvBY8/ifcVJSw+TrjBw6m/1O/5Hwe2
3t1uC5xezMXJ+jW2O06+bQzBwuGwjr0rJx+ez9H/dOXz
-----END CERTIFICATE-----


## 인증서 붙여넣기
$ cat > ca.crt
## -- 인증서 붙여넣기 --


```





### 3) hosts 파일등록

container 내부 에서 cluster kafka 에 접근하기 위해서는 host 별 IP(L4) 매핑이 필요하다.  

```sh
## /etc/hosts 파일에 아래 등록
$ vi /etc/hosts
...
10.217.166.67  sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-0-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-1-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
10.217.166.67  sa-cluster-kafka-route-2-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr


### kafka ###
211.254.212.105  bootstrap.myingress.com
211.254.212.105  broker-0.myingress.com
211.254.212.105  broker-1.myingress.com
211.254.212.105  broker-2.myingress.com


```





### 4) pub/sub 확인



```sh




export BROKERS=bootstrap.myingress.com:443
export KAFKAUSER=order-user
export PASSWORD=Kfix1IkttbDa
export TOPIC=order-intl-board-create
export GROUP=order-intl-board-create-group

  
## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SASL_SSL \
   -X sasl.mechanisms=SCRAM-SHA-512 \
   -X sasl.username=$KAFKAUSER \
   -X sasl.password=$PASSWORD \
   -X ssl.ca.location=./ca.crt -L

<-- 성공
% ERROR: Failed to acquire metadata: Local: Broker transport failure   <-- 주소인식이 안된경우



## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공


## 파일의 내용을 보내기
cat > msg.txt
abcdefg
---

## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1 msg.txt
<-- 성공



## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C
<-- 성공
 
 
## consumer 처음부터 읽어 확인할 경우
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C -o beginning
<-- 성공



## Consumer Group-id
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -G $GROUP  -C
<-- 성공

```





```sh


export BROKERS=bootstrap.211.254.212.105.nip.io:443
export TOPIC=my-topic
export GROUP=my-topic-group

  
## topic list
kafkacat -b $BROKERS \
  -X security.protocol=SSL \
  -X ssl.ca.location=./ca.crt \
  -L


## producer
kafkacat -b $BROKERS \
  -X security.protocol=SSL \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공




## Consumer
kafkacat -b $BROKERS \
  -t $TOPIC -C -o beginning

## Consumer Group-id
kafkacat -b $BROKERS \
  -t $TOPIC -G $GROUP  -C
<-- 성공


---
## topic list
kafkacat -b $BROKERS \
   -X security.protocol=SSL \
   -X ssl.ca.location=./ca.crt \
   -L

<-- 성공
% ERROR: Failed to acquire metadata: Local: Broker transport failure   <-- 주소인식이 안된경우



## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1
<-- 성공


## 파일의 내용을 보내기
cat > msg.txt
abcdefg
---

## producer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -P -X acks=1 msg.txt
<-- 성공



## Consumer
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C
<-- 성공
 
 
## consumer 처음부터 읽어 확인할 경우
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -C -o beginning
<-- 성공



## Consumer Group-id
kafkacat -b $BROKERS \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanisms=SCRAM-SHA-512 \
  -X sasl.username=$KAFKAUSER \
  -X sasl.password=$PASSWORD \
  -X ssl.ca.location=./ca.crt \
  -t $TOPIC -G $GROUP  -C
<-- 성공

```















# 8. python Native Connect

## 8.1 준비

### 1) python 준비 with docker



```sh
## docker 실행
docker run --name python --user root --rm -d nexus.dspace.kt.co.kr/python:3.9 sleep 365d

## cluster 에서 실행
oc create deploy python --image=nexus.dspace.kt.co.kr/python:3.9 -- sleep 365d

```





### 2) python library install

사내망에서 python nexus repo 참조하도록 설정

```


$ mkdir -p ~/.config/pip
$ cat > ~/.config/pip/pip.conf
[global]
trusted-host=10.217.59.89
index=http://10.217.59.89/nexus3/repository/pypi-group/pypi
index-url=http://10.217.59.89/nexus3/repository/pypi-group/simple


```



python 을 이용해서kafka 에 접근하기 위해서는 kafka 가아닌 kafka-python 을 설치해야 한다.

```bash
pip install kafka-python
```


### 3) kubernets port-forward

k8s 내에 존재하는 kafka(strimzi) 로 로컬에서 쉽게 접근하기 위해서 port-forward 를 이용해보자.

```bash
oka port-forward svc/sa-cluster-kafka-bootstrap 9092:9092

<사내에서 시도시>
KafkaAdminClient() 시도시 아래와 같은 에러 발생...
kafka.errors.NodeNotReadyError: NodeNotReadyError


<사외에서 시도시>
KafkaAdminClient() 시도시 아래와 같은 에러 발생...
target port 가 9092 가 아닌 50000 번대 port 로 변경

```



## 8.2 producer

### 1) no athentication

```python
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='sa-cluster-kafka-bootstrap.kafka-system-system.svc:9092')
producer.send('order-intl-board-create', b'python test2')
```

### 

```python
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='bootstrap.myingress.com:443',
                         security_protocol="SSL",
                         ssl_cafile='./ca.crt')
producer.send('my-topic', b'python test3')

-----------------
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='my-cluster-kafka-bootstrap:9092')
producer.send('my-topic', b'python test2')
------------------



consumer = KafkaConsumer(bootstrap_servers='sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443',
                        security_protocol="SASL_SSL",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='sa-edu-user',
                        sasl_plain_password='V1QEY60gW4lE',
                        ssl_check_hostname=True,
                        ssl_cafile='./ca.crt',
                        auto_offset_reset='earliest',
                        enable_auto_commit= True,
                        group_id='sa-edu-group-01')





```









### 2) SCRAM-SHA-512

```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='sa-cluster-kafka-bootstrap.kafka-system-system.svc:9092',
                        security_protocol="SASL_PLAINTEXT",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='order-user',
                        sasl_plain_password='Kfix1IkttbDa')
    
producer.send('order-intl-board-create', b'python test2')
producer.send('order-intl-board-create', b'python test3')
```



### 3) 대량 발송(성능테스트)

```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='sa-cluster-kafka-bootstrap.kafka-system.svc:9092',
                        security_protocol="SASL_PLAINTEXT",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='order-user',
                        sasl_plain_password='Kfix1IkttbDa')
    
producer.send('order-intl-board-create', b'python test2')
producer.send('order-intl-board-create', b'python test3')
producer.send('order-intl-board-create', b'{"eventName":"a","num":1,"title":"a", "writeId":"", "writeName": "", "writeDate":"" }')


# 20만건 테스트
for i in range(300000):
    print(i)
    producer.send('order-intl-board-create', b'{"eventName":"a","num":%d,"title":"a", "writeId":"", "writeName": "", "writeDate":"" }' % i)



# 20만건 테스트
for i in range(300000, 600000):
    print(i)
    producer.send('order-intl-board-create', b'{"eventName":"a","num":%d,"title":"a", "writeId":"", "writeName": "", "writeDate":"" }' % i)



```

- 4000 TPS 까지 테스트 완료함
- 7000 TPS 까지 테스트 완료함







## 8.3 consumer

### 1) no athentication

- topic 으로 읽을때

```python
>>> from kafka import KafkaConsumer
>>> consumer = KafkaConsumer('my_favorite_topic')
>>> for msg in consumer:
...     print (msg)
```





```python
from kafka import KafkaConsumer
consumer = KafkaConsumer(bootstrap_servers='my-cluster-kafka-bootstrap:9092,my-cluster-kafka-0:9094,my-cluster-kafka-1:9094,my-cluster-kafka-2:9094',
                        auto_offset_reset='earliest',
                        enable_auto_commit= True,
                        group_id='my-topic-group')
# topic 확인
consumer.topics()
# {'order-my-topic2', 'order-my-topic', 'order-my-topic3'}

# 사용할 topic 지정(구독)
consumer.subscribe("my-topic")
consumer.subscription()    ## {'order-my-topic3'}

# 메세지 읽기
for message in consumer:
   print("topic=%s partition=%d offset=%d: key=%s value=%s" %
        (message.topic,
          message.partition,
          message.offset,
          message.key,
          message.value))
```









- 
- 
- consumer group으로 읽을때

```python
from kafka import KafkaConsumer
consumer = KafkaConsumer(bootstrap_servers='sa-cluster-kafka-bootstrap.kafka-system.svc:9092',
                        auto_offset_reset='earliest',
                        enable_auto_commit= True,
                        group_id='order-group-cons1')
# topic 확인
consumer.topics()
# {'order-my-topic2', 'order-my-topic', 'order-my-topic3'}

# 사용할 topic 지정(구독)
consumer.subscribe("order-my-topic")
consumer.subscription()    ## {'order-my-topic3'}

# 메세지 읽기
for message in consumer:
   print("topic=%s partition=%d offset=%d: key=%s value=%s" %
        (message.topic,
          message.partition,
          message.offset,
          message.key,
          message.value))
          
```


### 2) SCRAM-SHA-512

```python
from kafka import KafkaConsumer
consumer = KafkaConsumer(bootstrap_servers='sa-cluster-kafka-bootstrap.kafka-system.svc:9092',
                        security_protocol="SASL_PLAINTEXT",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='order-user',
                        sasl_plain_password='Kfix1IkttbDa',
                        auto_offset_reset='earliest',
                        enable_auto_commit= True,
                        group_id='order-intl-board-group')

# topic 확인
consumer.topics()
# {'order-my-topic2', 'order-my-topic', 'order-my-topic3'}

# 사용할 topic 지정(구독)
consumer.subscribe("order-intl-board-create")
consumer.subscription()    ## {'order-intl-board-create'}

# 메세지 읽기
for message in consumer:
   print("topic=%s partition=%d offset=%d: key=%s value=%s" %
        (message.topic,
          message.partition,
          message.offset,
          message.key,
          message.value))

'''
---
topic=order-my-topic3 partition=0 offset=16: key=None value=b'python test2'
topic=order-my-topic3 partition=2 offset=7: key=None value=b'python test3'
topic=order-my-topic3 partition=0 offset=17: key=None value=b'python test4'

'''
```



### 3) External Access(Cluster 외부에서 접근시, 개발자 PC

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(bootstrap_servers='sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443',
                        security_protocol="SASL_SSL",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='sa-edu-user',
                        sasl_plain_password='V1QEY60gW4lE',
                        ssl_check_hostname=True,
                        ssl_cafile='./ca.crt',
                        auto_offset_reset='earliest',
                        enable_auto_commit= True,
                        group_id='sa-edu-group-01')

# topic 확인
consumer.topics()
# {'sa-edu-topic-49', 'sa-edu-topic-26', 'sa-edu-topic-45', 'sa-edu-topic-21', 'sa-edu-topic-34', 'sa-edu-topic-29', 'sa-edu-topic-47', 'sa-edu-topic-17', 'sa-edu-topic-44', 'sa-edu-topic-11', 'sa-edu-topic-13', 'sa-edu-topic-32', 'sa-edu-topic-09', 'sa-edu-topic-02', 'sa-edu-topic-28', 'sa-edu-topic-20', 'sa-edu-topic-16', 'sa-edu-topic-33', 'sa-edu-topic-14', 'sa-edu-topic-03', 'sa-edu-topic-41', 'sa-edu-topic-50', 'sa-edu-topic-06', 'sa-edu-topic-35', 'sa-edu-topic-22', 'sa-edu-topic-19', 'sa-edu-topic-37', 'sa-edu-topic-43', 'sa-edu-topic-01', 'sa-edu-topic-07', 'sa-edu-topic-27', 'sa-edu-topic-46', 'sa-edu-topic-10', 'sa-edu-topic-36', 'sa-edu-topic-31', 'sa-edu-topic-38', 'sa-edu-topic-08', 'sa-edu-topic-39', 'sa-edu-topic-25', 'sa-edu-topic-18', 'sa-edu-topic-04', 'sa-edu-topic-12', 'sa-edu-topic-42', 'sa-edu-topic-23', 'sa-edu-topic-15', 'sa-edu-topic-48', 'sa-edu-topic-30', 'sa-edu-topic-24', 'sa-edu-topic-05', 'sa-edu-topic-40'}


# 사용할 topic 지정(구독)
consumer.subscribe("sa-edu-topic-01")
consumer.subscription()    
## {'sa-edu-topic-01'}

# 메세지 읽기
for message in consumer:
   print("topic=%s partition=%d offset=%d: key=%s value=%s" %
        (message.topic,
          message.partition,
          message.offset,
          message.key,
          message.value))

'''
---
topic=order-my-topic3 partition=0 offset=16: key=None value=b'python test2'
topic=order-my-topic3 partition=2 offset=7: key=None value=b'python test3'
topic=order-my-topic3 partition=0 offset=17: key=None value=b'python test4'

'''
```





## 8.4 Consumer Group 

### 1) List 

namesapce 를 보내서 CG list 리턴

- test1

```python
from kafka.admin import KafkaAdminClient

admin_client = KafkaAdminClient(bootstrap_servers="sa-cluster-kafka-bootstrap.kafka-system-system.svc:9092", 
                        security_protocol="SASL_PLAINTEXT",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='order-user',
                        sasl_plain_password='Kfix1IkttbDa',
                        #client_id='test1'
                        )

list_cg = admin_client.list_consumer_groups()
print(type(list_cg))
print(list_cg )

'''
---
python getConsumerGroupList.py
---
<class 'list'>
[
 ('console-consumer-78969', 'consumer'), 
 ('console-consumer-12890', 'consumer'), 
 ('console-consumer-52141', 'consumer'), 
 ('order-consumer-group', 'consumer'),
 ('order-consumer-group2','consumer')
]
---
[
('order-group-cons2', 'consumer'), 
('console-consumer-76018', 'consumer')
]

  <-- 오케이 잘된다.
'''
```

- 주의사항
  - list_consumer_groups 함수는 Kafka에 offset을 저장한 Consumer Group 만 리턴한다. 그룹만 추가되었다고 해서 조회되지는 않는다.  그러므로 list로 조회되려면 사실상 아래와 같이 3단계가 지나야 한다.
    - 그룹생성
    - 구독
    - 첫1회 consuming
  - 그러므로 위 3단계를 그룹추가 로직에 포함시켜야 한다.
  - 리턴되는 튜블중 두번째 인자는 consumer group protocol type 이다.





### 2) Describe

CG 명을 던져서 topicname, partition, current-offset 이 리턴되어야 한다.

참조: https://kafka-python.readthedocs.io/en/master/apidoc/KafkaAdminClient.html

참조: https://github.com/dpkp/kafka-python/issues/1798



- test1

```python

from kafka.admin import KafkaAdminClient

# external
admin_client = KafkaAdminClient(bootstrap_servers='sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443',
                         security_protocol="SASL_SSL",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='sa-edu-user',
                        sasl_plain_password='V1QEY60gW4lE',
                        ssl_check_hostname=True,
                        ssl_cafile='./ca.crt'
                        )

# internal
admin_client = KafkaAdminClient(bootstrap_servers="sa-cluster-kafka-bootstrap.kafka-system.svc:9092", 
                        security_protocol="SASL_PLAINTEXT",
                        sasl_mechanism='SCRAM-SHA-512',
                        sasl_plain_username='sa-edu-user',
                        sasl_plain_password='V1QEY60gW4lE',
                        #client_id='test1'
                        )

# 그룹명을 인수로 보낼때는 반드시 리스트[] 로 보내야 한다.
cg_desc = admin_client.describe_consumer_groups(['sa-edu-group-01'])
print(type(cg_desc))
print(cg_desc)
print('')

# offset 정보
cg_offsets = admin_client.list_consumer_group_offsets('sa-edu-group-01')
print(type(cg_offsets))
print(cg_offsets)


'''

<class 'list'>
[
GroupInformation(error_code=0, group='sa-edu-group-01', state='Stable', 
protocol_type='consumer', protocol='range', 
members=[MemberInformation(member_id='kafka-python-2.0.2-92366493-3d61-4b74-bac4-6245b7ffae37', 
client_id='kafka-python-2.0.2', client_host='/10.130.2.2', 
member_metadata=ConsumerProtocolMemberMetadata(version=0, subscription=['sa-edu-topic-01'], user_data=b''), 
member_assignment=ConsumerProtocolMemberAssignment(version=1, assignment=[(topic='sa-edu-topic-01', partitions=[2])], 
user_data=None)), 
MemberInformation(member_id='consumer-sa-edu-group-01-2-4eec207f-6467-45df-9806-aa9955a1ff52', 
client_id='consumer-sa-edu-group-01-2', client_host='/10.129.2.2', 
member_metadata=ConsumerProtocolMemberMetadata(version=1, subscription=['sa-edu-topic-01'], user_data=None), 
member_assignment=ConsumerProtocolMemberAssignment(version=1, assignment=[(topic='sa-edu-topic-01', partitions=[0, 1])], 
user_data=None))], 
authorized_operations=None)
]

<class 'dict'>
{
TopicPartition(topic='sa-edu-topic-01', partition=0): OffsetAndMetadata(offset=736, metadata=''), 
TopicPartition(topic='sa-edu-topic-01', partition=2): OffsetAndMetadata(offset=619, metadata=''), 
TopicPartition(topic='sa-edu-topic-01', partition=1): OffsetAndMetadata(offset=639, metadata='')
}


'''
```






### 3) Delete

- CG 삭제
  kafka native 나 bridge 를 통해서 삭제기능을 제공하지 않는다.
  그러므로 shell 을 통해서만 가능하다.






## 8.5 kafka admin Client



- topic 생성

```python
from kafka.admin import KafkaAdminClient, NewTopic
admin_client = KafkaAdminClient(bootstrap_servers="sa-cluster-kafka-bootstrap.kafka-system.svc:9092", client_id='test')

topic_list = []
topic_list.append(NewTopic(name="example_topic", num_partitions=1, replication_factor=1))
admin_client.create_topics(new_topics=topic_list, validate_only=False)

'''
---
python kafkaAdminClient.py
---
'''
```





# 9. Java Native Connect



### 1) application.yml

```

  cloud:
    stream:
      kafka:
        binder:
          # applicationId: my-user
          brokers:
          - sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443,sa-cluster-kafka-route-1-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443,sa-cluster-kafka-route-2-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
            sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
          configuration:
            security:
              protocol: SASL_SSL
            sasl:
              mechanism: SCRAM-SHA-512
              jaas:
                config: org.apache.kafka.common.security.scram.ScramLoginModule required username="my-user" password="StzbQLCa1XHH";
           
            ssl:
              truststore.location: classpath:/truststore.jks
              truststore.type: JKS
              truststore.password: new1234
              
          - sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443,sa-cluster-
          
          
          
```



```


          brokers:
          - 
          sa-cluster-kafka-route-0-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443,
          sa-cluster-kafka-route-1-kafka.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443,
          sa-cluster-kafka-route-2-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr:443
          sa-cluster-kafka-route-bootstrap-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
          configuration:
          
          
          
```







## 9.1 producer

```java
	public static Properties setProperty() {
		String bootstrapServer = "sa-cluster-kafka-bootstrap.kafka-system.svc:9092";
		String username = "order-user";
		String password = "XVDMDpGgaTTu";

		Properties props = new Properties();
        props.put("bootstrap.servers", bootstrapServer);
        props.put("security.protocol", "SASL_PLAINTEXT");
        props.put("sasl.mechanism", "SCRAM-SHA-512");
        
        String jaasTemplate = "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";";
        String jaasCfg = String.format(jaasTemplate, username, password);
        props.put("sasl.jaas.config", jaasCfg);
		
        return props;
	}
	
	public static void Producer() {
		String topicName = "order-my-topic1";

        // property
		Properties props = setProperty();
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        // producer create
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
        TestCallback callback = new TestCallback();

        // Message send     
        for (int i = 0; i < 5; i++) {
            producer.send(new ProducerRecord<String, String>(topicName, "my-key", "hello Arsenal ES - " + i), callback);
        }

        // producer Close
        producer.flush();
        producer.close();
	}
	
	private static class TestCallback implements Callback {
       @Override
       public void onCompletion(RecordMetadata recordMetadata, Exception e) {
           if (e != null) {
               System.out.println("Error while producing message to topic :" + recordMetadata);
               e.printStackTrace();
           } else {
               String message = String.format("sent message to topic:%s partition:%s  offset:%s", recordMetadata.topic(), recordMetadata.partition(), recordMetadata.offset());
               System.out.println(message);
           }
       }
	}
```



## 9.2 consumer

```java
	public static Properties setProperty() {
		String bootstrapServer = "sa-cluster-kafka-bootstrap.kafka-system.svc:9092";
		String username = "order-user";
		String password = "XVDMDpGgaTTu";

		Properties props = new Properties();
        props.put("bootstrap.servers", bootstrapServer);
        props.put("security.protocol", "SASL_PLAINTEXT");
        props.put("sasl.mechanism", "SCRAM-SHA-512");
        
        String jaasTemplate = "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";";
        String jaasCfg = String.format(jaasTemplate, username, password);
        props.put("sasl.jaas.config", jaasCfg);
		
        return props;
	}
	
	public static void Consumer() {
		String topicName = "order-my-topic1";
		
        // property
		Properties props = setProperty();
        props.put("group.id", "order-consumer-group");
        props.put("enable.auto.commit", "true");
        props.put("auto.offset.reset", "latest");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        // Consumer create
		try {
			KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
			consumer.subscribe(Arrays.asList(topicName));
			while (true) {
				ConsumerRecords<String, String> records = consumer.poll(1000);
			    for (ConsumerRecord<String, String> record : records) {
			        System.out.printf("%s [%d] offset=%d, key=%s, value=\"%s\"\n",
									  record.topic(), record.partition(),
									  record.offset(), record.key(), record.value());
				}
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```



## 9.3 streams

```java
	public static void Pipe() {
		
        /*
         * 카프카 스트림 파이프 프로세스에 필요한 설정값
         * StreamsConfig의 자세한 설정값은
         * https://kafka.apache.org/10/documentation/#streamsconfigs 참고
         */

		String bootstrapServer = "sa-cluster-kafka-bootstrap.kafka-system.svc:9092";
		String username = "order-user";
		String password = "XVDMDpGgaTTu";

		Properties props = new Properties();
		// 카프카 스트림즈 애플리케이션을 유일할게 구분할 아이디
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-pipe");
		// 스트림즈 애플리케이션이 접근할 카프카 브로커정보
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServer);
		// 데이터를 어떠한 형식으로 Read/Write할지를 설정(키/값의 데이터 타입을 지정) - 문자열
		props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
		props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

		
		props.put(StreamsConfig.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");
        props.put("sasl.mechanism", "SCRAM-SHA-512");
        
        String jaasTemplate = "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";";
        String jaasCfg = String.format(jaasTemplate, username, password);
        props.put("sasl.jaas.config", jaasCfg);
        
        // consumer
        props.put("group.id", "order-consumer-group");
        props.put("enable.auto.commit", "true");
        props.put("auto.offset.reset", "latest");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        

		// 데이터의 흐름으로 구성된 토폴로지를 정의할 빌더
		final StreamsBuilder builder = new StreamsBuilder();

		// input에서 output으로 스트림 데이터 흐름을 정의 한다.
		// input-topic : order-my-topic1
		// output-topic : order-my-topic2
		/*
		 * KStream<String, String> source = builder.stream("order-my-topic1");
		 * source.to("order-my-topic2");
		 */
		builder.stream("order-my-topic1").to("order-my-topic2");

		// 최종적인 토폴로지 생성
		final Topology topology = builder.build();

		// 만들어진 토폴로지 확인
        System.out.println("Topology info: " + topology.describe());

		final KafkaStreams streams = new KafkaStreams(topology, props);

		try {
			streams.start();
			System.out.println("topology started");

		} catch (Throwable e) {
			System.exit(1);
		}
		System.exit(0);

	}
```







# 10. monitoring



## 10.1 prometheus 



### 1) 사전준비



```

이전
quay.io/prometheus/prometheus:v2.34.0

이후
nexus.dspace.kt.co.kr/prometheus/prometheus:v2.34.0


docker pull quay.io/prometheus/prometheus:v2.34.0

docker tag quay.io/prometheus/prometheus:v2.34.0 nexus.dspace.kt.co.kr/prometheus/prometheus:v2.34.0

docker push nexus.dspace.kt.co.kr/prometheus/prometheus:v2.34.0




```







### 2) 권한부여

- anyuid

```
# 권한부여시
oc adm policy add-scc-to-user    anyuid -z prometheus-server -n kafka-system

# 권한삭제시
oc adm policy remove-scc-from-user anyuid  -z prometheus-server -n kafka-system

```



- image pull 권한

```
//  secret  생성
oc create secret docker-registry dspace-nexus --docker-server=nexus.dspace.kt.co.kr \
    --docker-username=icistr-cmmn-readonly \
    --docker-password=icis1234 \
    --docker-email=icistr-sa@kt.co.kr \
    -n kafka-system
  
//  secret 조회
oc get secrets dspace-nexus -n kafka-system

//  secret 삭제
oc delete secrets dspace-nexus -n kafka-system
  
// istio-system default  serviceaccount 에 추가
oc patch serviceaccount prometheus-server -p '{"imagePullSecrets": [{"name": "dspace-nexus"}]}' -n kafka-system

```



### 3) helm deploy

```sh
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

$ helm3 fetch prometheus-community/prometheus


$ helm3 -n kafka-system list

$ helm3 -n kafka-system install prometheus . \
  --set alertmanager.enabled=false \
  --set configmapReload.prometheus.enabled=false \
  --set configmapReload.alertmanager.enabled=false \
  --set kubeStateMetrics.enabled=false \
  --set nodeExporter.enabled=false \
  --set server.enabled=true \
  --set server.image.repository=nexus.dspace.kt.co.kr/prometheus/prometheus \
  --set server.namespaces[0]=kafka-system \
  --set server.ingress.enabled=false \
  --set server.persistentVolume.enabled=false \
  --set pushgateway.enabled=false \
  --dry-run=true > dry-run.yaml


## 삭제
$ helm3 -n kafka-system delete prometheus 


```





### 4) exporter service 생성

exporter service 는 자동으로 생성되지 않는다.

아래와 같이 수동으로 추가해야 한다.

```
kind: Service
apiVersion: v1
metadata:
  name: prometheus-server
  namespace: kafka-system
  labels:
    app: prometheus
    app.kubernetes.io/managed-by: Helm
    chart: prometheus-15.8.4
    component: server
    heritage: Helm
    release: prometheus
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9090
  selector:
    app: prometheus
    component: server
    release: prometheus
  type: ClusterIP
```





### 5) route

````
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: prometheus-route
  namespace: kafka-system
  labels:
    app: prometheus
    app.kubernetes.io/managed-by: Helm
    chart: prometheus-15.8.4
    component: server
    heritage: Helm
    release: prometheus
spec:
  host: prometheus-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
  to:
    kind: Service
    name: prometheus-server
    weight: 100
  port:
    targetPort: http
````





### 6) configmap

exporter 주소를 추가해야 한다.

````
kind: ConfigMap
apiVersion: v1
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus
    meta.helm.sh/release-namespace: kafka-system
  name: prometheus-server
  namespace: kafka-system
  labels:
    app: prometheus
    app.kubernetes.io/managed-by: Helm
    chart: prometheus-15.8.4
    component: server
    heritage: Helm
    release: prometheus
data:
  alerting_rules.yml: |
    {}
  alerts: |
    {}
  prometheus.yml: |
    global:
      evaluation_interval: 1m
      scrape_interval: 1m
      scrape_timeout: 10s
    rule_files:
    - /etc/config/recording_rules.yml
    - /etc/config/alerting_rules.yml
    - /etc/config/rules
    - /etc/config/alerts
    scrape_configs: 
    - job_name: kafka-exporter
      metrics_path: /metrics
      scrape_interval: 1m
      scrape_timeout: 10s
      static_configs:
      - targets:
        - sa-cluster-kafka-exporter.kafka-system.svc
        ...
````







## 10.1 grafana.yaml

```yaml

 # This is not a recommended configuration, and further support should be available
# from the Prometheus and Grafana communities.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  labels:
    app: strimzi
spec:
  replicas: 1
  selector:
    matchLabels:
      name: grafana
  template:
    metadata:
      labels:
        name: grafana
    spec:
      containers:
      - name: grafana
        image: nexus.dspace.kt.co.kr/grafana/grafana:7.3.7
        ports:
        - name: grafana
          containerPort: 3000
          protocol: TCP
        volumeMounts:
        - name: grafana-data
          mountPath: /var/lib/grafana
        - name: grafana-logs
          mountPath: /var/log/grafana
        readinessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 15
          periodSeconds: 20
      volumes:
      - name: grafana-data
        emptyDir: {}
      - name: grafana-logs
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  labels:
    app: strimzi
spec:
  ports:
  - name: grafana
    port: 3000
    targetPort: 3000
    protocol: TCP
  selector:
    name: grafana
  type: ClusterIP

```





- route

```yaml
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: grafana-kafka-system-route
  namespace: kafka-system
  labels:
    app: strimzi
spec:
  host: grafana-kafka-system.apps.ktis-console.c01-okd4.cz-tb.paas.kt.co.kr
  to:
    kind: Service
    name: grafana
    weight: 100
  port:
    targetPort: grafana
  wildcardPolicy: None
```



- 로그인

기본 Grafana 사용자 이름과 암호는 모두 admin 이다.







# 15. docker-compose



## 1) docker-compose 설치



```sh

$ sudo curl -L https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose version
Docker Compose version v2.5.1

$ docker-compose -v
Docker Compose version v2.5.1
```





## 2) docker-compose.yaml

```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
  
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

```





```yaml
version: '2'
services:
  zookeeper:
    container_name: local-zookeeper
    image: wurstmeister/zookeeper:3.4.6
    ports:
      - "2181:2181"

  kafka:
    container_name: local-kafka
    image: wurstmeister/kafka:2.12-2.3.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_CREATE_TOPICS: "my-topic:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    
```





## 3) 실행



```sh
# 시작
$ sudo docker-compose -f docker-compose.yaml up -d

# 확인
$ sudo docker-compose ls

$ sudo docker-compose ps

# 종료
$ docker-compose down

# stop
$ docker-compose stop


# 로그 보기

docker container logs local-zookeeper


docker container logs local-kafka

```





## 4) python test



### (1) producer.py

```python
from kafka import KafkaProducer
from json import dumps
from time import sleep
import sys

def on_send_success(record_metadata):
    print("topic recorded:",record_metadata.topic)
    print("partition recorded:",record_metadata.partition)
    print("offset recorded:",record_metadata.offset)

def on_send_error(excp):
        print(excp)

# 카프카 서버
#bootstrap_servers = ["localhost:9095"]
bootstrap_servers = ["172.22.253.23:29092"]


# 카프카 producer 생성
producer = KafkaProducer(bootstrap_servers=bootstrap_servers,
                         key_serializer=None,
                         acks=0,
                         #linger_ms=500,
                         value_serializer=lambda x: dumps(x).encode('utf-8')
                        )

# 카프카 토픽
str_topic_name = 'my-topic'

#produce
for i in range(50):
    response = producer.send(str_topic_name,
                             "test"
                            ).add_callback(on_send_success).add_errback(on_send_error)
    print("----" + i)
    time.sleep(0.5)
    
```





```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='172.22.253.23:9092',
                         key_serializer=None,
                         acks=0)

producer.send('my-topic', b'python test2')

producer.send('my-topic', b'python test2').add_callback(on_send_success).add_errback(on_send_error)

    
    
    
```





### (2) consumer.py

```python
from kafka import KafkaConsumer
from json import loads
from time import sleep
import time


# 카프카 서버
#bootstrap_servers = ["localhost:9095"]
bootstrap_servers = ["172.22.253.23:9092"]

# 카프카 토픽
str_topic_name    = 'my-topic'

# 카프카 consumer group 생성
str_group_name = 'my-topic-group'

#-------------comsumption data-----------
consumer = KafkaConsumer(str_topic_name,                          # kafka topic name
                         bootstrap_servers= bootstrap_servers,    # kafka server
                         auto_offset_reset='earliest',            # 가장 처음 offset부터
                         enable_auto_commit=True,      # 마지막으로 읽은 offset 위치 commit
                         auto_commit_interval_ms=500,  # offset commit 주기, default : 5000
                         group_id=str_group_name,      # consumer가 생성될 consumer group
                         value_deserializer=lambda x: loads(x.decode('utf-8')) #serialize된 메시지를 deserialize
                        )
for event in consumer:
     event_data = event.value
     print(event_data)
     sleep(1)
```





```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(bootstrap_servers='172.27.224.1:9092',
    auto_offset_reset='earliest',
    enable_auto_commit= True,
    group_id='my-topic-group')



# topic 확인
consumer.topics()
# {'order-my-topic2', 'order-my-topic', 'order-my-topic3'}

# 사용할 topic 지정(구독)
consumer.subscribe("my-topic")
consumer.subscription()    ## {'order-my-topic3'}

# 메세지 읽기
for message in consumer:
   print("topic=%s partition=%d offset=%d: key=%s value=%s" %
        (message.topic,
          message.partition,
          message.offset,
          message.key,
          message.value))
```





## 5) kafka-client

```sh

# topic list
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# topic create
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic my-topic2

# producer
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic

# consumer
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning

```

