# QEMU
Qemu simulation on arm64. Booting linux kernel on arm64 simulation
**
Booting Linux Kernel on QEMU (AArch64)

1. Install Required Utilities and Toolchain
First, update the package list and install all necessary dependencies:

sudo apt update -y
sudo apt install -y zsh git gcc flex bison make ca-certificates curl gcc-aarch64-linux-gnu bi

2.  Download and Build the Linux Kernel
Clone the Linux kernel source code (version 6.6 in this case):

git clone --depth 1 --branch v6.6 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux

3. Generate a default configuration for the ARM64 architecture:

make ARCH=arm64 defconfig

Compile the kernel image and device tree blobs (DTBs):

make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image dtbs

5. Build BusyBox
Clone the BusyBox source code:

git clone --depth=1 https://git.busybox.net/busybox.git

cd busybox

Configure BusyBox and ensure static linking is enabled:

make menuconfig
- Navigate to Settings → Build Options.
- Enable the option: CONFIG_STATIC=y

5. Compile BusyBox:

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install

This will generate the _install directory containing the minimal root filesystem.

6. Create an Initramfs
Navigate into the BusyBox installation directory:
  
cd _install
Create a minimal init script:
cat > init << 'EOF'
#!/bin/sh
Simple init script
echo "Hello from the AArch64 initramfs!" > /dev/console
Set minimal shell environment
export PATH=/bin:/sbin:/usr/bin:/usr/sbin
Launch an interactive shell
exec /bin/sh
EOF
chmod +x init

7. Generate the initramfs image:

find . -print0 | cpio --null -ov --format=newc > ../../initramfs.cpio

This will produce the file ~/initramfs.cpio.

8. Boot the Kernel in QEMU
From the parent directory (where both linux/ and initramfs.cpio reside), launch QEMU with the following

qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -kernel linux/arch/arm64

At this point, the kernel should boot, and you will be greeted with the BusyBox shell via the initram
