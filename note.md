# Arch install in VMware

## pre-Installing
Your system should be `x86_64`.

Open UEFI mode
uefi 需要注意VM是否支持（开启？）



Varify whether UEFI mode is enabled.

```bash
ls /sys/firmware/efi/efivars
```

Network check

````bash
ping www.archlinux.org
````



## Installing

Set tty font

```bash
setfont /usr/shar/kbd/consolefonts/...
```

Maybe `LatGrkCyr-12*22 ` `ter-u22b` is god

### system clock

Update the system clock

```bash
timedatectl set-ntp true
```



### disk partition

See the current disk for partition

```bash
lsblk #list block?
```

You will see `loop0` device and the disk which will likely be `sda`. `fdisk -l ` also will list the device.

In this example, we assume `sda` is the disk. The disk partition can be created by entering the following command.

```bash
cfdisk /dev/sda
```

`fdisk` can be used, but I think `cfdisk` is more convenient, since it is the graphic interface.

Select `gpt` and start our partitions. There are three important partitions:

1. EFI partition (sda1)
2. swap partition (sda2)
3. root partition (sda3)

Set `EFI partiton` size `512M` for the `grub`; Set `swap partition` `1-2G` for swap; Set leftover space for `root partition`.

You can also seperate other partitons, such as `home`. It is up to you.

Then, enter `Write` and `Quit` to comfirm your operation. Enter `lsblk` to varify whether it works.

Next we can set the partition format.

```bash
# EFI
mkfs.fat -F32 /dev/sda1 # boot: fast32
# swap
mkswap /dev/sda2
swapon /dev/sda2
# root
mkfs.ext4 /dev/sda3 # root disk: ext4
```

### disk mount

mount the important disks for further settings.

```bash
# root disk -> /mnt
mount /dev/sda3 /mnt
# EFI -> /mnt/boot/efi
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

### downloading some essentials

[arch linux mirror link](https://archlinux.org/mirrors/status/) to find more mirrors.

edit `/etc/pacman.d/mirrorlist` .

```bash
vim /etc/pacman.d/mirrorlist
```

Here are some mirrors in China.

```
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrosr.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
```

install the essential packages

```bash
pacstrap /mnt base linux linux-firmware
```

You can enter `linux-lts` instead of `linux` for Long Time Support kernel.

Then, we can generate `fstab` so that system can know where to mount the partitions after boots.

```bash
genfstab /mnt >> /mnt/etc/fstab
```

### root setting

Now that we can change root into our system.

```bash
arch-chroot /mnt
```

Customize the timezone.

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

set your `Region` and  `City`. For exampe, `ln -sf /usr/share/zoneinfo/Japan/Tokyo /etc/localtime`

Editor is neccessary. We can use `vim`. 

```bash
pacman -S vim
```

Next, edit the file `/etc/locale.gen` and uncommenting any locale you need. For example `en_US.UTF-8`. Generate the locales and set them.

```bash
vim /etc/locale.gen
# ... uncomment locales
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

It is time to set our host name and hosts.

```bash
echo "archVM" >> /etc/hostname # you can set whatever name you want
vim /etc/hosts
```

Our entries would look like this.

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   archVM.localdomain  archVM # set you host name
```

We can set root password and users.

```bash
passwd # set root password
# enter root's password
mkdir /home/username
useradd -d /home/username username
passwd username # set username password
# enter username's password
passwd -d username # you can empty username's password
```







### packman server

```
vim /etc/pacman.conf
```

enter `mirrorlist` in `[community]` by `gf`

