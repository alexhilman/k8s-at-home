# Install Kubernetes Toolset: `kubeadm`, `kubelet`, and `kubectl`

1. Kubernetes doesn't play well with `swap`, so we'll disable it permanently

    ```shell script
    sudo swapoff -a
    sudo sed --in-place --regexp-extended '/none\s+swap\s+sw\s+/d' /etc/fstab
    ```
1. Configure modules and system for Kubernetes

    ```shell script
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    br_netfilter
    EOF
    
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sudo sysctl --system
    ```
1. Add the k8s repository and required packages

    ```shell script
    sudo apt-get update && sudo apt-get install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
