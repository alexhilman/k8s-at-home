# Bare Metal Hypervisor: `hydra.alexhilman.com`

I have a bare-metal Ubnutu 20.04 host running on an AMD Ryzen 7 2700 with 64 GB of RAM.

On this host I am using KVM to manage a set of virtual machines (all Ubuntu 20.04) which will provide the basic set of servers for raw infrastructure as well as the required nodes for Kubernetes.

## Hardware

| Component | Value |
| --- | --- |
| CPU | AMD Ryzen 7 2700 (8-core, 16-thread) |
| RAM | 64 GB |
| Disk | 1TB System partition, ext4 |
| Disk | 2TB ZFS storage container for VMs |
| Network | `10.0.0.10/8` |

_ZFS is limited to a maximum ARC cache size of 16GB._
