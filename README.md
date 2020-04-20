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
export KEDA_VERSION=1.4.0
wget https://github.com/kedacore/keda/releases/download/v${KEDA_VERSION}/keda-${KEDA_VERSION}.tar.gz
tar xzf keda-${KEDA_VERSION}.tar.gz
rm keda-${KEDA_VERSION}.tar.gz

kubectl apply -f keda-${KEDA_VERSION}/crds
kubectl apply -f keda-${KEDA_VERSION}/

rm -rf keda-${KEDA_VERSION}
```

The KEDA operator and the metrics apiserver is up and running in the `keda` namespace:

```shell
kubectl get pods -n keda
NAME                                    READY   STATUS    RESTARTS   AGE
keda-metrics-apiserver-96766f49-qg27t   1/1     Running   0          5m
keda-operator-884dfdccb-srhd4           1/1     Running   0          5m
```

## Deploy Apache Kafka application

Create the Kafka topic `my-topic` in the `kafka` namespace:

```shell
kubectl apply -f kafka-topic.yaml -n kafka
```

Deploy the Kafka consumer application which is currently scaled to 0:

```shell
kubectl apply -f kafka-consumer.yaml -n kafka
```

Deploy the KEDA scaled object:

```shell
kubectl apply -f kafka-scaledobject.yaml -n kafka
```

Deploy the Kafka producer application:

```shell
kubectl apply -f kafka-producer.yaml -n kafka
```

## Auto scaling up and down

When the consumer sent more messages than the `lagThreshold` specified in the scaled object, KEDA does a scale from 0 to 1 so one Kafka consumer pod is started consuming messages.

Increasce the frequency with which the Kafka producer sends messages from 1 sec to 10 msec and increase the number of producer replicas to 5.
The KEDA operator will scale the number of pods for the Kafka consumer up to three, which is anyway the maximim because the number of partitions of the `my-topic` topic.

Slowing down the frequency of the Kafka producer and back to 1 sec delay, after a `coolingDown` period (which is 5 minutes by default), the KEDA operator will scale the Kafka consumer to just one pod as the beginning.
Stopping the Kafka producer, the KEDA operator will scale the Kafka consumer to 0.


