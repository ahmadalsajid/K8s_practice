# K8s_practice

Practice Kubernetes

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
As we are done with the pod, let's delete it
```
$  kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          14m   10.244.0.3   minikube   <none>           <none>
$ kubectl delete pod nginx
pod "nginx" deleted
```







## References

* [Install Tools](https://kubernetes.io/docs/tasks/tools/)
* [Kubernetes Beginner To Expert Level In One Video](https://www.youtube.com/watch?v=JoHUi9KvnOA)