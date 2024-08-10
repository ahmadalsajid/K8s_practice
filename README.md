# K8s_practice

Notes on hands-on practice Kubernetes

## Table of Content

- [The beginning](#the-beginning)
- [Auto-scaling, Auto-healing](#auto-scaling-auto-healing)
- [Kubernetes Service](#kubernetes-service)

## The beginning

### Pod

Install `kubectl` and `minikube` following the official
[doc](https://kubernetes.io/docs/tasks/tools/) suitable for your platform.

Next, create a local cluster using `minikube` from the CLI

```
minikube start
```

Confirm that the cluster is running by

```
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   6m44s   v1.30.0
```

Alternatively, you can use

```
$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7db6d8ff4d-wq2dc           1/1     Running   0             20m
kube-system   etcd-minikube                      1/1     Running   0             20m
kube-system   kube-apiserver-minikube            1/1     Running   0             20m
kube-system   kube-controller-manager-minikube   1/1     Running   0             20m
kube-system   kube-proxy-lhxd5                   1/1     Running   0             20m
kube-system   kube-scheduler-minikube            1/1     Running   0             20m
kube-system   storage-provisioner                1/1     Running   1 (20m ago)   20m
```

Now, create a file [pod.yml](./pod.yml) and paste the below

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

We will create our first pod now

```
$ kubectl create -f pod.yml
pod/nginx created
```

Let's get the newly created pod

```
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          4m50s
```

If we want more details, we can do

```
$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          5m37s   10.244.0.3   minikube   <none>           <none>
```

we can log in to the pod and check using `curl`

```
$ minikube ssh
docker@minikube:~$ curl 10.244.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Get the details of the specific pod

```
 kubectl describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sat, 10 Aug 2024 01:06:56 +0600
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  nginx:
    Container ID:   docker://50292041f0935f3366794342e725974dd6e9071349290f6852236db94aa92a2e
    Image:          nginx:1.14.2
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 10 Aug 2024 01:06:56 +0600
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nqc6j (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-nqc6j:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  39m   default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulled     39m   kubelet            Container image "nginx:1.14.2" already present on machine
  Normal  Created    39m   kubelet            Created container nginx
  Normal  Started    39m   kubelet            Started container nginx
```

Print the logs for a pod

```
$ kubectl logs nginx
```

As we are done with the pod, let's delete it

```
$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          14m   10.244.0.3   minikube   <none>           <none>
$ kubectl delete pod nginx
pod "nginx" deleted
```

## Auto-scaling, Auto-healing

### Deployment

AS of Pods, they can be compared to a single Docker containers. For autoscaling,
we will be using `deployments`. Let's create another file,
[deployment.yml](./deployment.yml) and put the below definition

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Let's spin up the deployment

```
$ kubectl create -f deployment.yml 
deployment.apps/nginx-deployment created
```

Alternately, we can get the same by

```
$ kubectl apply -f deployment.yml 
deployment.apps/nginx-deployment created
```

List the deployment

```
$ kubectl get deploy             
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           9m55s
```

List the ReplicaSet

```
 kubectl get rs  
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-77d8468669   3         3         3       10m

```

And list the pods

```
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-2cnhp   1/1     Running   0          10m
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          10m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          10m
```

Now, open another terminal and watch for the pods

```
$ kubectl get pods -w            
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-2cnhp   1/1     Running   0          18m
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          18m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          18m
```

From the first terminal, delete one pod

```
$ kubectl delete pod nginx-deployment-77d8468669-2cnhp
pod "nginx-deployment-77d8468669-2cnhp" deleted
```

Now, if you list again the pods after some time, you will get 3 pods but one with a new name

```
$ kubectl get pods                                    
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          21m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          21m
nginx-deployment-77d8468669-ht2kj   1/1     Running   0          60s
```

And the watch log

```
$ kubectl get pods -w            
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-2cnhp   1/1     Running   0          18m
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          18m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          18m
nginx-deployment-77d8468669-2cnhp   1/1     Terminating   0          20m
nginx-deployment-77d8468669-ht2kj   0/1     Pending       0          0s
nginx-deployment-77d8468669-ht2kj   0/1     Pending       0          0s
nginx-deployment-77d8468669-ht2kj   0/1     ContainerCreating   0          0s
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-ht2kj   1/1     Running             0          1s
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
```

Let's increase the replicas from 3 to 5. update line 8 from `replicas: 3`
to `replicas: 5` in the file [deployment.yml](./deployment.yml), and apply
the change

```
$ kubectl apply -f deployment.yml
deployment.apps/nginx-deployment configured
```

On second terminal, we will get,

```
$ kubectl get pods -w            
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-2cnhp   1/1     Running   0          18m
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          18m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          18m
nginx-deployment-77d8468669-2cnhp   1/1     Terminating   0          20m
nginx-deployment-77d8468669-ht2kj   0/1     Pending       0          0s
nginx-deployment-77d8468669-ht2kj   0/1     Pending       0          0s
nginx-deployment-77d8468669-ht2kj   0/1     ContainerCreating   0          0s
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-ht2kj   1/1     Running             0          1s
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-2cnhp   0/1     Terminating         0          20m
nginx-deployment-77d8468669-w87p5   0/1     Pending             0          1s
nginx-deployment-77d8468669-w87p5   0/1     Pending             0          1s
nginx-deployment-77d8468669-qmfr5   0/1     Pending             0          0s
nginx-deployment-77d8468669-qmfr5   0/1     Pending             0          0s
nginx-deployment-77d8468669-w87p5   0/1     ContainerCreating   0          1s
nginx-deployment-77d8468669-qmfr5   0/1     ContainerCreating   0          0s
nginx-deployment-77d8468669-qmfr5   1/1     Running             0          0s
nginx-deployment-77d8468669-w87p5   1/1     Running             0          1s
```

List all the pods now,

```
$ kubectl get pods                                    
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77d8468669-9dcqk   1/1     Running   0          29m
nginx-deployment-77d8468669-czxg2   1/1     Running   0          29m
nginx-deployment-77d8468669-ht2kj   1/1     Running   0          9m2s
nginx-deployment-77d8468669-qmfr5   1/1     Running   0          86s
nginx-deployment-77d8468669-w87p5   1/1     Running   0          87s
```

Ok, we are done with the deployment for now, let's destroy what we've created!!

```
$ kubectl delete -f deployment.yml                                                                                                                                      
deployment.apps "nginx-deployment" deleted
```

## Kubernetes Service

### Cluster IP mode

only accessible from the same network/cluster

### Node Port mode

only accessible from organization

### Load Balancer mode

open to world

Let's create a deployment for the application in the file
[app_deployment.yml](./app_deployment.yml) as below

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  labels:
    app: python-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
      - name: fast-app
        image: ahmadalsajid/fast-app:latest
        ports:
        - containerPort: 8000
```

Spin up the deployment via

```
$  kubectl create -f app_deployment.yml
deployment.apps/python-app created
```

Now, let's create a service for this in [app_service.yml](./app_service.yml) file for NodePort mode.

```
apiVersion: v1
kind: Service
metadata:
  name: python-app-k8s-service
spec:
  type: NodePort
  selector:
    app: python-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

And create it by

```
$ kubectl apply -f app_service.yml 
service/python-app-k8s-service created
```

Check it with

```
$  kubectl get svc -v=7
I0810 20:12:09.709775    1847 loader.go:395] Config loaded from file:  /home/sajid/.kube/config
I0810 20:12:09.710265    1847 cert_rotation.go:137] Starting client certificate rotation controller
I0810 20:12:09.712302    1847 round_trippers.go:463] GET https://127.0.0.1:52630/api/v1/namespaces/default/services?limit=500
I0810 20:12:09.712326    1847 round_trippers.go:469] Request Headers:
I0810 20:12:09.712350    1847 round_trippers.go:473]     Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
I0810 20:12:09.712357    1847 round_trippers.go:473]     User-Agent: kubectl/v1.30.2 (linux/amd64) kubernetes/3968350
I0810 20:12:09.718214    1847 round_trippers.go:574] Response Status: 200 OK in 5 milliseconds
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1      <none>        443/TCP        20h
python-app-k8s-service   NodePort    10.99.195.89   <none>        80:30007/TCP   2m3s
```

Check for the APIs working

```
$ minikube ssh
docker@minikube:~$ curl http://10.99.195.89:80/
{"Hello":"World"}
docker@minikube:~$
```

Or, without SSHing to minikube, we can access it, from the same device where minikube is running,

```
$ minikube service --all
|-----------|------------|-------------|--------------|
| NAMESPACE |    NAME    | TARGET PORT |     URL      |
|-----------|------------|-------------|--------------|
| default   | kubernetes |             | No node port |
|-----------|------------|-------------|--------------|
😿  service default/kubernetes has no node port
|-----------|------------------------|-------------|---------------------------|
| NAMESPACE |          NAME          | TARGET PORT |            URL            |
|-----------|------------------------|-------------|---------------------------|
| default   | python-app-k8s-service |          80 | http://192.168.49.2:30007 |
|-----------|------------------------|-------------|---------------------------|
❗  Services [default/kubernetes] have type "ClusterIP" not meant to be exposed, however for local development minikube allows you to access this !
🏃  Starting tunnel for service kubernetes.
🏃  Starting tunnel for service python-app-k8s-service.
|-----------|------------------------|-------------|------------------------|
| NAMESPACE |          NAME          | TARGET PORT |          URL           |
|-----------|------------------------|-------------|------------------------|
| default   | kubernetes             |             | http://127.0.0.1:36571 |
| default   | python-app-k8s-service |             | http://127.0.0.1:37519 |
|-----------|------------------------|-------------|------------------------|
🎉  Opening service default/kubernetes in default browser...
👉  http://127.0.0.1:36571
🎉  Opening service default/python-app-k8s-service in default browser...
👉  http://127.0.0.1:37519
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

Open another terminal and do

```
$ curl http://127.0.0.1:37519/
{"Hello":"World"}%                                            
```

For now, we are done, and we can clear the system by

```
$ kubectl delete -f app_service.yml 
service "python-app-k8s-service" deleted
$ kubectl delete -f app_deployment.yml 
deployment.apps "python-app" deleted
```

## References

* [Install Tools](https://kubernetes.io/docs/tasks/tools/)
* [Kubernetes Beginner To Expert Level In One Video](https://www.youtube.com/watch?v=JoHUi9KvnOA)
* [Reset kubectl context](https://stackoverflow.com/questions/64805569/reset-the-kubectl-context-to-docker-desktop)
* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* [Unable to access a NodePort service on Minikube](https://stackoverflow.com/questions/55591468/unable-to-access-a-nodeport-service-on-minikube)
* [Unable to access minikube IP address](https://stackoverflow.com/questions/71536310/unable-to-access-minikube-ip-address)