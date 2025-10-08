# Kubernetes
Kubernetes helps us manage a lot of containers

#### Main benefits
- Speed of deployment
- Ability to absorb change quickly
- Ability to recover quickly
- Hide complexity in the cluster

#### Infrastructure Abstraction
One of the main features that kubernetes provides is **Infrastructure Abstraction**
Which means when deploying developers don't need to know or care about specific nodes or do any manual jobs to make sure the load balancer sends traffic to the application, it is all done under the hood

#### Desired State
We can describe how our application structure looks like (all different services/controllers) and Kubernetes ensures it maintains this **desired state** (all the services are up, how they interact with each other, etc.)

#### Kubernetes API
Provides a way to deploy and manage the Kubernetes services and also this is where Kubernetes Cluster components interact with each other

## Kubernets API Objects
Kubernetes API Objects are the components which help us configure and deploy applications to Kubernetes. Using these objects we define the state and structure of the system. We can do that both **Declaratively** (using .yml) and **Imperatively** (setting everything up using the command line)

### Pods
- One or more containers
- Most basic unit of work. The **single** application / service.
- Amount of resources needed can be described and kubernetes makes sure those resources are available to the **Pod**
- **Ephemeral** (no **Pod** is ever "redeployed" if **Pod** fails, new one will replace it)
- **Atomicity** - the **Pod** is either there or NOT (**Pod** **cannot be partially running**). This makes more sense when more than 1 container is running on the **Pod**. If at least 1 of the containers fail, **Pod** is no longer available.
- Using **Probes**, the health of the application inside a **Pod** can be checked. Even though the **Pod** is running, the inside application might be throwing errors and using **Probes** we can ensure that doesn't happen without us knowing.

### Controllers
Makes sure our Pods are running and healthy.
- Defines the desired state
- Can create and manage Pods for you
- Monitors and responds to Pod state and health

Types of controllers:
- *ReplicaSet.* - allows to define a number of replicas for a pod and makes sure all the replicas run and are healthy
- *Deployment* - ReplicaSets don't usually get created by themselves, they are defined in the *Deployment* controllers and are created by them. *Deployment* controller also manages the transition between 2 ReplicaSets (for example moving between 2 versions of an application)

### Services
Helps to keep Pods **persistent** (since they are *ephemeral*)
- Service is provided with an **IP** address and a **DNS** name
- Users can just simply reach new pods by communicating with the Serivce instead of individual pods, that get deleted and recreated
- Helps to manage easy **Scaling**
- Handles all the **routing** to pods
- Provides **Load Balancing**

### Storage
**Persistent Volume Claim** is used to reserve some *Cluster* storage for each Pod. Each pod uses their storage independently

## Cluster components

### Control Plane Node (formerly Master Node)
Implements main control functions of a cluster, does monitoring and scheduling, coordinates cluster operations

Contains these components:
- #### API Server
    Communications hub for the cluster<br>
    **kubectl** is used to interact with the **API Server**<br>
    Uses RESTful principle
- #### etcd
    Cluster datastore. Persists the **state** of the kubernetes **API Objects** using **key-value pairs**
- #### Scheduler
    Tells kubernetes on which nodes the Pod should be started based on the Pods **resource** requirements and other Pod requirements<br>
    **Watches** the **API server** and looks for **unscheduled Pods** and schedules them onto Nodes in the Cluster<br>
    **Constraints** can be defined (2 specific Pods to never be on the same Node, etc.) and **Scheduler** follows them
- #### Controller manager
    Responsible for keeping things in the desired state. Implements lifecycle functions for controllers that monitor and manage kubernetes objects state.

### Node
The worker node is where our application pods run. Starts pods and their containers 

Each node has these services:
#### Kubelet
- Monitors API Server for changes
- Responsible for Pod Lifecycle (starts and stops pods and their containers)
- Reports Node & Pod state
- Executes Pod *probes*

#### kube-proxy
- most commonly impemented in **iptables**
- implements Services
- Routes traffic to Pods
- Load balancing

#### Container Runtime
- Downloads images & runs containers
- uses Container Runtime Interface (CRI) which allows to choose a desired container runtime
- the default runtime since 1.20 is ***containerd*** before then it was *Docker*

### Cluster Add-on Pods
Special purpose Pods

Examples:
- #### DNS Server Pod
    Provides DNS for the Cluster
- #### Ingres controller
    Advanced load balancers

## Kubernets Networking
- Pods on a Node can communicate to all Pods on all Nodes without Network Address Translation (NAT)
- Agents (kubelet, System Daemon, etc.) on a Node can communicate with all Pods on that Node

## Installing Kubernetes with **kubeadm**
### Where to install?
- Cloud
    - IaaS - virtual machines (have to take care of the OS and Kubernetes software on the virtual machines)
    - PaaS - managed service (provided by major cloud providers, don't have to maintain anything)
- On-premises (have to take care of everything)
    - Bare metal
    - Virtual machines

While Cloud takes care of the harware and/or software maintenance, on-prem solution might be cheape and more flexible.

### Installation Requirements
- #### System
    - Linux OS
    - 2 CPUs
    - 2GB RAM
- #### Container runtime
    - Container Runtime Interface (CRI)
    - containerd
    - CRI-O
- #### Networking
    - Connectivity between all nodes
    - Unique hostname
    - Unique MAC address

### Installation Steps
#### 1. Define your Cluster structure and network topology
For example a Cluster with a control plane node and 2 worker nodes. 

Assign an internal IP address to each of the nodes.

#### 2. Install packages on each node
- containerd (or other container runtime like Docker)
- kubelet
- kubeadm
- kubectl

See [kubernetes package installation guide](kubernetes/0-PackageInstallation-containerd.sh)

#### 3. Bootstrap a Cluster with kubeadm - Control Plane Node
kubeadm init will: 
- execute a series of pre-flight checks:
    - ensures you have right permissions on the system
    - ensures you have enough system resources
    - ensures there's a container runtime and it is up and running
- create a Certificate Authority (CA)
    - secures cluster communication
    - creates certificates for user and cluster component authentication
    - CA files are stored in /etc/kubernetes/pki
- generate kubeconfig files so cluster components could locate each other
    - defines how to connect to your cluster
        - Client certificates
        - Cluster API network location
    - stored in /etc/kubernetes
        - admin.connf (kubernetes-admin)
        - super-admin.conf
        - controller-manager.conf
        - scheduler.conf
        - kubelet.conf
- generate static pod manifests
    - manifest describes a configuration
    - lives in /etc/kubernetes/manifests
    - watched by the **kubelet** to start automatically on system start or restart
- wait for the control plane pods to start
- taint the control plane node (so user pods won't be scheduled onto control plane node)
- generate a bootstrap token (used to add nodes to the cluster)
- start add-on pods

To create the Control Plane Node follow the [link](kubernetes/1-CreateControlPlaneNode-containerd.sh)

#### 4. Bootstrap a Cluster with kubeadm - Worker Node
**kubeadm join** will be used to join a node to a cluster

kubeadm join will:
- download cluster information
- submit CSR (Certificate Signing Request) and receive a certificate that node kubelet will use to join the cluster
- CA automatically signs the CSR, kubeadm join downloads the certificate and stores it in a file system of the node (/var/lib/kubelet/pki)
- generate kubelet.conf file

Follow the [link](kubernetes/2-CreateNodes-containerd.sh) for further instructions on how to join the worker node onto the cluster