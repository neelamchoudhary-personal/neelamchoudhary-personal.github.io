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


## Step 3 
After successfully installing docker and minikube start the minikube 
`minikube start` or if you are looking at specific version of kubernetes  `minikube start --kubernetes-version=v1.18.3`
`minikube dashboard` to view the dashboard of the minikube 

## Step 4 
Confirm if the setup is working fine
`kubectl get po -A`  should return something  
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

Deployment:A Kubernetes deployment is a resource object in Kubernetes that provides declarative updates to applications. A deployment allows you to describe an application’s life cycle, such as which images to use for the app, the number of pods there should be, and the way in which they should be updated. 

Service: An abstract way to expose an application running on a set of Pods as a network service.Kubernetes Pods are created and destroyed to match the state of your cluster. Pods are nonpermanent resources. If you use a Deployment to run your app, it can create and destroy Pods dynamically.

Each Pod gets its own IP address, however in a Deployment, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them "backends") provides functionality to other Pods (call them "frontends") inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?

Enter Services.

## Step 6 
Expose the service by 
`kubectl port-forward service/nginx-service 7080:80`

Ngnix is now available at http://localhost:7080 

Kubectl port-forward allows you to access and interact with internal Kubernetes cluster processes from your localhost. You can use this method to investigate issues and adjust your services locally without the need to expose them beforehand

There you go your first application on kubernetes.







