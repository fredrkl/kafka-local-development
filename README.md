# Local Kafka development with Strimzi and Kind

This repo is intended to be a sandbox for local development and testing. It has
local Kafka using the Strimzi operator, and Kubernetes(K8s). Additionally, is
has a full [LGTM OTel
setup](https://grafana.com/docs/opentelemetry/docker-lgtm/#run-lgtm-in-kubernetes)
for local throubleshooting. The goal is to create a compelling inner-loop
development experience.

## Local Kafka development

Strimzi is an open-source project that provides a way to
run Apache Kafka on Kubernetes and OpenShift. It simplifies the deployment and
management of Kafka clusters on Kubernetes by providing custom resources and
operators that handle the complexity of running Kafka in a cloud-native
environment.

## Kind cluster setup

You can use the provided `kind-config.yaml` file to create a local Kubernetes
cluster using Kind.

To create the cluster, run the following command:

```bash
kind create cluster --config kind-config.yaml
kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```

Watch the Strimzi operator logs to ensure it is running correctly. Then we want
to instantiate a Kafka cluster using the provided `kafka-cluster.yaml` file:

```bash
kubectl apply -f kafka-cluster.yaml -n kafka
```

To delete:

```bash
# Delete:  kind delete cluster --name strimzi
```

## Local kafka GUI

In order to visualize and locally investigate topics, messages, and other Kafka
resources, we can use a local Kafka GUI. Popular options is
[Kafdrop](https://github.com/obsidiandynamics/kafdrop) and
[Kafbat](https://www.kafbat.io), which provides a web-based interface for
managing Kafka clusters.

### Kafbat UI

- [Documentation](https://ui.docs.kafbat.io/configuration/helm-charts/quick-start)

```bash
kubectl run nettest -n kafka --rm -it --image=busybox --restart=Never -- \
  sh -c 'nslookup my-cluster-kafka-bootstrap.kafka.svc.cluster.local \
  && nc -zv my-cluster-kafka-bootstrap.kafka.svc.cluster.local 9092'
```

This command verfies that the Kafka cluster is reachable from within the
Kubernetes cluster. If the command succeeds, you should see output indicating
that the connection was successful.

```bash
helm repo add kafbat-ui https://kafbat.github.io/helm-charts
helm install kafbat-ui kafbat-ui/kafka-ui -f kafbat-values.yaml
```

We can then access the Kafbat UI by port-forwarding the service to our local
machine:

```bash
kubectl port-forward svc/kafbat-ui 8080:80 -n kafka
```

## Kafka topics

```bash
kubectl apply -f ./kafka-topics/
```

Verify that the topics have been created in the UI for example.

## Local Kafka producer and consumer

```bash
brew install kcat
```

### Produce messages to a topic

```bash
kcat -b localhost:30092 -X broker.address.family=v4 -t orders -P -K:
customer-42:{"orderId": 2, "item": "tea", "qty": 1}
customer-7:{"orderId": 3, "item": "cake", "qty": 3}
customer-8:{"orderId": 5, "item": "cake", "qty": 3}
```

### Consume messages from a topic

```bash
kcat -b localhost:30092 -X broker.address.family=v4 -t \
orders -C -f 'p%p o%o  %k => %s\n' -e
```

## Observability

In order to setup the observability stack, we can use the provided
`observability/lgtm.yaml` file to deploy LGTM (Grafana, Loki, Tempo, and
Prometheus) in the Kind cluster.

We will have a local OTel collector exposes on our local machine.

Please notice, that everything is running in memory, and this is **not**
intended for production.

### Verify

In order to verify the LGTM inner-loop stack, we can use the following
`telemetrygen` tool:

```bash
kubectl apply -f observability/telemetrygen.yaml -n observability
```

### Local Grafana

In order to access the Grafana dashboard, and the open telemetry collector
endpoints, run:

```bash
kubectl port-forward service/lgtm 3000:3000 4317:4317 4318:4318 \
-n observability
```

Then open <http://localhost:3000> in your browser.
