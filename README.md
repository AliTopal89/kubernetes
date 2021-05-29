### Kubernetes Clusters 

Kubernetes coordinates a highly available cluster of computers that are connected to work as a single unit. The abstractions in Kubernetes allow you to deploy containerized applications to a cluster without tying them specifically to individual machines.

A Kubernetes cluster consists of two types of resources:

The Control Plane coordinates the cluster
- Control Plane coordinates all activities in your cluster, such as scheduling applications, maintaining applications' desired state, scaling applications, and rolling out new updates.
Nodes are the workers that run applications
- Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes control plane.
- The node should also have tools for handling contaniers and operations such as *containerd* or *docker*
  - The **docker** command line tool can build container images, pull them from registries, create, start and manage containers. ... commands. **containerd**: This is a daemon process that manages and runs containers
- A Kubernetes cluster that handles production traffic should have a minimum of three nodes.

### notes from kubernetes cluster installation tutorial

install cloud servers ssh cloud_user@[ip] then password for control panel and workers

`sudo swapoff -a`

`sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`

 on all nodes disable swap, because the idea of kubernetes is to pack instance to %100
 utilized capacity. If memory swapping is allowed to occur on a host system, this can lead to performance and stability issues within Kubernetes. 

sudo apt-mark hold <package> - even when you apt-get update the packages stay the same version

`sudo kubeadm init --pod-network-cidr x.x.x.x/x` - initialize the cluster and setup kube command line tool (kubectl)
  - if you run in to prefilight errors such as
  
  ```
    error execution phase preflight: [preflight] Some fatal errors occurred:
	  [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
  ```

    - this was the solution for me
      ```s
      vi /etc/sysctl.conf
      net.bridge.bridge-nf-call-iptables = 1
      sysctl -p
      ```

kubeadm join 10.0.1.101:6443<x.x.x:port> --token blah.blah \
    --discovery-token-ca-cert-hash sha256:abcd123abc123abcd090909

or you can do 
 `kubeadm token create --print-join-command`
Get the join command (this command is also printed during ` kubeadm init`.)

You should now deploy a pod network to the cluster.
Run `"kubectl apply -f [podnetwork].yaml"` with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/


`Calico` is a networking and network policy provider. Calico supports a flexible set of networking options so you can choose the most efficient option for your situation, including non-overlay and overlay networks, with or without BGP. Calico uses the same engine to enforce network policy for hosts, pods, and (if using Istio & Envoy) applications at the service mesh layer.

`kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml` apply calico manifest

run `kubectl version`

you should have 
`Server Version: version.Info{Major:"1", Minor:"20", GitVersion:....`
means kubectl was able to communicate with our cluster and pull the version info

`kubectl get pods -n kube-system`

check the calico related pods to verify that everything is working so far

then on your control plane run 
`kubeadm token create --print-join-command`
and sudo run the commands on your workers otherwise you might see errors such as
```
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR IsPrivilegedUser]: user is not running as root
[preflight] If you know what you are doing, you can make a check non-fatal with "--ignore-preflight-errors=..."
```

from control plane run to see status of control plane and worker nodes

`kubectl get nodes`

```
NAME          STATUS   ROLES                  AGE   VERSION
k8s-control   Ready    control-plane,master   24m   v1.20.1
k8s-worker1   Ready    <none>                 99s   v1.20.1
k8s-worker2   Ready    <none>                 82s   v1.20.1
```
### Kubectl API basic cheat sheet

- Get a list of Pods in the kube-system namespace:
  `kubectl get pods -n kube-system`
- Get the same list of Pods using the raw Kubernetes API:
  `kubectl get --raw /api/v1/namespaces/kube-system/pods`
- Get information on a single Pod using the raw API:
  `kubectl get --raw /api/v1/namespaces/kube-system/pods/etcd-k8s-control`
- Read status of the specified Namespace
  `kubectl get --raw /api/v1/namespaces/kube-system/status`
- Show Merged kubeconfig settings
  `kubectl config view`
- List all pods in all namespaces
 `kubectl get pods --all-namespaces`
- Describe specific pod with verbose output (hint: use -n for namespace)
`kubectl describe pods -n kube-system calico-kube-controllers-blah-123`


#### Notes

1.[Kubernetes API ](https://kubernetes.io/docs/reference/kubernetes-api/)
1.[Kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Kubernetes objects

**Kubernetes objects** are persistent data entities stored by Kubernetes. They represent the state of your cluster. Pods are the kubernetes objects that are used to run and manage containers.

Kubernetes object includes two nested object fields
For objects that have a `spec`, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its *desired state*.

The `status` describes the *current state* of the object, supplied and updated by the Kubernetes system and its components.

The control plane works to implement the state represented by the object and the contanier is spun up in one of the nodes. Worker nodes constantly notify the API about the status of each contanier to control plane.

### Container with pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  contaniers:
  - name: nginx
    image: nginx
```

- `kind` - specifies the object type as Pod, (this could be Service, Deployments etc), we are telling kubernetest to create a pod object.
- `metadata.name` - The name of the pod object (this could be something like nginx-deployment etc.)
- `spec.containers` - provides a list of one or more contaniers
- `spec.containers[].image` - container image which contains the software we want our contanier to actually run. (docker image name) 

### Kubectl lab

```kubectl get pods -n default
NAME                READY   STATUS             RESTARTS   AGE
auth-microservice   1/1     Running            0          119m
data-backend        1/1     Running            0          119m
web-frontend        0/1     CrashLoopBackOff   28         119m
```

`kubectl -n default describe pod web-frontend`
is to describe your pod, which can be used to see any error in pod creation and running the pod like lack of resource, etc.

`kubectl delete pod web-frontend`

`vi webfront-end.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-frontend
spec:
  containers:
  - name: nginx
    image: nginx
```

`kubectl create -f web-frontend.yml`

```
kubectl get pods -n default
NAME                READY   STATUS    RESTARTS   AGE
auth-microservice   1/1     Running   0          49m
data-backend        1/1     Running   0          49m
web-frontend        1/1     Running   0          71s
```

### LAB create nginx and redis pod

`kubectl apply -f https://k8s.io/examples/application/deployment.yaml`

or vi and use an example from above from contanier with pods tutorial on control plane

- `kubectl create -f nginx-pod.yml`

- `kubectl get pods -o wide `- get verbose output of runnning pod

- `curl -kv $NGINX_IP_FROM_ABOVE`

- do the same but for redis for example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    ports:
    - containerPort: 6379
```