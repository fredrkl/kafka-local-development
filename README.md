# Local Kafka development

This repository contains a local Kafka development environment using Docker
Compose. It allows you to quickly set up a Kafka cluster for testing and
development purposes.

I plan to setup Strimzi on a local Kind cluster.

## Kind cluster setup

We have a `kind` cluster setup in this repo. You can use the provided
`kind-config.yaml` file to create a local Kubernetes cluster using Kind.

To create the cluster, run the following command:

```bash
kind create cluster --config kind-config.yaml
```

To delete:

```bash
# Delete:  kind delete cluster --name strimzi
```

## Tilt

This repo also aims at setting up a tint environment for local development.
Tilt is a tool that automates the process of building, deploying, and testing
your applications in a Kubernetes cluster. It provides a fast feedback loop for
developers, allowing them to see changes in real-time.
