
# Configure the prereq

```bash
# Check required ports are open
nc 127.0.0.1 6443 -zv -w 2
```

```bash
# Disable swap memory
swapoff -a

# Permanently disable swap memory
sudo sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```bash
# Enable br_netfilter to process bridged network traffic
modprobe br_netfilter
```

```bash
# Enable IP forwarding
sysctl -w net.ipv4.ip_forward=1
```

# Install a container runtime 

```bash
# Define the Kubernetes version and used CRI-O stream
KUBERNETES_VERSION=v1.34
CRIO_VERSION=v1.34
```

```bash
# Add the Kubernetes repository
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/repodata/repomd.xml.key
EOF
```

```bash
# Add the CRI-O repository
cat <<EOF | tee /etc/yum.repos.d/cri-o.repo
[cri-o]
name=CRI-O
baseurl=https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/rpm/repodata/repomd.xml.key
EOF
```

```bash
# Install crio
dnf install -y cri-o
```

```bash
# Enable and start CRI-O
systemctl enable --now crio.service
```


# Installing kubeadm, kubelet and kubectl 

```bash
# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

```bash
dnf install -y kubelet kubeadm kubectl container-selinux

# Enable kubelet
sudo systemctl enable --now kubelet
```


# Bootstrap a cluster

```bash
# --apiserver-advertise-address sets the advertised address for the control-plane's node
# --control-plane-endpoint is the LB IP used to distribute traffic to the control plane nodes
kubeadm init --apiserver-advertise-address x.x.x.x --control-plane-endpoint x.x.x.x
```


