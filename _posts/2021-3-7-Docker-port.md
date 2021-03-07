---
layout: post
title: Docker port
---

A docker container does not publish any of its ports to the outside world. In order to expose the service running on port in the docker container 


### Docker 
Use the publish flag `-p 8080:80`	Map TCP port 80 in the container to port 8080 on the Docker host.

### Docker Compose

```
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
 db:
   image: postgres
```
Map port on 8080 on container to Docker host 


### Kubernetes
```
 spec:
      containers:
        - name: api
          image: <image-link>
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
          volumeMounts:
          - name: nginx-certs
            mountPath: /nginx_certs/
            readOnly: true
         
```
config of containerPort defines the port on which the service running on 


### DockerFile 
EXPOSE instruction in the dockerfile **does not actually expose the port**. It is a way of documention for the image builder to the image user.

