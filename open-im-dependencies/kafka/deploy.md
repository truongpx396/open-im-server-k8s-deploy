# kafka(3 broker)helm charts部署文档
## 部署步骤
### 1. 创建namespace
```
kubectl create ns kafka
```
### 2. 创建kafka storageClass
```
  kubectl apply -f ./sc.yaml;
```

### 3. 修改values.yaml
#### 1. values.yaml常用参数说明
|参数名   | 参数值|  参数说明    |
|  ----  | ----  | --- |
| zookeeperChrootPath | "zookeeper-cluster-0.zookeeper-cluster-headless.zookeeper", "zookeeper-cluster-1.zookeeper-cluster-headless.zookeeper", "zookeeper-cluster-2.zookeeper-cluster-headless.zookeeper" | k8s集群内部zk地址 |
| provisioning.topics|  name: ws2ms_chat partitions: 12 replicationFactor: 1 flush.messages: 1 | kafka topic分区数， 副本数， 刷盘策略 |
|global.storageClass| kafka-data-sc |存储类名，需要和sc.yaml中storageClass保持一致|
| affinity | 示例如下| 亲和性 |

#### 2. 亲和性配置
pod反亲和性，保证kafka broker的pod不会被调度到同一个node上运行
```
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - topologyKey: kubernetes.io/hostname
      labelSelector:
        matchExpressions: 
          - key: app.kubernetes.io/name
            operator: In 
            values: 
            - kafka
```
pod反亲和性，尽量保证kafka broke的pod不会被调度到同一个node上运行，如果无法满足这个规则，也会将kafka broke调度到同一个node
```
podAntiAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    podAffinityTerm:
      labelSelector:
        matchExpressions:
        - key: app.kubernetes.io/name
          operator: In
          values:
          - kafka
      topologyKey: kubernetes.io/hostname
```


### 3. 安装kafka集群
安装kafka集群
```
helm install kafka-cluster -f values.yaml bitnami/kafka -n kafka
```

卸载kafka集群
```
helm delete kafka-cluster -n kafka
```

通过values.yaml更新kafka
```
helm upgrade kafka-cluster bitnami/kafka -f values.yaml -n kafka
```