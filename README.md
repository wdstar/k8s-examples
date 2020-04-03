# k8s-examples
Kubernetes examples.

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
    Applying manifest
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

### Nginx

1. Apply manifest.
    ```bash
    $ kubectl apply -f bases/nginx.yaml
    ```
1. Add the following DNS entry to your `hosts` file
    ```
    <microk8s host IP> nginx.default.uk8s.example.com
    ```
1. Access http://nginx.default.uk8s.example.com