## K8s version- 1.28.0 install on Ubuntu 22.04.2 LTS
Official Documentation Link: https://www.cherryservers.com/blog/install-kubernetes-on-ubuntu
Official Youtube Link: https://www.youtube.com/watch?v=lede7TJk1PE


## For master node 

### Step 1: Set up hostnames
    vi /etc/hostname
    vi /etc/hosts  127.0.1.1 k8s-master

### Step 2: Update the /etc/hosts File for Hostname Resolution
Setting up host names is not enough. We have to map hostnames to their IP addresses as well. 
You should update the /etc/hosts file of all nodes(or at least of the master node), as shown below. 
(Remember that you have to use the IP addresses of your nodes. I have only given holder values.) 

### To open hosts file use the Command
    vi /etc/hosts
## In the hosts file we have to add ip address to all nodes like:    
    10.209.99.10 k8s-master--- master node ip address
    10.209.99.11 k8s-worker1--- worker1 node ip address
    10.209.99.12 k8s-worker2--- worker2 node ip address
    
## Step 3: Set up the IPV4 bridge on all nodes 
to configure the IPV4 bridge on all nodes, execute the following commands on each node.

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
## Step 5: Install kubelet, kubeadm, and kubectl on each node

Let’s install kubelet, kubeadm, and kubectl on each node to create a Kubernetes cluster. 
They play an important role in managing a Kubernetes cluster.

Then we need to install kubeadm, which is used to bootstrap a 
Kubernetes cluster, including setting up the master node and helping worker nodes join the cluster.

Kubectl is a CLI tool for Kubernetes to run commands to perform various actions such 
as deploying applications, inspecting resources, and managing cluster operations directly from the terminal.

Before installing them, you must update the package index with the command:

    apt-get update
Next, we have to ensure that we can download and install packages from the internet securely.

    sudo apt-get install -y apt-transport-https ca-certificates curl
This key is important to verify that the Kubernetes packages we download are genuine and haven't been tampered with.

    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
Next, we need to tell the apt package manager where to find Kubernetes packages for downloading.

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
Let’s refresh the apt package index to see new items by running the update command again.

    apt-get update
Now we are ready to install kubelet, kubeadm, and kubectl by running the command below (for specific version).

    apt install -y kubelet=1.28.0-00 kubeadm=1.28.0-00 kubectl=1.28.0-00
Step 5: Install Docker

    sudo apt install docker.io
Next, configure containerd on all nodes to ensure its compatibility with Kubernetes. First, create a folder for the configuration file with the 

    sudo mkdir /etc/containerd
Then, create a default configuration file for containerd and save it as config.toml using the command:

    sudo sh -c "containerd config default > /etc/containerd/config.toml"
After running these commands, you need to modify the config.toml file to locate the entry that sets "SystemdCgroup" to false and changes its value to true. 
This is important because Kubernetes requires all its components, and the container runtime uses systemd for cgroups.

    sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml

Next, restart containerd and kubelet services to apply the changes you made on all nodes.

    sudo systemctl restart kubelet.service
    sudo systemctl restart containerd.service
You will want to start kubelet service whenever the machine boots up, which you can do by running the command below:

    sudo systemctl enable kubelet.service














