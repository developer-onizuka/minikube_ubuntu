# 0. server's functions

| deployment | VM | ExposedIP | clusterIP | container's port |
| --- | --- | --- | --- | --- |
| nginx-test | minikube | 192.168.99.100 | don't care. resolved by dns | 80 | 
| mongo-test | minikube-m03 | N/A | don't care. resolved by dns | 27017 | 
| employee-test | minikube-m02 | N/A | don't care. resolved by dns | 5001 |


# 1. Install each package on Ubuntu20.4
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

# 6. Make employee's yaml file and run it. Check where the pod is running.
```
$ cat employee_latest_nodeSelector.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: employee-test
spec:
  selector:
    matchLabels:
      run: employee-test
  replicas: 1
  template:
    metadata:
      labels:
        run: employee-test
    spec:
      containers:
      - name: employee
        image: developeronizuka/employee
        ports:
        - containerPort: 5001
        - containerPort: 5000
        env:
        - name: MONGO
          value: mongo-test
        command: ["/usr/local/dotnet/publish/Employee"]
      nodeSelector:
        location: telaviv

$ kubectl create -f employee_latest_nodeSelector.yaml 

$ kubectl describe pod employee-test
Name:         employee-test-84b567445f-lmgf9
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.99.101
Start Time:   Sun, 18 Apr 2021 20:33:31 +0900
Labels:       pod-template-hash=84b567445f
              run=employee-test
Annotations:  <none>
Status:       Running
IP:           10.244.1.4
IPs:
  IP:           10.244.1.4
Controlled By:  ReplicaSet/employee-test-84b567445f
Containers:
  employee:
    Container ID:  docker://83a72ed7c6bcbd220f89838064adf0270997b03ee89ecf2470410a7904c83036
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Sun, 18 Apr 2021 20:35:15 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  mongo-test
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
Node-Selectors:  location=telaviv
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m26s  default-scheduler  Successfully assigned default/employee-test-84b567445f-lmgf9 to minikube-m02
  Normal  Pulling    2m25s  kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     42s    kubelet            Successfully pulled image "developeronizuka/employee" in 1m43.358795194s
  Normal  Created    42s    kubelet            Created container employee
  Normal  Started    42s    kubelet            Started container employee

$ kubectl -it exec dnsutils -- /bin/sh
# curl https://10.244.1.4:5001 -k
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title> - Employee</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" href="/">Employee</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/Home/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            
<h1>List of Employees</h1>

<h2></h2>

<a href="/Home/Insert"> Add New Employee</a>

<br /><br />


<table border="1" cellpadding="10">
</table>

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2020 - Employee - <a href="/Home/Privacy">Privacy</a>
        </div>
    </footer>
    <script src="/lib/jquery/dist/jquery.min.js"></script>
    <script src="/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/site.js"></script>
    
</body>
</html>
```

# 7. Expose employee's deployment.
```
$ kubectl expose deployment employee-test
service/employee-test exposed

$ minikube service list
|-------------|---------------|--------------|-----|
|  NAMESPACE  |     NAME      | TARGET PORT  | URL |
|-------------|---------------|--------------|-----|
| default     | employee-test | No node port |
| default     | kubernetes    | No node port |
| default     | mongo-test    | No node port |
| kube-system | kube-dns      | No node port |
|-------------|---------------|--------------|-----|
```

# 8. Check if it works correcly 
```
$ kubectl -it exec dnsutils -- /bin/sh
# curl https://employee-test:5001 -k
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title> - Employee</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" href="/">Employee</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/Home/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            
<h1>List of Employees</h1>

<h2></h2>

<a href="/Home/Insert"> Add New Employee</a>

<br /><br />


<table border="1" cellpadding="10">
</table>

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2020 - Employee - <a href="/Home/Privacy">Privacy</a>
        </div>
    </footer>
    <script src="/lib/jquery/dist/jquery.min.js"></script>
    <script src="/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/site.js"></script>
    
</body>
</html>
```
# 9. Create the persistent volume and make nginx's config on it.
```
$ cat nginx-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
spec:
  storageClassName: standard
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/data"
    type: DirectoryOrCreate

$ cat nginx-pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath-pvc
spec:
  storageClassName: standard
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi

$ kubectl create -f nginx-pv.yaml 
$ kubectl create -f nginx-pvc.yaml 
  
$ ssh docker@192.168.99.100
docker@192.168.99.100's password: tcuser

$ cat /var/data/default.conf 
upstream proxy.com {
        server employee-test:5001;
}

server {
        listen 80;
        server_name localhost;
        location / {
                root /usr/share/nginx/html;
                index index.html index.htm;
                proxy_pass https://proxy.com;
        }
}

```
# 10. Make nginx's yaml file and run it. 
```
$ cat nginx_1.14.2_nodeSelector.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      run: nginx-test
  replicas: 1
  template:
    metadata:
      labels:
        run: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d
        ports:
        - containerPort: 80
      volumes:
      - name: nginx-conf
        persistentVolumeClaim:
         claimName: hostpath-pvc
      nodeSelector:
        location: tokyo

$ kubectl create -f nginx_1.14.2_nodeSelector.yaml 
deployment.apps/nginx-test created

```

# 11. Expose nginx's demployment. Access the following URL from Host's blowser.
```
$ kubectl expose deployment nginx-test --type=LoadBalancer
service/nginx-test exposed

$ minikube service list
|-------------|---------------|--------------|-----------------------------|
|  NAMESPACE  |     NAME      | TARGET PORT  |             URL             |
|-------------|---------------|--------------|-----------------------------|
| default     | employee-test | No node port |
| default     | kubernetes    | No node port |
| default     | mongo-test    | No node port |
| default     | nginx-test    |           80 | http://192.168.99.100:30025 |
| kube-system | kube-dns      | No node port |
|-------------|---------------|--------------|-----------------------------|

```
# 12. The status of All Pods (but clusterIP's are deferent from the above, because I took it from another tries.)
```
$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/dnsutils                         1/1     Running   1          78m
pod/employee-test-84b567445f-5wb28   1/1     Running   0          71m
pod/mongo-test-7c6f94fcc4-b6jq7      1/1     Running   0          83m
pod/nginx-test-77889fd64-fct9m       1/1     Running   0          49m

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/employee-test   ClusterIP      10.105.6.20      <none>        5001/TCP,5000/TCP   67m
service/kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP             88m
service/mongo-test      ClusterIP      10.101.120.214   <none>        27017/TCP           80m
service/nginx-test      LoadBalancer   10.107.107.112   <pending>     80:31238/TCP        48m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/employee-test   1/1     1            1           71m
deployment.apps/mongo-test      1/1     1            1           83m
deployment.apps/nginx-test      1/1     1            1           49m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/employee-test-84b567445f   1         1         1       71m
replicaset.apps/mongo-test-7c6f94fcc4      1         1         1       83m
replicaset.apps/nginx-test-77889fd64       1         1         1       49m

```
