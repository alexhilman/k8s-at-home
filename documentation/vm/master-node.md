# Master Node

I have three identical master nodes:
* `kmaster-01.alexhilman.com`
* `kmaster-02.alexhilman.com`
* `kmaster-03.alexhilman.com`

From the KVM host:

```bash
virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kmaster-01.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kmaster-01.alexhilman.com.qcow2 32G
virsh autostart kmaster-01.alexhilman.com
virsh setmem kmaster-01.alexhilman.com $((2*1024*1024)) --config
virsh setvcpus kmaster-01.alexhilman.com 2 --config
virsh start kmaster-01.alexhilman.com

virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kmaster-02.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kmaster-02.alexhilman.com.qcow2 32G
virsh autostart kmaster-02.alexhilman.com
virsh setmem kmaster-02.alexhilman.com $((2*1024*1024)) --config
virsh setvcpus kmaster-02.alexhilman.com 2 --config
virsh start kmaster-02.alexhilman.com

virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kmaster-03.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kmaster-03.alexhilman.com.qcow2 32G
virsh autostart kmaster-03.alexhilman.com
virsh setmem kmaster-03.alexhilman.com $((2*1024*1024)) --config
virsh setvcpus kmaster-03.alexhilman.com 2 --config
virsh start kmaster-03.alexhilman.com
```

_I assigned the IP addresses via MAC address on my router in a static DHCP registry and created DNS names matching these hosts._

## Guest Setup

Connect to each host in sequence and set the hostname
```bash
sudo hostnamectl set-hostname # kmaster-01.alexhilman.com
```
* I specifically commented out the hostname to force the user to think about the system they are connecting to

Then run this script

```bash
sudo sed -i s/focal-template/$(hostname --short)/g /etc/hosts
sudo sgdisk --move-second-header /dev/sda
sudo parted /dev/sda resizepart 3 100%
sudo pvresize /dev/sda3 
sudo lvextend /dev/ubuntu-vg/ubuntu-lv /dev/sda3 --resizefs
```