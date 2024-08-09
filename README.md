# K8s_practice

Notes on hands-on practice Kubernetes

## Table of Content

- [tb](#the-beginning)
- [2nd](#auto-scaling-auto-healing)

## The beginning

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

## References

* [Install Tools](https://kubernetes.io/docs/tasks/tools/)
* [Kubernetes Beginner To Expert Level In One Video](https://www.youtube.com/watch?v=JoHUi9KvnOA)
* [Reset kubectl context](https://stackoverflow.com/questions/64805569/reset-the-kubectl-context-to-docker-desktop)