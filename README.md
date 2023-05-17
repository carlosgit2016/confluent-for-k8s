## Confluent for k8s example

This repository uses CFK to deploy and manage Kafka connectors and can be extensible to deploy any other object supported by the operator (Apache Kafka®, Connect workers, ksqlDB, Schema Registry, Confluent Control Center, Confluent REST Proxy).

### Components
#### [CFK](https://docs.confluent.io/operator/current/overview.html#co-long) (Confluent for Kubernetes)

The confluent operator need to be configured in the cluster before using the objects in `confluent-platform.yaml` https://docs.confluent.io/operator/current/co-quickstart.html#co-long-quickstart. 

In this example I am using [minikube](https://minikube.sigs.k8s.io/docs/start/) so those are all necessary commands to execute:
```sh
# Start minikube
minikube start

# Create namespace confluent
kubectl create namespace confluent

# Set this namespace to default for your Kubernetes context.
kubectl config set-context --current --namespace confluent

# Add the Confluent for Kubernetes Helm repository.
helm repo add confluentinc https://packages.confluent.io/helm
helm repo update

# Install Confluent for Kubernetes.
helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes
```

You can then port forward to the control central plane
```sh
kubectl port-forward svc/controlcenter 9021 -nconfluent
```

#### Connectors
Connectors can be deployed to the cluster after the operator is healthy and running, to deploy a connector use, create a new file or edit the existent one in `connectors/` folder, then run

```sh
# Deploy connectors
kubectl apply -f connectors/
```

#### Producer App
The producer App can be used to performance test or simple generate a certain amount of records in the chosen topic, the amount of records can be changed in the container `command`, the topic is configured in `confluent-platform.yaml`

```sh
# Run the producer app
kubectl apply -f kafka-producer.yaml

# To re execute you can simple delete and re create
kubectl delete -f kafka-producer.yaml && kubectl apply -f kafka-producer.yaml
```

You can also follow the logs of the producer-app using
```sh
kubectl logs -l app=producer-app -nconfluent
```

#### Rabbitmq deployment
There is a deployment of rabbitmq in `rabbitmq.yaml` that can be used to spin up a simple rabbitmq service 

```sh
kubectl apply -f rabbitmq.yaml
```

#### Rabbitmq producer script
The producer script written in python can be used to generate messages to certain rabbitmq queue after the rabbitmq service is deployed

```sh
# Port foward to the rabbitmq service first
kubectl port-forward svc/rabbitmq 5672:5672

# Usage
python scripts/producer.py <rabbitmq_queue> <message_count>
```