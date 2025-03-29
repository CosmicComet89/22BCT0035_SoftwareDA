# EDULA VENKATA SWAPNIL REDDY 22BCT0035_SoftwareDA
Repo containing all files for implementation and deployment of a simple flask application using docker and kubernetes


# Deploying and Managing a Scalable Web Application Using Kubernetes

## Assignment Overview
This guide covers the step-by-step process to set up, deploy, and manage a web application using Kubernetes. The deployment includes features like auto-scaling, rolling updates, persistent storage, and logging.

## 1. Setup Kubernetes Cluster
### Local Deployment:
- Install [Minikube](https://minikube.sigs.k8s.io/docs/) and start a cluster:
  ```sh
  minikube start --nodes=2
  ```

### Cloud Deployment:
- Use Google Kubernetes Engine (GKE), Amazon EKS, or Azure AKS.
- Ensure the cluster has at least two worker nodes.

## 2. Deploy a Web Application
### Steps:
1. Choose a simple Node.js or Python Flask-based web application.
2. Containerize the application using Docker.
3. Push the container image to Docker Hub or a private registry:
   ```sh
   docker build -t <your-dockerhub-username>/flask-app:v1 .
   docker push <your-dockerhub-username>/flask-app:v1
   ```

## 3. Create Kubernetes Resources
### Deployments:
```yaml
defaultDeployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-container
        image: <your-dockerhub-username>/flask-app:v1
        ports:
        - containerPort: 5000
```
Apply it:
```sh
kubectl apply -f deployment.yaml
```

### Services:
```yaml
defaultService.yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: NodePort
```
Apply it:
```sh
kubectl apply -f service.yaml
```

### ConfigMaps & Secrets:
- Store environment variables securely.
```sh
kubectl create configmap app-config --from-literal=ENV=production
kubectl create secret generic app-secret --from-literal=SECRET_KEY=mysecret
```

## 4. Implement Auto-scaling
Enable Horizontal Pod Autoscaler:
```sh
kubectl autoscale deployment flask-app --cpu-percent=50 --min=2 --max=5
```
Check autoscaler:
```sh
kubectl get hpa
```

## 5. Implement Persistent Storage (Optional)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: flask-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
Apply it:
```sh
kubectl apply -f pvc.yaml
```

## 6. Rolling Updates & Rollbacks
### Rolling Update:
```sh
kubectl set image deployment/flask-app flask-container=<your-dockerhub-username>/flask-app:v2
```
Verify:
```sh
kubectl rollout status deployment/flask-app
```

### Rollback:
```sh
kubectl rollout undo deployment/flask-app
```

## 7. Logging
View logs:
```sh
kubectl logs <pod-name>
```

## Testing Scenarios
### 1. Application Availability Tests
```sh
kubectl get services
curl http://<minikube-ip>:<NodePort>
```

### 2. Scaling Tests
```sh
kubectl get hpa
kubectl run --rm -it --image=busybox stress-test -- /bin/sh
while true; do wget -q -O- http://<service-ip>:5000; done
```

### 3. Rolling Update & Rollback Test
```sh
kubectl set image deployment/flask-app flask-container=newimage:v2
kubectl rollout undo deployment/flask-app
```

### 4. Pod Failure and Self-Healing Test
```sh
kubectl delete pod <pod-name>
kubectl get pods -w
```

### 5. Persistent Storage Test (If Implemented)
```sh
kubectl delete pod <pod-name>
kubectl get pods -w
```

### 6. Logging Test
```sh
kubectl logs <pod-name>
```



