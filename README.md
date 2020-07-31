# Horizontal Pod Autoscaler

The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics). Note that Horizontal Pod Autoscaling does not apply to objects that can't be scaled, for example, DaemonSets.
The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by the user.


### Step 1: Setup metrix server
```
# git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
# cd  kubernetes-metrics-server
# kubectl create -f .
# kubectl top nodes
# kubectl top pods 
```
How does the Horizontal Pod Autoscaler work? 

### Step 2: Create a deployment with resource limits
```
vi nginx-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
```
# kubectl apply -f nginx-deployment.yaml 
# kubectl edit deployment nginx-deployment
resources
    limits
       cpu “100m”
     requests
         cpu “100m” 
```
```
# kubectl expose deployment nginx-deployment --name=httpd-nodeport --port=80 --type=NodePort
# kubectl get all
```
### Step 3: To create Horizontal Pod Autoscaller
```
# git clone https://github.com/justmeandopensource/kubernetes
# cd kubernetes/yamls
```
```
# vi 10-hpa.yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
Metadata:
  name: nginx
spec:
  maxReplicas: 5
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  targetCPUUtilizationPercentage: 20
```
```
# kubectl apply -f 10-hpa.yaml 
```



### Step 4: Test with Load
```
# kubectl describe hpa nginx
# kubectl top pods
# watch kubectl get all
# kubectl run -it --rm load-generator --image=busybox /bin/sh
# while true; do wget -q -O- <<HOSTNAME OF POD or IP>>; done 
```










 
