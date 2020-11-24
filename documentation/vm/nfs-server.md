# NFS Server: `nfs.alexhilman.com`

1. From the KVM Host:

```bash
virt-clone --auto-clone \
           --mac RANDOM \
           --original focal-template \
           --name nfs.alexhilman.com
virsh autostart nfs.alexhilman.com
virsh setmem nfs.alexhilman.com $((4*1024*1024)) --config
virsh setvcpus nfs.alexhilman.com 4 --config
sudo qemu-img create -f qcow2 \
                     /var/lib/libvirt/images/nfs.alexhilman.com-k8sdata.qcow2 \
                     256G
virsh attach-disk nfs.alexhilman.com \
                  /var/lib/libvirt/images/nfs.alexhilman.com-k8sdata.qcow2 \
                  sdb \
                  --subdriver qcow2 \
                  --config
virsh start nfs.alexhilman.com
```

## Guest Setup

I want to use ZFS for my backing storage for my NFS volumes. This affords me the safety to snapshot an application's K8s volume just before app updates in my pods; a backup without a large file copy to another drive.

```bash
sudo hostnamectl set-hostname nfs.alexhilman.com
sudo sed -i s/focal-template/nfs/g /etc/hosts
sudo apt install -y nfs-kernel-server zfsutils-linux
sudo parted /dev/sdb mklabel gpt
sudo zpool create -m none -o autoexpand=on root /dev/sdb
sudo zfs create -o mountpoint=/nfs root/nfs
```