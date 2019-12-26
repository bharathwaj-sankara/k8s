# Certified Kubernetes Administrator

## Certificate
Certification web page is www.cncf.io/certification/cka. Exam fees is $300 with 1 retake
It's proctored online for 3 hours and is not a multiple choice exam.

Exam Curriculum (topics) can be found at https://github.com/cncf/curriculum. 

Candidate Handbook is at https://www.cncf.io/certification/candidate-handbook

Exam Tips is at http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD

```
Use the code - DEVOPS15 - while registering for the CKA or CKAD exams at 
Linux Foundation to get a 15% discount.
```

## Core Concepts

### Kubernetes Architecture

K8s consists of two types of nodes:

* Master Nodes
* Worker Nodes


#### Master Node

Master Nodes is responsible for the control plane on the cluster. Master nodes contains 
the following components:

1. `ETCD Cluster` - Etcd key-value store is the data store for the entire cluster state.
2. `kube-scheduler` - responsible for scheduling pods on the worker nodes.
3. `kube-control-manager` - Various controllers like node controller, replication controller etc.
4. `kube-apiserver` - is the main orchestrator of all the k8s function

#### Worker Node

Worker nodes actually host the application container. These nodes contain:
1. `Container Runtime Enginer` - Something like docker or rkt or containerd to actually run containers
2. `kubelet` - the service that manages the node. 
3. `kubeproxy` - that helps containers within the worker node communicate.

Kubernetes can be setup manually by installing all the base conponents on master and 
worker nodes. Or it can be setup using kubeadm where all the components are brought up as pods
on the master node.

### ETCD 
ETCD is a distributed reliable key-value store that is simple, fast and secure.
Etcd stores information about:
* Nodes
* Pods
* Configs
* Secrets
* Accounts
* Roles
* Bindings
* Others

`kubectl` commands fetches information from etcd server through api-server.


For manual K8s setup, etcd can be downloaded and installed manually on the master.
etcd runs natively on the master and can be queried directly for the k8s kv.
For kubeadm deploys, to list all the keys k8s has stored in kubernetes try:

```
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

### Kube-apiserver

Kube-apiserver is the primary control plane manager for the k8s cluster. 
A `kubectl` command like `kubectl get nodes` connects to the kube-apiserver, 
authenticates the user, validates the requests and retrieves the data from etcd server.
Alternatively, we can use http queries to fetch this information instead of kubectl.

kube-apiserver is responsible for:

1. Authenticate user
2. Validate Request
3. Retrieve data
4. Update ETCD - infact kube-apiserver is the only engine which can update etcd
5. Scheduler and kubelet communicate with api-server to do their tasks.

For instance to create a new pod:
1. kube-apiserver authenticates and validates the request
2. Updates in etcd to create the pod 
3. responds back to the user that the pod is created.
4. the kube-scheduler contantly monitors the kube-apiserver and finds a new pod is created with no worker nodes.
5. kubeserver will create a worker node and let the kube-apiserver know about this.
6. kube-apiserver will update this in etcd 
7. kube-apiserver will communicate this with the kubelet and ask it to create the new pod.
8. kubelet will instruct the container runtime engine to download the image and start the contianer.
9. Once the container is up, kubelet will ket the kube-apiserver know 
10. kube-apiserver will update the etcd cluster about this.

If k8s is setup using kubeadm, then apiserver is run as a pod in the kube-system namespace.

```
kubectl get pods -n kube-system
```
The way apiserver is run with all it's option within the pod definition in:

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

If k8s is setup manually, the run command can be seen at:

```
cat /etc/systemd/system/kube-apiserver.service
```

### Kube Controller Manager

Controller is a process that continuously monitors the state of various components and 
works towards getting the entire system to the desired state. For instance: 
`Node controller` - monitor state of every node every 5s, and marks unreachable (after 40s) and 
gives 5 mins to come backup and after this moves pods from the unhealthy nodes to the healthy node.
`Replication Controller` - desired number of pods are always available

All the controller runs as a part of kube-controller-manager.

Options like node eviction period etc are options to starting the kube-controller-manager.
It also has an option called `controllers` which allows to selectively choose the controllers
to run, be default all controllers are run.

### Kube-Scheduler

Scheduler decides which pods goes on which nodes. Kubelet actually manage launching the container.
To find a node for a pod, scheduler goes through two phases:
1. filter phase - the nodes that cannot fit the pod are discarded from selection
2. ranking phase - the nodes that can accomodate the pod are then ranked based on resource availability.

The node with the highest rank is chosen for the pod.

### Kubelet
1. Registers the node with the kubernetes master
2. Works with apiserver and when a pod is assigned to it, will ask the container runtime engine to download the image and run the pod.
3. Contantly monitors the status of the node and the pods and reports it to the apiserver.

kubeadm does NOT deploy kubelet. It needs to be manually downloaded and installed on the nodes.

### Kube-proxy
In a k8s system, every pod can reach every pod. Pod network is a virtual network in the k8s cluster with which pods communicate.
kube-proxy is a process run on each node and looks for new services and everytime a new service is found it will learn that
and provide the routing the backend pods whenever a request to the service is made. It does this using iptable rules. 

Using kubeadm deploy kube-proxy as deamon-set on each node.