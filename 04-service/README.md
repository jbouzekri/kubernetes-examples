# Service

I like this quote : *A service is the logical bridge between the "mortal" pods and other services or end-users*

the `kubectl expose` in a previous example created a service.

Creating a service creates an endpoint for the pod :

* **ClusterIp** : virtual IP only reachable from within the cluster
* **NodePort** : a port that is the same on each node that is also reachable externally
* **LoadBalancer** : a loadbalancer created by the cloud provider that will route external traffic to every node on the NodePort (for production)

Let's restart the pod from the `01-first-app` example :

```shell
$ kubectl create -f ../01-first-app/helloworld-pod.yml
pod "nodehelloworld.example.com" created
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nodehelloworld.example.com   1/1       Running   0          6s
$ kubectl describe pod nodehelloworld.example.com
Name:         nodehelloworld.example.com
Namespace:    default
Node:         minikube/10.0.2.15
Start Time:   Mon, 07 May 2018 21:07:40 +0200
Labels:       app=helloworld
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
Containers:
  k8s-demo:
    Container ID:   docker://0c0c6cb1bf73046f4f29b4965d2c8be4bc7dbe33f386ca631b1496473dd16a45
    Image:          wardviaene/k8s-demo
    Image ID:       docker-pullable://wardviaene/k8s-demo@sha256:2c050f462f5d0b3a6430e7869bcdfe6ac48a447a89da79a56d0ef61460c7ab9e
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 07 May 2018 21:07:43 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-q7xb8 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-q7xb8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-q7xb8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              48s   default-scheduler  Successfully assigned nodehelloworld.example.com to minikube
  Normal  SuccessfulMountVolume  47s   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-q7xb8"
  Normal  Pulling                47s   kubelet, minikube  pulling image "wardviaene/k8s-demo"
  Normal  Pulled                 46s   kubelet, minikube  Successfully pulled image "wardviaene/k8s-demo"
  Normal  Created                46s   kubelet, minikube  Created container
  Normal  Started                45s   kubelet, minikube  Started container
```

Let's create the service :

```shell
$ kubectl create -f helloworld-nodeport-service.yml
service "helloworld-service" created
```

Retrieve the service url from minikube :

```shell
$ minikube service helloworld-service --url
http://192.168.99.100:31001
$ curl http://192.168.99.100:31001
Hello World!
$ kubectl describe service helloworld-service
Name:                     helloworld-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.100.167.15
Port:                     <unset>  31001/TCP
TargetPort:               nodejs-port/TCP
NodePort:                 <unset>  31001/TCP
Endpoints:                172.17.0.4:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Note : IP is a virtual Cluster IP only reachable from within the cluster

```shell
$ kubectl delete service helloworld-service
service "helloworld-service" deleted
$ kubectl create -f helloworld-nodeport-service.yml
service "helloworld-service" created
$ kubectl describe service helloworld-service
Name:                     helloworld-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.101.117.96
Port:                     <unset>  31001/TCP
TargetPort:               nodejs-port/TCP
NodePort:                 <unset>  31001/TCP
Endpoints:                172.17.0.4:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Note that the virtual IP changes but the port number not because we defined it statically in the yaml file. You can define a static IP in the yaml file too.

Cleanup :

```shell
$ kubectl delete service helloworld-service
service "helloworld-service" deleted
$ kubectl delete pod nodehelloworld.example.com
pod "nodehelloworld.example.com" deleted
```