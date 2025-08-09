---
published: true 
title: Enabling Networking in QEMU VM with Ubuntu Cloud Server Image 
tags: qemu Tips
---

This is just a short post logging something I newly learned. It turned out that when launching VMs with QEMU, if the official cloud server images (available at [this link](https://cloud-images.ubuntu.com/)) by Ubuntu is used, the networking will be missing by default (seemed to be related to some cloud-init issues).

To fix it, I referred to [this page](https://documentation.ubuntu.com/public-images/public-images-how-to/launch-qcow-with-qemu/), the simple takeaway here is that adding a home-made ```seed.img``` file can fix this issue.

For example, the original command I used to launch a Ubuntu VM using a x86 cloud server disk image was:

```bash
qemu-system-x86_64 \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 -m 4000 \
  -nographic \
  -kernel linux-6.11.7/arch/x86/boot/bzImage \
  --append "console=ttyS0 root=/dev/vda1" \
  -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=hostnet0,mac=52:54:00:12:34:56 \
  -drive if=none,file=noble-server-cloudimg-amd64.img,id=hd0 \
  -device virtio-blk-pci,drive=hd0
```

However, to get networking, I need to first install cloud-image-utils with

```bash
sudo apt install cloud-image-utils
```

Then create a file named ```user-data```:

```
#cloud-config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
```

And a file named ```meta-data```:

```
instance-id: iid-001
```

Then create the ```seed.img```:

```bash
cloud-localds seed.img user-data meta-data
```

Add this ```seed.img``` into the QEMU command, and network is configured correctly.

```bash
qemu-system-x86_64 \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 -m 4000 \
  -nographic \
  -kernel linux-6.11.7/arch/x86/boot/bzImage \
  --append "console=ttyS0 root=/dev/vda1" \
  -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=hostnet0,mac=52:54:00:12:34:56 \
  -drive if=none,file=noble-server-cloudimg-amd64.img,id=hd0 \
  -drive if=virtio,format=raw,file=seed.img \
  -device virtio-blk-pci,drive=hd0
```

Meanwhile, when I tested cloud images for **ARM64**, it turned out that this extra step is not necessary, the cloud-init feature came out of the box.

If you happen to catch what mistake I made here in the QEMU commands, please let me know!