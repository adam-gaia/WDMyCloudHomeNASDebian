# WIP: Debian on a WD My Cloud Home
Instructions and scripts to install Debian on a WD My Cloud Home NAS

## Motivation
I was unsatisfied with my [WD My Cloud Home 4T](https://www.westerndigital.com/products/cloud-storage/wd-my-cloud-home#WDBVXC0040HWT-NESN). It is less featurefull than WD My Cloud (sans Home) devices and does not give the user access to the OS.

## Disclaimer
* Follwing this guide may void your warrenty and brick your device
* Don't just copy+paste the commands here in case I've got a typo or something

## Refrences
Very little of work in this repo is my own.
* I stood on the shoulders of the giants in this [this WD community thread](https://community.wd.com/t/install-debian-on-wd-my-cloud-home/250061?page=2)
* They somehow found [this russian fourm](https://4pda.to/forum/index.php?showtopic=467828&st=12140#entry87961189)  where someone found a way to flash a custom debian image to the device.
    * Translating the page with the google chrome translate browser extension worked suprisingly well
* [This blog post](https://blog.loetzimmer.de/2020/09/debian-on-wd-my-cloud-home-single-bay.html) (author is a commenter in the WD forum) details how to repalce the sketchy custom debian image with a fresh one
    * subsequent posts detail how to set up OpenMediaVault
* [This other blog post](https://nerdprojekte.wordpress.com/2021/02/22/wd-my-cloud-home-to-linux-server-2-installation/) provides another sketchy custom image resulting from the steps in the first blog post
* [Help formating the usb drive](https://askubuntu.com/a/571340)
* [Tare down video](https://www.youtube.com/watch?v=CfLzUWs6R1c) You shouldn't need this. I just had a hard time pressing the reset button with the case on. The tabs will break as proof you took the device appart, voiding your warrenty.


## Instructions
The links above all have their own instructions, but this is the process I went with.


### Install a sketchy pre-built debian image
I started with Nerdprojekte's compiled debian image. I feel bad calling it sketchy since his blog was a huge help, but I don't like installing binaries from random blogs on internet.
I used it because
* I wanted to compile the kernel on the MCH instead of cross compiling
* The instructions to install the russian image were harder to follow


#### Format a usb drive
Multiple forum users noted that they had trouble with this step on linux, so take extra care
```bash
# Find your usb device with lsblk
lsblk
...
# Mine was /dev/sdb
# You know the drill, replace `/dev/sdb` with your device from here on out

# Erace the drive (optional). This will take a while
sudo dd status=progress if=/dev/zero of=/dev/sdb bs=4k && sync

# Partition the drive
sudo fdisk /dev/sdb
o # create partition table
t # set the partition type
b # this should corespond to the `W95 FAT32` hex code
n # add a new partition

w # Write and save chages
# This will end the fdisk session

# Find the new partion
lsblk
...
# Mine was /dev/sdb1

# Format the new partition
sudo mkfs.vfat /dev/sdb1 # Use the partition not the device

# Extract Nerdprojekte's zip file's contents to the usb drive
# I found the instructions confusing here. The usb drive should *not* be bootable
# The contents of Nerdprojekte's zip should not be in a sub directory
# ls /path/to/mounted/partition/
#     bluecore.audio  omv  password.txt  rescue.root.sata.cpio.gz_pad.img  rescue.sata.dtb  sata.uImage
# If you encounter issues later, try again with theses files in the subdirectory they came in
# I tried both ways and dont remember which worked

# Eject the drive
sudo eject /dev/sdb # Eject the device not the partition
```

#### Install debian image on the MCH
This is the black magic. I dont know how the OG Forth32 found out that we can load arbitrary image on the MCH.
* Unplug the powersupply from the MCH
* Plug in the usb flash drive
* Hold down the reset button. A ballpoint pen wasnt skinny enough to press the reset button. I found it easiest to remove the cover to gain access to the button
* While keeping the reset button held down, plug the power supply back in
* Keep the reset button held down untill the light blinks steadly. 


### Build our own version of debian to replace the pre-built one
Though it was redundant, I then followed Alex N's to curate my own image

#### Debootstrap

```bash
ssh bullseye@<IP> # password is 'bullseye'
su root # password is also 'bullseye'

# Install debootstrap
apt update && apt install debootstrap

# Add debootstrap and other system utils to the path
export PATH="${PATH}:/sbin"

# Debootstrap script expects these other scripts in a very specific place. Idk why they aren't installed to that location
ln -s /usr/share/debootstrap/scripts /debootstrap/scripts

# Finally actually run the debootstrap utility
debootstrap bullseye /mnt http://ftp.debian.org/debian

# Copy settings into our debootstrap target so we dont have to set them up again later
cp /etc/network/interfaces /mnt/etc/network/interfaces
cp /etc/fstab /mnt/etc/fstab

# Chroot our debootstap
cd /mnt/
mount -t proc proc proc/
mount -t sysfs sys sys/
mount -o bind /dev dev/
chroot .

# Install open ssh server so we can get back in
apt update
apt upgrade
apt install openssh-server -y --no-install-recommends
apt clean
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/archive/*.deb

# Create a non-privileged user
USERNAME='agaia'
adduser --home "/home/${USERNAME}" --shell /bin/bash "${USERNAME}"
# set a password so you can ssh in later

# Exit chroot
exit

# Clean up
umount /mnt/proc/
umount /mnt/sys/
umount /mnt/dev/

# Mount your flash drive so we can write back to it
mkdir -p /media/usb-drive
# Find the flash drive
lsblk
...
# My device was /dev/sdb1
mount /dev/sdb1 /media/usb-drive/
mkdir /media/usb-drive/new

# Save the image for use later
mkdir /new
mv /mnt/usr /new/
mv /mnt/var /new/
mkdir /mnt/usr /mnt/var
cd /mnt
# Creating these archives will take a while
tar --create --gzip --file /media/usb-drive/new/20-root.tar.gz *
cd /new/var
tar --create --gzip --file /media/usb-drive/new/21-var.tar.gz *
cd /new/usr
tar --create --gzip --file /media/usb-drive/new/22-usr.tar.gz *

```



## Exploritory
```
$ uname -a
Linux testnas 4.1.17 #1 SMP PREEMPT Sat Aug 10 20:17:46 MSK 2019 aarch64 GNU/Linux
```
