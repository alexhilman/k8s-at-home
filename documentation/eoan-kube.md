# `eoankube` System Specifications

I have a bare-metal Ubnutu 19.10 host running on an AMD FX-8150 with 16 GB of RAM.

On this host I have reserved 6-cores and 8 GB of RAM via a KVM hardware-based virtual machine named `eoankube`.

## Hardware

| Component | Value |
| --- | --- |
| CPU | 6-cores |
| RAM | 6 GB |
| Disk | 500 GB |

## Software and Configuration

| Key | Value |
| --- | --- |
| OS | Ubuntu 19.10 |
| Virtual? | Yes, via `kvm` |
| Hostname | `eoankube` |
| IP Address | `192.168.1.25` |
| Network | bridged |
