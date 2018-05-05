# First app

This example runs a first application in the cluster.

First starts the nodejs `pod` :

```
$ kubectl create -f ./helloworld-pod.yml
```

Wait for it to spin up, you can check the advancement with the command `kubectl describe pod nodehelloworld.example.com`

Then we expose the pod to the outside world using the port forwarding kubectl command :

```
$ kubectl port-forward nodehelloworld.example.com 8081:3000
Forwarding from 127.0.0.1:8081 -> 3000
```

```
$ curl localhost:8081
Hello World!
```

The second solution is to use a service :

```
$ kubectl expose pod nodehelloworld.example.com --type=NodePort --name=nodehelloworld-service
```

Use `minikube` command to get the url and port affected to this service :

```
$ minikube service nodehelloworld-service --url
http://<your cluster ip>:<the port used by kubernetes to expose the hello world service>
$ curl http://<your cluster ip>:<the port used by kubernetes to expose the hello world service>
Hello World!
```

You can see that the `kubectl expose pod` command has created a service automatically :

```
$ kubectl get service
...
nodehelloworld-service   NodePort    10.100.133.116   <none>        3000:<the port used by kubernetes to expose the hello world service>/TCP   32s
```

Some useful `kubectl` commands :

* `kubectl attach <pod>` : display the stdout of the pod
* `kubectl exec <pod> -- <command>` : execute `command` in pod
* `kubectl describe <resource typ> <cluster resource>` : get description of the resource

Examples :

```
$ kubectl describe service nodehelloworld-service
Name:                     nodehelloworld-service
Namespace:                default
Labels:                   app=helloworld
Annotations:              <none>
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.100.133.116
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  30957/TCP
Endpoints:                172.17.0.4:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # telnet 172.17.0.4 3000
GET /

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
ETag: W/"c-7Qdih1MuhjZehB6Sv8UNjA"
Date: Sat, 05 May 2018 20:25:35 GMT
Connection: close

Hello World!Connection closed by foreign host
```

*Note : port `3000` can be used locally by container in the same pod or by using the `endpoint` as we did in the previous example and `30957` from outside*

Cleanup :

```
$ kubectl delete service nodehelloworld-service
service "nodehelloworld-service" deleted
$ kubectl delete pod busybox
pod "busybox" deleted
$ kubectl delete pod nodehelloworld.example.com
pod "nodehelloworld.example.com" deleted
```