---
layout: post
title: Run your first kubernetes application
---

# Run your first kubernetes app - with minikube

In this post will show you all the steps to run a app running as docker container running on local kuberentes cluster via minikube.
If you looking for running kuberenets workload on docker desktop click here
If you need to decide if you want to run the kubernetes workload on minikube or docker desktop click here

### Prerequisite
1. Docker image
2. Port number on which the service is running in docker container

## Step 1
Install docker desktop depending on your os - https://docs.docker.com/docker-for-windows/install/

## Step 2 
Install minikube https://minikube.sigs.k8s.io/docs/start/
x

## Step 3 
After successfully installing docker and minikube start the minikube 
`minikube start` or if you are looking at specfic version of kubernetes  `minikube start --kubernetes-version=v1.18.3`
`minikube dashboard` to view the dashboard of the minikube 

## Step 4 
Confirm if the setup is working fine
`kubectl get po -A`  should return soemthing  
```
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-9l9mm            1/1     Running   1          15h
kube-system   etcd-minikube                      1/1     Running   1          15h
kube-system   kube-apiserver-minikube            1/1     Running   1          15h
kube-system   kube-controller-manager-minikube   1/1     Running   1          15h
kube-system   kube-proxy-lxdz6                   1/1     Running   1          15h
kube-system   kube-scheduler-minikube            1/1     Running   1          15h
kube-system   storage-provisioner                1/1     Running   2          15h
```

## Step 5 
Setup a simple deployment of ngnix with 3 replica along with a service 
```
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

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

Deployment:

Service:

## Step 6 
Expose the service by 
`kubectl port-forward service/nginx-service 7080:80`

Ngnix is now available at http://localhost:7080 

There you go your first application on kubernetes.







