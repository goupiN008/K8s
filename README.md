## Standalone Kubernetes Installation Manual

##### Component

- Kubernetes 1.13.12
- kubeadm 1.13.12
- kubelet 1.13.12
- kubernetes-cni 0.7.5
- Docker CE 19.03.5
- calico 3.3
- Ubuntu 18.04.3 LTS

#### VMs

- 1 VM for Kubernetes





### 1.) Install Docker and Kubernetes for all workers and masters 

- Run command as root user.

- Apply these steps to all master and worker nodes

  

##### 1.1) Base Prerequisite 

Update DNS

```shell
mv /etc/resolv.conf /etc/resolv.conf.org
printf "nameserver 127.0.0.53\nnameserver 8.8.8.8\n" > /etc/resolv.conf
cat  /etc/resolv.conf
```



 Change Time Zone

```shell
timedatectl set-timezone Asia/Bangkok
date
```



Define proxy if needed.

```shell
echo "export http_proxy="  >> /etc/bash.bashrc
echo "export https_proxy="  >> /etc/bash.bashrc
echo "export no_proxy="  >> /etc/bash.bashrc
echo "export http_proxy="  >> /etc/environment
echo "export https_proxy="  >> /etc/environment
```



Turn-off swap & Permanently disable swap

```shell
swapoff -a
sed -e '/swap/ s/^#*/#/' -i /etc/fstab
```



Update your base system with the latest available packages.

```shell
sudo apt-get update
```



Disable Firewall

```shell
apt-get install ufw
ufw disable
systemctl disable ufw
```



##### 1.2) Install docker

Setup Proxy for Docker.

```shell
mkdir /etc/systemd/system/docker.service.d
echo "[Service]" > /etc/systemd/system/docker.service.d/http-proxy.conf
echo "Environment=\"http_proxy=\""  >> /etc/systemd/system/docker.service.d/http-proxy.conf
echo "Environment=\"https_proxy=\""  >> /etc/systemd/system/docker.service.d/http-proxy.conf
echo "Environment=\"no_proxy=\""  >> /etc/systemd/system/docker.service.d/http-proxy.conf
```



Turn on Json file configuration.

```shell
mkdir -p /etc/docker/
echo "{" > /etc/docker/daemon.json
echo "  \"bip\": \"10.200.1.1/16\"," >> /etc/docker/daemon.json
echo "  \"default-address-pools\": [" >> /etc/docker/daemon.json
echo "  {" >> /etc/docker/daemon.json
echo "    \"base\": \"10.200.0.0/16\"," >> /etc/docker/daemon.json
echo "    \"size\": 16" >> /etc/docker/daemon.json
echo "  }" >> /etc/docker/daemon.json
echo "  ]," >> /etc/docker/daemon.json
echo "  \"log-driver\": \"json-file\"," >> /etc/docker/daemon.json
echo "  \"log-opts\": {" >> /etc/docker/daemon.json
echo "    \"max-size\": \"5m\"," >> /etc/docker/daemon.json
echo "    \"max-file\": \"1\"" >> /etc/docker/daemon.json
echo "  }" >> /etc/docker/daemon.json
echo "}" >> /etc/docker/daemon.json
```



Install required packages to add Docker repository.

```shell
apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```



Download and add Docker’s GPG key.

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
```



Add Docker repository.

```shell
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```



Update the repository.

```shell
apt-get update
apt-cache showpkg docker-ce
```



Install Docker.

```shell
apt-get -y install docker-ce=5:19.03.5~3-0~ubuntu-bionic

docker version
```



Add your username to the docker group to avoid sudo.

```shell
usermod -aG docker root					
usermod -aG docker ubuntu
```



##### 1.4) Install Kubernetes

Download and GPG key.

```shell
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```



Add Kubernetes repository.

```shell
echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```



Update the repository

```shell
cd /tmp
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.13.12/bin/linux/amd64/kubectl
echo "deb https://packages.cloud.google.com/apt/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
apt-get update -y
```



Install Kubernetes

```shell
yes Y | apt-get install kubeadm=1.13.12-00 kubelet=1.13.12-00 kubectl=1.13.12-00 kubernetes-cni=0.7.5-00
```





### 3.) Initialize the Master Node

- Run command as root user

Initialize Master.

```shell
kubeadm init --ignore-preflight-errors=all
```



-Copy the “kubeadm join” command line printed as the result of the previous command for other Masters and Workers to join the cluster (in step 3.4 and 6) as example below.

```shell
kubeadm join 172.31.115.113:6443 --token kkjdt2.b24hf6269e4ruyaz --discovery-token-ca-cert-hash sha256:3cd9489da728c9306e1f7a979ed936e73ae15cd74fa04b6a58bb3e6bfad58c98		
```



#### 4.) Move .kube configuration files.

- Run command as ubuntu user.
- Apply this step to all Master nodes.

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



#### 5.) Apply CNI Network.

- Run command as ubuntu user.
- Apply this step to any Master node.

```shell
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml

kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```



#### 6.) Join the Cluster for Worker nodes.

- Run command as root
-  user.
- Apply this step to all Worker nodes.

Paste the “kubeadm join” command line printed as the result of the 3.1) step.

```shell
<kubeadm join ...>
```
