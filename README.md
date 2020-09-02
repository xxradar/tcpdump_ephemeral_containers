# How to tcpdump using ephemeral containers and ```kubectl debug```
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
## Create a nginx pod
```
kubectl run nginx-demo --image nginx:latest 
```
## Create an ephemeral debug container
```
kubectl alpha debug -it nginx-demo  --image=dockersec/tcpdump --target nginx-demo
```

## Generate some traffic via a seperate console
```
kubectl run -it --rm load --image xxradar/hackon -- bash
    curl http://<nginx-demo-ip-address>
```
See the output in the ephemeral container
```
kubectl alpha debug -it nginx-demo  --image=dockersec/tcpdump --target nginx-demo
Defaulting debug container name to debugger-jml67.
If you don't see a command prompt, try pressing enter.
09:41:59.177177 ARP, Request who-has nginx-demo tell 128-199-42-214.kubernetes.default.svc.cluster.local, length 28
09:41:59.177199 ARP, Reply nginx-demo is-at 46:9f:0a:24:f4:81 (oui Unknown), length 28
09:41:59.177203 IP 10.48.38.201.46526 > nginx-demo.80: Flags [S], seq 3083611759, win 64400, options [mss 1400,sackOK,TS val 2789751669 ecr 0,nop,wscale 7], length 0
09:41:59.177225 ARP, Request who-has 169.254.1.1 tell nginx-demo, length 28
09:41:59.177228 ARP, Reply 169.254.1.1 is-at ee:ee:ee:ee:ee:ee (oui Unknown), length 28
09:41:59.177230 IP nginx-demo.80 > 10.48.38.201.46526: Flags [S.], seq 1011782564, ack 3083611760, win 65236, options [mss 1400,sackOK,TS val 3814256463 ecr 2789751669,nop,wscale 7], length 0
09:41:59.177263 IP 10.48.38.201.46526 > nginx-demo.80: Flags [.], ack 1, win 504, options [nop,nop,TS val 2789751669 ecr 3814256463], length 0
09:41:59.183283 IP nginx-demo.40204 > kube-dns.kube-system.svc.cluster.local.53: 33453+ PTR? 201.38.48.10.in-addr.arpa. (43)
```
