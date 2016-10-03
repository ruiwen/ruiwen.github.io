---
Categories:
- Development
- GoLang
Description: ""
Tags:
- raspberry pi 3
- raspberry pi
- ubuntu
- ubuntu 16.04
date: 2016-10-03T01:30:42+08:00
menu: main
title: Ubuntu 16.04 on Raspberry Pi 3
---

I recently wanted to unify my dev environments, including my (very handy!) Raspberry Pi 3.

I'd gotten started with it with a [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/), but felt the new [Ubuntu 16.04](https://ubuntu-pi-flavour-maker.org/xenial/ubuntu-minimal-16.04-server-armhf-raspberry-pi.img.xz.torrent) was really more my speed. Thus began the journey to upgrade Pi OSes


# Images

First up, finding the right image. The good thing about the Pi community is that there's so much activity, and so many community images. The bad thing is, well, that there are so many community images that it's sometimes hard to tell which is going to work.

Let me save you the trouble — [this one does](https://ubuntu-pi-flavour-maker.org/xenial/ubuntu-minimal-16.04-server-armhf-raspberry-pi.img.xz.torrent)

It's a link to a `.torrent` file from [Ubuntu Pi Flavour Maker](https://ubuntu-pi-flavour-maker.org/). You'll end up with a `.img.xz` file which you'll have to unpack (depending on your OS) to get the original `.img` image.

After which, it's a quick write to the prepared SD Card (I suggest formatting the card via your OS's disk management tools, so you'll want to back anything important up) with something similar to

	sudo dd bs=1m if=path_of_your_image.img of=/dev/diskn

(The official raspberrypi.org has [handy documentation on how you can write a Raspberry Pi `.img` to your SD card](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).)


# Booting Up

Once you're done writing the image to your SD card, pop it into your Pi, connect the peripherals, plug in the power, and... _wait_.

At this stage, unless you have a configured ethernet connection all set up for your Pi and are confident of SSH'ing in, you'll probably want a monitor and keyboard attached as well so you can work directly on the console after it boots.

If you don't have a ready ethernet connection, keep in mind that there's a script that waits for an address to be assigned to `eth0` via DHCP, and times out only after 5 mins. If your Pi isn't responsive after plugging the power in, grab a coffee or something.

After a while, your Pi should be running, and if you've got a monitor connected to it, you should have a login prompt ready to go.

_Just in case — the default username and password for RPi Ubuntu images is usually: `ubuntu` / `ubuntu`_


# Connecting to Wifi

The next thing you'll want to do is probably to connect your Pi to your wifi network. If you don't, and already have an ethernet connection good to go, then feel free to skip the rest of this section.

Wireless connections on the RPi are handled by a package called `wpasupplicant`. It's not installed by default, so you'll need to grab it with a familiar `apt-get install`. This is also why you need an ethernet connection to at least get started.

_Pro-tip: If your router/switch is too far away from where you have the rest of your RPi set up, bridging your desktop/laptop's wifi connection over an adapter to an ethernet connection is something I found really handy._

You'll also notice that if you entered `ifconfig` at the prompt, the wireless interface `wlan0` (by default) has not automatically come up on boot, so we'll need to configure it to do so.

Find and edit the file `/etc/network/interfaces`

	$ sudo vim /etc/network/interfaces

Then add a block to configure the `wlan0` interface with the requisite wifi credentials

	auto wlan0
	allow-hotplug wlan0
	iface wlan0 inet dhcp
	wpa-ssid "YourWifiSSIDHere"
	wpa-psk "y0uRs3cretw1f1p@55w0rdHere"

Save the file, and you should be able to restart the networking service with `sudo service networking restart` and have the `wlan0` interface come up.

	wlan0     Link encap:Ethernet  HWaddr b8:27:eb:49:2d:dc
	          inet addr:192.168.5.103  Bcast:192.168.5.255  Mask:255.255.255.0
	          inet6 addr: fe80::bf27:ebdf:fd41:369c/64 Scope:Link
	          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
	          RX packets:165550 errors:0 dropped:4 overruns:0 frame:0
	          TX packets:125635 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:131148832 (131.1 MB)  TX bytes:26205017 (26.2 MB)

Ensure that the interface is properly connected by looking out for the `inet addr: 192.168.5.103` line. This means that the RPi is connected and has successfully obtained an IP address on that interface. 

To be extra certain, you might want to disconnect the ethernet connection at this stage and reboot the RPi and see if it comes up. Remember the IP address that was issued to `wlan0` here, and use that to try to reconnect to the RPi after it reboots. Most routers will issue the same IP via DHCP to the same host if there is no contention, and on a home/small office network, this usually isn't a problem. This Ubuntu image comes with OpenSSH installed and running, so you'll just need to attempt to connect via SSH once it boots. _Keep in mind that even though the ethernet cable has been disconnected, the script that waits for the ethernet interface to come up still times out at 5 minutes, so you have at least that long to wait._


With that running, you're good to go!

# Resizing the partition

Once your network is up and running rejoice! Until you run a `df -h` or similar and realise that your partition seems a lot smaller that you thought it'd be.

I'm running my Pi off a 64GB microSD card, but when I first looked, `df` was showing me only a *3GB* drive thereabouts. Talk about a shock!

A quick google revealed that the Ubuntu image didn't eat up the rest of my 60GB though. Turns out that RPi image is based off of a 4GB disk image, and when I `dd`ed that over to my sdcard, it just wrote the exact same partition table as well. 

Not to fear though, the [Ubuntu MATE guys have a good writeup on how to resize your RPi's partitions](http://ubuntu-mate.org/raspberry-pi/) _(Search for "Re-size file system")_

In summary though, when you need to do is to delete the smaller ~4GB partition, and rewrite a larger version back to the partition table.

The mounted sdcard is recognised in the RPi usually as `/dev/mmcblk0`, with the `/boot` partition on `/dev/mmcblk0p1` and `/dev/mmcblk0p2`. We'll need to use `fdisk` to delete the partition on `/dev/mmcblk0p2`, and rewrite it back as a larger one.

	$ sudo fdisk /dev/mmcblk0

This gets you into `fdisk`, where you can then `p`rint the partition table, and `d`elete partition `2`. 

Then, create a `n`ew `p`rimary partition, accepting the defaults `fdisk` offers, which would be to use all the remaining space on the sdcard. Remember to `w`rite the changes to disk before quitting `fdisk`, otherwise your changes would not take effect.

Once that's done, reboot the system.

You'll notice if you run `df -h` again once the Pi has booted, that it still says your data is sitting on a smaller ~4GB partition. _What gives?_

Turns out that even though your partition has been resized, the _filesystem_ itself needs to be made aware of its comfy new environment and spread out accordingly.

With a simple

	$ resize2fs /dev/mmcblk0p2

the filesystem is resized, and `df -h` should finally show you the values you've been expecting all along.

# Conclusion

Once all this is done, your RPi should be up and ready to use! Happy hacking!

