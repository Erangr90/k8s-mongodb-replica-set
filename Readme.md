### Replica set initiate

    1. Go to the first container shell
    kubectl exec -it mongo-0 -- mongosh

    2. Initiate the replica set:
    rs.initiate()

    var cfg = rs.conf()

    cfg.members[0].host="mongo-0.mongo:27017"

    rs.reconfig(cfg)

    rs.status()

    3. Add the others containers to the replica set

    rs.add("mongo-1.mongo:27017")

    rs.add("mongo-2.mongo:27017")

    rs.status()

    4. The connection string inside the kubernetes cluster is:
    kubectl run mongo --rm -it --image mongo -- sh

    mongosh mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo/

### Install metal LB

Install guide:
https://metallb.universe.tf/installation/

    1. Preparation
    editing kube-proxy config in current cluster:

    kubectl edit configmap -n kube-system kube-proxy

    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    kind: KubeProxyConfiguration
    mode: "ipvs"
    ipvs:
      strictARP: true

    on vime: Then press "esc" then type ":x" then press "enter"

    2. Installation by manifest:
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml

    3. Configuration -> Layer 2 configuration:
    https://metallb.universe.tf/configuration/

    Metallb.yaml file:

    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: first-pool
      namespace: metallb-system
    spec:
      addresses:
        - 192.168.7.100-192.168.7.110
    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: example
      namespace: metallb-system



    3. Change the address to your free ip address range:
    3.1 on windows shell : ipconfig
    3.2 under Wireless LAN adapter Wi-Fi -> IPv4 Address

    4. kubectl apply -f metallb.yaml

### Expose ip address

kubectl expose pod mongo-0 --port 27017 --target-port 27017 --type LoadBalancer
