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

## Kubernetes API Objects
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
- Users can just simply reach new pods by communicating with the Service instead of individual pods, that get deleted and recreated
- Helps to manage easy **Scaling**
- Handles all the **routing** to pods
- Provides **Load Balancing**

### Storage
**Persistent Volume Claim** is used to reserve some *Cluster* storage for each Pod. Each pod uses their storage independently

### ConfigMap
ConfigMap is an API object used to store non-confidential data in key-value pairs

Example: 
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true  
```

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
- most commonly implemented in **iptables**
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

## Kubernetes Networking
- Pods on a Node can communicate to all Pods on all Nodes without Network Address Translation (NAT)
- Agents (kubelet, System Daemon, etc.) on a Node can communicate with all Pods on that Node

### CNI (Container Network Interface) can be used to satisfy Kubernetes networking requirements
Kubernetes by itself doesn’t implement networking. Instead, it defines networking requirements that can be satisfied by a Container Network Interface (CNI) plugin.
**Calico** is one of the most popular CNI implementations.

## Installing Kubernetes with **kubeadm**
### Where to install?
- Cloud
    - IaaS - virtual machines (have to take care of the OS and Kubernetes software on the virtual machines)
    - PaaS - managed service (provided by major cloud providers, don't have to maintain anything)
- On-premises (have to take care of everything)
    - Bare metal
    - Virtual machines

While Cloud takes care of the hardware and/or software maintenance, on-prem solution might be cheaper and more flexible.

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
        - admin.conf (kubernetes-admin)
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

## Working with Clusters
The main tool to control a running Cluster is **kubectl**

usage: kubectl [operation] [resource] [name] [flags (e.g. output)]<br>
Note: name is optional if you want to work with multiple resources

More on usage in [kubernetes kubectl documentation](https://kubernetes.io/docs/reference/kubectl/kubectl/) and [cheat sheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

### kubectl operations
- apply/create - create resource
- run - start a pod from an image (a pod that is not managed by a controller)
- explain - documentation of resources
- delete - delete resource
- get - list resources
- describe - detailed resource information
- exec - execute a command on a container
- logs - view logs on a container

### kubectl resources
- nodes (no)
- pods (po)
- services (svc)
- ...more resources [here](https://kubernetes.io/docs/reference/kubectl/overview/#resource-types)

### kubectl output
By adding additional flags, kubectl output can be modified. Output flag is -o
- wide - output additional information
- yaml - YAML formatted API object
- json - JSON formatted API object
- dry-run - print an object without sending it to the API Server (good for generating YAML on how to create resources)

Examples of kubectl usage and how to enable bash kublectl completion [here](kubernetes/1-workingwithyourcluster.sh)

## Application Deployment in Kubernetes
### Imperative configuration (executing commands in cli one at a time)
- kubectl create deployment nginx -- image=nginx
- kubectl run nginx -image=nginx
### Declarative (Configured in a file with desired state)
- Define desired state in code
- Manifest in YAML or JSON
- kubectl apply -f deployment.yaml

Using imperative way with a dry run flag (--dry-run=client -o yaml > deployment.yaml) could help generate correct Declarative configuration

Instructions on how to deploy your application into the cluster and how to modify existing deployments [here](kubernetes/2-deployingapplications.sh)

## Cluster Maintenance
### Cluster Upgrade Process
- Upgrade Control Plane Node -> Upgrade any other Control Plane Nodes -> Upgrade Worker Nodes
- Can only upgrade minor versions
    - 1.28 -> 1.29
    - 1.27 X 1.29
- Read the release notes before each upgrade
### Worker Node Maintenance
- Version upgrades, OS updates and hardware updates
#### Steps:
- ##### Drain/Cordon the Node
    - kubectl drain NODE_NAME
        - Marks the node unschedulable
        - Gracefully terminates Pods (Controller automatically reschedules them on other Nodes)
- ##### Keep resources in mind
    Before draining the Node be sure you have enough resources on the other Nodes to take over the work of the Node you're about to drain and upgrade

### Upgrade Control Plane Node
- Update kubeadm package (apt or yum upgrade)
- kubeadm upgrade plan
    - does pre-flight checks like ensuring you're upgrading to an appropriate version, nodes are healthy, etc.
- kubeadm upgrade apply
    - runs pre-flight checks
    - pre-pulls the container images
    - updates certificates for each control plane pod
    - creates new static pod manifests in /etc/kubernetes/manifests
    - saves previous manifests in /etc/kubernetes/temp
- drain the Control Plane Node
    - drain only non-control plane pods
    - control plane pods will stay on control plane node, since they are static
- upgrade kubelet and kubectl
- uncordon/undrain the Control Plane Node

Instructions on how to do it in terminal [here](./kubernetes/1-ControlPlaneUpgrade.sh)

### Upgrade Worker Nodes
Be sure to upgrade worker nodes to the **same versions** as the **Control Plane Nodes**
- update kubeadm
- kubeadm upgrade node
- drain the node
- update kubelet and kubectl
- uncordon the node

Instructions on how to do it in terminal [here](./kubernetes/2-NodeUpgrade.sh)


## Probes
### Liveness probes
Used by kubelet to know when to restart a container
### Readiness probes
Used by kubelet to know when the container is ready to start accepting traffic. A pod is considered ready when all of its containers are ready. This is used for services. When a pod is not ready, it is removed from service load balancers.
### Startup probes
Used by kubelet to know when a container application has started. This can be used to adopt liveness checks on slow starting containers, and will avoid them getting killed by the kubelet before they are up and running.

## Init Containers
These containers are used to support the main containers of the pod and are run before app containers start. They are mainly used for setup scripts.

## Kustomize
Community-led solution for managing **Kubernetes configuration**. Allows to take configuration expressed as raw YAML and customize the YAML for our purposes. It leaves the **original YAML** definitions **untouched**. A **good use case** is to use **Kustomize** to apply **DRY principle** while writing Kubernetes configuration for different environments (dev, stage, prod). Meaning to not rewrite configurations for each environment and to reuse the same one, but add minor changes related to that environment.

Uses 2 broad classes to achieve the result:
- Generators - create new Kubernetes objects when they don't exist and transform existing object when they do exist
- Transformers - take existing Kubernetes objects and overlay alternative values and fields to make different variations of the configuration. 

Kustomize key facts:
- Originates from Google. It's a sub-project of the Kubernetes community CLI SIG (SIG-CLI)
- Kustomize is both, a standalone binary, and also an integral part of Kubernetes CLI (kubectl)
- Kustomize has many features, but if a required feature is missing this can be provided using a plugin.


### Kubernetes Resource Model (KRM)
A declarative way to express Kubernetes clusters. Kustomize adopts the familiar KRM approach for defining operations on configuration.

The configuration is described in `kustomization.yaml` file.<br>
Example:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

### Built-in Generators
- ConfigMap - Creates, replaces or merges one or more ConfigMap objects
- Secret - Creates, replaces or merges one or more Secret objects

### Built-in Transformers
- Namespace - Sets namespace for some or all objects
- ImageTag - Sets image name, tag or digest
- PrefixSuffix - Adds a prefix or suffix to a name
- ReplicaCount - Increase or decrease number of replicas

### Built-in Metadata Transformers
- Label - Adds labels to the metadata for objects and their corresponding selectors
- Annotation - Adds annotation key/value pairs to the metadata for all applicable objects

### Resources
- List of directories pointing to configuration files that are going to be extended / changed
- Web URLs can be used to provide remote resources. Web URLs are separated by // to provide a path location to resource within the remote locations (e.g. `https://git.com/myloc//path/to/deployments`)

### Setting namespaces
An example of how to set namespaces with kustomize:
```
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base
  # Include the namespace to add the managed-by label
  - namespace.yaml
buildMetadata:
  - managedByLabel
namespace: dev
```

namespace.yaml:
```
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Namespaces can also be **changed** using NamespaceTransformer instead

### Adding labels
You can add label to all objects (resources) metadata or make **common** labels, that will be applied to not only the metadata, but the selectors and the templates
```
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base
  - namespace.yaml
buildMetadata:
  - managedByLabel
namespace: dev
# appears only in the metadata
labels: 
  - pairs:
      app.kubernetes.io/env: dev
# appears everywhere
commonLabels:
  app.kubernetes.io/version: v1.0

```

### Generating ConfigMap
Use `configMapGenerator` to generate a **ConfigMap**<br>
Multiple **ConfigMaps** can be generated using `configMapGenerator`
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
configMapGenerator:
  - name: db
    literals:
      - SQLITE_DB_LOCATION=/tmp/todos/todo.db
```

Similarly `Kubernetes Secretes` can be managed too:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
secretGenerator:
  - name: registry-credentials
    files:
      - config.json
    type: "kubernetes.io/dockerconfigjson"
```
The main difference is that when specifying `Kubernetes Secrets` you can specify a **type**

### Patching
- #### Strategic Merge Patch
- #### JSON Patch
    Patch operations: Add, Remove, Replace, Move, Copy, Test<br>
    Example:
    ```
    # deployment-patch.yaml
    - op: replace
      path: /spec/template/spec/containers/0/imagePullPolicy
      value: Always
    ```