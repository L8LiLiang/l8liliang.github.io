---
layout: article
tags: Linux
title: image mode
mathjax: true
key: Linux
---

## covert image mode to qcow2
```
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_image_mode_for_rhel_to_build_deploy_and_manage_operating_systems/creating-bootc-compatible-base-disk-images-with-bootc-image-builder_using-image-mode-for-rhel-to-build-deploy-and-manage-operating-systems#creating-qcow2-images-by-using-bootc-image-builder_creating-bootc-compatible-base-disk-images-with-bootc-image-builder
https://docs.fedoraproject.org/en-US/bootc/qemu-and-libvirt/

podman login quay.io

# Create config.toml for bootc-image-builder
[[customizations.user]]
name = "root"
password = "redhat"
groups = ["root"]

# Pull the bootable container image
CONTAINER_IMAGE=images.paas.redhat.com/bootc/rhel-bootc:latest-9.6
sudo podman pull $CONTAINER_IMAGE
# Create an output directory to write the qcow2 file to
mkdir -p output

sudo podman run \
    --rm \
    -it \
    --privileged \
    --pull=newer \
    --security-opt label=type:unconfined_t \
    -v $(pwd)/output:/output \
    -v ./config.toml:/config.toml \
    -v /var/lib/containers/storage:/var/lib/containers/storage \
    quay.io/centos-bootc/bootc-image-builder:latest \
    --local \
    --type qcow2 \
    --config /config.toml \
    $CONTAINER_IMAGE
```


## install with libvirt
```
rhel_version=rhel$(rpm -E %rhel)
  
# libvirt && kvm
yum -y install virt-install
yum -y install libvirt
yum install -y python3-lxml.x86_64
rpm -qa | grep qemu-kvm >/dev/null || yum -y install qemu-kvm
if (($rhel_version < 7)); then
        service libvirtd restart
else
        systemctl restart libvirtd
        systemctl start virtlogd.socket
fi

# work around for failure of virt-install
chmod 666 /dev/kvm


# define default vnet
virsh net-define /usr/share/libvirt/networks/default.xml
virsh net-start default
virsh net-autostart default

# define vm name and mac
vm_name=v3
mac4vm=a4:a4:a4:a4:a4:a3

# download image
#wget http://netqe-bj.usersys.redhat.com/share/vms/rhel8.4.qcow2 -O /var/lib/libvirt/images/$vm_name.qcow2
\cp /root/output/qcow2/disk.qcow2 /var/lib/libvirt/images/$vm_name.qcow2 
# install vm
virt-install \
        --name $vm_name \
        --vcpus=2 \
        --ram=2048 \
        --disk path=/var/lib/libvirt/images/$vm_name.qcow2,device=disk,bus=virtio,format=qcow2 \
        --network bridge=virbr0,model=virtio,mac=$mac4vm \
        --boot hd \
        --accelerate \
        --graphics vnc,listen=0.0.0.0 \
        --force \
	--os-variant=rhel-unknown \
        --noautoconsol 

```

## build aarch64 image
```
# cat Containerfile 
FROM registry.stage.redhat.io/rhel9/rhel-bootc:9.5

RUN echo "root:redhat" | chpasswd
ADD ./resolv.conf /etc/resolv.conf
RUN echo 1 > /etc/secert

#ADD http://lab-02.rhts.eng.rdu.redhat.com/beaker/anamon3 /usr/local/sbin/anamon
#RUN chmod 755 /usr/local/sbin/anamon
#
ADD https://certs.corp.redhat.com/certs/Current-IT-Root-CAs.pem  /etc/pki/ca-trust/source/anchors/Current-IT-Root-CAs.pem
RUN update-ca-trust
#
#RUN dnf -y install restraint restraint-rhts

RUN dnf -y clean all

RUN bootc container lint

podman login quay.io
podman login registry.stage.redhat.io

# when executing on aarch64 ststem
podman build -t rhel95 -f Containerfile .
# or podman build -t rhel95 -f Containerfile --platform linux/arm64 .

podman tag localhost/rhel95  quay.io/liali/rhel9.5-aarch64

podman push quay.io/liali/rhel9.5-aarch64
```
