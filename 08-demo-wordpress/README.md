# Demo wordpress

A not fully functionnal wordpress setup but with secrets.

```shell
$ kubectl create -f ./wordpress-secrets.yml
secret "wordpress-secrets" created
$ kubectl create -f ./wordpress-single-deployment-no-volumes.yml
deployment.extensions "wordpress-deployment" created
$ kubectl get pods
NAME                                    READY     STATUS              RESTARTS   AGE
wordpress-deployment-6f47769b85-bqgt4   0/2       ContainerCreating   0          4s
$ kubectl create -f ./wordpress-service.yml
service "wordpress-service" created
$ minikube service wordpress-service --url
Waiting, endpoint for service is not ready yet...
Waiting, endpoint for service is not ready yet...
http://192.168.99.100:31001
$ curl http://192.168.99.100:31001/wp-admin/install.php
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en-US" xml:lang="en-US">
<head>
	<meta name="viewport" content="width=device-width" />
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="robots" content="noindex,nofollow" />
	<title>WordPress &rsaquo; Installation</title>
	<link rel='stylesheet' id='buttons-css'  href='http://192.168.99.100:31001/wp-includes/css/buttons.min.css?ver=4.9.5' type='text/css' media='all' />
<link rel='stylesheet' id='install-css'  href='http://192.168.99.100:31001/wp-admin/css/install.min.css?ver=4.9.5' type='text/css' media='all' />
<link rel='stylesheet' id='dashicons-css'  href='http://192.168.99.100:31001/wp-includes/css/dashicons.min.css?ver=4.9.5' type='text/css' media='all' />
</head>
<body class="wp-core-ui language-chooser">
...
</body>
</html>
```

You can open your browser and go to the service URL to complete the wordpress installation. Remember that this pod does not use volumes for the wordpress installation or database so everything you do will be lost if you restart the pod.

Cleanup:

```shell
$ kubectl delete deployment wordpress-deployment
deployment.extensions "wordpress-deployment" deleted
$ kubectl delete service wordpress-service
service "wordpress-service" deleted
$ kubectl delete secret wordpress-secrets
secret "wordpress-secrets" deleted
```