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


### Pods-YAML

ALl YAML has 4 basic fields: `apiVersion`, `kind`, `metadata`, `spec`.

Here is an example:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
```

### Replication Controller

`NOTE:` Replication Controller is the old technology and is replaced by replica sets. Replication Controller is deprecated

Replicaset Template:


api version is `apps/v1`
`selector` is the main difference between replication controller and replica set. It is a mandatory property for replicaset.
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rc
  labels:
    type: app
spec:
  template:
    metadata:
        name: nginx
        labels:
            type: app
    spec:
        containers:
            - name: nginx
            image: nginx
  replicas: 3  
  selector:
    matchLabels:
      type: app
```

To change replica set:

1. Change the replicas in the file and run:
```
kubectl replace -f replicaset-definition.yml
```

or

```
kubectl apply -f replicaset-definition.yml
```

2. Change the replicas in command line

```
kubectl scale -replicas=6 -f replicaset-definition.yaml
```

or

```
kubectl scale --replicas=6 replicaset nginx-rc
```



### Difference between kubectl create/replace and kubectl apply

From https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create

```
Those are two different approaches. kubectl create is what we call Imperative Management. On this approach you tell the Kubernetes API what you want to create, replace or delete, not how you want your K8s cluster world to look like.

kubectl apply is part of the Declarative Management approach, where changes that you may have applied to a live object (i.e. through scale) are maintained even if you apply other changes to the object.

You can read more about imperative and declarative management in the Kubernetes Object Management documentation.
```

They do different things. kubectl create will throw an error if the resource already exists. kubectl apply won't. The difference is that kubectl create specifically says "create this thing" whereas kubectl apply says "do whatever is necessary (create, update, etc) to make it look like this"

https://stackoverflow.com/questions/47241626/what-is-the-difference-between-kubectl-apply-and-kubectl-replace

### Deployments

Definition is same as replicaset, just `Kind` is `Deployment`.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-rc
  labels:
    type: app
spec:
  template:
    metadata:
        name: nginx
        labels:
            type: app
    spec:
        containers:
            - name: nginx
            image: nginx
  replicas: 3  
  selector:
    matchLabels:
      type: app
```


### Certification Tip 

```
As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/
```

Example:

```
kubectl run my-app --image=mycompany/myapp:latest --replicas=1 --port=8080
```

1. Create an nginx pod:

```
kubectl run --generator=run-pod/v1 nginx --image=nginx
```

To create with a label

```
kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db
```

2. Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
```

3. Create a deployment

(a) with run - deprecated
```
kubectl run --generator=deployment/v1beta1 nginx --image=nginx [--replicas=6 --dry-run -o yaml]
```

Or recommended

```
kubectl create deployment --image=nginx nginx [--dry-run -o yaml]
```

4. Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

```
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
```

kubectl create deployment does not have a --replicas option. 
You could first create it and then scale it using the kubectl scale command.

5. Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

```
kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
```

(This will automatically use the pod's labels as selectors)

Or

```
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml  
```

(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

To add a label to the service:

```
kubectl expose pod redis --port=6379 --name redis-service
```

6. Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

```
kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml
```

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml
```


(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

7. Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas

```
kubectl create deployment webapp --image=kodekloud/webapp-color
kubectl scale deployment/webapp --replicas=3
```


#### References
https://kubernetes.io/docs/reference/kubectl/conventions/


### Namespaces

When a new service starts a DNS entry is added of the format:

```
{service-name}.{namespace}.svc.k8s-cluster-domain
```

For instance:

```
db-service.dev.svc.cluster.local
```

To create a namespace:

```
apiVersion: v1
kind: Namespace
metadata: 
  name: dev
```

or

```
kubectl create namespace dev
```

To switch namespace from default to another namespace:

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

To view pods in all namespaces:

```
kubectl get pods --all-namespaces
```

To create quota for a namespace:

```
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: compute-quota-dev
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```


### Services
#### NodePort

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                                                                                  │
│                                                                                  │
│                                                                                  │
├──────┐         ┌─────────────────┐                                               │
│      │         │                 │                                               │
│      │         │                 │                                               │
│      │         │  Service        │                                               │
│      │         │  10.106.1.12    │                                               │
│30008 │         │            ┌────┤ (2) Port                                      │
│      │◀────────┼────────────│ 80 │◀────────────────────────────▲                 │
│      │         │            └────┤                             └┐                │
│      │         │                 │                              │ (1) Target Port│
│      │         │                 │                    ┌─────┬───────┬─────┐      │
├──────┘         │                 │                    │     │  80   │     │      │
│(3) NodePort    └─────────────────┘                    │     │       │     │      │
│ 30000 - 32767                                         │     └───────┘     │      │
│                                                       │                   │      │
│                                                       │                   │      │
│                                                       │                   │      │
│                                                       │                   │      │
│                                                       │  POD (10.106.1.10)│      │
│                                                       │                   │      │
│                                                       │                   │      │
│                                                       └───────────────────┘      │
│      Node (IP: 192.168.10.3)                                                     │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

Service Definition with NodePort:

```
apiVersion: v1
kind: Service
metadata: 
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

Note - port is mandatory. If targetPort is not specified, it becomes same as port. 
If note port is not specified then it is randonmly choosen among available ports in the range.


## Scheduling

### Manual Scheduling
K8s scheduler looks at all pods without a node and assigns a node to a pod only when it does not have one.
If scheduler is not running in k8s, then all the pods will be in pending state. To schedule a pod manually
without scheduler, use `nodeName` attribute. Ex:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: node01
```

Manually specifying `nodeName` in spec file takes effect only in pod creation time.

In case the pod is already in `Pending` state, setting nodeName will not work. In that case we need to create
a binding object and POST to k8s api server.

Example:

```
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

Convert this yaml to json and do:

```
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion": "v1", ...}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
```

### Selector and Labels

Add labels for a pod so that pods can be grouped based on need.

to select a certain pods:

```
kubectl get pods --selector app=App1
```

selectors are also used to connect features in k8s.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rc
  labels:     ### ---> This is the label of the replicaset itself, 
              ##       in case we want to select this replicaset elsewhere.
    type: app
spec:
  template:
    metadata:
        name: nginx
        labels:          ### ---> This is the label of the pod itself.
            type: app    ###      "pod label"
    spec:
        containers:
            - name: nginx
            image: nginx
  replicas: 3  
  selector:
    matchLabels:         ### ---> This is the selector label - which has to match
      type: app          ###      the pod label.
```

Also annotations can be added to the spec file for documentation purposes which can also be used for some integration later.

From practice test:

```
# To select all pods in default namespace with env prod
kubectl get all --selector env=prod

# If this is used to filter pods in env=prod finance bu and frontend tier, will throw an error 
kubectl  get pod --selector env=prod bu=finance tier=frontend
error: name cannot be provided when a selector is specified

# Right way is:
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

We can also filter using `-l` option as in:

```
kubectl get all -l env=prod,bu=finance,tier=frontend
```

### Taints and Tolerations
Taints/Tolerations can be set to restrict what pods can be placed on what nodes.

Taints are attributes added to the node and only nodes with the toleration to the taint can 
be placed on that node.

To restrict pod placement on a node:

1. Add a taint on the node.
2. By default pods have no toleration for the taint, so nothing can be placed on it.
3. Then add a toleration to the pod.

#### Tainting node:

```
kubectl taint nodes node-name key=value:taint-effect
```

1. `node-name` is the name of the node
2. `key=value` is the taint KV
3. `taint-effect`: what happens to the pod that do not tolerate the taint. There are 3 values for this:
NoSchedule|PreferNoScheduler|NoExecute.

Example:

```
kubectl taint nodes node1 app=blue:NoSchedule
```

To remove a taint:

```
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
```

#### Tolerations
Add `tolerations` to the pod spec file

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

A taint is automatically set on the master node, so no application pod can be scheduled on the master node.
To see the masterNode taint:

```
kubectl describe node kubemaster | grep Taint
```

### Node Selection/Affinity
Taint/Toleration prevents unwanted pods from being scheduled on a node, but does NOT guarantee placement of 
certain pods on specific nodes.

There are two ways to do this:

#### Node Selector

1. Label the node with some kv

```
kubectl label nodes <node-name> <label-key>=<label:value>
```

Ex:

```
kubectl label nodes node1 size=large
```


2. In the pod definition add a `nodeSelector` property.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:
    size: large
```

Cannot achieve more complex placement like do not place on small nodes or place on large or medium nodes.

#### Node Affinity
Ensures pods are scheduled on particular nodes.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large 
            - Merge
```

operator can be `In|NotIn|Exists`.

If nodeAffinity cannot be met, it is handled by the type of nodeAffinity - `requiredDuringSchedulingIgnoredDuringExecution`.
Two types right:

1. `requiredDuringSchedulingIgnoredDuringExecution`
2. `preferedDuringSchedulingIgnoredDuringExecution`

There is plan for `requiredDuringSchedulingRequiredDuringExecution` in the future.

#### Labs
To add a label to a node:

```
kubectl label node node01 color=blue
```

### Resource Limits & Requirements

#### Requests

Default resource request is 0.5 CPU and 256MiB Memory.

1 count of CPU is 1vcpu (1 AWS vCPU, 1 Azure core, 1 GCP core == 1 hyperthread).
0.1 CPU is also called 100Mili. We can go as low as 1Mili. 

#### Limits
Default limits is 1 vCPU and 512MiB.


To change requests and lmits:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    type: app
spec:
  containers:
    - name: nginx
      image: nginx
  resources:
    requests:
      cpu: 1
      memory: "1Gi"
    limits:
      cpu: 2
      memory: "2Gi"
    
```


### Editing PODs and Deployments

#### Edit a POD
Remember, you CANNOT edit specifications of an existing POD other than the below.

1. spec.containers[*].image
2. spec.initContainers[*].image
3. spec.activeDeadlineSeconds
4. spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the kubectl edit pod <pod name> command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.



A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

```kubectl delete pod webapp```



Then create a new pod with your changes using the temporary file

``` kubectl create -f /tmp/kubectl-edit-ccvrq.yaml ```



2. The second option is to extract the pod definition in YAML format to a file using the command

```kubectl get pod webapp -o yaml > my-new-pod.yaml```

Then make the changes to the exported file using an editor (vi editor). Save the changes

vi my-new-pod.yaml

Then delete the existing pod

``` kubectl delete pod webapp ```

Then create a new pod with the edited file

``` kubectl create -f my-new-pod.yaml ```



#### Edit Deployments
With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

```kubectl edit deployment my-deployment```


### Daemon Sets

DaemonSets runs a pod (only one instance) in each of the nodes. When the node joins the cluster the daemon sets runs on it and when it leaves it gets deleted.
DaemonSets are useful for monitoring/logging type pods. kube-proxy runs as daemon set too.

To create a daemonset.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-rc
  labels:
    type: app
spec:
  template:
    metadata:
        name: nginx
        labels:
            type: app
    spec:
        containers:
          - name: nginx
            image: nginx

  selector:
    matchLabels:
      type: app
```

No replicas allowed in daemonsets.
Until k8s v1.12 daemon sets were implemented with nodeName for each pod but after that it is using nodeAffinity.

### Static Pods

`kubelet` can manage the node indepedently without `kube-apiserver` or `scheduler`. Usually the `kube-apiserver` will co-ordinate bringing up new instances with the help of scheduler. But if there is no master node, we can schedule pods using static pods.

A pod-config directory can be designated in the worker node. Kube-let will periodically look for changes in this directory and when there is a change will bringup the pods. It will monitor for new files and will also montior changes in existing files and apply them as needed. Only pod definition files can be placed in this directory and not replicaSet or deployments.

The configuration path can be any directory on the host and it needs to be passed as command line arg for kubelet using `--pod-manifest-path` option. 

Another way is to provide a config option `--config=kubeletconfig.yaml`, and in that path provide the 
config path as :

```
staticPodPath: /a/b/c/d/
```

To check if the static pods running, use `docker ps` .
If kube api server is running, kubelet would automatically update kube api server. WIth this we can check if the pod is running using kubectl command, but that's all we can do. To delete the pod, we have to remove the config file from the directory.


#### Use cases for Static Pod
kubeadmin tool will use this to setup all the master components like controller, apiserver, etcd etc. We install kubelet on the master node and place these files in the pod-manifest-path. Since these are static pods, even if they crash for any reason kubelet will relaunch them.

#### Difference between static PODs and DaemonSets

|:Static PODs:|:DaemonSets:|
|-------------|------------|
|:Created by Kubectl|:Created by Kube-API server (DaemonSet Controller)|
|:Deploy Control Plane componets as static pods|:Deploy Monitoring Agents, Logging Agents on nodes|
| Ignored by the Kube-Scheduler |
