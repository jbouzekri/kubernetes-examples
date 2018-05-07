# Kubernetes Examples

## Introduction

This repository contains all the exercices done when I followed the Udemy course [Learn DevOps: The Complete Kubernetes Course](https://www.udemy.com/learn-devops-the-complete-kubernetes-course/). I encourage you to buy and follow this course yourself, it is a really great introduction that covers almost all the tools you need to know to be productive. Of course, it is an introduction course so it does not make you an expert.

Note that a lot of examples comes directly from the [course repository](https://github.com/wardviaene/kubernetes-course).

## Installation

I am using [minikube](https://github.com/kubernetes/minikube) to setup a fully functionnal kubernetes cluster for testing purpose. Follow the installation steps from the [minikube](https://github.com/kubernetes/minikube) documentation.

Once minikube is installed. Spin up a kubernetes cluster using `minikube` :

```shell
minikube start
```

You will need the kubernetes admin tool `kubectl` too. Follow the [installation documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl). Note that if minikube does not find one, it installs it automatically so you may not need to do this step.

`minikube` sets up automatically a kube config file to connect to it :

```shell
$ cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/<username>/.minikube/ca.crt
    server: https://<ip of the kubernetes virtual machine>:<port>
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/<username>jobou/.minikube/client.crt
    client-key: /home/<username>/.minikube/client.key
```

To test it :

```shell
$ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
deployment.apps "hello-minikube" created
# wait a little
$ kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
hello-minikube-c8b6b4fdc-tjdwh   1/1       Running   0          1m
$ kubectl expose deployment hello-minikube --type=NodePort
service "hello-minikube" exposed
$ minikube service hello-minikube --url
http://<the ip of the hello service>:<the port of the hello service>
$ curl http://<the ip of the hello service>:<the port of the hello service>
# in the body, you should see :
-no body in request-
$ minikube stop
```

Cleanup `minikube` :

```shell
$ kubectl delete service hello-minikube
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube
deployment.extensions "hello-minikube" deleted
```

## Summary

* [01 - First app](./01-first-app/README.md)
* [02 - Replication Controller](./02-replication-controller/README.md)
* [03 - Deployment](./03-deployment/README.md)
* [04 - Service](./04-service/README.md)