# Web UI

Web UI is accessible from `https://<kubernetes-master>/ui`

if you cannot accessit, you can install it manually using :

```shell
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```

If a password is asked, you can retrieve the password with `kubectl config view`

```shell
$ minikube dashboard --url
http://192.168.99.100:30000
```