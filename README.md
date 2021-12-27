# Debian on a WD My Cloud Home
Instructions and scripts to install Debian on a WD My Cloud Home NAS

## Motivation
I was unsatisfied with my [WD My Cloud Home 4T](https://www.westerndigital.com/products/cloud-storage/wd-my-cloud-home#WDBVXC0040HWT-NESN). It is less featurefull than WD My Cloud (sans Home) devices and does not give the user access to the OS.

## Disclaimer
Follwing this guide may void your warrenty and brick your device

## Refrences
Very little of work in this repo is my own.
* I stood on the shoulders of the giants in this [this WD community thread](https://community.wd.com/t/install-debian-on-wd-my-cloud-home/250061?page=2)
* They somehow found [this russian fourm](https://4pda.to/forum/index.php?showtopic=467828&st=12140#entry87961189)  where someone found a way to flash a custom debian image to the device.
    * Translating the page with the google chrome translate browser extension worked suprisingly well
* [This blog post](https://blog.loetzimmer.de/2020/09/debian-on-wd-my-cloud-home-single-bay.html) (author is a commenter in the WD forum) details how to repalce the sketchy custom debian image with a fresh one
    * subsequent posts detail how to set up OpenMediaVault
* [This other blog post](https://nerdprojekte.wordpress.com/2021/02/22/wd-my-cloud-home-to-linux-server-2-installation/) provides another sketchy custom image resulting from the steps in the first blog post

## Instructions
The links above all have their own instructions, but this is the process I followed.





```bash
apt update && apt install debootstrap
export PATH="${PATH}:/sbin" # add debootstrap to the path
ln -s /usr/share/debootstrap/scripts /debootstrap/scripts # Debootstrap script expects these other scripts in a very specific place. Idk why they aren't installed to that location
```



## Exploritory
```
$ uname -a
Linux testnas 4.1.17 #1 SMP PREEMPT Sat Aug 10 20:17:46 MSK 2019 aarch64 GNU/Linux
```
