---
lastmod: 2021-12-11
date: 2017-01-21
linktitle: To Kubernetes
title: Deploying to Kubernetes
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
weight: 30
---

Deploying KrakenD in Kubernetes requires a straightforward configuration.

Create a `Dockerfile` that includes the configuration of the service. Read how to generate a [Docker artifact](/docs/deploying/docker/) for detailed instructions. You could also use a ConfigMap, although the recommendation is to use immutable artifacts.

From here you need to create a `NodePort` and send all the traffic to KrakenD.

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

There is no official Helm chart for KrakenD. But you can see as example [Mikescandy's contribution](https://github.com/mikescandy/krakend-helm)
