### Container Orchestration with Kubernetes

Overall, in this lesson we will explore:

Docker for Application Packaging
Container Orchestration with Kubernetes
Kubernetes Resources
Declarative Kubernetes Manifests


VMs
In the past years, VMs (Virtual Machines) were the main mechanism to host an application. VMs encapsulate the code, configuration files, and dependencies necessary to execute the application.

containers
There was a clear need to optimize the usage of the available infrastructure. As a result, the virtualization of the Operating System was introduced. This prompted the appearance of containers, which represent the bare minimum an application requires for a successful execution e.g. code, config files, and dependencies. By default, there is a better usage of available infrastructure.

Dockerfile
A Dockerfile is a set of instructions used to create a Docker image. 

Dockerfile - set of instructions used to create a Docker image
Docker image - a read-only template used to spin up a runnable instance of an application
Docker registry - a central mechanism to store and distribute Docker images


The following snippet showcases the Dockerfile for the application:

FROM golang:alpine

WORKDIR /go/src/app

ADD . .

RUN go build  -o helloworld

EXPOSE 6111

CMD ["./helloworld"]

To build tag and push the image to DockerHub, use the following commands:

# build the image
docker build -t go-helloworld .

# tag the image
docker tag go-helloworld pixelpotato/go-helloworld:v1.0.0

# push the image
docker push pixelpotato/go-helloworld:v1.0.0

### Kubernetes - The Container Orchestrator Framework

A container orchestrator framework is capable to create, manage, configure thousands of containers on a set of distributed servers while preserving the connectivity and reachability of these containers. In the past years, multiple tools emerged within the landscape to provide these capabilities, including Docker Swarm, Apache Mesos, CoreOS Fleet, and many more. However, Kubernetes took the lead in defining the principles of how to run containerized workloads on a distributed amount of machines.


This is because Kubernetes solutionizes portability, scalability, resilience, service discovery, extensibility, and operational cost of containers.

Kubernetes Architecture
A Kubernetes cluster is composed of a collection of distributed physical or virtual servers. These are called nodes. Nodes are categorized into 2 main types: master and worker nodes. The components installed on a node, determine the functionality of a node, and identifies it as a master or worker node.

The suite of master nodes, represents the control plane, while the collection of worker nodes constructs the data plane.



Kubeconfig
To access a Kubernetes cluster a kubeconfig file is required. 

A Kubeconfig file has 3 main distinct sections:
Cluster - encapsulates the metadata for a cluster, such as the name of the cluster, API server endpoint, and certificate authority used to check the identity of the user.
User - contains the user details that want access to the cluster, including the user name, and any authentication metadata, such as username, password, token or client, and key certificates.
Context - links a user to a cluster. If the user credentials are valid and the cluster is up, access to resources is granted. Also, a current-context can be specified, which instructs which context (cluster and user) should be used to query the cluster.


Here is an example of a kubeconfig file:

apiVersion: v1
# define the cluster metadata 
clusters:
- cluster:
    certificate-authority-data: {{ CA }}
    server: https://127.0.0.1:63668
  name: udacity-cluster
# define the user details 
users:
# `udacity-user` user authenticates using client and key certificates 
- name: udacity-user
  user:
    client-certificate-data: {{ CERT }}
    client-key-data: {{ KEY }}
# `green-user` user authenticates using a token
- name: green-user
  user:
    token: {{ TOKEN }}
# define the contexts 
contexts:
- context:
    cluster: udacity-cluster
    user: udacity-user
  name: udacity-context
# set the current context
current-context: udacity-context



Kubernetes Resources Part 1

Kubernetes provides a rich collection of resources that are used to deploy, configure, and manage an application. Some of the widely used resources are:

Pods - the atomic element within a cluster to manage an application
Deployments & ReplicaSets - oversees a set of pods for the same application
Services & Ingress - ensures connectivity and reachability to pods
Configmaps & Secrets - pass configuration to pods
Namespaces - provides a logical separation between multiple applications and their resources
Custom Resource Definition (CRD) - extends Kubernetes API to support custom resources


A pod is the anatomic element within a cluster that provides the execution environment for an application. 


Deployments and ReplicaSets

To deploy an application to a Kubernetes cluster, a Deployment resource is necessary. A Deployment contains the specifications that describe the desired state of the application. Also, the Deployment resource manages pods by using a ReplicaSet. A ReplicaSet resource ensures that the desired amount of replicas for an application are up and running at all times.



New terms
Pod - smallest manageable uint within a cluster that provides the execution environment for an application
ReplicaSet - a mechanism to ensure a number of pod replicas are up and running at all times
Deployment - describe the desired state of the application to be deployed



Kubernetes Resources Part 2

Services
A Service resource provides an abstraction layer over a collection of pods running an application. A Service is allocated a cluster IP, that can be used to transfer the traffic to any available pods for an application.


There are 3 widely used Service types:
ClusterIP - exposes the service using an internal cluster IP. If no service type is specified, a ClusterIP service is created by default.
NodePort - expose the service using a port exposed on all nodes in the cluster.
LoadBalancer - exposes the service through a load balancer from a public cloud provider such as AWS, Azure, or GCP. This will allow the external traffic to reach the services within the cluster securely.


Ingress
To enable the external user to access services within the cluster an Ingress resource is necessary. 


New terms
Service - an abstraction layer over a collection of pods running an application
Ingress - a mechanism to manage the access from external users and workloads to the services within the cluster



Kubernetes Resources Part 3

Application Configuration And Context

Kubernetes has 2 resources to pass data to an application: Configmaps and Secrets.


ConfigMaps
ConfigMaps are objects that store non-confidential data in key-value pairs. A Configmap can be consumed by a pod as an environmental variable, configuration files through a volume mount, or as command-line arguments to the container.
To create a Configmap use the kubectl create configmap command, with the following syntax:



Secrets
Secrets are used to store and distribute sensitive data to the pods, such as passwords or tokens. Pods can consume secrets as environment variables or as files from the volume mounts to the pod. It is noteworthy, that Kubernetes will encode the secret values using base64.
To create a Secret use the kubectl create secret generic command, with the following syntax:



Namespaces
A Kubernetes cluster is used to host hundreds of applications, and it is required to have separate execution environments across teams and business verticals.
To create a Namespace we can use the kubectl create namespace command, with the following syntax:



New terms
Configmap - a resource to store non-confidential data in key-value pairs.
Secret - a resource to store confidential data in key-value pairs. These are base64 encoded.
Namespace - a logical separation between multiple applications and associated resources.



Solution: Kubernetes Resources

# create the namespace 
# note: label option is not available with `kubectl create`
kubectl create ns demo

# label the namespace
kubectl label ns demo tier=test

# create the nginx-alpine deployment 
kubectl create deploy nginx-alpine --image=nginx:alpine  --replicas=3 --namespace demo

# label the deployment
kubectl label deploy nginx-alpine app=nginx tag=alpine --namespace demo

# expose the nginx-alpine deployment, which will create a service
kubectl expose deployment nginx-alpine --port=8111 --namespace demo

# create a config map
kubectl create configmap nginx-version --from-literal=version=alpine --namespace demo



Declarative Kubernetes Manifests

Kubernetes is widely known for its support for imperative and declarative management techniques. The imperative approach enables the management of resources using kubectl commands directly on the live cluster. 


On the other side, the declarative approach uses manifests stored locally to create and manage Kubertenest objects. This approach is recommended for production releases, as we can version control the state of the deployed resources.


YAML Manifest structure
A YAML manifest consists of 4 obligatory sections:

apiversion - API version used to create a Kubernetes object
kind - object type to be created or configured
metadata - stores data that makes the object identifiable, such as its name, namespace, and labels
spec - defines the desired configuration state of the resource


Deployment YAML manifest


New terms
Imperative configuration - resource management technique, that operates and interacts directly with the live objects within the cluster.
Declarative configuration - resource management technique, that operates and manages resources using YAML manifests stored locally.


Solution: Declarative Kubernetes Manifests

kubectl apply -f exercises/manifests/



Edge Case: Failing Control Plane for Kubernetes
Some of these resources are:

ReplicaSets - to ensure that the desired amount of replicas is up and running at all times
Liveness probes - to check if the pod is running, and restart it if it is in an errored state
Readiness probes - ensure that traffic is routed to that pods that are ready to handle requests
Services - to provide one entry point to all the available pods of an application




Glossary
Dockerfile - set of instructions used to create a Docker image
Docker image - a read-only template used to spin up a runnable instance of an application
Docker registry - a central mechanism to store and distribute Docker images
CRD - Custom Resource Definition provides the ability to extend Kubernetes API and create new resources
Node - a physical or virtual server
Cluster - a collection of distributed nodes that are used to manage and host workloads
Master node - a node from the Kubernetes control plane, that has installed components to make global, cluster-level decisions
Worker node - a node from the Kubernetes data plane, that has installed components to host workloads
Bootstrap - the process of provisioning a Kubernetes cluster, by ensuring that each node has the necessary components to be fully operational
Kubeconfig - a metadata file that grants a user access to a Kubernetes cluster
Pod - smallest manageable uint within a cluster that provides the execution environment for an application
ReplicaSet - a mechanism to ensure a number of pod replicas are up and running at all times
Deployment - describe the desired state of the application to be deployed
Service - an abstraction layer over a collection of pods running an application
Ingress - a mechanism to manage the access from external users and workloads to the services within the cluster
Configmap - a resource to store non-confidential data in key-value pairs.
Secret - a resource to store confidential data in key-value pairs. These are base64 encoded.
Namespace - a logical separation between multiple applications and associated resources.
Imperative configuration - resource management technique, that operates and interacts directly with the live objects within the cluster.
Declarative configuration - resource management technique, that operates and manages resources using YAML manifests stored locally.


