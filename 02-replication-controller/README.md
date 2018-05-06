# Replication controller

A replication controller which ensures that 2 pods of the helloworld app are always running.

Create the replication controller :

```shell
$ kubectl create -f ./helloworld-controller.yml
replicationcontroller "helloworld-controller" created
```

Wait for it to spin up, you can check the advancement with the command `kubectl describe rc helloworld`

You should have 2 pods running :

```shell
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-l6vlw   1/1       Running   0          6s
helloworld-controller-qj9xr   1/1       Running   0          6s
```

Test to delete a pod :

```shell
$ kubectl delete pod helloworld-controller-l6vlw
pod "helloworld-controller-l6vlw" deleted
```

You see the replication controller in action, the pod you have deleted is terminating and the controller spins up a new pod to replace it automatically.

```shell
$ kubectl get pods
NAME                          READY     STATUS              RESTARTS   AGE
helloworld-controller-l6vlw   1/1       Terminating         0          2m
helloworld-controller-qj9xr   1/1       Running             0          2m
helloworld-controller-tl4vm   0/1       ContainerCreating   0          3s
```

You can increase the number of pod you want the replication controller to start :

```shell
$ kubectl scale --replicas=4 -f helloworld-controller.yml
replicationcontroller "helloworld-controller" scaled
```

And it spins up 2 additionnal pods :

```shell
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
helloworld-controller-2n5sw   1/1       Running   0          42s
helloworld-controller-qj9xr   1/1       Running   0          5m
helloworld-controller-tl4vm   1/1       Running   0          2m
helloworld-controller-vbw5c   1/1       Running   0          42s
```

We used the `scale` command with the replication controller configuration file, but you can update the configuration directly in your cluster :

```shell
$ kubectl scale --replicas=1 rc/helloworld-controller
replicationcontroller "helloworld-controller" scaled
$ kubectl get pods
NAME                          READY     STATUS        RESTARTS   AGE
helloworld-controller-2n5sw   1/1       Terminating   0          2m
helloworld-controller-qj9xr   1/1       Running       0          7m
helloworld-controller-tl4vm   1/1       Terminating   0          4m
helloworld-controller-vbw5c   1/1       Terminating   0          2m
```

Cleanup :

```shell
$ kubectl delete rc helloworld-controller
replicationcontroller "helloworld-controller" deleted
```