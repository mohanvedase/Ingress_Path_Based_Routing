# Ingress_Path_Based_Routing
This project provides a simple example of how to implement path-based routing in Kubernetes using Nginx Ingress. It's a great starting point for understanding how to manage multiple microservices under a single domain.

## Architecture
![Demo-project - Page 1 (1)](https://github.com/user-attachments/assets/dd3b012f-76ad-48f9-9d91-37aad71d6553)

# Prerequisites
- We need to install the following before starting the project.

1. **aws-cli**
2. **EKSCTL**
3. **Kubectl**

## Step 1: Create the YAML Manifest files
### Step 1.1 Create Deployment files
- Take Nginx as a base image and create 5 deployment files for the Demo project.
- Modify the docker image with the help of /bin/bash to display as welcome to microservices.
- Deployment Object: Defines the desired state of a set of Pods. This manifest creates a Deployment named "service2".
- Replica Count: Specifies the desired number of Pods for the Deployment. In this case, it's set to 1.
- Pod Template: Describes the configuration for each Pod in the Deployment.
- Container Image: Defines the container image to be used for the Pod. Here, it's using the Nginx image.
- Container Port: Specifies the port exposed by the container. In this case, it's port 80.
- Container Command: Sets the command to be executed inside the container. It's configured to create an index.html file with a welcome message and then start the Nginx server.
- Resource Requests and Limits: Defines the minimum and maximum resource requirements for the container, ensuring it doesn't consume more resources than allocated.
### Step 1.2 Create Service Manifest files
- Defines a network service for a set of Pods.
- Exposes Pods as a single endpoint using a virtual IP.
- In the service files there are different Types to expose the application. they are ClusterIP, NodePort, LoadBalancer, ExternalName.
- Don't mention the type of service by default it takes  as *clusterip*.
- if you want to expose the service as NodePort or LoadBalancer mention in the service as Type= nodeport / LoadBalancer.
- Service Object: Defines a network service for a set of Pods. This manifest creates a Service named "service1".
- Selector: Specifies the label selector to identify which Pods belong to this Service. In this case, Pods with the label "app: service1" will be part of this Service.
- Port: Defines the port on which the Service will listen for traffic. The protocol is TCP, the port is 80 (external port), and the targetPort is 80 (port on the Pod).
- This means that traffic to port 80 of the Service will be forwarded to port 80 of the Pods selected by the selector.
      


## Step 2: Set Up the AWS EKS Environment

- configure the AWS CLI in the computer.
- ![image](https://github.com/user-attachments/assets/a59f3c39-4d1d-4926-bea5-3bb9a760f844)

- Create a Cluster with EKSCTL with the help command.
- Take T2.medium as instance. 
```
  eksctl create cluster --name demo-cluster-7 --region ap-south-1 --version 1.24 --nodegroup-name standard-workers --node-type t2.medium --nodes 2 --nodes-min 1 --nodes-max 2 --managed
```
![image](https://github.com/user-attachments/assets/198ca947-a43f-4409-a9b9-cacba955abcc)

- Update the.Kube-config with the following command
- It Stores configuration for accessing Kubernetes clusters.
- It Contains cluster endpoints, user credentials, and contexts.
- It is used to Update to switch between clusters, users, or namespaces.
 ```
  aws eks --region ap-south-1 update-kubeconfig --name demo-cluster-7
 ```
  ![image](https://github.com/user-attachments/assets/d02f2d48-be93-496f-bbac-1a24e75e1b04)

  ![image](https://github.com/user-attachments/assets/a498fb4f-eef4-4a23-8f17-96a71c32f0ae)

  ![image](https://github.com/user-attachments/assets/36fecf8d-47cc-4429-a29e-313447d0cb30)

  ![image](https://github.com/user-attachments/assets/b87f544d-52af-4032-904d-af0ed01c22bf)

### Commands
#### Deployment Commands
- kubectl create -f microservice_1.yaml
- kubectl create -f microservice_2.yaml
- kubectl create -f microservice_3.yaml
- kubectl create -f microservice_4.yaml
- kubectl create -f microservice_5.yaml
#### Service Commands
- kubectl create -f service1.yaml
- kubectl create -f service2.yaml
- kubectl create -f service3.yaml
- kubectl create -f service4.yaml
- kubectl create -f service5.yaml


## Step 3: Install the Nginx ingress controller in the cluster 
- from the ingress documentation copy the yaml script run in the cluster .
- ingress-nginx is an Ingress controller for Kubernetes using [NGINX](https://www.nginx.org/) as a reverse proxy and load
balancer.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/aws/deploy.yaml
```
![image](https://github.com/user-attachments/assets/79a44544-839d-4a59-b64d-273166c20fec)

![image](https://github.com/user-attachments/assets/d45190d7-ed94-48cf-8bc9-6f1b07fcd906)


## Ingress Manifest Breakdown:

* **API Version:** `networking.k8s.io/v1` - Specifies the API version used for Ingress resources.
* **Kind:** `Ingress` - Defines the type of resource being created (an Ingress).
* **Metadata:**
    * **Name:** `sample-ingress` - Assigns a name to the Ingress resource.
    * **Annotations:** (Optional) Additional information for the Ingress controller.
        * `nginx.ingress.kubernetes.io/rewrite-target: /` - Re-writes incoming requests to remove the path component (useful for root path access).

* **Spec:** Defines the configuration details for the Ingress.
    * **ingressClassName:** `nginx` - Specifies the Ingress controller to be used (likely Nginx Ingress Controller in this case).
    * **Rules:** An array of rules for routing incoming traffic.
        * Each rule has the following properties:
            * **host:** (Optional) The hostname to match for the rule. Empty string (`""`) matches any hostname.
            * **http:** Defines HTTP routing configuration.
                * **paths:** An array of path definitions.
                    * Each path definition has the following properties:
                        * **path:** The path prefix to match incoming requests. 
                            * `/service1` - Matches requests starting with "/service1".
                        * **pathType:** Defines how to match the path.
                            * `Prefix` - Matches any request starting with the specified path.
                        * **backend:** Defines the backend service to route traffic to.
                            * **service:** 
                                * **name:** The name of the Service to target.
                                * **port:** 
                                    * **number:** Specifies the port on the Service to forward traffic to (80 in all cases here).
                    * This Ingress defines multiple paths:
                        * `/service1` - Routes traffic to `service1` on port 80.
                        * `/service2` - Routes traffic to `service2` on port 80.
                        * Similar rules exist for `service3`, `service4`, and `service5`.


**this Ingress creates a single entry point for your application with different access paths. It routes traffic based on the path prefix in the request URL to the corresponding Service.**
**The Interaction Process**
#### *Ingress Creation:*

- We create and apply the Ingress manifest to our Kubernetes cluster. This defines the desired routing rules.
#### *Ingress Controller Watches:*

- The Nginx Ingress Controller constantly monitors for changes in Ingress resources.
- Upon detecting the new Ingress, it processes the configuration.
#### *Configuration Generation:*

- The Ingress controller translates the Ingress rules into Nginx configuration. This involves creating Nginx virtual servers, locations, and proxy configurations to match the Ingress rules.
#### *Nginx Configuration Update:*

- The controller reloads Nginx to apply the new configuration. This typically involves sending a signal to the Nginx process or using configuration hot reloading.
#### *Traffic Routing:*

- Incoming traffic is directed to the Nginx Ingress controller.
- Nginx uses the configured virtual servers and locations to route traffic to the appropriate backend service based on the Ingress rules.

### Creating Elastic Load Balancer (ELB) with Ingress.yaml

- Ingress Manifest: Defines rules for external traffic routing to services.
- Ingress Controller: Watches for Ingress changes and translates rules into configurations.
- Cloud Provider Integration: The Controller interacts with the cloud provider (AWS, GCP, Azure) to create ELB.
- ELB Creation: ELB is created based on Ingress rules, including listeners, target groups, and health checks.
- DNS Resolution: The Cloud provider assigns a public DNS name to the ELB.
- Traffic Routing: Incoming traffic to the ELB is distributed to backend services based on Ingress rules.



![image](https://github.com/user-attachments/assets/0383b5e1-6073-4323-baf0-b75d2b656f8b)
![image](https://github.com/user-attachments/assets/acaf4fe2-9e15-4b46-91a7-4b47adca4b31)

## Step 4: Delete the Resources

- After the demonstration is done delete the Cluster. 

![image](https://github.com/user-attachments/assets/5a9ff650-df1d-4bce-b744-775af44bc8d6)
