## K8s version- 1.28.0 install on Ubuntu 22.04.2 LTS
Official Documentation Link: https://www.cherryservers.com/blog/install-kubernetes-on-ubuntu
Official Youtube Link: https://www.youtube.com/watch?v=lede7TJk1PE


## For master node 

### Step 1: Set up hostnames (For master and worker nodes)
    vi /etc/hostname
    vi /etc/hosts  127.0.1.1 k8s-master

### Step 2: Update the /etc/hosts File for Hostname Resolution

Setting up host names is not enough. We have to map hostnames to their IP addresses as well. 
You should update the /etc/hosts file of all nodes(or at least of the master node), as shown below. 
(Remember that you have to use the IP addresses of your nodes. I have only given holder values.) 

### To open hosts file use the Command
    vi /etc/hosts
### In the hosts file we have to add ip address to all nodes like (For master and worker nodes):
    10.209.99.10 k8s-master--- master node ip address
    10.209.99.11 k8s-worker1--- worker1 node ip address
    10.209.99.12 k8s-worker2--- worker2 node ip address
    
### Step 3: Disable swap (For master and worker nodes)

    swapoff -a
    sed -i '/ swap / s/^/#/' /etc/fstab
    
### Step 4: Set up the IPV4 bridge on all nodes 
to configure the IPV4 bridge on all nodes, execute the following commands on each node (For master and worker nodes).

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    # Apply sysctl params without reboot
    sudo sysctl --system
### Step 5: Install kubelet, kubeadm, and kubectl on each node

Let’s install kubelet, kubeadm, and kubectl on each node to create a Kubernetes cluster. 
They play an important role in managing a Kubernetes cluster.

Then we need to install kubeadm, which is used to bootstrap a 
Kubernetes cluster, including setting up the master node and helping worker nodes join the cluster.

Kubectl is a CLI tool for Kubernetes to run commands to perform various actions such 
as deploying applications, inspecting resources, and managing cluster operations directly from the terminal.

Before installing them, you must update the package index with the command (For master and worker nodes):

    apt-get update
Next, we have to ensure that we can download and install packages from the internet securely (For master and worker nodes):

    sudo apt-get install -y apt-transport-https ca-certificates curl
This key is important to verify that the Kubernetes packages we download are genuine and haven't been tampered with (For master and worker nodes):

    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
Next, we need to tell the apt package manager where to find Kubernetes packages for downloading (For master and worker nodes):

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
Let’s refresh the apt package index to see new items by running the update command again (For master and worker nodes)

    apt-get update
Now we are ready to install kubelet, kubeadm, and kubectl by running the command below (for specific version) For (master and worker nodes):

    apt install -y kubelet=1.28.0-00 kubeadm=1.28.0-00 kubectl=1.28.0-00
Step 5: Install Docker (For master and worker nodes):

    sudo apt install docker.io
Next, configure containerd on all nodes to ensure its compatibility with Kubernetes. First, create a folder for the configuration file with the (For master and worker nodes).

    sudo mkdir /etc/containerd
Then, create a default configuration file for containerd and save it as config.toml using the command (For master and worker nodes):

    sudo sh -c "containerd config default > /etc/containerd/config.toml"
After running these commands, you need to modify the config.toml file to locate the entry that sets "SystemdCgroup" to false and changes its value to true. 
This is important because Kubernetes requires all its components, and the container runtime uses systemd for cgroups (For master and worker nodes).

    sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml

Next, restart containerd and kubelet services to apply the changes you made on all nodes (For master and worker nodes).

    sudo systemctl restart kubelet.service
    sudo systemctl restart containerd.service
You will want to start kubelet service whenever the machine boots up, which you can do by running the command below (For master and worker nodes):

    sudo systemctl enable kubelet.service

### Step 6: Initialize the Kubernetes cluster on the master node:

When you initialize a Kubernetes control plane using kubeadm, several components are deployed to manage and orchestrate the cluster. 
Some examples of these components are kube-apiserver, kube-controller-manager, kube-scheduler, etcd, kube-proxy. 
‘We need to download the images of these components by running the following command Only for master node.

    sudo kubeadm config images pull
Next, initialize your master node. The --pod-network-cidr flag is setting the IP address range for the pod network for master node.

    sudo kubeadm init --pod-network-cidr=10.10.0.0/16
To manage the cluster, you should configure kubectl on the master node. 
Create the .kube directory in your home folder and copy the cluster's admin configuration to your personal .
kube directory. Next, change the ownership of the copied configuration file to give the user the permission 
to use the configuration file to interact with the cluster.

Here are the commands you need to do this on master node.

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Step 7: Configure kubectl and Calico Run the following commands on the master node to deploy the Calico operator:

    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
Next, download the custom resources file for Calico, which contains definitions of the various resources that Calico will use.

    curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O

Using the following command, modify the CIDR in the downloaded custom resources to match your pod network. 
Here, you're using the sed command to change the default CIDR value in the Calico custom resources to match the CIDR you used in the kubeadm init command.

    sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.10.0.0\/16/g' custom-resources.yaml
Finally, tell kubectl to create the resources defined in the custom-resources.yaml file.

    kubectl create -f custom-resources.yaml
### Step 8: Add worker nodes to the cluster only for worker node:
    kubeadm join 10.209.99.150:6443 --token 4ohoxu.pmnnk0gmtffta1rk         --discovery-token-ca-cert-hash sha256:859ac308bf59b59ac3a344799e840cc8b8651e8fa5453f9da23f0685ada550ee
### Step 9: Verify the cluster and test
Finally, we want to verify whether our cluster is successfully created. By running the kubectl get no command, we can list all nodes that are part of the cluster

    kubectl get no
we can list the pods from all namespaces in the cluster  by running the command:

    kubectl get po -A 














