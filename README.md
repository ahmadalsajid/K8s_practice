# K8s_practice

Notes on hands-on practice Kubernetes

## Table of Content

- [tb](#the-beginning)
- [2nd](#auto-scaling-auto-healing)

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
## References

* [Install Tools](https://kubernetes.io/docs/tasks/tools/)
* [Kubernetes Beginner To Expert Level In One Video](https://www.youtube.com/watch?v=JoHUi9KvnOA)
* [Reset kubectl context](https://stackoverflow.com/questions/64805569/reset-the-kubectl-context-to-docker-desktop)
* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)