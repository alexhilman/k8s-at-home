# Worker Node

I have three worker nodes:
* `kworker-01.alexhilman.com`
* `kworker-02.alexhilman.com`
* `kworker-03.alexhilman.com`

From the KVM host:

```bash
virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kworker-01.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kworker-01.alexhilman.com.qcow2 32G
virsh autostart kworker-01.alexhilman.com
virsh setmem kworker-01.alexhilman.com $((12*1024*1024)) --config
virsh setvcpus kworker-01.alexhilman.com 4 --config
virsh start kworker-01.alexhilman.com

virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kworker-02.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kworker-02.alexhilman.com.qcow2 32G
virsh autostart kworker-02.alexhilman.com
virsh setmem kworker-02.alexhilman.com $((12*1024*1024)) --config
virsh setvcpus kworker-02.alexhilman.com 4 --config
virsh start kworker-02.alexhilman.com

virt-clone --auto-clone \
           --original focal-template \
           --mac RANDOM \
           --name kworker-03.alexhilman.com
sudo qemu-img resize /var/lib/libvirt/images/kworker-03.alexhilman.com.qcow2 32G
virsh autostart kworker-03.alexhilman.com
virsh setmem kworker-03.alexhilman.com $((12*1024*1024)) --config
virsh setvcpus kworker-03.alexhilman.com 4 --config
virsh start kworker-03.alexhilman.com
```

_I assigned the IP addresses via MAC address on my router in a static DHCP registry and created DNS names matching these hosts._

## Guest Setup

Connect to each host in sequence and set the hostname
```bash
sudo hostnamectl set-hostname # kworker-01.alexhilman.com
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
