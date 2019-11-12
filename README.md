# K8S at Home

K8S at Home name is documentation for myself as well as a guide to other developers they may set up their own Kubernetes infrastructure for their own use. This stack will be for long-term use (at-home production) and has the capability to be scaled out at a later time.

Kubernetes has a fairly steep learning curve; this repository contains condensed information to get bring devs up to speed fairly quickly.

On a final note, we will be using a [management tool called `helm`](https://github.com/helm/helm) to make things easier for us to install the vanilla services we need in the stack. 

## License

This project uses the following license: [Apache 2.0 License](LICENSE).

## Prerequisites

Before you begin, ensure you have met the following requirements:
* You have a working understanding of Linux operating systems and utilities like SSH
* You have a basic understanding of containerization; ideally with [Docker](https://docs.docker.com/)
* You have a bare-metal or vm-based Linux distribution with the following requirements:
    * [`iptables` version >= 1.6.2](documentation/iptables-race-condition.md)
    * at least 4 GB of RAM
    * at least 2 CPU cores
    * at least 100 GB of free disk space

[Here are my system specs for this example](documentation/eoan-kube.md).

## Preparing your Stack

### Static IP

I highly recommended that you pick a static IP for your first Kubernetes node. This will be your `master` node.

This guide uses `192.168.1.25/24` as that is my server's IP on my network. This IP address has been defined under the host name `eoankube`.

## Installation

### Docker

These instructions are condensed from the [Ubuntu Docker install guide](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

_At the time of writing, Docker is not directly supported on Ubuntu 19.10, so we'll need to use the packages distributed for 19.04 (`disco`)._

```shell script
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   disco \
   stable"
sudo apt-get install -y docker-ce=18.09 docker-ce-cli containerd.io
sudo usermod --append --groups docker "$USER"
```

Docker should work for you. Here's a simple test:

```text
$ sudo docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### Kubernetes

Condensed from https://kubernetes.io/docs/setup

1. Configure Docker for use with `systemd` as the control group driver (note: if you already have Docker configured, then you will need to merge the configs instead of this overwrite operation)

    ```shell script
    cat <<EOF | sudo tee /etc/docker/daemon.json >/dev/null
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
    EOF
    sudo mkdir -p /etc/systemd/system/docker.service.d
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```
1. Kubernetes doesn't play well with `swap`, so we'll disable it permanently

    ```shell script
    sudo swapoff -a
    sudo sed --in-place --regexp-extended '/none\s+swap\s+sw\s+/d' /etc/fstab
    ```
1. Kubernetes (and Docker) rely on the legacy `iptables` functions to route packets and lock down communications. Since Ubuntu 19.04 iptables uses `nftables` as a back-end for `iptables`; to work around this we need to switch to legacy mode:

    ```shell script
    sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
    sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
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
1. Initialize your `kubelet`

    ```shell script
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=eoankube
    ```
    1. There will be a warning: Kubernetes is currently at version 1.16.2 which lists the maximum supported version of Docker at 18.09; since we are using the latest Docker (19.3) we'll need to accept this to continue
        1. Kubernetes will still work in this condition

1. To easily interact with the cluster with your Linux user, run the following commands:

    ```shell script
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1. Apply the Flannel networking add-on

    ```shell script
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
    ```

Your instance should be up and running and ready to accept deployments. Here's a simple test:

```text
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3m51s
```

### Single-Node Installs

If you only intend to run one and only one node, then you will need to make the master node available for pod scheduling. However, **if you want multiple nodes for your cluster, you can skip this step**.

```shell script
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Helm

1. Create a service account for helm usage


```shell script
sudo snap install helm --classic
helm init
```
