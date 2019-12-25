# Certified Kubernetes Administrator

## Certificate
Certification web page is www.cncf.io/certification/cka. Exam fees is $300 with 1 retake
It's proctored online for 3 hours and is not a multiple choice exam.

Exam Curriculem (topics) can be found at https://github.com/cncf/curriculum. 

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

`kubectl` commands fetches information from etcd server.


K8s can be setup manually where etcd can be downloaded and installed manually on the master.
Or K8s can be setup using kubeadm and in this etcd is brought up on containers running on master.

On installation with kubeadm to list all the keys k8s has stored in kubernetes try:

```
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

### Kube-apiserver

Kube-apiserver is the primary control plane manager for the k8s custer. 
A `kubectl` command like `kubectl get nodes` connects to the kube-apiserver, 
authenticates the user, validates the requests and retrieves the data from etcd server.
We dont need kubectl command to fetch information, we can use http queries to fetch this information.

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

