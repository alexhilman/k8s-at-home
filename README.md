# K8S at Home

**This is currently a work-in-progress. This note will be removed once the setup is fully documented.**

K8S at Home name is documentation for myself as well as a guide to other administrators who wish to set up their own Kubernetes infrastructure on a small server.

Kubernetes has a fairly steep learning curve; this repository contains condensed information to bring administrators up to speed fairly quickly. The primary documentation is in abridged format with more interesting details linked to in other pages.

On a final note, we will be using a [management tool called `helm`](https://github.com/helm/helm) to make things easier for us to install the vanilla services we need in the stack. 

## License

This project uses the following license: [Apache 2.0 License](LICENSE).

## Prerequisites

Before you begin, ensure you have
* a working understanding of Linux operating systems and utilities like SSH and Bash;
* a basic understanding of containerization; and
* access to one or more servers (or VMs) which will become nodes for K8s - each with the following minimum specs
    * [`iptables` version >= 1.6.2](documentation/iptables-race-condition.md)
    * at least 4 GB of RAM
    * at least 2 CPU cores
    * at least 100 GB of free disk space (more or less; depends on your intended use-case for the node)

[Here are my system specs for this example](documentation/hydra.md).

## Preparing your Stack

For a minimum stack we will need

1. A file server (NFS) accessible by all K8s nodes
1. 1 K8s master node
1. 1 K8s worker node

_If you desire HA services then you should consider placing additional worker nodes in your cluster._

### NFS Server

Install a [NFS server](documentation/vm/nfs-server.md) for use with Kubernetes.

## K8s Installation

Ultimate goal: minimum requirements for a HA cluster (3 master nodes; 3 worker nodes) with the following services/applications added to it
1. HTTP(S) Ingress
1. Let's Encrypt CA for automated certificate installation
1. Smallstep CA for internal network identities; primarily for signed SSH certificates
1. GitLab CE for code, CI, etc
1. Nexus OSS for artifact storage (I'm a Java developer)
1. Room to grow

### Node Prep: Master and Workers

1. Install [`containerd`](documentation/containerd/install-containerd.md)
1. Install [`kubeadm`, `kubelet`, and `kubectl`](documentation/k8s/install-tools.md)

### Master Node

_You will need two terminals to your master node open at once._

1. Initialize your master Kubernetes node _and very quickly run the next step in your second terminal_

    ```shell script
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```
1. Wait for the following output line from the `kubeadm init` command:
    
    ```plain
    Waiting for the kubelet to boot up the control plane as static Pods from directory
    ```
    
    Now run the following script on your **second** session
    
    ```shell script
    echo "cgroupDriver: systemd" | sudo tee --append /var/lib/kubelet/config.yaml
    sudo systemctl restart kubelet
    ```
    1. If the above commands were issued on time, then the master node should initialize correctly
    1. **TODO** find a way of making this seamless; this seems really janky
1. Note and save the printed `kubeadm join` command in the first terminal's output; this will be used for any worker nodes you wish to add to your cluster
1. Copy the Kubernetes config in the master node to your Linux user's home directory:
    ```shell script
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1. Apply the [Flannel networking add-on](https://github.com/coreos/flannel#flannel)

    ```shell script
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```

Your instance should be up and running and ready to accept deployments. Here's a simple test:

```text
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3m51s
```

### Single-Node Installs

**If** you only intend to run one and only one node (master and worker), then you will need to make the master node available for pod scheduling. However, **if you want multiple nodes for your cluster, you can skip this step**.

```shell script
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Client PC Prep: `kubectl` administration

Install `kubectl` to your client PC using the instructions for your distribution.

Copy the master node's Kube config file to your client PC

```shell script
scp -rp kmaster-01.alexhilman.com:.kube $HOME
```

### Worker Nodes

I'm going to use three [identical virtual machines](documentation/vm/worker-node.md) for applications and services.

Run these steps on **each** worker node:

1. Run your `kubeadm join` command **without the `--control-plane` argument** which was printed out in your master node's initialization output
1. Configure `kubelet` for `systemd`

    ```shell script
    echo "cgroupDriver: systemd" | sudo tee --append /var/lib/kubelet/config.yaml
    sudo systemctl restart kubelet
    ```
### Helm

```shell script
sudo snap install helm --classic
helm init
```
