# 1. Install each package at Ubuntu20.4
```
<minikube>
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
$ sudo dpkg -i minikube_latest_amd64.deb

<kubectl>
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
```
# 2. Run minikube
```
$ minikube start --driver=virtualbox --nodes=3
😄  minikube v1.19.0 on Ubuntu 20.04
✨  Using the virtualbox driver based on user configuration
💿  Downloading VM boot image ...
    > minikube-v1.19.0.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
    > minikube-v1.19.0.iso: 244.49 MiB / 244.49 MiB  100.00% 2.75 MiB p/s 1m29.
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.20.2 preload ...
    > preloaded-images-k8s-v10-v1...: 491.71 MiB / 491.71 MiB  100.00% 2.26 MiB
🔥  Creating virtualbox VM (CPUs=2, Memory=2633MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting node minikube-m02 in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=2633MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.99.100
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.4 ...
    ▪ env NO_PROXY=192.168.99.100
🔎  Verifying Kubernetes components...

👍  Starting node minikube-m03 in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=2633MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.99.100,192.168.99.101
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.4 ...
    ▪ env NO_PROXY=192.168.99.100
    ▪ env NO_PROXY=192.168.99.100,192.168.99.101
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# 3. Check the node list and status
```
$ minikube node list
minikube	192.168.99.100
minikube-m02	192.168.99.101
minikube-m03	192.168.99.102

$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

minikube-m02
type: Worker
host: Running
kubelet: Running

minikube-m03
type: Worker
host: Running
kubelet: Running
```

# 3. Put labels on each node
```
$ kubectl label nodes minikube location=tokyo
node/minikube labeled
$ kubectl label nodes minikube-m02 location=telaviv
node/minikube-m02 labeled
$ kubectl label nodes minikube-m03 location=houston
node/minikube-m03 labeled
$ kubectl describe node |grep location
                    location=tokyo
                    location=telaviv
                    location=houston
```

# 4. Make mongo's yaml file and run it. Check where the pod is running.
```
$ cat mongo_latest_nodeSelector.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-test
spec:
  selector:
    matchLabels:
      run: mongo-test
  replicas: 1
  template:
    metadata:
      labels:
        run: mongo-test
    spec:
      containers:
      - name: mongodb
        image: docker.io/mongo
        ports:
        - containerPort: 27017
      nodeSelector:
        location: houston

$ kubectl create -f mongo_latest_nodeSelector.yaml 
deployment.apps/mongo-test created

$ kubectl describe pod mongo-test
Name:         mongo-test-7cb544b5bd-29c4v
Namespace:    default
Priority:     0
Node:         minikube-m03/192.168.99.102
Start Time:   Sun, 18 Apr 2021 19:00:54 +0900
Labels:       pod-template-hash=7cb544b5bd
              run=mongo-test
Annotations:  <none>
Status:       Running
IP:           10.244.2.2
IPs:
  IP:           10.244.2.2
Controlled By:  ReplicaSet/mongo-test-7cb544b5bd
Containers:
  mongodb:
    Container ID:   docker://c80969dccf56c3f34043493572d0394ba6128fdc941dc8a976e369f43441484f
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:b66f48968d757262e5c29979e6aa3af944d4ef166314146e1b3a788f0d191ac3
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 18 Apr 2021 19:02:31 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-chlbn (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-chlbn:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-chlbn
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  location=houston
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  105s  default-scheduler  Successfully assigned default/mongo-test-7cb544b5bd-29c4v to minikube-m03
  Normal  Pulling    105s  kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled     8s    kubelet            Successfully pulled image "docker.io/mongo" in 1m36.500647634s
  Normal  Created    8s    kubelet            Created container mongodb
  Normal  Started    8s    kubelet            Started container mongodb
```
# 5. Expose mongo's deployment.
```
$ kubectl expose deployment mongo-test
service/mongo-test exposed

$ minikube service list
|-------------|------------|--------------|-----|
|  NAMESPACE  |    NAME    | TARGET PORT  | URL |
|-------------|------------|--------------|-----|
| default     | kubernetes | No node port |
| default     | mongo-test | No node port |
| kube-system | kube-dns   | No node port |
|-------------|------------|--------------|-----|

$ cat dnsutils_latest.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  labels:
    name: dnsutils
spec:
  containers:
  - name: dnsutils
    image: tutum/dnsutils
    command:
    - sleep
    - "3600"

$ kubectl create -f dnsutils_latest.yaml 
pod/dnsutils created

$ kubectl exec -it dnsutils -- /bin/sh
# nslookup mongo-test
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	mongo-test.default.svc.cluster.local
Address: 10.110.22.192

# apt-get update
# apt install curl
# curl 10.110.22.192:27017
It looks like you are trying to access MongoDB over HTTP on the native driver port.

# curl mongo-test:27017
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```
