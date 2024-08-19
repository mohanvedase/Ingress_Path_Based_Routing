# Ingress_Path_Based_Routing
This project provides a simple example of how to implement path-based routing in Kubernetes using Nginx Ingress. It's a great starting point for understanding how to manage multiple microservices under a single domain.
# Prerequisites
- We need to install the below before starting the project.

1. **aws-cli**
2. **EKSCTL**
3. **Kubectl**

## Step 1: create the deployment yaml  files
- Take Nginx as base image create 5 deployment files for the Demo project .
- Modify the docker image with the help of /bin/bash to display as welcome to microservices.
- create the service files to run on port no 80
- Dont mention the type of service by default it take  as *clusterip*.
- if you want to expose the service as NodePort or LoadBalancer mention in the service as Type= nodeport / LoadBalancer    


## Step 1: Set Up the AWS EKS Environment

- configure the AWS CLI in the computer . Create a Cluster with EKSCTL with the help command.
- Take T2.medium as instance . 
```
  eksctl create cluster --name demo-cluster-7 --region ap-south-1 --version 1.24 --nodegroup-name standard-workers --node-type t2.medium --nodes 2 --nodes-min 1 --nodes-max 2 --managed
```
![image](https://github.com/user-attachments/assets/198ca947-a43f-4409-a9b9-cacba955abcc)

- Update the .Kube-config with
  ```
  aws eks --region ap-south-1 update-kubeconfig --name demo-cluster-7
  ```
  ![image](https://github.com/user-attachments/assets/d02f2d48-be93-496f-bbac-1a24e75e1b04)

## Install the Nginx ingress controller in the cluster 
- from the ingress documentation copy the yaml script run in the cluster .
- ingress-nginx is an Ingress controller for Kubernetes using [NGINX](https://www.nginx.org/) as a reverse proxy and load
balancer.
  ```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/aws/deploy.yaml
  ```


