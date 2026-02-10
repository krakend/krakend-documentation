---
lastmod: 2021-12-11
old_version: true
date: 2017-01-21
linktitle: To Kubernetes
title: Deploying KrakenD API Gateway on Kubernetes
description: Discover the best practices and guidelines for deploying KrakenD API Gateway on Kubernetes, enabling scalable and efficient API management
menu:
  community_v2.12:
    parent: "190 Deployment and Go-Live"
weight: 30
---

Deploying KrakenD in Kubernetes requires a straightforward configuration.

Create a `Dockerfile` that includes the configuration of the service. Read how to generate a [Docker artifact](/docs/v2.12/deploying/docker/) for detailed instructions. You could also use a ConfigMap, although the recommendation is to use immutable artifacts.

From here you need to create a `NodePort` and send all the traffic to KrakenD.

{{< note title="Run as user 1000" type="tip" >}}
Whether you run KrakenD on Kubernetes, OpenShift, or any other platform with the capability to run the container as a specific user UID, make sure you use the **UID 1000**
{{< /note >}}


## Deployment definition YAML
The KrakenD `deployment` definition, in a file called `deployment-definition.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krakend-deployment
spec:
  selector:
    matchLabels:
      app: krakend
  replicas: 2
  template:
    metadata:
      labels:
        app: krakend
    spec:
      containers:
      - name: krakend
        image: YOUR-KRAKEND-IMAGE:1.0.0
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
        command: [ "/usr/bin/krakend" ]
        args: [ "run", "-d", "-c", "/etc/krakend/krakend.json", "-p", "8080" ]
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
            add:
              - NET_BIND_SERVICE
        env:
        - name: KRAKEND_PORT
          value: "8080"
```


## Service definition yaml

The KrakenD `service` definition, in a file called `service-definition.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: krakend-service
spec:
  type: NodePort
  ports:
  - name: http
    port: 8000
    targetPort: 8080
    protocol: TCP
  selector:
    app: krakend
```

## Registering the service

Using the `kubectl` command:

{{< terminal title="Register deployment">}}
kubectl create -f deployment-definition.yaml
{{< /terminal >}}

{{< terminal title="Register service">}}
kubectl create -f service-definition.yaml
{{< /terminal >}}

For a more step by step process see [this blog entry](/blog/krakend-on-kubernetes/).

## Helm Chart

There is no official Helm chart for KrakenD. However, there is a Helm chart here
tracking Krakend releases maintained by [Equinix Metal](https://github.com/equinixmetal-helm/krakend).
