# Launch K8S Cluster using containerd and calico network add on

For this exercise I am using Ubuntu 20.04 and I will install K8S 1.23.0
 

### The following steps (1-17) must be performed on all three nodes (1 control plane and 2 workers).

1. Create configuration file for containerd:

```
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

2. Load modules:

```
$ sudo modprobe overlay
```
```
$ sudo modprobe br_netfilter
```

3. Set system configurations for Kubernetes networking:


$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
4. Apply new settings:


$ sudo sysctl --system
5. Update OS packages Install containerd:


$ sudo apt-get update && sudo apt-get install -y containerd
6. Create default configuration file for containerd:


$ sudo mkdir -p /etc/containerd
7. Generate default containerd configuration and save to the newly created default file:


$ sudo containerd config default | sudo tee /etc/containerd/config.toml
8. Restart containerd to ensure new configuration file usage:


$ sudo systemctl restart containerd
9. Verify that containerd is running:


sudo systemctl status containerd
10. Disable swap:


$ sudo swapoff -a
11. Disable swap on startup in /etc/fstab:


$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
12. Install dependency packages:


$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl
13. Download and add GPG key:


$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
14. Add Kubernetes to repository list:


$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
15. Update package listings:


$ sudo apt-get update
16. Install Kubernetes packages (Note: If you get a dpkg lock message, just wait a minute or two before trying the command again):


$ sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
17. Turn off automatic updates:


$ sudo apt-mark hold kubelet kubeadm kubectl
 

 

In control plane only perform steps (18-23)

 

initialising cluster

18. Initialize the Kubernetes cluster on the control plane node using kubeadm:


$ sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.23.0

I0627 13:09:14.363678   13411 version.go:248] remote version is much newer: v1.24.2; falling back to: stable-1.15
[init] Using Kubernetes version: v1.23.0
[preflight] Running pre-flight checks
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.101.33.147:6443 --token qe67n8.fckzqwmllwyrpqs2 \
    --discovery-token-ca-cert-hash sha256:365374078c253404dfb46e539e8ae7a40fd20136854c29722bad973696d37d8a
19. Set kubectl access:


$ mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
20. Test access to cluster (should be not ready yet):


$ kubectl get nodes

NAME                                     STATUS       ROLES                  AGE    VERSION
ip-10-101-33-147.srv101.dsinternal.org   Not Ready    control-plane,master   2d3h   v1.23.0
Install the Calico Network Add-On

21. On the control plane node, install Calico Networking:


$ kubectl apply -f <https://docs.projectcalico.org/manifests/calico.yaml>
22. Check status of the control plane node (wait, control plane will be in ready status):


$ kubectl get nodes

NAME                                     STATUS   ROLES                  AGE    VERSION
ip-10-101-33-147.srv101.dsinternal.org   Ready    control-plane,master   2d3h   v1.23.0
Getting token for joining worker nodes

23. In the control plane node, create the token and copy the kubeadm join command (NOTE:The join command can also be found in the output from kubeadm init command):


$ kubeadm token create --print-join-command

kubeadm join 10.101.33.147:6443 --token qe67n8.fckzqwmllwyrpqs2 --discovery-token-ca-cert-hash sha256:365374078c253404dfb46e539e8ae7a40fd20136854c29722bad973696d37d8a
Perform steps (24-25) in Worker Nodes only

24. In both worker nodes, paste the kubeadm join command to join the cluster. Use sudo to run it as root:


$ sudo kubeadm join 10.101.33.147:6443 --token qe67n8.fckzqwmllwyrpqs2 --discovery-token-ca-cert-hash sha256:365374078c253404dfb46e539e8ae7a40fd20136854c29722bad973696d37d8a
25. In the control plane node, view cluster status (Note: You may have to wait a few moments to allow all nodes to become ready):


$ kubectl get nodes

NAME                                     STATUS   ROLES                  AGE    VERSION
ip-10-101-32-196.srv101.dsinternal.org   Ready    <none>                 2d3h   v1.23.0
ip-10-101-33-147.srv101.dsinternal.org   Ready    control-plane,master   2d3h   v1.23.0
ip-10-101-35-71.srv101.dsinternal.org    Ready    <none>                 2d3h   v1.23.0
Conclusion
Your Cluster should be up and running!!!!!!
