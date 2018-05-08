# Healthcheck

Your application could malfunctions but the pods and their containers could still be running. You can run health checks to detect these states.

2 different types of healthchecks :

* running a command in the container periodically
* periodic checks on a url (HTTP)

We create a deployment with a configured healthcheck.

```shell
$ kubectl create -f helloworld-healthcheck.yml
deployment.extensions "helloworld-deployment" created
jobou@maison-linux:~/workspace/kubernetes-examples/06-healthcheck$ kubectl get pods
NAME                                     READY     STATUS    RESTARTS   AGE
helloworld-deployment-77664648b9-2gh2s   1/1       Running   0          33s
helloworld-deployment-77664648b9-lnznz   1/1       Running   0          33s
helloworld-deployment-77664648b9-ltn69   1/1       Running   0          33s
jobou@maison-linux:~/workspace/kubernetes-examples/06-healthcheck$ kubectl describe pod helloworld-deployment-77664648b9-2gh2s
Name:           helloworld-deployment-77664648b9-2gh2s
Namespace:      default
Node:           minikube/10.0.2.15
Start Time:     Tue, 08 May 2018 10:46:34 +0200
Labels:         app=helloworld
                pod-template-hash=3322020465
Annotations:    <none>
Status:         Running
IP:             172.17.0.6
Controlled By:  ReplicaSet/helloworld-deployment-77664648b9
Containers:
  k8s-demo:
    Container ID:   docker://d739b9ec4c5e9b78a6367799399603672233d5e923bf3ec4b7f590e8f051ca39
    Image:          wardviaene/k8s-demo
    Image ID:       docker-pullable://wardviaene/k8s-demo@sha256:2c050f462f5d0b3a6430e7869bcdfe6ac48a447a89da79a56d0ef61460c7ab9e
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 May 2018 10:46:39 +0200
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:nodejs-port/ delay=15s timeout=30s period=10s #success=1 #failure=3
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
  Normal  Scheduled              44s   default-scheduler  Successfully assigned helloworld-deployment-77664648b9-2gh2s to minikube
  Normal  SuccessfulMountVolume  43s   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-q7xb8"
  Normal  Pulling                43s   kubelet, minikube  pulling image "wardviaene/k8s-demo"
  Normal  Pulled                 39s   kubelet, minikube  Successfully pulled image "wardviaene/k8s-demo"
  Normal  Created                39s   kubelet, minikube  Created container
  Normal  Started                39s   kubelet, minikube  Started container
```

Note the `Liveness` line which indicates that a healthcheck is setup.
