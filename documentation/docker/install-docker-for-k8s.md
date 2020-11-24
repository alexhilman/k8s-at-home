# Install Docker for Kubernetes

Condensed from https://kubernetes.io/docs/setup

1. Install a supported version of Docker CE

    ```shell script
    sudo apt-get update
    sudo apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    sudo apt-get update && sudo apt-get install -y \
      containerd.io=1.2.13-2 \
      docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
      docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
    sudo usermod --append --groups docker "$USER"
    ```
    1. Docker should work for you. To make sure, [here's a simple test](test-docker.md)

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
    sudo systemctl enable docker
    ```