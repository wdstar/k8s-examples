# k8s-examples
Kubernetes examples.

## Table of Contents

- [k8s-examples](#k8s-examples)
    - [Table of Contents](#table-of-contents)
    - [Install MicroK8s](#install-microk8s)
    - [Examples](#examples)
        - [Anchore Engine](#anchore-engine)
        - [chaoskube](#chaoskube)
        - [Cloudprober](#cloudprober)
        - [crond](#crond)
        - [Falco](#falco)
        - [Grafana](#grafana)
        - [Kubernetes Operational View](#kubernetes-operational-view)
        - [mtail](#mtail)
        - [Nginx](#nginx)
        - [PowerfulSeal](#powerfulseal)
        - [Sysdig](#sysdig)
        - [Telegraf](#telegraf)
        - [Telepresence](#telepresence)
        - [WeaveScope](#weavescope)

## Install MicroK8s

1. Install MicroK8s and addons.
    ```bash
    $ sudo snap install microk8s --channel latest/stable --classic
    ...

    $ sudo microk8s.status --wait-ready
    microk8s is running
    addons:
    cilium: disabled
    dashboard: disabled
    ...

    $ sudo microk8s.enable dns ingress rbac
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
    ...
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

### [Anchore Engine](https://engine.anchore.io/)

1. Install required MicroK8s addons.
    ```bash
    $ sudo microk8s.enable helm3 storage
    ...
    ```
1. Install Anchore Engine.
    ```bash
    $ sudo microk8s.helm3 repo add anchore https://charts.anchore.io
    "anchore" has been added to your repositories
    $ sudo microk8s.helm3 install anchore-demo anchore/anchore-engine
    ...
    $ kubectl get po
    NAME                                                       READY   STATUS    RESTARTS   AGE
    anchore-demo-anchore-engine-analyzer-7886755c84-tdnbr      1/1     Running   0          2m7s
    anchore-demo-anchore-engine-api-584576cd9d-phwmz           1/1     Running   0          2m7s
    anchore-demo-anchore-engine-catalog-5db69ddd68-vc9dn       1/1     Running   0          2m7s
    anchore-demo-anchore-engine-policy-59d9c66f5-sxjfg         1/1     Running   0          2m7s
    anchore-demo-anchore-engine-simplequeue-66df84ffd9-cdgwj   1/1     Running   0          2m7s
    anchore-demo-postgresql-69c55f9c5-plp2k                    1/1     Running   0          2m7s
    ```
1. Run an anchore-cli container.
    ```bash
    $ ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-demo-anchore-engine -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)
    $ kubectl run -i --tty anchore-cli --restart=Always --image anchore/engine-cli \
      --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} \
      --env ANCHORE_CLI_URL=http://anchore-demo-anchore-engine-api.default.svc.cluster.local:8228/v1/
    ...
    # check system status.
    [anchore@anchore-cli anchore-cli]$ anchore-cli system status
    ...
    Engine DB Version: 0.0.14
    Engine Code Version: 0.9.2
    [anchore@anchore-cli anchore-cli]$ exit
    # re-attach the container.
    $ kubectl attach anchore-cli -c anchore-cli -i -t
    [anchore@anchore-cli anchore-cli]$
    ```
1. Analyze a image.
    ```bash
    [anchore@anchore-cli anchore-cli]$ anchore-cli image add docker.io/library/nginx:latest
    ...
    # check analysis status.
    [anchore@anchore-cli anchore-cli]$ anchore-cli image list
    Full Tag                              Image Digest
                            Analysis Status
    docker.io/library/nginx:latest        sha256:48d56bae87c65ca642b0a1d13c3dc97c4430994991e5531ff123f77cdf975fae
                            analyzed
    # show vulnerabilities.
    [anchore@anchore-cli anchore-cli]$ anchore-cli image vuln docker.io/library/nginx:latest all
    ...
    ```
1. Manage policies.
    ```bash
    [anchore@anchore-cli anchore-cli]$ anchore-cli policy list
    Policy ID                                   Active        Created                     Updated
    2c53a13c-1765-11e8-82ef-23527761d060        True          2021-03-13T02:09:43Z        2021-03-13T02:09:43Z
    [anchore@anchore-cli anchore-cli]$ anchore-cli policy get 2c53a13c-1765-11e8-82ef-23527761d060 --detail
    ... (output as JSON)
    ```

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

### [Falco](https://github.com/falcosecurity/falco)

Ref.: https://falco.org/docs/installation/

1. Add `--allow-privileged=true` to `/var/snap/microk8s/current/args/kube-apiserver` and activate it. 
    ```bash
    $ sudo systemctl restart snap.microk8s.daemon-apiserver.service
    ```
1. Apply manifests.
    ```bash
    $ kubectl apply -k falco/bases
    ```
1. Verify Falco started correctly.
    ```bash
    $ kubectl logs -l app=falco-example
    ```
1. Delete manifests.
    ```bash
    $ kubectl delete -k falco/bases
    ```

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

### [Kubernetes Operational View]()

1. Apply manifests.
    ```bash
    $ kubectl apply -k kube-ops-view/bases
    ```
1. Add the following DNS entry to your `hosts` file.
    ```
    <microk8s host IP> kube-ops-view.default.uk8s.example.com
    ```
1. Access http://kube-ops-view.default.uk8s.example.com
1. Delete manifests.
    ```bash
    $ kubectl delete -k kube-ops-view/bases
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

### [Sysdig](https://github.com/draios/sysdig)

#### Local

1. [Install Sysdig](https://github.com/draios/sysdig/wiki/How-to-Install-Sysdig-for-Linux).
1. Create a proxy server to the Kubernetes API server.
    ```bash
    $ kubectl proxy --port=8080
    ```
1. On another terminal, run `csysdig`.
    ```bash
    $ sudo csysdig -k http://127.0.0.1:8080 -pc --cri /var/snap/microk8s/common/run/containerd.sock
    ```
    - Tips: Using eBPF instrumentation instead of Sysdig kernel module.
        1. Confirm the Kernel version (eBPF support >= `4.14`) and install `clang` and `llvm`.
            ```bash
            $ uname -r
            4.15.0-99-generic
            $ sudo apt-get install clang llvm
            ```
        1. Run `csysdig` with `-B` (or `--bpf=""`) option.
            ```bash
            $ sudo csysdig -B -k http://127.0.0.1:8080 -pc --cri /var/snap/microk8s/common/run/containerd.sock
            ```

#### MicroK8s

1. Add `--allow-privileged=true` to `/var/snap/microk8s/current/args/kube-apiserver` and activate it. 
    ```bash
    $ sudo systemctl restart snap.microk8s.daemon-apiserver.service
    ```
1. Apply manifests.
    ```bash
    $ kubectl apply -k sysdig/bases
    ```
    1. If you use Sysdig Inspect too.
        ```bash
        $ kubectl apply -k sysdig/inspect
        ```
    1. Add the following DNS entry to your `hosts` file.
        ```
        <microk8s host IP> sysdig-inspect.default.uk8s.example.com
        ```
1. Execute `bash` in a sysdig-daemonset pod and hit `sysdig` or `csysdig`.
    ```bash
    $ kubectl exec -it sysdig-daemonset-<pod id> -c sysdig -- bash
    root@sysdig-daemonset-rgqnb:/# csysdig -K /var/run/secrets/kubernetes.io/serviceaccount/token -k https://${KUBERNETES_SERVICE_HOST} -pk -pc
    ```
1. If you use Sysdig Inspect, access http://sysdig-inspect.default.uk8s.example.com
1. Delete manifests.
    ```bash
    $ kubectl delete -k sysdig/bases
    or
    $ kubectl delete -k sysdig/inspect
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

### [WeaveScope](https://github.com/weaveworks/scope)

1. Add `--allow-privileged=true` to `/var/snap/microk8s/current/args/kube-apiserver` and activate it. 
    ```bash
    $ sudo systemctl restart snap.microk8s.daemon-apiserver.service
    ```
1. Create `weave` namespace and apply manifests.
    ```bash
    $ kubectl apply -f weavescope/ns.yaml
    $ kubectl apply -k weavescope/bases
    ```
1. **Note**: Dev. environment only, access Web UI via Ingress.
    1. Apply the Ingress manifest.
        ```bash
        $ kubectl apply -f weavescope/ingress.yaml
        ```
    1. Add the following DNS entry to your `hosts` file.
        ```
        <microk8s host IP> weave-scope.default.uk8s.example.com
        ```
    1. Access http://weave-scope.default.uk8s.example.com
1. Access Web UI by the following port forwarding.
    1. Setup port forwarding.
        ```bash
        $ kubectl port-forward svc/weave-scope-app -n weave 4040:80
        ```
    1. Access http://localhost:4040
1. Delete manifests.
    ```bash
    $ kubectl delete -k weavescope/bases
    $ kubectl delete -f weavescope/ingress.yaml
    $ kubectl delete -f weavescope/ns.yaml
    ```
