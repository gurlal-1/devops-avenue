### update hostname
sudo hostnamectl set-hostname controlplane
sudo hostnamectl set-hostname worker1
sudo hostnamectl set-hostname worker2

### check mac and uuid 
ip link
cat /sys/class/dmi/id/product_uuid

### turn swap off
swapoff -a

### enable ip packet forwarding 

echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/k8s.conf
sysctl --system
sysctl net.ipv4.ip_forward

### install containerd
apt update && sudo apt upgrade -y
apt-get install containerd
ctr --version

### Install cni plugins
mkdir -p /opt/cni/bin
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.2.tgz

### configure containerd

mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml
head /etc/containerd/config.toml
systemctl restart containerd

*** configure the systemd cgroup driver
vi /etc/containerd/config.toml

Within [plugins.”io.containerd.grpc.v1.cri”.containerd.runtimes.runc.options] section
SystemdCgroup = true
systemctl restart containerd

### Add Kubernetes repos and install tools
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm
apt-mark hold kubelet kubeadm

### Install kubectl on controlplane

apt-get install -y kubectl


### Intialize Cluster
kubeadm init --pod-network-cidr=192.168.0.0/16

### Configure a regular user for kubectl

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Install network plugin - calico
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.1/manifests/custom-resources.yaml
watch kubectl get pods -n calico-system









######################################### Mi version nueva


 1  swapoff -a
    2  echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/k8s.conf
    3  sysctl --system
    4  sysctl net.ipv4.ip_forward
    5  apt update && sudo apt upgrade -y
    6  apt-get install containerd
    7  ctr --version
    8  mkdir -p /opt/cni/bin
    9  https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
   10  wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
   11  tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.2.tgz
   12  mkdir /etc/containerd
   13  containerd config default > /etc/containerd/config.toml
   14  head /etc/containerd/config.toml
   15  systemctl restart containerd
   16  nano /etc/containerd/config.toml
   17  systemctl restart containerd
   18  apt-get update
   19  apt-get install -y apt-transport-https ca-certificates curl gpg
   20  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   21  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   22  apt-get update
   23  apt-get install -y kubelet kubeadm
   24  apt-mark hold kubelet kubeadm
   25  apt-get install -y kubectl
   26  kubeadm init --pod-network-cidr=192.168.0.0/16
   27  mkdir -p $HOME/.kube
   28  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   29  sudo chown $(id -u):$(id -g) $HOME/.kube/config
   30  kubectl create -f  https://raw.githubusercontent.com/projectcalico/calico/v3.31.1/manifests/tigera-operator.yaml
   31  kubectl create -f  https://raw.githubusercontent.com/projectcalico/calico/v3.31.1/manifests/custom-resources.yaml
   32  watch kubectl get pods -n calico-system

