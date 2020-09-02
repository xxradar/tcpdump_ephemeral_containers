# How to tcpdump using ephemeral containers
In order to use ephemeral containers, the K8S cluster needs to be created using the EphemeralContainers feature gate.

## Create a cluster enabling EphemeralContainers
The following configuration ```cluster.yaml``` helps installing a K8S cluster using ```kubeadm```.
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    "feature-gates": "EphemeralContainers=true"
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
networking:
  serviceSubnet: 10.49.0.0/16
  podSubnet: 10.48.0.0/16
  dnsDomain: cluster.local
apiServer:
  extraArgs:
    "feature-gates": "EphemeralContainers=true"
scheduler:
  extraArgs:
    "feature-gates": "EphemeralContainers=true"
controllerManager:
  extraArgs:
    "feature-gates": "EphemeralContainers=true"
```
Initialise the cluster. See https://github.com/xxradar/onprem-k8s-calico-oss for more info on prerequisites.
```
sudo kubeadm init --config cluster.yaml
```
## Create a nginx container
```
kubectl create deployment nginx --image nginx:latest
```
