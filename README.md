# Odroid U2/U3 Ubuntu 18.04 Desktop image

## Downloads
<https://github.com/IOOI-SqAR/odroid/releases/download/1.0.0/ubuntu-18.04-4.16.0-v7-desktop-odroid-u2-u3-20190308.img.xz>

### Checksums:
**compressed image:**
md5sum: daf7312bc65e98ecfdc1b4d9fbf44f03 ubuntu-18.04-4.16.0-v7-desktop-odroid-u2-u3-20190308.img.xz

**uncompressed image:**
md5sum: e46df26b7e08f0e1d6cb3297f91b11da ubuntu-18.04-4.16.0-v7-desktop-odroid-u2-u3-20190308.img

### Credentials:

**console:**  
user: root  
pass: odroid

**graphical UI and console:**  
user: odroid  
pass: ubuntu1804

**Please change this as soon as you can!**

## What is this?
A xc compressed image containing an installation of Ubuntu 18.04 Desktop for the Odroid U2 or U3 (and possibly the X2, but tested only on Odroid U2 for now). 

The uncompressed Size of this Image is **128GB** (119 GiBiByte or exacly 127865454592 Bytes), to use it you need a microSDXC card of at least this size (I used a SanDisk Ultra R100 microSDXC 128GB Kit, UHS-I U1, A1, Class 10 (SDSQUAR-128G-GN6MA), which has exactly this size — hence the size of the .img). Burn the image onto the Card using [balenaEtcher](https://www.balena.io/etcher/) works best, no need to uncompress the image!

## How came this into life?
I was looking for an image to run Ubuntu 18.04 on my Odroid U2 and found [this thread](https://forum.odroid.com/viewtopic.php?t=31765), however, the [offered image](https://www.odroid.in/mirror/dn.odroid.com/4412/trial_18.04_minimal/) did not work for me (and others). So, after a while browsing [the forum](https://forum.odroid.com/index.php), I found [this post](https://forum.odroid.com/viewtopic.php?f=77&t=30654#p232918), which inspired me to try the same. The result is above download.

## How I did this
For those, who don't trust me or want to do it on their own (for instance, because they don't own such a large microSD-card) I post the instructions here:

### Download the minimal Ubuntu 16.04 image and install it

* Use <https://dn.odroid.com/4412/Linux/ubuntu-u2-u3/trial_kernel4.16/ubuntu-odroidu3-4.16.img.zip>
(or do it like <https://www.nico-maas.de/?p=1491> as [posted here](https://forum.odroid.com/viewtopic.php?f=77&t=30654))

* burn the image onto a microSD-card using `dd` or use balenaEtcher

* then see <https://forum.odroid.com/viewtopic.php?f=52&t=2948>
to resize the partition and filesystem:

```
#!/bin/bash

fdisk_first() {
		p2_start=`fdisk -l /dev/mmcblk0 | grep mmcblk0p2 | awk '{print $2}'`
		echo "Found the start point of mmcblk0p2: $p2_start"
		fdisk /dev/mmcblk0 << __EOF__ >> /dev/null
d
2
n
p
2
$p2_start

p
w
__EOF__

		sync
		touch /root/.resize
		echo "Ok, Partition resized, please reboot now"
		echo "Once the reboot is completed please run this script again"
}

resize_fs() {
	echo "Activating the new size"
	resize2fs /dev/mmcblk0p2 >> /dev/null
	echo "Done!"
	echo "Enjoy your new space!"
	rm -rf /root/.resize
}


if [ -f /root/.resize ]; then
	resize_fs
else
	fdisk_first
fi
```
* log in onto your odroid using `ssh root@<your_odroid's_ip>` from your Laptop/Desktop and then paste the above script into `vi`,`nano` or whatever editor you prefer. Save the script as `resize.sh`

* then run the script before and after a reboot:

```
chmod +x resize.sh
./resize.sh
reboot
./resize.sh
```

### Upgrade to Ubuntu 18.04
* to upgrade to 18.04 (see also <https://forum.odroid.com/viewtopic.phpf=77&t=30654&p=232918#p232918> and <https://www.cyberciti.biz/faq/ubuntu-bash-do-release-upgrade-command-not-found/>):

```
sudo apt install ubuntu-release-upgrader-core

sudo apt update

sudo apt upgrade

sudo reboot

sudo do-release-upgrade -d

sudo reboot
```


### From Server to Desktop

(see also: <https://askubuntu.com/questions/322122/switching-from-server-to-desktop>)

```
sudo apt-get update

sudo apt-get install tasksel

sudo tasksel

sudo reboot
```
### Some additional things

For some reasons unknown to me **gnome-terminal** is not working, it crashes on launch. To solve this problem I just installed xterm:

```
sudo apt-get install xterm
```

When examing the first image I created in a hex editor, I found that a lot of clutter from deleted Android files was still there (which probably was a leftover of the original Odroid Android from the 16.04 image I downloaded) So I decided to remove this by first defragmenting the disk and then wiping the free disk space:

```
sudo apt-get install e2fsprogs
sudo apt-get install secure-delete

sudo e4defrag -c /
sudo e4defrag -v /

sudo sfill -llz -v /
sudo sfill -llz -v /media/boot
```
This also resulted in a smaller xz compressed image file since free space is so much easier to compress … :)

## Some things for you to do
* **It is strongly recommended to disable remote ssh login for the user `root` since this makes remote attacks much harder!** ssh login for `root` came in handy during the installation process and is enabled in the minimal Ubuntu 16.04 image I used as a starting point.
* You might want to pin the MAC address of your Odroid to a fixed value so you don't get always a new IP address after each reboot. I found this: <https://forum.odroid.com/viewtopic.php?f=77&t=30654#p245732> a working solution (the way like the first solution offered in this thread (post 11) did not work for me)
* When you created the image file yourself (instead of using mine) you might experience a file system damage on the minimal Ubuntu 16.04 image. The boot partition (FAT16) had a damage, which I later repaired (the downloadable image should be fine). You can repair such images using a second SD-Card to boot from and then plug in an the SD-Card you want to repair using an USB-Adapter.
* **Report back to me whether this image works on a Odroid U3 or X2.**
* **Have fun!**