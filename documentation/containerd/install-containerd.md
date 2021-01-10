# Install `containerd`

1. Configure modules and the system

    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    # Setup required sysctl params, these persist across reboots.
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF
    
    # Apply sysctl params without reboot
    sudo sysctl --system
    ```
1. Install `containerd`

    ```bash
    sudo apt-get update && sudo apt-get install -y containerd
    ```
1. Configure `containerd` with the initial defaults

    ```bash
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    ```
1. Configure `containerd` to use SystemD as the cgroup driver by editing `/etc/containerd/config.toml`
    ```toml
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      ...
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
    ```

1. Restart `containerd`

    ```bash
    sudo systemctl restart containerd
    ```