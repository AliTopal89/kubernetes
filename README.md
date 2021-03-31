notes from kubernetes cluster installation tutorial

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