### install `CRE`
```
wget https://github.com/containerd/containerd/releases/download/v2.1.0/containerd-2.1.0-linux-amd64.tar.gz
tar Czxvf /usr/local containerd-2.1.0-linux-amd64.tar.gz
```
### install `service` containerd
```
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv containerd.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd
```
### install `runc`
```
wget https://github.com/opencontainers/runc/releases/download/v1.3.0/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```
### install `CNI` plugin
```
mkdir -p /opt/cni/bin
wget https://github.com/containernetworking/plugins/releases/download/v1.7.1/cni-plugins-linux-amd64-v1.7.1.tgz
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.7.1.tgz
```
### config `containerd`
```
sudo mkdir -p /etc/containerd/
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```
### To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
```
[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options] 
SystemdCgroup = true
sudo systemctl restart containerd
```
### Forwarding IPv4 and letting iptables see bridged traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system
```
### swap off
```
vim /etc/fstab
comment line
#/swapfile none swap sw 0 0
```
### install `kubernetes`
```
https://kubernetes.io/releases/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```
### installing `kubeadm`, `kubelet` and `kubectl`
```
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt list -a kubeadm
apt update
apt install kubelet=1.30.12-1.1 kubeadm=1.30.12-1.1 kubectl=1.30.12-1.1 -y
sudo apt-mark hold kubelet kubeadm kubectl
```
### init cluster on node `master`
```
kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=192.168.228.130 --kubernetes-version 1.30.12
```
### install network overlay `calico`
```
https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml
CHANGE cidr with cidr Kubernetes
wget https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml
  calicoNetwork:
    nodeAddressAutodetectionV4:
      interface: "ens33"
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 10.10.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

kubectl create -f custom-resources.yaml
kubectl get po -A -o wide
```
### join `worker` to cluster
```
kubeadm token list
kubeadm token create --print-join-command --ttl 10h
```
### change `label` Kubernetes node
```
kubectl label node worker1 kubernetes.io/role=worker1
```
### install `nerdctl`
```
GitHub:https://github.com/containerd/nerdctl?tab=readme-ov-file
wget https://github.com/containerd/nerdctl/releases/download/v2.1.2/nerdctl-full-2.1.2-linux-amd64.tar.gz
tar Cxzvvf /usr/local nerdctl-full-2.1.2-linux-amd64.tar.gz
for use  namespace containerd k8s.io
nerdctl -n k8s.io ps
```
