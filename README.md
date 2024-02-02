# Build a minimal linux system ( Abstract )
This are my personal notes on how to build a minimal linux system from scratch. The goal is to understand the boot process and the minimal requirements to boot a linux system.

# Overview
bootloader: syslinux -> kernel: bzImage (linux) -> busybox

## Build the environment
```bash
sudo docker run -it --privileged -v ./boot-files:/boot-files ubuntu
```
```bash
apt update
apt install bzip2 git vim make gcc libncurses-dev flex bison bc cpio libelf-dev libssl-dev syslinux dosfstools
```


# 1. Build the kernel
```
git clone --depth=1 https://github.com/torvalds/linux
cd linux
make menuconfig
make -j8
cp arch/x86/boot/bzImage /boot-files/
```
tools used to build the kernel:
The kernel is built using make. This are the most common commands used to build the kernel:
1. CC: The C compiler
2. AR: The archiver
3. LD: The linker
4. AS: The assembler
5. NM: The symbol lister

# 2. Build busybox
```
git clone --depth=1 https://git.busybox.net/busybox
cd busybox
make menuconfig
# Enable the static compilation option. Settings -> Build Static Binary (No shared libs)
make -j8
```
# 3. Build the initramfs
This is the first file system that is loaded by the kernel. It contains the minimal tools to boot the system. The initramfs is a cpio archive that contains the minimal tools to boot the system. The kernel will load the initramfs into memory and execute the /init file.

We will put busybox into the initramfs. The busybox binary is statically compiled and contains the minimal tools to boot the system.

```bash
mkdir /boot-files/initramfs
make CONFIG_PREFIX=/boot-files/initramfs install
cd /boot-files/initramfs
vim init
```
Our init file will look like this:
[init](./boot-files/initramfs/init)
This is a minimal init file that uses the shell to spawn a shell
```bash
rm linuxrc
chmod +x init
find . | cpio -o -H newc > ../initramfs.cpio
```
> Now, we have the initramfs.cpio file that we will use to boot the system.

# 4. Bootloader
```bash
cd /boot-files
dd if=/dev/zero of=boot bs=1M count=50
mkfs -t fat boot
syslinux boot
mkdir m 
# mknod -m 660 /dev/loop0 b 7 0 , if you get an error mounting the file
mount boot m
cp bzImage initramfs.cpio m
umount m
```
# 5. Boot the system
```bash
# launching with wayland gtk
# using xhost + does not work
sudo -E qemu-system-x86_64 boot
```

## References
- https://www.kernel.org/doc/Documentation/x86/boot.txt
- https://www.kernel.org/doc/
