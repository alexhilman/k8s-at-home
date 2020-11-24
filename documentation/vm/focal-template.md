# Template VM: `focal-template`

1. From the KVM host

    ```bash
    virt-install --name focal-template \
                 --description "Template Ubuntu 20.04 Server" \
                 --hvm \
                 --os-type generic \
                 --graphics none \
                 --network bridge=br0,model=virtio \
                 --location $HOME/iso/ubuntu-20.04.1-live-server-amd64.iso,kernel=casper/vmlinuz,initrd=casper/initrd \
                 --memory memory=16384,currentMemory=2048 \
                 --vcpus 2,maxvcpus=16 \
                 --disk size=16,sparse=true \
                 --autostart \
                 --extra-args 'console=ttyS0'
    ```
1. Proceed with the server install enabling only the SSH server option
1. Default disk layout with LVM enabled

## Guest Setup

1. Install `htop`
1. Trust an SSH key from a client host
