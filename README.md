# K8S at Home

**This is currently a work-in-progress. This note will be removed once the setup is fully documented.**

K8S at Home name is documentation for myself as well as a guide to other administrators who wish to set up their own Kubernetes infrastructure on a small server.

Kubernetes has a fairly steep learning curve; this repository contains condensed information to bring administrators up to speed fairly quickly. Anecdotes will be provided via links to other documentation spaces in this repository to provide more detail.

On a final note, we will be using a [management tool called `helm`](https://github.com/helm/helm) to make things easier for us to install the vanilla services we need in the stack. 

## License

This project uses the following license: [Apache 2.0 License](LICENSE).

## Prerequisites

Before you begin, ensure you have
* a working understanding of Linux operating systems and utilities like SSH and Bash;
* a basic understanding of containerization; ideally with [Docker](https://docs.docker.com/); and
* access to one or more servers in which to create nodes for K8s with the following minimum specs (each)
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

### Node Prep: Master and Worker

1. Install [Docker for Kubernetes](documentation/docker/install-docker-for-k8s.md)
1. Install [`kubeadm`, `kubelet`, and `kubectl`](documentation/k8s/install-tools.md)

### Primary Master Node

**Only on `kmaster-01`**.

1. Initialize your first Kubernetes node

    ```shell script
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=$(hostname --fqdn)
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

```bash
mkdir $HOME/.kube
chmod go-rwx $HOME/.kube
scp kmaster-01.alexhilman.com:.kube/config $HOME/.kube
```

### Worker Nodes

I'm going to use three [identical virtual machines](documentation/vm/worker-node.md) for applications and services.

### Helm

```shell script
sudo snap install helm --classic
helm init
```

### NFS Server

This part may be a bit complicated as it is a volume-in-volume approach, so I'll describe what we are going to do:

1. Make a `local` Kubernetes volume and claim
1. Install an NFS service in K8s

This NFS service will provide a network-available storage solution to other pods as exposed NFS volumes. Each NFS volume will be hosted on the single bare-metal `local` volume (and claim) on the node in step 1.

1. Create the `local` volume's storage directory on the host which will serve the NFS service

    ```shell script
    sudo mkdir -p /srv/k8s/nfs-server
    ```
1. Create the NFS volumes (change any values in these files to suit your needs)

    ```shell script
    kubectl apply -f nfs/volumes/nfs-server-pv.yml
    kubectl apply -f nfs/volumes/nfs-server-pvc.yml
    ```

1. Create the NFS service

    ```shell script
    kubectl apply -f nfs/nfs.yml
