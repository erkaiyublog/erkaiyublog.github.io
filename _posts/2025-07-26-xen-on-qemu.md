---
published: true 
title: Xen on ARM in QEMU 
tags: hypervisor qemu
---

The official documentation for running Xen on ARM in QEMU is quite old (latest update on 2019 by the time this post is published). Luckily, I managed to get it working, so I'm writing this post for future reference. 

Major components and their versions:
* Linux: *6.11.7*
* Xen: *4.18.5* 
* qemu-system-aarch64: *8.2.2* 
* gcc: *13.3.0*

## Install Dependencies

```bash
sudo apt update

sudo apt install git build-essential ninja-build pkg-config libglib2.0-dev \
libpixman-1-dev libcap-ng-dev python3-pip python3-venv wget gcc qemu-system-arm \
flex bison libelf-dev bc libssl-dev curl zstd gcc-aarch64-linux-gnu
```

## Cross Compile Linux for ARM
Use the following script to compile Linux-6.11.7 for ARM.

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.11.7.tar.xz -O linux-6.11.7.tar.xz
tar Jxvf linux-6.11.7.tar.xz
rm linux-6.11.7.tar.xz

cd linux-6.11.7
ARM_FLAGS='ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-'

make $ARM_FLAGS defconfig
./scripts/config -d CONFIG_WERROR
make $ARM_FLAGS olddefconfig
make $ARM_FLAGS -j$(nproc)
```

## Download the Xen Server Disk Image 
QEMU works well with UEFI firmware for the emulated machine. To get the system working, download an Aarch64 UEFI ready distro image, 

```bash
wget https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-arm64-uefi1.img
```

## Build Xen for ARM
First, download the Xen repository.

```bash
sudo git clone https://xenbits.xen.org/git-http/xen.git
sudo chown -R $USER:$USER xen
cd xen
git checkout RELEASE-4.18.5 
```

Cross compile the Xen hypervisor.

```bash
make distclean
make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

The result of the compilation is stored as ```xen/xen/xen.efi```, this file will be used later.

## Run QEMU (1)
Use the following command to launch QEMU for the first time, the goal is to **set root password**. The MAC address used in the command is just a common convention for QEMU.

```bash
qemu-system-aarch64 \
   -machine virt,gic_version=3 -machine virtualization=true \
   -cpu cortex-a57 -machine type=virt -nographic \
   -smp 4 -m 4000 \
   -kernel linux-6.11.7/arch/arm64/boot/Image.gz --append "console=ttyAMA0 root=/dev/vda1 init=/bin/sh" \
   -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=hostnet0,mac=52:54:00:12:34:56 \
   -drive if=none,file=xenial-server-cloudimg-arm64-uefi1.img,id=hd0 -device virtio-blk-device,drive=hd0
```

### Remount Root FS and Set Root Password
Once the shell shows up in QEMU, remount the root fs:

```bash
mount -o remount,rw /dev/vda1 /
```

Then, set the root password with 
```bash
passwd
```

This command triggers a session for setting root password, set the password as a non-empty string.

### Turn Off QEMU

After setting root password, turn off QEMU by typing ```ctrl-a x```.

## Run QEMU (2)
Launch QEMU for the second time, but without the init shell. Using the following command:

```bash
qemu-system-aarch64 \
   -machine virt,gic_version=3 -machine virtualization=true \
   -cpu cortex-a57 -machine type=virt -nographic \
   -smp 4 -m 4000 \
   -kernel linux-6.11.7/arch/arm64/boot/Image.gz --append "console=ttyAMA0 root=/dev/vda1" \
   -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=hostnet0,mac=52:54:00:12:34:56 \
   -drive if=none,file=xenial-server-cloudimg-arm64-uefi1.img,id=hd0 -device virtio-blk-device,drive=hd0
```

Note that this QEMU launch will **take much longer**, a ```cloud-init``` feature during booting will take 120 seconds before its timeout.

### Set Up SSH Access
Inside the VM, create the ```.ssh/authorized_keys``` file:

```bash
mkdir -p /root/.ssh
echo "YOUR_PUBLIC_KEY_CONTENTS" >> /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

Where ```YOUR_PUBLIC_KEY_CONTENTS``` is the contents of **host machine**'s ```~/.ssh/id_rsa.pub``` or ```~/.ssh/id_ed25519.pub```.

### SSH from Host (Optional)
Now it is possible to ssh into the VM from host,

```bash
ssh -p 2222 root@localhost
```

### SCP the Xen EFI and the Dom0 Image
Copy the Xen EFI and the Dom0 image from host to QEMU with ```scp```,

```bash
scp -P2222 linux-6.11.7/arch/arm64/boot/Image.gz root@127.0.0.1:/boot/efi/kernel
scp -P2222 xen/xen/xen.efi root@127.0.0.1:/boot/efi
```

### SCP the Xen Config File and the Device Tree Blob
Write a Xen config file named ```xen.cfg``` on the host machine, 

```
 options=console=dtuart noreboot dom0_mem=512M
 kernel=kernel root=/dev/vda1 init=/bin/sh rw console=hvc0
 dtb=virt-gicv3.dtb
```

SCP the config file into the QEMU VM,

```bash
scp -P2222 xen.cfg root@127.0.0.1:/boot/efi
``` 

On the host machine, generate a device tree blob file with the following command,

```bash
qemu-system-aarch64 \
   -machine virt,gic_version=3 \
   -machine virtualization=true \
   -cpu cortex-a57 -machine type=virt \
   -smp 4 -m 4096 -display none \
   -machine dumpdtb=virt-gicv3.dtb
```

Before copying the device tree blob file, it needs to be modified with device tree compiler. Install device tree compiler:

```bash
sudo apt install device-tree-compiler
```

Update the ```dtb``` file:

```bash
dtc -I dtb -O dts virt-gicv3.dtb > virt-gicv3.dts ;
sed 's/compatible = "arm,pl061.*/status = "disabled";/g' virt-gicv3.dts > virt-gicv3-edited.dts ;
dtc -I dts -O dtb virt-gicv3-edited.dts > virt-gicv3.dtb ;
rm virt-gicv3-edited.dts virt-gicv3.dts
```

SCP the config file into the QEMU VM,

```bash
scp -P2222 virt-gicv3.dtb root@127.0.0.1:/boot/efi
```

### Turn Off QEMU
With all the files copied, turn off QEMU by typing ```ctrl-a x```.

## Obtain the QEMU EFI Binary
Install ```qemu-efi-aarch64``` on the host machine, it comes with a QEMU EFI binary.

```bash
sudo apt install qemu-efi-aarch64
```

Copy the QEMU EFI file to the working directory,

```bash
cp /usr/share/qemu-efi-aarch64/QEMU_EFI.fd QEMU_EFI.bin
```

## Run QEMU (3)
Finally, start the Xen emulation with:

```bash
qemu-system-aarch64 \
   -machine virt,gic_version=3 \
   -machine virtualization=true \
   -cpu cortex-a57 -machine type=virt \
   -smp 4 -m 4096 -display none \
   -serial mon:stdio \
   -bios QEMU_EFI.bin \
   -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=hostnet0,mac=52:54:00:12:34:56 \
   -drive if=none,file=xenial-server-cloudimg-arm64-uefi1.img,id=hd0 -device virtio-blk-device,drive=hd0 -boot order=d
```

### Launch Xen System
Once the QEMU emulation starts, keep on pressing the ```esc``` key to enter the UEFI manual, pick ```Boot Manager```, then ```EFI Internal Shell```. 

In the shell, type the commands below to start Dom0, with the ```ls``` command, you are supposed to see ```kernel```, ```xen.cfg```, and ```virt-gicv3.dtb``` being listed.

```
Shell> fs0:
FS0:\> ls
FS0:\> xen.efi
```

## References
1. [Xen ARM with Virtualization Extensions/qemu-system-aarch64](https://wiki.xenproject.org/wiki/Xen_ARM_with_Virtualization_Extensions/qemu-system-aarch64).
2. [Building Xen on ARM](https://wiki.xenproject.org/wiki/Xen_ARM_with_Virtualization_Extensions/qemu-system-aarch64#Install_Xen).
5. [How do I run XEN on arm64 and Qemu 6.0.0 with Linux 4.20.11 as Dom0](https://stackoverflow.com/questions/68372448/how-do-i-run-xen-on-arm64-and-qemu-6-0-0-with-linux-4-20-11-as-dom0).
6. [qemu-alpine-arm64.sh](https://github.com/xen-project/xen/blob/9bc9fff04ba077c4a9782f12578362d8947c534b/automation/scripts/qemu-alpine-arm64.sh).
