# k8s-examples
Kubernetes examples.

## Table of Contents

- [k8s-examples](#k8s-examples)
    - [Table of Contents](#table-of-contents)
    - [Install MicroK8s](#install-microk8s)
    - [Examples](#examples)
        - [chaoskube](#chaoskube)
        - [Cloudprober](#cloudprober)
        - [crond](#crond)
        - [Grafana](#grafana)
        - [mtail](#mtail)
        - [Nginx](#nginx)
        - [PowerfulSeal](#powerfulseal )
        - [Telegraf](#telegraf)
        - [Telepresence](#telepresence)

## Install MicroK8s

1. Install MicroK8s and addons.
    ```bash
    $ sudo snap install microk8s --channel 1.17/stable --classic
    ...

    $ sudo microk8s.status --wait-ready
    microk8s is running
    addons:
    cilium: disabled
    dashboard: disabled
    ...

    $ sudo microk8s.enable dns ingress
    Enabling DNS
    Applying manifests
    serviceaccount/coredns created
    configmap/coredns created
    ...
    Restarting kubelet
    DNS is enabled
    Enabling Ingress
    namespace/ingress unchanged
    serviceaccount/nginx-ingress-microk8s-serviceaccount unchanged
    clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole unchanged
    ...
    Ingress is enabled
    ```
1. Alias the `microk8s.kubectl` command or import microk8s' conf. to your existing `kubectl`.
    ```bash
    # Alias
    $ sudo snap alias microk8s.kubectl kubectl

    # Or import the following conf.
    $ sudo microk8s.config
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0...
        server: https://192.168.1.211:16443
    name: microk8s-cluster
    contexts:
    - context:
        cluster: microk8s-cluster
        user: admin
    name: microk8s
    current-context: microk8s
    kind: Config
    preferences: {}
    users:
    - name: admin
    user:
        username: admin
        password: ************************************************************
    ```

## Examples

### [chaoskube](https://github.com/linki/chaoskube)

1. Apply manifests.
    ```bash
    $ kubectl apply -k chaoskube/bases
    ```
1. Delete manifests.
    ```bash
    $ kubectl delete -k chaoskube/bases
    ```

### [Cloudprober](https://github.com/google/cloudprober)

1. Apply manifests.
    ```bash
    $ kubectl apply -k cloudprober/bases
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> cloudprober.default.uk8s.example.com prom-cb.default.uk8s.example.com
    ```
1. Access http://cloudprober.default.uk8s.example.com/status and http://prom-cb.default.uk8s.example.com
1. Delete manifests.
    ```bash
    $ kubectl delete -k cloudprober/bases
    ```

### crond

See: https://github.com/wdstar/crond-image

### [Grafana](https://grafana.com/)

1. Apply manifests.
    ```bash
    $ kubectl apply -k grafana/bases
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> grafana.default.uk8s.example.com
    ```
1. Access http://grafana.default.uk8s.example.com
1. Data source examples.
    - Prometheus:
        - `http://prom.default.svc.cluster.local:9090`: See [Mtail example](#mtail).
        - `http://prom-cb.default.svc.cluster.local:9090`: See [Cloudprober example](#cloudprober).
        - `http://prom-tg.default.svc.cluster.local:9090`: See [Telegraf example](#telegraf).
    - InfluxDB:
        - `http://influxdb.default.svc.cluster.local:8086`: See [Telegraf example](#telegraf).
1. Delete manifests.
    ```bash
    $ kubectl delete -k grafana/bases
    ```

### mtail

See: https://github.com/wdstar/mtail-image

### [Nginx](https://www.nginx.com/)

1. Apply manifests.
    ```bash
    $ kubectl apply -k nginx/bases
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> nginx.default.uk8s.example.com
    ```
1. Access http://nginx.default.uk8s.example.com
1. Delete manifests.
    ```bash
    $ kubectl delete -k nginx/bases
    ```

### [PowerfulSeal](https://github.com/bloomberg/powerfulseal)

1. Apply manifests.
    ```bash
    $ kubectl apply -k powerfulseal/bases
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> powerfulseal.default.uk8s.example.com powerfulseal-exp.default.uk8s.example.com
    ```
1. Access [Web UI](http://powerfulseal.default.uk8s.example.com), [Prometheus exporter](http://powerfulseal-exp.default.uk8s.example.com/metrics)
1. Delete manifests.
    ```bash
    $ kubectl delete -k powerfulseal/bases
    ```

### [Telegraf](https://github.com/influxdata/telegraf)

#### with InfluxDB

1. Apply manifests.
    ```bash
    $ kubectl apply -k telegraf/influxdb
    ```
1. Access InfluxDB
    ```bash
    $ kubectl exec -it telegraf-<pod id> -c influxdb -- influx
    Connected to http://localhost:8086 version 1.8.0
    InfluxDB shell version: 1.8.0
    >
    ```
1. Delete manifests.
    ```bash
    $ kubectl delete -k telegraf/influxdb
    ```

#### with Prometheus

1. Apply manifests.
    ```bash
    $ kubectl apply -k telegraf/prom
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> prom-tg.default.uk8s.example.com
    ```
1. Access http://prom-tg.default.uk8s.example.com
1. Delete manifests.
    ```bash
    $ kubectl delete -k telegraf/prom
    ```

### [Telepresence](https://www.telepresence.io/)

#### with Nginx example

1. Apply Nginx example's manifests.
    ```bash
    $ kubectl apply -k nginx/bases
    ```
1. Access Nginx service (welcome page) by the Kubernetes internal DNS.
    ```bash
    $ telepresence --run curl http://nginx.default.svc.cluster.local
    ...
    $ telepresence --run curl http://nginx.default
    ...
    # by Docker
    $ telepresence --docker-run --rm -it pstauffer/curl curl http://nginx.default.svc.cluster.local
    ...
    ```
