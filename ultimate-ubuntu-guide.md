## Index
- [Introduction](#why-i-wrote-this-another-things-to-do-guide) 
- [Basic OS installation](#ubuntu-installation-process)
  - [Install basic developer packages](#install-some-basic-developer-whistles)
- [Edit `/etc/fstab` mount options](#edit-etcfstab-mount-options)
- Force automatic SSD TRIM
- Configure Intel HD graphic
- Edit `/etc/default/grub boot` options
- Reduce system temperature
- Edit `/etc/sysctl.conf`
- Save battery with TLP daemon
- Wi-Fi speed tweak with REG domain
- Speed up browsing
- Use Prelink and Preload
- Configure Firewall
- Set back-up daemon
- Disable unwanted services
- Fix font-rendering
- Change GTK theme, icons and cursor
- Use Powerline for Gnome Terminal
- Disable guest account
- Fix bug in Avahi
- Install Upwork Desktop App

## Why I wrote this ~~another things to do~~ guide
The main reason why I wrote this guide is because I need it as my own memo.

Also because I'm sick of other Ubuntu guides that are often written by idiots!
Most often they are useless and written with only one purpose - to collect traffic from Google.
And they all are almost the same, copy each other or even tell you such stupid dead-simple things like
Skype installation from Ubuntu Software Center.
Hi SEO! :fu: you SEO!

Also most of guides are written for the very beginners or for fresh "swithchers".
I'm pretty sure that you've seen a lot of Ubuntu Linux  installation guides here and there across the web.
All these blogs for n00bs like OMG!Ubuntu! or Webupd8 you know them...
So something more complicated must be written.

OK. So what makes this guide different?  
It is for more hard-cored PC users and **It is applicable to reality.**  

And thats why:

- I have been using Linux as my only OS for desktop and servers for more than 10 years on all my desktops and servers
- I always use this guide as my memo to set up my new machine
- I do work, travel and I make money using my Linux laptop
- Also this guide tries to be simple yet extremely effective in a real life

So enough talking and let the fun begin!

## Ubuntu installation process
Here is my current laptop with pretty common configuration:

- Model: Thinkpad x240 :shit:
- CPU: Core i5-4300U
- Mem: 8GB
- Storage: 240GB SSD
- Wi-Fi: Intel 7260 adapter

> **N.B.!**  
> - *Do not buy current Thinkpads because they are crap* :shit:  
> - *Do not mess with damned Lenovo who ruin IBM's Thinkpad line* :cry:  

Also I ripped out that f#ckin useless x240 touchpad and replaced it with touchpad from x250 that have those lovely real
buttons for using with trackpoint. So I can't tell you how to configure x240 touchpad with those virtual buttons.
I was trying different configs and even patched `libinput` for it. But no matter what it is impossible to use it
without pain and tears. That's why I ripped it out. And I recommend you to do the same.

OK. Here are the steps to reproduce:  

1. Download Ubuntu 16.04 LTS image and create bootable USB stick.
2. Go to BIOS and **disable** "Secure Boot" and "Security Chip", **enable** "USB UEFI BIOS Support" and all in "I/O Port Access", also **set** "Legacy Only" boot option.
3. Boot in "Live Mode" by choosing your bootable media in BIOS.
4. Now when you are in "Live Mode" run "Gnome Disks utility" or "GParted" to format your SSD with zero-rewriting.
5. Run Ubuntu installer and when it will ask you about your target disk for installation choose "Manual Partitioning" option and create the **only one** partition with BTRFS file system and root mount-point (`/`). It will automatically create BTRFS subvolumes `@/` and `@home` which is useful.
6. Next set your username, pass, e.t.c and just wait until installation process will finish.
7. Now boot to your fresh installed Ubuntu and check "Software updates" options and enable "Partner repository". Also enable (:heavy_check_mark:) `intel-microcode` on "Drivers" tab.
8. Then update Ubuntu to current state by running this command:
```
sudo apt update && sudo apt -y upgrade && sudo apt -y dist-upgrade
```
Clean up and Reboot after command finishes:
```
sudo apt -y autoremove && sudo fstrim --all && sudo reboot now
```
Thats it. Now you have installed and updated Ubuntu 16.04 LTS ready for configuration and tweaks.

### Install some basic developer whistles
Run this command:
```
sudo apt -y install autoconf automake build-essential libssl-dev gettext git-core git gitg subversion git-svn checkinstall deborphan wget curl cdbs devscripts dh-make fakeroot check libtool gcc liblzo2-dev g++ libglib2.0-dev libdbus-1-dev libdbus-glib-1-dev libxml2-dev unace unrar zip unzip p7zip-full p7zip-rar sharutils rar uudeview mpack arj cabextract file-roller genisoimage gtk2-engines-pixbuf gtk2-engines-murrine inkscape libgdk-pixbuf2.0-dev librsvg2-dev libsass0 libxml2-utils pkg-config pysassc libqt4-svg unity-tweak-tool dconf-editor sysfsutils libcurl4-gnutls-dev libexpat1-dev libz-dev
```
I will not explain this step ok? I think you are not so stupid and can read packages names.

## Edit `/etc/fstab` mount options

Here we will set mount options for our partitions.  
Edit file `/etc/fstab` with your favorite editor and change options line for every partition or BTRFS subvolume:
```
UUID=some_UUID    /        btrfs    rw,ssd_spread,space_cache,compress=lzo,autodefrag,noatime,subvol=@        0    1
UUID=some_UUID    /home    btrfs    rw,ssd_spread,space_cache,compress=lzo,autodefrag,noatime,subvol=@home    0    2
```
Here we explicitly define the most useful and proper settings for our SSD.  
You can read more about BTRFS mount options on [btrfs.wIki.kernel.org](https://btrfs.wiki.kernel.org/index.php/Mount_options)

> **N.B.!**  
> - *Do not use option `nodatacow`!
It is widely recommended over the internet by random idiots but the thruth is - perfomance gained from this option
is usually about zero. But it will ruin most advantages of BTRFS! For examle it will disable compression.*

> - *Do not use option `nodirtime`. It is very common in tutorials but you don't need it.
I thought everybody knew that `noatime` automatically implements `nodirtime`.*

> - *Do not use  "continuous trim" with `discard` option. This is stupid and will slow down your iops.
Use periodic trim. For example once a week - thats enough for desktop.*

> - *Do not use `/tmp`, `/var/log`, e.t.c in `tmpfs` for "prolonging" SSD life. It really doesn't affect SSDs life
but can cause you a lot of problems with some applications.*

Save and reboot when you'll be done.

After that run this commands:
```
sudo btrfs property set / compression lzo
sudo btrfs property set /home compression lzo
sudo btrfs filesystem defragment -r -v -clzo /
sudo chattr +c /
sudo btrfs filesystem defragment -r -v -clzo /home
sudo chattr +c /home
sudo btrfs balance start /
sudo btrfs balance start /home
```
Here we will defragment our `@/` and `@home` subvolumes with `lzo-compression`. And set attribute `compressed` to them.  
And at last we will [balance](https://btrfs.wiki.kernel.org/index.php/Manpage/btrfs-balance) block groups on a BTRFS.

Also because of our `/etc/fstab` config all new files will be compressed and our FS will be defragmented automatically.

> **N.B.!**  
> - *Do not forget to turn on `write cache feature` in "Gnome Disks utility".
> Just go to your SSD settings in this utility and turn on this feature.*

Reboot again.

> **N.B.!**  
> - *Because of BTRFS design it is recommended to disable `copy-on-write` by setting special attribute for directories
> and/or files with heavy R-W usage such as catalog with virtual machines images or catalog/files that you share via torrents.*

You can do it with this command:
```
sudo chattr +C /some_dir/
```
Be carefull - letter `C` is capitalized in that command.

## Force automatic TRIM for SSD
First of all check if your SSD supports TRIM feature. Run command:
```bash
sudo hdparm -I /dev/sda | grep "TRIM supported"
```
Ok. Supported! If not then your SSD is a shit and you need to replace it ASAP!

Now check if we have periodic trim job by running this command:
```
cat /etc/cron.weekly/fstrim
```
Ok. We got it. But I don't believe in it. So I will force periodic TRIM run by these commands:
```
sudo cp /usr/share/doc/util-linux/examples/fstrim.service /etc/systemd/system
sudo cp /usr/share/doc/util-linux/examples/fstrim.timer /etc/systemd/system
sudo systemctl enable fstrim.timer
sudo systemctl enable fstrim.service
```
Since now once a week our SSD will be trimmed automatically.
We can forget about this task and live our life.
