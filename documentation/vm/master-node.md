# Master Node

From the KVM host:

```bash
virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kmaster-01.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kmaster-01.alexhilman.com.qcow2 32G
virsh autostart kmaster-01.alexhilman.com
virsh setmem kmaster-01.alexhilman.com $((3*1024*1024)) --config
virsh setvcpus kmaster-01.alexhilman.com 2 --config
virsh start kmaster-01.alexhilman.com
```

_I assigned the IP addresses via MAC address on my router in a static DHCP registry and created DNS names matching these hosts._

## Guest Setup

1. Assign the host name to the new VM
    ```bash
    sudo hostnamectl set-hostname kmaster-01.alexhilman.com
    ```
1. Expand the disk and correct the `hosts` file

```bash
sudo sed -i s/focal-template/$(hostname --short)/g /etc/hosts
sudo sgdisk --move-second-header /dev/sda
sudo parted /dev/sda resizepart 3 100%
sudo pvresize /dev/sda3 
sudo lvextend /dev/ubuntu-vg/ubuntu-lv /dev/sda3 --resizefs
```
