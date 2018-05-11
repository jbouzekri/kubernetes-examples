# Secret

Secrets provides a way in Kubernetes to distribute credentials, keys, passwords or "secret" data to the pods.

Note : *external secret vault can be used instead of the default secret mecanism*

Secrets can be used :

* as environment variables
* as a file in a pod (via a volume mounted in a container)
* as external images (from a private image registry)

Generate secrets using files :

```shell
$ echo -n "root" > ./username.txt
$ echo -n "password" > ./password.txt
$ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
secret "db-user-pass" created
```

Or store an SSH key or certificates :

```shell
kubectl create secret generic ssl-certificate --from-file=ssh-private-key=~/.ssh/id_rsa --ssl-certs=ssl-certs=mysslcert.crt
```

Or using yaml definitions :

```shell
$ echo -n "root" | base64
cm9vdA==
$ echo -n "password" | base64
cGFzc3dvcmQ=
$ cat ./secrets-db-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: cm9vdA==
  password: cGFzc3dvcmQ=
$ kubectl create -f ./secrets-db-secret.yml
secret "db-secret" created
```

And in the pod, exposes the secret as environment variables :

```yaml
spec:
  containers:
    - name: k8s-demo
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: SECRET_PASSWORD
          [...]
```

Or in a mounted file

```yaml
spec:
  containers:
    - name: k8s-demo
      volumeMounts:
        - name: credvolume
          mountPath: /etc/creds
          readOnly: true
  volumes:
    - name: credvolume
      secret:
        secretName: db-secret
```

As a file, secrets will be stored in :

* `/etc/creds/username`
* `/etc/creds/password`

Example :

```shell
$ kubectl create -f ./helloworld-secrets.yml
secret "db-secrets" created
$ kubectl create -f ./helloworld-secrets-volumes.yml
deployment.extensions "helloworld-deployment" created
$ kubectl exec helloworld-deployment-6d4c6b79d9-5lthj -it -- /bin/bash
$ ls /etc/creds/
password  username
$ cat /etc/creds/username
root
$ cat /etc/creds/password
password
```

Cleanup :

```shell
$ kubectl delete deployment helloworld-deployment
deployment.extensions "helloworld-deployment" deleted
$ kubectl delete secret db-secrets
secret "db-secrets" deleted
```