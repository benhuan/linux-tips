# uEnv

booting sequence:
`MLO->u-boot.img->kernel->root file system`
Loading `uEnv.txt` after loading u-boot.img,
U-boot allows the user a 2-3 seconds window to stop the boot process.
Hit any key in the UART terminal application and the u-boot prompt will
be displayed as shown below:

`U-Boot#`

There are several useful commands to remember here.

To display the environment variables, type `printenv` or just `pri`.

to display perticular virable, type `env print VAR`

to set virable, type `setenv NAME VALUE`

to get more help, type `help` or `help COMMAND`

To continue the boot from u-boot, simply type `boot` and the process will continue as if you did not stop the boot by hitting a key.  This is also equivalent to typing `run bootcmd`.  The bootcmd environment variable is actually a sequence of conditional statements \(separated by semicolons\) which perform checks on the hardware and software to complete the boot process.

These commands can do all the work, but we will need `uEnv.txt` if we want to make things work
automaticaly.

Actually it works like [GRUB](http://www.gnu.org/software/grub/), you will easily understand the whole process if you know [GRUB](http://www.gnu.org/software/grub/) well.

Let me show my uEnv.txt first:

```bash
#uEnv.txt boot kernel from TFTP, NFS or SD

#basic setting for booting from tftp
ipaddr=10.42.0.12
serverip=10.42.0.1
gatewayip=10.42.0.1
staticip=${ipaddr}:${serverip}:${gatewayip}:255.255.255.0:::off
rootpath=/home/longqi/BBB/rootfs

#setting for kernel loading
kloadaddr=0x82000000
fdtaddr=0x83000000

##Disable HDMI/eMMC
cape_disable=capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN,BB-BONE-EMMC-2G
#cape_enable=capemgr.enable_partno=BB-BONE-LCD7-01

#parameters for kernel
mmc_args=setenv bootargs console=ttyO0,115200n8 quiet root=/dev/mmcblk0p2 ro rootfstype=ext4 rootwait ${cape_disable} ${cape_enable}
nfs_args=setenv bootargs console=ttyO0,115200n8 root=/dev/nfs rw nfsroot=${serverip}:${rootpath} ip=${staticip} ${cape_disable} ${cape_enable}


#load kernel and device tree
load_u_kernel=tftp ${kloadaddr} ${rootpath}/boot/uImage
load_z_kernel=tftp ${kloadaddr} ${rootpath}/boot/zImage
loadfdt=tftp ${fdtaddr} ${rootpath}/boot/dtbs/am335x-boneblack.dtb


#booting options
boot_uimage=bootm ${kloadaddr} - ${fdtaddr}
boot_zimage=bootz ${kloadaddr} - ${fdtaddr}

#to boot
uenvcmd=run load_u_kernel; run loadfdt; run mmc_args; run boot_uimage

#uenvcmd=run load_z_kernel; run loadfdt; run mmc_args; run boot_zimage

```

### format

`NAME=VALUE` in `uEnv.txt` is same with `setenv NAME VALUE` in u-boot prompt, they do the same thing:

**define virables**

`${NAME}` means call this variable

`run NAME` means run the `NAME` as command

### parameters for booting

`kloadaddr` this is the loading address for the kernel
`fdtaddr` this the loading addressfor the binary device tree
`initrd_addr` this is the loading address for the [initrd](https://en.wikipedia.org/wiki/Initrd) ramdisk

```
In computing, initrd (initial ramdisk) is a scheme for loading a temporary root file system
into memory in the boot process of the Linux kernel. initrd and initramfs refer to two
different methods of achieving this. Both are commonly used to make preparations before the
real root file system can be mounted.
```

