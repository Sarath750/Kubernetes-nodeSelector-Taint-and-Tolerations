# kubernetes-nodeSelector-taint-and-tolerations

nodeSelector
-----------
to select particular node in which our pods should run.



alias k=kubectl


k get nodes
----------
ip-192-168-7-130.ec2.internal
ip-192-168-36-131.ec2.internal



to check labels 
---------------
k get nodes --show-labels


creating label in node for applying nodeSelector  (because i want my nginx pods to run on only on ip-192-168-36-131.ec2.internal)
-------------------------------------------------
--> k label nodes ip-192-168-36-131.ec2.internal app=nginx-node
(now pods are running only in ip-192-168-36-131.ec2.internal)


nginx.yml with nodeSelector
---------------------------
------------
vi nginx.yml
------------
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        app: nginx-node
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80



for deleting created label(once nodeSlector removed from deployment.yml file, delete created label as well with below mentioned command)
--------------------------
kubectl label nodes ip-192-168-36-131.ec2.internal app-  

nginx.yml without nodeSelector
------------------------------
------------
vi nginx.yml
------------
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80







Taint and Toleration
--------------------
to add taint on particular-node
-------------------------------
k taint nodes ip-192-168-36-131.ec2.internal app=nginx-taint:NoExecute   
(now all the nginx 5 pods will be running in ip-192-168-7-130.ec2.internal and also all the pods which are running earlier in ip-192-168-36-131.ec2.internal will also be removed and be moved to ip-192-168-7-130.ec2.internal, because taint has been applied on ip-192-168-36-131.ec2.internal)


vi springboot.yml
-----------------
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-deployment
  labels:
    app: springboot
spec:
  replicas: 5
  selector:
    matchLabels:
      app: springboot
  template:
    metadata:
      labels:
        app: springboot
    spec:
	tolerations:
      - key: "app"
        operator: "Equal"
        value: "nginx-taint"
        effect: "NoExecute"
      containers:
      - name: springboot
        image: sarath750/springboot-hello:v1
        ports:
        - containerPort: 8080


Now taint is on node ip-192-168-36-131.ec2.internal, so if we want to deploy our springboot application in that tainted-node we need to add tolerations in deployment.yml file. So, now our springboot application will run on both nodes ip-192-168-36-131.ec2.internal, ip-192-168-7-130.ec2.internal.



Now i want my springboot application to run only on node ip-192-168-36-131.ec2.internal, so i need to add label for that node and need to add nodeSelector in deployment.yml

 k label nodes ip-192-168-36-131.ec2.internal app=nginx-node

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-deployment
  labels:
    app: springboot
spec:
  replicas: 5
  selector:
    matchLabels:
      app: springboot
  template:
    metadata:
      labels:
        app: springboot
    spec:
      nodeSelector:
        app: nginx-node
	tolerations:
      - key: "app"
        operator: "Equal"
        value: "nginx-taint"
        effect: "NoExecute"
      containers:
      - name: springboot
        image: sarath750/springboot-hello:v1
        ports:
        - containerPort: 8080


to remove taint from that node
------------------------------
kubectl taint nodes ip-192-168-36-131.ec2.internal app=nginx-taint:NoSchedule-
