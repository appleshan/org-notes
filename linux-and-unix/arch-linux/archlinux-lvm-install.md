# Arch Linux LVM Installation
In this guide I am utilizing [Virtualbox](https://www.virtualbox.org/) as the Hypervisor due to it being easily obtainable and opensource.

## Prerequisites
- Virtualbox Installation
- ArchLinux ISO

### Installation Steps
In this setup I presented two block devices to the virtual machine.
- root.vdi - 25GB - used for root directory / /boot and swap
- home.vdi - 20GB - used for the home directory /home

Once you have your VM created and the arch linux iso image inserted into the cd rom boot it and select Archlinux.
On the terminal we will use fdisk to partition our drives.

#### List block devices
Check if the drives are available
```sh
dmesg | grep sd[ab]
```
You should see sda and sdb devices as attached.
Next, ensure that your drives are available for partitioning.
```sh
fdisk -l
```
You should see both sda and sdb.
#### Partitioning
```sh
fdisk /dev/sda
n - for new partition
p  - for primary (3 primary, 0 extended, 1 free)
Enter - use defaults
Enter - use defaults
Enter - use defaults
w - to write the changes
```
Repeat the steps for /dev/sdb like fdisk /dev/sdb

#### Create the Physical Volumes
Create the two physical volumes to reside in your volume group.
```sh
pvcreate /dev/sda
pvcreate /dev/sdb
```
#### Display the Physical Volumes
```sh
pvdisplay
```

#### Create the Volume Groups
Here I create two volume groups one for root and one for home.  That way we can easily expand each volume group in the event that we need additional space.
```sh
vgcreate vg_root /dev/sda
vgcreate vg_home /dev/sdb
```

#### List the Volume Groups
```sh
vgdisplay
```

#### Create the Logical Volumes
Create the logical volumes that we will eventually mount to their respective locations.
```sh
lvcreate -L 250M -n boot vg_root
lvcreate -L 20G -n root vg_root
lvcreate -L 4G -n swap vg_root
lvcreate --extents 100%FREEVG -n home vg_home
```

#### List the Logical Volumes
```
lvdisplay
```

#### Create the Filesystems
```sh
mkfs.ext4 /dev/vg_root/boot
mkfs.ext4 /dev/vg_root/root
mkswap /dev/vg_root/swap
swapon /dev/vg_root/swap
mkfs.ext4 /dev/vg_home/home
```

#### Mount the Volumes to Directories
```sh
mount /dev/vg_root/root /mnt
mkdir /mnt/boot
mount /dev/vg_root/boot /mnt/boot
mkdir /mnt/home
mount /dev/vg_home/home /mnt/home
```

#### System Clock Update
```sh
timedatectl set-ntp true
```

#### Pacstrap the system
```sh
pacstrap /mnt base base-devel
```

#### Generate your fstab
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

#### Change root to the new system location
```sh
arch-chroot /mnt /bin/bash
```

#### Setup the Locale
Open the file /etc/locale.gen
nano /etc/locale.gen
Uncomment the line en_US.UTF-8 UTF-8 or whatever your local is.
Save the file with Ctrl-W
```sh
locale-gen
touch /etc/locale.conf
```
Here I just use EOF to quickly add a line to the locale.conf file. Pressing Enter after each typed line.
**NOTE:** When using cat with EOF you cannot go back up a line after Enter is pressed.  If you have a mistake you must rm the file and start over. 
```sh
cat <<EOF>> /etc/locale.conf
LANG=en_US.UTF-8
EOF
```

#### Set Hostname in /etc/hostname
Set the hostname to your hostname of choice
```sh
cat <<EOF>> /etc/hostname
archlab
EOF
```

#### Set Hostname with hostnamectl
```sh
hostnamectl set-name archlab
```

#### Add the hostname to /etc/hosts
```sh
cat <<EOF>> /etc/hosts
archlab 127.0.0.1
EOF
```

#### Setup name resolution
```sh
cat <<EOF>> /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF
```

#### Setup Network
Find the adapter name
```sh
ls /sys/class/net
```
enp0s3 and lo were listed use the adapter listed for your system
```sh
touch /etc/systemd/network/enp0s3.network
cat <<EOF>> /etc/systemd/network/enp0s3.network
[Match]
name=en*
[Network]
DHCP=yes
EOF
```
Restart and enable the systemd unit
```sh
systemctl restart systemd-networkd
systemctl enable systemd-networkd
```

#### Setup the Timezone
Select the appropriate information in the setup for your timezone.
```sh
tzselect
ln -s /usr/share/zoneinfo/Zone/SubZone /etc/localtime
```

#### Set UTC time
```sh
hwclock --systohc --utc
```

#### Install mkinitcpio and linux
```sh
pacman -S mkinitcpio linux
```

#### Modify the mkinitcpio.conf
Ensure that filesystems and lvm2 are listed in the HOOKS line so it looks like this.
```sh
HOOKS=(base udev autodetect modconf block filesystems lvm2 keyboard fsck)
```

#### Generate the initramfs
```sh
mkinitcpio -p linux
```

#### Install Grub Bootloader
```sh
pacman -S grub
```

#### Install grub to the arch block device
```sh
grub-install --target=i386-pc /dev/sda
```

#### Generate the grub configuration file
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

#### Install the Video Driver
Search for the VGA driver
```sh
lspci | grep VGA
```
Since we are using VirtualBox it uses the VMWare Driver so we search for that
```sh
pacman -Ss VMWare
```
This driver was listed so install the driver
```
pacman -S xf86-video-vmware
```

#### Install man
```sh
pacman -S man
```

#### Change root password
```sh
passwd
```

#### Create User Account
```sh
useradd -m yourusername
```

#### Create your password
```sh
passwd
```

#### Add sudo capability to your user
```sh
EDITOR=nano visudo
```
Add the following line under root ALL=(ALL) ALL
```sh
yourusername ALL=(ALL) ALL
```
Save the file with Ctrl-W

#### Shutdown the system
```sh
shutdown -P now
```

#### Note: Eject the archlinux DVD from your host and then start it