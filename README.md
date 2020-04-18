# Auto-Scaling data streaming applications with Strimzi and KEDA

## Strimzi operator and Apache Kafka cluster

Create a new `kafka` namespace:

```
kubectl create namespace kafka
```

### Install Strimzi operator

Install the Strimzi operator:

```shell
export STRIMZI_VERSION=0.17.0
curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/${STRIMZI_VERSION}/strimzi-cluster-operator-${STRIMZI_VERSION}.yaml \
  | sed 's/namespace: .*/namespace: kafka/' \
  | kubectl apply -f - -n kafka
```

The Strimzi operator is up and running in the `kafka` namespace:

```shell
kubectl get pods -n kafka
NAME                                          READY     STATUS    RESTARTS   AGE
strimzi-cluster-operator-d5b6c6458-vx85b      1/1       Running   0          1m
```

### Provision the Apache Kafka Cluster

Provision an Apache Kafka cluster:

```shell
export STRIMZI_VERSION=0.17.0
kubectl apply -f https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/${STRIMZI_VERSION}/examples/kafka/kafka-persistent.yaml -n kafka
```

An Apache Kafka cluster with three ZooKeeper nodes and three Kafka brokers is up and running in the `kafka` namespace:

```shell
kubectl get pods -n kafka
NAME                                          READY     STATUS    RESTARTS   AGE
my-cluster-entity-operator-7c5697d596-z6tkg   3/3     Running   0          8s
my-cluster-kafka-0                            2/2     Running   0          37s
my-cluster-kafka-1                            2/2     Running   0          37s
my-cluster-kafka-2                            2/2     Running   0          37s
my-cluster-zookeeper-0                        2/2     Running   0          1m
my-cluster-zookeeper-1                        2/2     Running   0          1m
my-cluster-zookeeper-2                        2/2     Running   0          1m
strimzi-cluster-operator-d5b6c6458-vx85b      1/1     Running   0          2m
```

## KEDA operator

Install the KEDA operator:

```shell
git clone https://github.com/kedacore/keda && cd keda

kubectl apply -f deploy/crds/keda.k8s.io_scaledobjects_crd.yaml
kubectl apply -f deploy/crds/keda.k8s.io_triggerauthentications_crd.yaml
kubectl apply -f deploy/
```

The KEDA operator and the metrics apiserver is up and running in the `keda` namespace:

```shell
kubectl get pods -n keda
NAME                                    READY   STATUS    RESTARTS   AGE
keda-metrics-apiserver-96766f49-qg27t   1/1     Running   0          5m
keda-operator-884dfdccb-srhd4           1/1     Running   0          5m
```
