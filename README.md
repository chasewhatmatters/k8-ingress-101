# k8-ingress-101

---

## Create NameSpace and Deploy APP

kubectl create namespace dev
namespace/dev created

kubectl apply -f hello-app.yaml
deployment.apps/hello-app created

kubectl get deploy -n dev
NAME READY UP-TO-DATE AVAILABLE AGE
hello-app 3/3 3 3 23s

---

## Create Service

kubectl apply -f hello-app-service.yaml
kubectl apply -f hello-app-service-node.yaml

C:\Users\ajay\_\github_projects\k8-ingress-101>kubectl describe svc hello-service-np -n dev
Name: hello-service-np
Namespace: dev
Labels: app=hello
Annotations: <none>
Selector: app=hello
Type: NodePort
IP Family Policy: SingleStack
IP Families: IPv4
IP: 10.99.198.4
IPs: 10.99.198.4
LoadBalancer Ingress: localhost
Port: <unset> 80/TCP
TargetPort: 8080/TCP
NodePort: <unset> 30001/TCP
Endpoints: 10.1.0.70:8080,10.1.0.71:8080,10.1.0.72:8080
Session Affinity: None
External Traffic Policy: Cluster
Events: <none>

---

## List all Deployment & Svc

kubectl get all -n dev  
NAME READY STATUS RESTARTS AGE
pod/hello-app-78f957775f-s47dz 1/1 Running 0 37m
pod/hello-app-78f957775f-xmlf7 1/1 Running 0 37m
pod/hello-app-78f957775f-zzbkv 1/1 Running 0 37m

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/hello-service ClusterIP 10.103.41.80 <none> 80/TCP 36m
service/hello-service-np NodePort 10.99.198.4 <none> 80:30001/TCP 14m >> We will use this sevrice

NAME READY UP-TO-DATE AVAILABLE AGE
deployment.apps/hello-app 3/3 3 3 37m

NAME DESIRED CURRENT READY AGE
replicaset.apps/hello-app-78f957775f 3 3 3 37m

---

## Connect Using Curl and Service

$ curl -s http://localhost:30001/
Hello, world!
Version: 2.0.0
Hostname: hello-app-78f957775f-zzbkv

---

## Create Ingres

kubectl apply -f ingress-np.yaml

kubectl describe ingress test-ingress-np -n dev
Name: test-ingress-np
Namespace: dev
Address:
Default backend: default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
Host Path Backends

---

demo.apps.cwm.com
/ hello-service-np:80 (10.1.0.70:8080,10.1.0.71:8080,10.1.0.72:8080)
Annotations: <none>
Events: <none>

---

## Update local windows hosts file C:\Windows\System32\drivers\hosts ( start cmd as administrator and ien in notepad)

# Added by Ajay for Demo on NodePort

127.0.0.1 demo.apps.cwm.com

---

## Connect Using Curl and Service, but here due to ingress you can manipulate Path

$ curl -s http://demo.apps.cwm.com:30001/
Hello, world!
Version: 2.0.0
Hostname: hello-app-78f957775f-s47dz

---

## Connect Using Curl but here due to ingress you can manipulate Path

UpdateIngress and add /app path prefix

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: test-ingress-np
namespace: dev
spec:
ingressClassName: nginx
rules:

- host: "demo.apps.cwm.com"
  http:
  paths: - pathType: Prefix  
   path: "/"
  backend:
  service:
  name: hello-service-np
  port:
  number: 80 - pathType: Prefix  
   path: "/app"
  backend:
  service:
  name: hello-service-np
  port:
  number: 80

---

## Delete and Recreate Ingress Object

kubectl delete ingress test-ingress-np -n dev  
ingress.networking.k8s.io "test-ingress-np" deleted

kubectl apply -f ingress-np.yaml  
ingress.networking.k8s.io/test-ingress-np created

---

## Connect Using Curl and access same APP on 2 differemnt path

$ curl -s http://demo.apps.cwm.com:30001/app
Hello, world!
Version: 2.0.0
Hostname: hello-app-78f957775f-xmlf7

$ curl -s http://demo.apps.cwm.com:30001/app
Hello, world!
Version: 2.0.0
Hostname: hello-app-78f957775f-zzbkv

$ curl -s http://demo.apps.cwm.com:30001
Hello, world!
Version: 2.0.0
Hostname: hello-app-78f957775f-s47dz
