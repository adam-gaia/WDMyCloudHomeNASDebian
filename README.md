# WIP: Debian on a WD My Cloud Home
Instructions and scripts to install Debian on a WD My Cloud Home NAS

## Motivation
I was unsatisfied with my [WD My Cloud Home 4T](https://www.westerndigital.com/products/cloud-storage/wd-my-cloud-home#WDBVXC0040HWT-NESN). It is less featurefull than WD My Cloud (sans Home) devices and does not give the user access to the OS.

## Disclaimer
* Following this guide may void your warranty and brick your device
* Don't just copy+paste the commands here in case I've got a typo or something

## References
Very little of work in this repo is my own.
* I stood on the shoulders of the giants in this [this WD community thread](https://community.wd.com/t/install-debian-on-wd-my-cloud-home/250061?page=2)
* They somehow found [this russian forum](https://4pda.to/forum/index.php?showtopic=467828&st=12140#entry87961189)  where someone found a way to flash a custom debian image to the device.
    * Translating the page with the google chrome translate browser extension worked surprisingly well
* [This blog post](https://nerdprojekte.wordpress.com/2021/02/22/wd-my-cloud-home-to-linux-server-2-installation/) provides another sketchy custom image resulting from the steps in the first blog post
* [Help formatting the usb drive](https://askubuntu.com/a/571340)
* [Tare down video](https://www.youtube.com/watch?v=CfLzUWs6R1c) You shouldn't need this. I just had a hard time pressing the reset button with the case on. The tabs will break as proof you took the device apart, voiding your warranty.
* https://nerdprojekte.wordpress.com/2021/12/28/wd-mycloud-home-to-linux-server-17-install-openmediavault-6-and-debian-11-directly-new-method/


## Instructions
The links above all have their own instructions, but this is the process I went with. After I started the process, I found 
[Nerdprojekte's latest update](https://nerdprojekte.wordpress.com/2021/12/28/wd-mycloud-home-to-linux-server-17-install-openmediavault-6-and-debian-11-directly-new-method/) which contained instructions to use Rah-66's installer.



### Format a usb drive
Multiple forum users noted that they had trouble with this step on linux, so take extra care
```bash
# Find your usb device with lsblk
lsblk
...
# Mine was /dev/sdb
# You know the drill, replace `/dev/sdb` with your device from here on out

# Erase the drive (optional). This will take a while
sudo dd status=progress if=/dev/zero of=/dev/sdb bs=4k && sync

# Partition the drive
sudo fdisk /dev/sdb
o # create partition table
n # create a new partition
t # set the partition type
b # as in 0x0b. This should correspond to the `W95 FAT32` hex code

w # Write and save changes
# This will end the fdisk session

# Find the new partition
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

### Install Debian and OMV
This is the black magic. I dont know how the OG Forth32 found out that we can load arbitrary image on the MCH.
* Unplug the powersupply from the MCH
* Plug in the usb flash drive
* Hold down the reset button. A ballpoint pen wasnt skinny enough to press the reset button. I found it easiest to remove the cover to gain access to the button
* While keeping the reset button held down, plug the power supply back in
* Keep the reset button held down until the light blinks steadily. 

### Run setup process
See https://nerdprojekte.wordpress.com/2021/12/28/wd-mycloud-home-to-linux-server-17-install-openmediavault-6-and-debian-11-directly-new-method/


### House keeping
```bash
# Change root password

# Create a new user

# Disable ssh as root


```

