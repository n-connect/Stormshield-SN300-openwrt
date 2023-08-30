# Stormshield-SN300-openwrt
Image and use Stormshield SN300 with openwrt 

I have already written a little about the Stormshield SN300, about the components and specifications. You can read more about it here -> https://github.com/bmn-m/Stormshield-SN300-opnsense#stormshield-sn300

OPNSense runs on the Stormshioeld SN300, but rather bad than good. Especially the load of the singlecore CPU is quite high with OPNSense.
In the Linux world there are often already suitable distributions for the needed purpose. 
We have a piece of hardware that we want to use as a firewall router? So what is more obvious than to adapt OpenWrt for our purposes...
Especially the low hardware requirements speak for the use of OpenWrt. Don't ask me for the exact hardware requirements for OpenWrt, I couldn't find them. Make yourselves a picture: https://openwrt.org/toh/recommended_routers.

## The image
First of all we need to download the OpenWrt image, write it to flash memory and customize it.
I have downloaded the ```https://downloads.openwrt.org/releases/22.03.5/targets/x86/64/openwrt-22.03.5-x86-64-generic-ext4-combined.img.gz``` image.
Unpack it with:

```gzip -dk openwrt-22.03.5-x86-64-generic-ext4-combined.img.gz```

Next we write the unpacked image to the 2GB flash memory.
With ```lsblk``` you can check which blockdevice you have to write to, in my case /dev/sda:

```dd if=openwrt-22.03.5-x86-64-generic-ext4-combined.img of=/dev/sda```

If you don't like the console, you can also use gnome-disk-utilitys.
Next we mount the rootfs to edit the network config an add a little programm we need later.

```/dev/sda2 /media/bmn-m/rootfs ext4 (rw,nosuid,nodev,relatime,uhelper=udisks2)```

## Adjustments
Now we have to copy the "network" file i created to (your mountpoint)/etc/config/. Please edit the IP to your needs.

As the device is still on your PC, lets rezise the rootfs partition:

```
fdisk /dev/sda1
e2fsck -f -y -v -C 0 '/dev/sda2' 
resize2fs -p '/dev/sda2
```

Open fdisk and observe with p where the partition starts, remove it (d), and create (n) larger one on the same place. Pay attention on its start sector and use exactly the same sector when creating the new partition. Now set the ending sector a little smaller than the last usable sector. Exit fdisk with writing the changes (w). Also you can use Gparted to extend partition.
Sync and unmount your flash drive, mount the hardware back into the device and boot. Now you should be able to reach the Stormshield SN300 at the IP address you set via http in the Network file.

Packages needed to nex step (network enabling) :

Then we have to download the screen package and dependencies from :

```https://downloads.openwrt.org/releases/22.03.5/packages/x86_64/packages/screen_4.8.0-2_x86_64.ipk```
```https://downloads.openwrt.org/releases/22.03.5/packages/x86_64/base/libncurses6_6.3-2_x86_64.ipk```
```https://downloads.openwrt.org/releases/22.03.5/packages/x86_64/base/terminfo_6.3-2_x86_64.ipk```

And copy it to (your mountpoint)/root/.

## Network Enabling / Switching :

OpenWRT does not detect an active network interface after booting, even though I have plugged a network cable into one of the eight switch ports. Unfortunately, there is also no LED that shows a link. Only one interface and no link, for a firewall rather bad. But this should not stop us in our mission. As we have seen on the mainboard, there seems to be a switch chipset. You usually connect to a switch with a serial console, at least for the initial configuration
On OpenWRT :

Install packages :

```
cd /root
opkg install terminfo_6.3-2_x86_64.ipk libncurses6_6.3-2_x86_64.ipk screen_4.8.0-2_x86_64.ipk
```

Connect to switch :

```
screen /dev/ttyS1 19200
```

Show ports state (all ports disabled by default) :

```port state```

Enable ports :

```port state 1-9 enable```

For more informations about switching : https://github.com/bmn-m/Stormshield-SN300-opnsense#switching

## Web-Frontend
Now if you login you should be able to observe smt. like this:
![stpa](https://user-images.githubusercontent.com/18091782/204138820-70eb0475-6b1c-4af8-a175-3fce82d41ef9.png)

Interfaces

![intf](https://user-images.githubusercontent.com/18091782/204138829-c3d80d6a-9615-43f6-a788-acccae8a6633.png)

Devices

![devi](https://user-images.githubusercontent.com/18091782/204138834-500d616f-ca30-434b-a05f-9580284f489f.png)

## Serial
If you were not able to connect to the Stormshield via Web interface, then we have to config the built in switch via console.




