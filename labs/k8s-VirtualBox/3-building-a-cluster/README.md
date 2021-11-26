# 3 - Building a K8S Cluster

This lab will guide you through the process of K8S cluster creation.  
By now, you should have already done [labs-1](https://github.com/YuvalShaul/kubernetes/tree/main/labs/k8s-VirtualBox/1-infastructure-lab), [lab-2](https://github.com/YuvalShaul/kubernetes/tree/main/labs/k8s-VirtualBox/2-network-lab)  
Note:  
If, during your efforts commands stop working (not found) when you use **sudo**, do the following:

- to go to user root
**sudo su**
- edit the sudoers file:
**visudo /etc/sudoers**
- find and disable (put in a comment) the line starting with:
**# Defaults    secure_path = ...**

## Setting host names
It is easier to work with meaningfull hostname.  
Command:  
  **sudo hostnamectl set-hostname <host-name>**  (e.g: k8s-a, k8s-b, k8s-c k8s-control)  
Then, logout and login, to enable the change.

## Edit /etc/hosts

Make it easier to access each host.  
Update all /etc/hosts files so that all noeds know each other.  
Here are the lines I added to all of my master and nodes:  
192.168.122.10 k8s-control  
192.168.122.11 k8s-a  
192.168.122.12 k8s-b  
192.168.122.13 k8s-c  

## Enable kernel modules for containerd

**cat << EOF | sudo tee /etc/modules-load.d/containerd.conf  
overlay  
br_netfilter  
EOF**

Explanation:
- The **cat** command is reading from a [here document](https://tldp.org/LDP/abs/html/here-docs.html).
To understand here docs, try them with a word different from EOF.  
Here's and example with my name as a marker:  
**cat << yuval  
Hello  
from a  
here doc.  
yuval**  

- The **tee** command is reading the output of cat (that's because of the **|** pipeline symbol).
It will output the text to both the file in action, and the standard output.
(You can **cat** that file, and see that the commands are there).
So the kernel will load these module.

I'll also load those modules right now:
- **sudo modprobe overlay**
- **sudo modprobe br_netfilter**

## Some networking configurations for containerd

- Enable some required abilities:  
**cat << EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf  
net.bridge.bridge-nf-call-iptables = 1  
net.ipv4.ip_forward = 1  
net.bridge.bridge-nf-call-ip6tables = 1  
EOF**  

- To set those immediatelly:  
**sudo sysctl --system**  
- Install iproute-tc package:  
**sudo dnf install -y iproute-tc**  
- Stop and disable the firewalld service (not recommended for production):  
**systemctl stop firewalld.service**  
**systemctl disable firewalld.service**  
(there is a better option - enable those specific ports we really need, see [here](https://www.tecmint.com/install-a-kubernetes-cluster-on-centos-8/)).

## Install containerd

- Set up the Docker repository for CentOS:  
  - Install the yum-utils package:  
  **sudo yum install -y yum-utils**  
  - Add the repository:  
  **sudo yum-config-manager \  
  --add-repo \  
  https://download.docker.com/linux/centos/docker-ce.repo**  
- Remove conflicting packages:  
**sudo yum remove -y podman buildah**  
- Install the containerd.io package:  
**sudo yum install -y containerd.io**  

## Configure containerd

- **Now configure containerd:**
  - **sudo mkdir -p /etc/containerd**
  - **sudo /usr/local/bin/containerd config default | sudo tee /etc/containerd/config.toml**
  - Edit the created config.toml file, and change the fields that are specified [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)  
  Do not restart containerd service, as you have not created the service yet!
- **Create and run a containerd service**  
  - Create a systemd containerd.service file as described [here](https://github.com/containerd/containerd/blob/main/docs/ops.md) and put it in /etc/systemd/system (or better in /usr/lib/systemd/system, and a soft-link)
  - The ExecStart property points to where your binary file really is.
  - run and enable the service:  
    - **systemctl start containerd**
    - **systemctl enable containerd**
  - Verify it is running:
  systemctl status containerd

## Install K8S packages

- Turn swap off:
  - **sudo swapoff -a**
  - **sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab**  
  (this will put a single # character at the beginning of the swap line)
- Install packages:  
  [(see here for more details)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
  - cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo  
[kubernetes]  
name=Kubernetes  
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch  
enabled=1  
gpgcheck=1  
repo_gpgcheck=1  
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl  
EOF  
- Disable SELinux:  
  - **sudo setenforce 0**
  - **sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config**  
  This will actually turn off SELinux, which is required.
- Now for the REAL installations:  
  **sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes**
- **sudo systemctl enable --now kubelet**

## Initializing the Cluster

Initialization is done just on the control node.
Worker nodes are then joined to the cluster.
- sudo kubeadm init --pod-network-cidr 172.16.0.0/16 --kubernetes-version 1.22.3