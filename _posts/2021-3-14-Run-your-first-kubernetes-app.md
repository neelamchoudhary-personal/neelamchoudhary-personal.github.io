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







