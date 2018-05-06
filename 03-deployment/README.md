# Deployment

Some useful deployment commands to remember :

* `kubectl get deployments` : get information on current deployments
* `kubectl get rs` : get information on replica sets
* `kubectl get pods --show-labels` : get pods and also show attached labels
* `kubectl rollout status deployment/helloworld-deployment` : get deployment status
* `kubectl set image deploument/helloworld-deployment k8s-demo=k8s-demo:2` Run k8s-demo with the image label version 2
* `kubectl edit deployment/helloworld-deployment` : edit the deployment object
* `kubectl rollout history deployment/helloworld-deployment` : get the rollout history
* `kubectl rollout undo deployment/helloworld-deployment` : rollback to the previous version
* `kubectl rollout undo deployment/helloworld-deployment --to-revision=n` : rollback to any version

Create the deployment :

```shell
$ kubectl create -f helloworld-deployment.yml
deployment.extensions "helloworld-deployment" created
$ kubectl get deployments
NAME                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment   3         3         3            3           27s
$ kubectl get rs
NAME                               DESIRED   CURRENT   READY     AGE
helloworld-deployment-67bd889c49   3         3         3         1m
$ kubectl get pods --show-labels
NAME                                     READY     STATUS    RESTARTS   AGE       LABELS
helloworld-deployment-67bd889c49-6cfbp   1/1       Running   0          1m        app=helloworld,pod-template-hash=2368445705
helloworld-deployment-67bd889c49-fwfrl   1/1       Running   0          1m        app=helloworld,pod-template-hash=2368445705
helloworld-deployment-67bd889c49-h8zxw   1/1       Running   0          1m        app=helloworld,pod-template-hash=2368445705
$ kubectl rollout status deployment/helloworld-deployment
deployment "helloworld-deployment" successfully rolled out
```

Create a service to expose the deployment :

```shell
$ kubectl expose deployment helloworld-deployment --type=NodePort
service "helloworld-deployment" exposed
$ kubectl get service
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
helloworld-deployment   NodePort    10.111.95.23   <none>        3000:31167/TCP   23s
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP          22h
$ kubectl describe service helloworld-deployment
Name:                     helloworld-deployment
Namespace:                default
Labels:                   app=helloworld
Annotations:              <none>
Selector:                 app=helloworld
Type:                     NodePort
IP:                       10.111.95.23
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  31167/TCP
Endpoints:                172.17.0.4:3000,172.17.0.5:3000,172.17.0.6:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

In my case port `31167` has been exposed as a node port.

Get minikube to give you the url to call this service :

```shell
$ minikube service helloworld-deployment --url
http://192.168.99.100:31167
$ curl http://192.168.99.100:31167
Hello World!
```

Rollout a new version of the deployment :

```shell
$ kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
deployment.apps "helloworld-deployment" image updated
$ kubectl rollout status deployment/helloworld-deployment
deployment "helloworld-deployment" successfully rolled out
$ curl http://192.168.99.100:31167
Hello World v2!
$ kubectl rollout history deployment/helloworld-deployment
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```

No change cause because we forgot `--record` option on the initial `kubectl create` command

Rollback to the previous version :

```shell
$ kubectl rollout undo deployment/helloworld-deployment
deployment.apps "helloworld-deployment"
$ kubectl rollout status deployment/helloworld-deployment
deployment "helloworld-deployment" successfully rolled out
$ kubectl get pods
NAME                                     READY     STATUS        RESTARTS   AGE
helloworld-deployment-575b5464bf-f9ffz   1/1       Terminating   0          3m
helloworld-deployment-575b5464bf-fml7s   1/1       Terminating   0          3m
helloworld-deployment-575b5464bf-xc7vw   1/1       Terminating   0          3m
helloworld-deployment-67bd889c49-54hc2   1/1       Running       0          19s
helloworld-deployment-67bd889c49-gjjbv   1/1       Running       0          22s
helloworld-deployment-67bd889c49-h54gk   1/1       Running       0          21s
$ curl http://192.168.99.100:31167
Hello World!
```

Check the history :

```shell
$ kubectl rollout history deployment/helloworld-deployment
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

We only see the last 2 revisions. We can increase this number :

```shell
$ kubectl edit deployment/helloworld-deployment
# Update revisionHistoryLimit to the value 100
deployment.extensions "helloworld-deployment" edited
```

Then let's create some history :

```shell
$ kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
deployment.apps "helloworld-deployment" image updated
$ kubectl rollout history deployment/helloworld-deployment
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
3         <none>
4         <none>

$ kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:1
deployment.apps "helloworld-deployment" image updated
$ kubectl rollout history deployment/helloworld-deployment
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
5         <none>


```

And rollback to revison 3 :

```shell
$ kubectl rollout undo deployment/helloworld-deployment --to-revision=3
deployment.apps "helloworld-deployment"
jobou@maison-linux:~/workspace/kubernetes-examples/03-deployment$ kubectl rollout history deployment/helloworld-deployment
deployments "helloworld-deployment"
REVISION  CHANGE-CAUSE
4         <none>
5         <none>
6         <none>

```

It has created a new revision but rollbacked to the version of the image from revision 3.

Cleanup :

```shell
$ kubectl delete deployment helloworld-deployment
deployment.extensions "helloworld-deployment" deleted
$ kubectl delete service helloworld-deployment
service "helloworld-deployment" deleted
```