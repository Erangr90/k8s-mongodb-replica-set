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
        - 127.0.10.200-127.0.10.203

    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: example
      namespace: metallb-system


    4. kubectl apply -f metallb.yaml

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

### Expose ip address

    kubectl expose pod mongo-0 --port 27017 --target-port 27017 --type LoadBalancer
    kubectl expose pod mongo-1 --port 27017 --target-port 27017 --type LoadBalancer
    kubectl expose pod mongo-2 --port 27017 --target-port 27017 --type LoadBalancer

    On Windows, the hosts file is located at C:\Windows\System32\drivers\etc\hosts. Edit it with a text editor run as Administrator. Add the following lines to map the IP addresses to hostnames:

    127.0.10.200 mongo-0.mongo
    127.0.10.201 mongo-1.mongo
    127.0.10.202 mongo-2.mongo

