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



