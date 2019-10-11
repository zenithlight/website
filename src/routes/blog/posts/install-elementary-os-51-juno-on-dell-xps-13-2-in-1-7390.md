---
title: Install Elementary OS 5.1 Juno on Dell XPS 13 2-in-1 7390
date: 2019-10-07T22:29:44.000Z
---

Ubuntu variants on this computer are a bit more of a hassle than normal, but after trying a bunch of different things and reading a bunch of posts on Reddit and StackExchange, I've found a process that mostly works. I've distilled my learnings into this post for anyone else who's trying to do the same.

You will need a Ethernet-to-USB adapter for the initial setup. Once we have WiFi working you can stop using it.

You will also need a USB installation media for the latest elementary OS (and the included dongle if your adapter/media require it). Instructions to download and create media can be found on the [elementary website](https://elementary.io).

# Installation

There is a bug in the `intel_lpss` driver which causes booting to fail with Secure Boot disabled. This applies to both the installation media and the finished installation. In BIOS settings, you need Secure Boot **enabled** for the installation. We'll disable it later. However, you need to do the fixes described later so you can turn off Secure Boot, allowing WiFi drivers and other drivers to be installed.

If you're trying to dual boot, make sure you have Windows installed already and you shrunk the Windows partition to create free space for elementary. Otherwise, you can just go through the installer as normal.

# Kernel patch

The easy fix described on [ArchWiki](https://wiki.archlinux.org/index.php/Dell_XPS_13_2-in-1_(7390)) is to blacklist the `intel_lpss_pci` module. However, it doesn't work with the Ubuntu-flavored kernel. You'll need a mainline kernel. And if you just blacklist the module, you'll lose support for touchscreen, 802.11ac WiFi, and other useful things.

The best fix is to recompile the Linux kernel with a patched `intel_lpss` driver.

```
git clone https://github.com/torvalds/linux.git

cd linux

git checkout v5.3 # or latest stable

wget https://pastebin.com/raw/sqPv8ShP

git apply sqPv8ShP
```

The source of the patch is [this Reddit thread](https://www.reddit.com/r/Dell/comments/cx0fkc/xps_13_2_in_1_7390_linux_boot_attempt/eyxevml/).

Build the patched kernel using the second method in [this answer](https://askubuntu.com/a/163348). Then install the generated dpkg files. The compilation will take 30-120 minutes.

At this point you should be able to boot using your new kernel. In the GRUB menu, select "Advanced options for Elementary OS" and you should be able to see your new kernel and boot from it.

Verify your computer boots properly and if so, you probably want to set your new kernel to be the default. Follow [this guide](https://unix.stackexchange.com/questions/198003/set-default-kernel-in-grub) to do so.

# Wifi

The XPS uses a Killer AX1650 wireless chip. The drivers are not included in the installation by default. You can follow [this guide](https://support.killernetworking.com/knowledge-base/killer-ax1650-in-debian-ubuntu-16-04/) to install them.

# Bluetooth

You already have the `linux-firmware` repository from the last step, so

```
cp -r linux-firmware/intel/* /lib/firmware/intel
```

and reboot. Your WiFi and Bluetooth should be working now.

# Remaining drivers

Open AppCenter and update to get the 5.1 update and drivers for various peripherals, including video acceleration. Reboot again and check out the new greeter.

# Pen

Not tested due to missing hardware. However, `Wacom HID 48EE` device is detected, so it may be working.

# Camera

Not working due to missing driver.

# Fingerprint sensor

Not working due to missing driver.

# Power management

You may want to install power management tools to get more battery life out of your device.

```
sudo apt install tlp powertop
```

# Boot

Sometimes the system refuses to POST while connected to AC power. I suspect it's related to the "low battery" BIOS warning. I haven't spent much time trying to find a configuration to avoid this, but you can usually connect to power for a couple minutes so the device has at least 2% battery, disconnect until you can get to the GRUB menu, then reconnect to power.

# Contact

If you run into issues, have questions, or see something out of date, please contact me at alexis@alexisanand.com.
