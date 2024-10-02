## Service
Service is the method for exposing a network application that is running as one or more Pods in the cluster.

Let's create a service to expose the port to access the nginx pod we created earlier. Remember labels added will be the key here.
```
$ kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service -n kube-tut
service/nginx-service exposed
$ kgs -n kube-tut -o wide
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
nginx-service   NodePort   172.17.51.181   <none>        80:32125/TCP   8s    demo=true,env=dev
```

Now we can access the pod from the outside:
```
$ curl http://localhost:32125/
```
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```