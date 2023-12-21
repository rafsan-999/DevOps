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

