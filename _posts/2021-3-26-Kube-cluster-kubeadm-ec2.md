---
layout: post
title: Kube cluster setup with kubeadm in ec2
---

# Setup Kube cluster in ec2 with kubeadm

If you want to try to setup a kube cluster on aws with kubeadm.
So I had previous experience of setting local cluster using minikube or microk3s and in production used AKS and Docker EE kubernetes cluster.
As part of one of job I needed to setup the cluster on AWS with kubeadm on ec2 with out kops. 
I thoughts it should not be too difficult I have setup the cluster locally and used them good 2-3 years.
During the setup I ran in multiple problem which left a technical itch to under stand this all.
After spending an entire day I managed. I hope this helps someone who need the setup in particular way due organization requirements 
This will make you appreciate the complexity of setting up a cluster.
Disclaimer:: This was setup with objective trying to setup the cluster. It never ran any production workload's

### Prerequisite
1. AWS account

## Step 1
Create a role in aws for the ec2 for the instance.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:*",
                "elasticloadbalancing:*",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:BatchGetImage",
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:UpdateAutoScalingGroup"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}

```

## Step 2 
Create 2 instance of ec2 with RHEL-8. 
Minimum required spec for running master and worker on the day of publishing is 2vcpu and 2 GB RAM.

1. Please select appropriate instance. 
![Instance](/images/Instance.png "Instance")
2. When creating the ec2 instance please set the ec2 role to ec2 instance
3. Setup security group. I have tested with wides version of security which not appropriate for production
![Security Group](/images/Security-group.png "Security Group")
4. Setup the tags. It is very important setup. The below are for master, same for the worker except kube-master would become kube-worker
![Tags](/images/tags.png "tags") 

## Step 3 - Setup Master
1. Install docker 
    * Add repo to add docker 
    ```
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    ```
    *  install docker 
    ```
    sudo yum install docker-ce
    ```
    * enable docker
    ```
    sudo systemctl enable --now docker 
    ```
    * add current user to docker group. as some of the docker command need alleviated access.
    ```
    sudo usermod -a -G docker ec2-user
    ```
    * restart docker after changes 
    ```
    sudo systemctl restart docker
    ```
    * enable service 
    ```
    sudo systemctl enable docker.service
    ```
    * validation - At this point you should be able to run this command with normal user  
    ```
    docker ps 
    ```
    * troubleshooting command - If you are facing the any issue with above commands have a look at
    ```$xslt
    sudo dockerd
    ```
    If issue is related to iptables - running the following commands
    ```
    sudo systemctl stop firewalld - is already stop not present continue with step
    sudo systemctl disable firewalld - is already stop not present continue with step
    sudo yum install iptables-services
    sudo systemctl start iptables
    sudo systemctl enable iptables
    sudo systemctl status iptables
    sudo iptables -nvL - validation everything is working as expected
    ``` 
    if you cannot run the docker ps as normal user trying running with sudo. 
    If that run the setup to run the docker as normal user may need some additional troubleshooting        
2. Install - kubeadm, kubelet 
   * Add the repo where the kuberneted packages are 
   ```
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
    ``` 
   * install the - kubeadm , kubelet
   ```
    sudo yum install -y kubelet kubeadm
    ``` 
   * install - kubernetes-cni . If it ask you downgrade say yes.
    ```
    sudo yum install -y kubernetes-cni
    ```
   * reload daemon 
   ```
    sudo systemctl daemon-reload  
    ```
   * start kubelet 
   ```
    sudo systemctl start kubelet
    ```  
   * enable kubelet service
   ```
    sudo systemctl enable kubelet.service
    ```
   * validation - 
   ```$xslt
    sudo systemctl status kubelet  
   ``` 
   * view logs - for troubleshooting 
   ```
    sudo journalctl -u kubelet
   ```
3. Now all the basic are setup we are ready to initialize the cluster
    * generate token - keep it handy for next step
    ```
    sudo kubeadm token generate
    ```    
    * create config file for the initialization - kube-master.yaml
    ```
    apiVersion: kubeadm.k8s.io/v1beta1
    kind: InitConfiguration
    bootstrapTokens:
    - groups:
      - "system:bootstrappers:kubeadm:default-node-token"
      token: "<token from above step>"
      ttl: "0s"
      usages:
      - signing
      - authentication
    nodeRegistration:
      name: "<master private dns>"
      kubeletExtraArgs:
        cloud-provider: "aws"
    ---
    apiVersion: kubeadm.k8s.io/v1beta1
    kind: ClusterConfiguration
    kubernetesVersion: "v1.18.17"
    apiServer:
    #  timeoutForControlPlane: 10m0s
      certSANs:
      - "<private ip address of master>"
      extraArgs:
        cloud-provider: "aws"
    clusterName: kubernetes # same name as one defined in the tag for the ec2 instances
    controlPlaneEndpoint: "<private ip address of master>:6443"
    controllerManager:
      extraArgs:
        cloud-provider: "aws"
        allocate-node-cidrs: "false"
    ```  
    * run the init - keep the join token generated
    ```
    sudo kubeadm init --config kube-master.yaml --ignore-preflight-errors=all
    ```
    * troubleshooting - command 
    ```
    journalctl -xeu kubelet
    docker ps shoudl show the kube-api and other pods running 
    ```
    * run kubectl get nodes - if returns error's saying 'Connection refused'
    ```
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    * access to user - 'admin'
    ```
    sudo kubectl create clusterrolebinding admin-cluster-binding --clusterrole=cluster-admin --user=admin
    ```
    * setup network 
    ```
    curl https://docs.projectcalico.org/manifests/calico.yaml -O
    kubectl apply -f calico.yaml
    ```
    * setup other mandatory add on's
    ```
     kubectl apply -f https://aws-quickstart.s3.amazonaws.com/quickstart-vmware/scripts/dashboard.yaml
     kubectl apply -f https://aws-quickstart.s3.amazonaws.com/quickstart-vmware/scripts/default.storageclass.yaml
     kubectl apply -f https://aws-quickstart.s3.amazonaws.com/quickstart-vmware/scripts/network-policy.yaml
    ```
    * sudo systemctl restart kubelet    
    
## Step 4 - Setup Worker node
1. Same as master
2. Same as master
3. Initialize the node as worker
    * if misplaced the generated join token
    ```
    sudo kubeadm token create --print-join-command
    ``` 
    * run the join command
    ```
    kubeadm join <from above command> --token <from the above command> --discovery-token-ca-cert-hash <from the above comman>
    ```
    
## Step 5 - Set PVC 
1. create pvc.yaml 
```
apiVersion: v1
kind: PersistentVolumeClaim````
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
```
2. apply 
```
kubectl apply -f pvc.yaml
``` 







