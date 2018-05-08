# Label

They are key/value pairs that can be attached to objects (like tags in cloud providers). Label your objects following an organizational structure :

* key: environment, value: dev, staging, ...
* key: department, value: engineering, finance, ...

Labels are not unique and multiple labels can be added to one object. Once you have labelled an object, you can use filters to narrow down results (this is called Label Selectors). You can also tag nodes using labels then with label selectors, you can setup pod to only run on specific nodes.

1. First tag the node

```shell
kubectl label nodes node1 hardware=high-spec
kubectl label nodes node2 hardware=low-spec
```

2. Add a `nodeSelector` to your pod configuration

```yaml
...
kind: Pod
spec:
  ...
  nodeSelector:
    hardware: high-spec
```

The following example is not really useful as `minikube` only runs on one node.

The labels on the node are :

```shell
$ kubectl get nodes --show-labels
NAME       STATUS    ROLES     AGE       VERSION   LABELS
minikube   Ready     master    2d        v1.10.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=minikube,node-role.kubernetes.io/master=
```

There is no label `high-spec`. We create a deployment with a nodeSelector having such a label.

```shell
$ kubectl create -f helloworld-nodeselector.yml
deployment.extensions "helloworld-deployment" created
$ kubectl get deployments
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment   3         3         3            0           16s
$ kubectl get pods
NAME                                     READY     STATUS    RESTARTS   AGE
helloworld-deployment-6576b7749d-2ztbx   0/1       Pending   0          20s
helloworld-deployment-6576b7749d-8c6kf   0/1       Pending   0          20s
helloworld-deployment-6576b7749d-l4wkt   0/1       Pending   0          20s
$ kubectl describe pod helloworld-deployment-6576b7749d-l4wkt
Name:           helloworld-deployment-6576b7749d-l4wkt
Namespace:      default
Node:           <none>
Labels:         app=helloworld
                pod-template-hash=2132633058
Annotations:    <none>
Status:         Pending
IP:
Controlled By:  ReplicaSet/helloworld-deployment-6576b7749d
Containers:
  k8s-demo:
    Image:        wardviaene/k8s-demo
    Port:         3000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-q7xb8 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  default-token-q7xb8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-q7xb8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  hardware=high-spec
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  16s (x6 over 31s)  default-scheduler  0/1 nodes are available: 1 node(s) didn't match node selector.
```

No node matches our `nodeSelector` so the scheduler cannot deploy our pods. If we add the label to the node :

```shell
$ kubectl label nodes minikube hardware=high-spec
node "minikube" labeled
$ kubectl get nodes --show-labels
NAME       STATUS    ROLES     AGE       VERSION   LABELS
minikube   Ready     master    2d        v1.10.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,hardware=high-spec,kubernetes.io/hostname=minikube,node-role.kubernetes.io/master=
$ kubectl get pods
NAME                                     READY     STATUS    RESTARTS   AGE
helloworld-deployment-6576b7749d-2ztbx   1/1       Running   0          3m
helloworld-deployment-6576b7749d-8c6kf   1/1       Running   0          3m
helloworld-deployment-6576b7749d-l4wkt   1/1       Running   0          3m
$ kubectl describe pod helloworld-deployment-6576b7749d-l4wkt
Name:           helloworld-deployment-6576b7749d-l4wkt
Namespace:      default
Node:           minikube/10.0.2.15
Start Time:     Tue, 08 May 2018 10:32:49 +0200
Labels:         app=helloworld
                pod-template-hash=2132633058
Annotations:    <none>
Status:         Running
IP:             172.17.0.4
Controlled By:  ReplicaSet/helloworld-deployment-6576b7749d
Containers:
  k8s-demo:
    Container ID:   docker://f7fb236494cf39d3ed61330553f31a9770456a8f32a85cc9972f805522afc29f
    Image:          wardviaene/k8s-demo
    Image ID:       docker-pullable://wardviaene/k8s-demo@sha256:2c050f462f5d0b3a6430e7869bcdfe6ac48a447a89da79a56d0ef61460c7ab9e
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 May 2018 10:32:52 +0200
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
Node-Selectors:  hardware=high-spec
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age               From               Message
  ----     ------                 ----              ----               -------
  Warning  FailedScheduling       1m (x14 over 4m)  default-scheduler  0/1 nodes are available: 1 node(s) didn't match node selector.
  Normal   Scheduled              1m                default-scheduler  Successfully assigned helloworld-deployment-6576b7749d-l4wkt to minikube
  Normal   SuccessfulMountVolume  1m                kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-q7xb8"
  Normal   Pulling                1m                kubelet, minikube  pulling image "wardviaene/k8s-demo"
  Normal   Pulled                 1m                kubelet, minikube  Successfully pulled image "wardviaene/k8s-demo"
  Normal   Created                1m                kubelet, minikube  Created container
  Normal   Started                1m                kubelet, minikube  Started container
```

The pods are running.

Cleanup :

```shell
$ kubectl label nodes minikube hardware-
node "minikube" labeled
$ kubectl delete deployment helloworld-deployment
deployment.extensions "helloworld-deployment" deleted
```
