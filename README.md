# K8S at Home

K8S at Home name is documentation for myself as well as a guide to other developers they may set up their own Kubernetes infrastructure for their own use. This stack will be for long-term use (at-home production) and has the capability to be scaled out at a later time.

Kubernetes has a fairly steep learning curve; this repository contains condensed information to get bring devs up to speed fairly quickly.

On a final note, we will be using a [management tool called `helm`](https://github.com/helm/helm) to make things easier for us to install the vanilla services we need in the stack. 

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

_At the time of writing Docker is not directly supported on Ubuntu 19.10, so we'll need to use the packages distributed for 19.04 (`disco`)._

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
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

## License

This project uses the following license: [Apache 2.0 License](LICENSE).