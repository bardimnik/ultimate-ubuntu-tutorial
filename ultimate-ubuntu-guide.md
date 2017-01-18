## Index
- [Introduction](#why-i-wrote-this-another-things-to-do-guide) 
- [Basic OS installation](#ubuntu-installation-process)
  - [Install basic developer packages](#install-some-basic-developer-whistles)
- [Edit `/etc/fstab` mount options](#edit-etcfstab-mount-options)
- [Force automatic SSD TRIM](#force-automatic-trim-for-ssd)
- [Configure Intel HD graphic](#configure-intel-hd-graphic)
- [Edit `/etc/default/grub boot` options](#edit-etcdefaultgrub-for-kernel-options)
- [Reduce system temperature](#pay-attention-to-system-temperature)
- [Edit `/etc/sysctl.conf`](#adding-sysctl-parameters)
- [Save battery with TLP daemon](#save-battery-with-tlp-daemon)
- [Wi-Fi speed tweak with REG domain](#additional-wi-fi-speed-tweak-by-setting-reg-domain)
- [Speed up browsing](#speed-up-browsing-with-profile-sync-daemon)
- [Use Prelink and Preload](#using-prelink-and-preload-for-system-speed-ups)
- [Configure Firewall](#enable-firewall)
- [Set back-up daemon](#use-déjà-dup-for-simple-diff-backups)
- [Disable unwanted services](#disabling-unwanted-services)
- [Fix font-rendering](#fixing-fonts-rendering)
- [Change GTK theme, icons and cursor](#set-google-material-gtk-theme-icons-and-cursor)
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

## Configure Intel HD graphic
Since we use Ubuntu on desktop we need smooth and tearing-free graphic experience.

Let's tune it. Start with this command:
```
sudo apt -y install gksu policykit-1 gdebi mesa-utils
```
Ok. Now dowload [Intel Graphics Update Tool for Linux](https://download.01.org/gfx/ubuntu/16.04/main/pool/main/i/intel-graphics-update-tool/intel-graphics-update-tool_2.0.2_amd64.deb) and install it using "GDebi" installer which will automatically fix and resolve all depencies.  
Next run "Intel Graphics Update Tool" from Ubuntu app menu and install graphics stack.  
Reboot after that.  

Next create this catalog and file:
```
sudo mkdir /etc/X11/xorg.conf.d/
sudo nano /etc/X11/xorg.conf.d/20-intel.conf
```
Put all these lines into this file:
```
Section "Device"
    Identifier    "Intel Graphics"
    Driver        "intel"
    Option        "AccelMethod"    "sna"
    Option        "TearFree"       "true"
    Option        "DRI"            "3"
EndSection
```
Save it and reboot.

Now check if it works by reading output of these commands:
```
cat /var/log/Xorg.0.log | grep sna
cat /var/log/Xorg.0.log | grep Tear
cat /var/log/Xorg.0.log | grep DRI
glxinfo | grep rendering
```
This configuration will force "SNA rendering" and "TearFree option" for our Intel HD graphics card. In other words it forces the use of hardware rendering.  
So these settings will help us to run extremely smoooooooth video playback and animations in Unity GUI.
Also we enable "DRI3" infrastructure becauses it works faster.
Read this [article at Phoronix](http://www.phoronix.com/scan.php?page=article&item=intel-skylake-dri3&num=1)
about difference between "DRI2" and "DRI3" performance.

> **N.B.!**  
> - *Do not use any PPAs with graphic drivers like Oibaf's or Xorg-edgers.
> We have already installed all we need for graphics.
> And any updates will come to our system directly from official Intel repository.*

## Edit `/etc/default/grub` for kernel options
Open file `/etc/default/grub` and change kernel boot options line:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_pstate=enable pcie_aspm=force i915.semaphores=1 i915.i915_enable_rc6=7 i915.i915_enable_fbc=1 i915.lvds_downclock=1 drm.vblankoffdelay=1 ipv6.disable=1"
```
Here we explicitly set CPU governor to `intel_pstate` and set kernel options for Intel graphics that will increase FPS
and decrease power consumption.  
And also we explicitly disable IPv6 because it's a big security mess right now.
Don't use it while community is thinking how to secure this back-door.

Read this [article with test of different kernel options and battery draining.](https://www.phoronix.com/scan.php?page=article&item=intel_i915_power&num=1)

> **N.B.!**  
> - *Diverting from the defaults will mark the kernel as tainted from Linux 3.18 onwards.
> This basically implies using other options than the per-chip defaults is considered experimental
> and not supported by the developers. But that's OK for us. Forget about it.*

Also change this line in `/etc/default/grub`:
```
GRUB_CMDLINE_LINUX="scsi_mod.use_blk_mq=y dm_mod.use_blk_mq=y"
```
This options will enable implementation of the new I/O block layer Multi-queue model.
It's better than default "Deadline" scheduler.

Read this [article about this scheduler.](https://www.thomas-krenn.com/en/wiki/Linux_Multi-Queue_Block_IO_Queueing_Mechanism_(blk-mq))

After all changes update grub records by this command:
```
sudo update-grub
```
Reboot now.

> **N.B.!**  
> - *If you don't really know why you need them do not add any other options from random tutorials in `/etc/default/grub`.
> Also we don't need any acpi options here because we will force-load `thinkpad_acpi` module and all acpi functions
> like Fn-kyes will works just fine. Also we don't need any other `i915` options because they are useless.*

## Pay attention to system temperature
Ok. It's time to think about overheating. First run this:
```
sudo apt -y install lm-sensors thermald intel-microcode smartmontools
sudo service thermald start
sudo sensors-detect
```
When running `sudo sensors-detect` just agree with everything you've been asked.
After that run this command:
```
sudo /etc/init.d/kmod start
sudo systemctl status thermald.service
sudo modprobe -v tcp_westwood
```
Ok. And now edit file `/etc/modules` and put these lines into it:
```
coretemp
thinkpad_acpi
tcp_westwood
```
At this step we install and start sensors and disk-smart detecting utils, detect all suported sensors in our system
and load special acpi module for thinkpad laptops.
Also load `tcp_westwood` module which we will use later for wifi speedups.
Read about `tcp_westwood` and other congestion controls [here](https://www.hindawi.com/journals/jcnc/2012/806272/) and [there](https://arxiv.org/abs/1212.1621).

Reboot now.

> **N.B.!**  
> - *Do not use any fan controlling utils (like "thinkfan") because we use special `thermald` daemon
> who do all the magic automatically. So our laptop will never fry up. I hope.*

Read about [thermald](https://01.org/linux-thermal-daemon/documentation/introduction-thermal-daemon).

## Adding sysctl parameters
In simple terms this configuration speeds up disk subsystem, extremely speeds up those slow-and-laggy Wi-Fi connections
in hotels and airports and defend you from stupid script-kiddies.

Edit file `/etc/sysctl.conf` remove everything it contains and add these lines into it:
```
fs.file-max = 655360
kernel.pid_max = 65535
kernel.kptr_restrict = 1
fs.suid_dumpable = 0
kernel.randomize_va_space = 2
kernel.msgmnb = 65535
kernel.msgmax = 65535
kernel.shmmni = 4096
vm.swappiness = 0
vm.vfs_cache_pressure = 50
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.min_free_kbytes = 131072
net.ipv4.tcp_congestion_control = westwood
net.core.somaxconn = 1024
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_moderate_rcvbuf = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rfc1337 = 1
net.ipv4.route.flush = 1
net.ipv4.ip_no_pmtu_disc = 0
net.ipv4.tcp_fastopen = 1
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_sack = 1
net.ipv4.tcp_fack = 1
net.ipv4.tcp_ecn = 0
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_frto = 2
net.core.rmem_max = 8388608
net.core.wmem_max = 8388608
net.core.rmem_default = 8388608
net.core.wmem_default = 8388608
net.core.optmem_max = 40960
net.ipv4.tcp_rmem = 4096 87380 8388608
net.ipv4.tcp_wmem = 4096 65536 8388608
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.ip_forward = 0
net.ipv4.conf.all.forwarding = 0
net.ipv4.conf.default.forwarding = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.tcp_syncookies = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
```
Save and close this file.
Now run this command for test and load new settings by running command:
```
sudo sysctl -p
```
Reboot now.

This settings affect memory allocation, different network parameters, routings and add a little more security
to network connections.

> **N.B!**  
> - *Those tweaks are optimized for my situation and may not work properly for you. You must know what are you doing
> before blindly copy-paste all these settings. Also you must recalculate all the values if you have any other amount
> of memory rather than 8GB or if you use 1GE wired connection.*

Read [kernel.org](http://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt), [Wikipedia](http://en.wikipedia.org/wiki/Sysctl) and my [reference sysctl.conf](here will be link to my file) to understand what, where and why.

> **N.B.!**  
> - *Do not forget to load `tcp_westwood` module and then add it in the startup loading modules in `/etc/modules`.*

Also do not forget to disable IPv6 in grub or add these parameters in `/etc/sysctl.conf`:
```
net.ipv6.route.flush = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

## Save battery with TLP daemon
Now we will install "TLP utility" that will dramatically decrease power consumption and will increase battery-driven working time. And yes it was designed especially for Thinkpads. Absolutely must have!

Even if TLP packages are available via the official Ubuntu repository we want to use PPA for newer versions:
```
sudo add-apt-repository ppa:linrunner/tlp
sudo apt update
sudo apt -y install tlp tlp-rdw tp-smapi-dkms acpi-call-dkms
```
Now we need to enable all related services and disable `ondemand` governor because we are using `intel_pstate`:
```
sudo systemctl enable tlp.service
sudo systemctl enable tlp-sleep.service
sudo systemctl enable NetworkManager-dispatcher.service
sudo systemctl mask systemd-rfkill.service
sudo tlp start
sudo tlp stat
sudo update-rc.d ondemand disable
sudo systemctl disable ondemand.service
sudo systemctl mask ondemand.service
sudo update-rc.d -f ondemand remove
```
Ok. Everything is running. Now we need to fine tune config file `/etc/default/tlp`. Remove all lines in it and add these:
```
TLP_ENABLE=1
TLP_DEFAULT_MODE=AC
DISK_IDLE_SECS_ON_AC=0
DISK_IDLE_SECS_ON_BAT=2
MAX_LOST_WORK_SECS_ON_AC=60
MAX_LOST_WORK_SECS_ON_BAT=120
CPU_SCALING_GOVERNOR_ON_AC=performance
CPU_SCALING_GOVERNOR_ON_BAT=powersave
CPU_BOOST_ON_AC=1
CPU_BOOST_ON_BAT=0
SCHED_POWERSAVE_ON_AC=0
SCHED_POWERSAVE_ON_BAT=1
NMI_WATCHDOG=0
ENERGY_PERF_POLICY_ON_AC=performance
ENERGY_PERF_POLICY_ON_BAT=powersave
DISK_DEVICES="sda"
DISK_APM_LEVEL_ON_AC="254 254"
DISK_APM_LEVEL_ON_BAT="128 128"
SATA_LINKPWR_ON_AC=max_performance
SATA_LINKPWR_ON_BAT=min_power
AHCI_RUNTIME_PM_TIMEOUT=15
PCIE_ASPM_ON_AC=performance
PCIE_ASPM_ON_BAT=powersave
WIFI_PWR_ON_AC=off
WIFI_PWR_ON_BAT=on
WOL_DISABLE=Y
SOUND_POWER_SAVE_ON_AC=0
SOUND_POWER_SAVE_ON_BAT=1
SOUND_POWER_SAVE_CONTROLLER=Y
BAY_POWEROFF_ON_BAT=0
BAY_DEVICE="sr0"
RUNTIME_PM_ON_AC=on
RUNTIME_PM_ON_BAT=auto
RUNTIME_PM_ALL=1
USB_AUTOSUSPEND=1
RESTORE_DEVICE_STATE_ON_STARTUP=1
START_CHARGE_THRESH_BAT0=75
STOP_CHARGE_THRESH_BAT0=85
START_CHARGE_THRESH_BAT1=77
STOP_CHARGE_THRESH_BAT1=87
DEVICES_TO_DISABLE_ON_LAN_CONNECT="wifi"
DEVICES_TO_ENABLE_ON_LAN_DISCONNECT="wifi"
```
Save and exit.
These settings will enable power-saving functions for CPU, SSD, Wi-Fi adapter and Sounblaster.
Also it will set battery charging thresholds to save our battery life-cycle and minimize wearing.
And also it will automatically switch on/off WIFI when we will connect/disconnect LAN wire.

Run these commands while on charger:
```
sudo tlp start
sudo tlp stat
sudo tlp-stat -w
sudo tlp setcharge 75 85 BAT0
sudo tlp setcharge 77 87 BAT1
```
And now using laptop battery run these:
```
sudo tlp start
sudo tlp stat
sudo tlp-stat -w
```
Read all the outputs and check if perfomance/power-save switching works. Also check if your governor is `intel_pstate`
and that TLP knows about it.

Reboot and charge your battery with full factory thresholds last time:
```
sudo tlp fullcharge BAT0
sudo tlp fullcharge BAT1
```
All next charges will be battery-life-optimized by TLP.
Read for more information [here](http://linrunner.de/en/tlp/docs/tlp-linux-advanced-power-management.html) and [there](https://github.com/linrunner/TLP/issues/183#issuecomment-175228097)

> **N.B.!**  
> - *Once after kernel updated I saw a strange thing when TLP stops working. So now just in case after every new kernel update I simply reinstall all TLP-related stuff with this command:*
```
sudo apt install --reinstall tlp tlp-rdw tp-smapi-dkms acpi-call-dkms lm-sensors thermald intel-microcode smartmontools
```

## Additional Wi-Fi speed tweak by setting REG domain
> **N.B.!**  
> - *This dirty hack can be illegal in your country. But who cares right?*

This hack will tell your Wi-Fi adapter that he is on vacation in Venezuela (the most free country ha-ha) and thats why
he must increase `tx_power` of adapter in 2402MHz-2482MHz range.

Viva la poor Venezuela radio regulation!

Run these commands:
```
sudo iw reg get
sudo iw reg set VE
```
Now edit this file `/etc/default/crda` and set our new reg domain:
```
REGDOMAIN=VE
```
Reboot and check if it works:
```
sudo iw reg get
```
Easy-peasy! Enjoy!

## Speed up browsing with "profile-sync-daemon"
Recent browsers are hungry fat pigs. They can eat all your memory and shit out all your disk.
So using this small helpful daemon is absolutely must!
It is tiny pseudo-daemon designed to manage browser profile(s) in tmpfs and to periodically sync back
to the physical disc (HDD/SSD). This is accomplished by an innovative use of rsync to maintain synchronization between
a tmpfs copy and media-bound backup of the browser profile(s). Additionally, `psd` features several
crash recovery features. Since we have enough memory let's use it!

We need to add PPA and install daemon:
```
sudo add-apt-repository ppa:graysky/utils
sudo apt update
sudo apt -y install profile-sync-daemon
```
Run psd the first time which will create `$XDG_CONFIG_HOME/psd/psd.conf`:
```
psd
```
Now uncomment and edit these lines in config file `/home/user_name/.config/psd/psd.conf`:
```
USE_OVERLAYFS="yes"
BROWSERS="google-chrome" # set here you browser name from list
USE_BACKUPS="yes"
```
Save and exit.
Now parse config file:
```
psd p
```
To use overlayfs feature we need to add this line in `/etc/sudoers`:
```
your_username_here ALL=(ALL) NOPASSWD: /usr/bin/psd-overlay-helper
```
And run `psd p` command one more time.
Now enable daemon:
```
systemctl --user start psd
systemctl --user enable psd
```
Reboot and check if it works with `psd p` command.
This daemon will speed up browsing by keeping browser profile in memory.
Additional reading is in famous [Arch Wiki](https://wiki.archlinux.org/index.php/profile-sync-daemon)

## Using Prelink and Preload for system speed ups
Ok. First - **DO NOT USE PRELINK! NEVER!**
Neither on desktop nor on server.
It is useless and deprecated. It gains no speedups on modern hardware but can cause a lot of problems.
Read this [discussion](https://pagure.io/fesco/issue/1183).

But we still can use preload daemon.
It is a tool that monitors and keeps a history of the user's most frequently used applications and the files
that those applications load upon their execution. And based on that data, it then tries to guess what app the user
will be most likely to open in the near future and then loads that data from the disk to the page cache,
before they're requested. In plain simple terms - once preload is installed, after a while you should be able to open
your frequently used applications much faster. Sounds good right?

Install it:
```
sudo apt -y install preload
```
Now edit parameter "sortstrategy" in `/etc/preload.conf`, set it to "0" because we use SSD.
After that restart the daemon:
```
sudo /etc/init.d/preload restart
```
OK. It is up and running! After some time you will notice some perfomance increasing.

## Enable Firewall
Yep we need firewall. As we are using laptop then our firewall strategy will be simple:
*DENY* all incoming, *DENY* and *DISABLE* routing and *ALLOW* all outgoing traffic.

So run this commands:
```
sudo apt -y install ufw gufw
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
```
Ok. Now our firewall is up and enabled.
Now open "Gufw" utility from dash and check everything.
Also you can disable logs here. I am pretty sure that you don't really need them.
That's it!

## Use Déjà Dup for simple diff-backups
Déjà Dup is a simple backup tool. It hides the complexity of backing up the
"Right Way" (encrypted, off-site, and regular) and uses `duplicity` as the back-end.

It is nicely integrated in Ubuntu and Nautilus and runs smooth.
Just don't forget to config it properly. And also you can edit more parameters of it in dconf-editor by opening
this path `org.gnome.DejaDup`. Here you can set custom periods of back-ups or set on-line storage.

## Disabling unwanted services
> **N.B.!**  
> - *You can ruin everything at this step! You absolutely must know what are you doing and why!*

Run this command to unhide all apps:
```
sudo sed -i "s/NoDisplay=true/NoDisplay=false/g" /etc/xdg/autostart/*.desktop
```
Then open Autostarts utility in Unity Dash and disable startups that you don't really need.

Next go to services.
Here are commands to list all active services:
```
service --status-all
systemctl list-unit-files | grep enabled
systemctl --state=active
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```
Read the output and think about it.

Now stop and disable all useless junk:
```
sudo systemctl stop bluetooth.service bluetooth.target brltty.service brltty-udev.service apport.service apport-forward.socket apt-daily.service apt-daily.timer cups-browsed.service cups.path cups.service cups.socket saned.service saned.socket whoopsie.service kerneloops.service rsyslog.service syslog.socket syslog.service speech-dispatcher.service

sudo systemctl disable bluetooth.service bluetooth.target brltty.service brltty-udev.service apport.service apport-forward.socket apt-daily.service apt-daily.timer cups-browsed.service cups.path cups.service cups.socket saned.service saned.socket whoopsie.service kerneloops.service rsyslog.service syslog.socket syslog.service speech-dispatcher.service

sudo systemctl mask bluetooth.service bluetooth.target brltty.service brltty-udev.service apport.service apport-forward.socket apt-daily.service apt-daily.timer cups-browsed.service cups.path cups.service cups.socket saned.service saned.socket whoopsie.service kerneloops.service rsyslog.service syslog.socket syslog.service speech-dispatcher.service
```
Also `mask` action will prevent activation of those services even when you accidentally update them or reinstall.

OK. Here is the basic start list of things I don't use and always disable:

- I don't print anything - disable `cups`.
- I don't scan any images - disable `saned`.
- I am not blind - disable `brltty`.
- I do all my system updates in manual mode - disable all `apt` services.
- I don't want to send any reports - disable `kerneloops`, `apport` and `whoopsie` services.
- I don't want logs on the laptop - disable them. (We stil can use systemd `journalctl`).
- I don't want my laptop speaking - disable `speech-dispatcher`.

Also you can kill AppArmor service that is useless for my taste (because of his outdated and useless profiles).
But it will also kill ability of proper installing and using Ubuntu Snap packages. What a shame!
But I think they are useless too. Just do some research and find what services you want to be disabled.

By the way there are a lot of useless hardcoded services for my taste.
I was having an experiment - If I disable all of them the memory footprint of Ubuntu will be about 250-300Mb.
But because of poor architecture and coding quality in Linux and Ubuntu in particular we don't want to do this because
it will make our OS unstable and useless (more than now). So give your memory to that hungry monster.

> **N.B.!**  
> - *You can kill a lot of junk services. Just read about them find their functions and dependencies and
> if you don't want it you kill it. Actually I always disable and remove Zeitgeist, Avahi, AppArmor, Snap, Evolution
> and most of the preinstalled bloat-ware because I am sure that I don't need them.*

## Fixing fonts rendering
If you hate default fat Ubuntu fonts and how they render this part is for you!

> **N.B.!**  
> - *Do not use "infinality patch" for font rendering. It's abandoned and deprecated.
> And it surely will kill all letters geometry. Actually I am completely don't understand why this stupid *infinality patch* was so popular.*

Because my laptop has only 12.5" low-quality screen I will use Roboto hinted fonts for everything and
set Chrome OS like font rendering. I found that it is nice for my eyes and save space on a such micro-screen.

First we must install new fonts:
```
sudo apt -y install fonts-croscore fonts-roboto-hinted ttf-mscorefonts-installer
```
After installation set all fonts in "Unity Tweak Tool" to Roboto at 10.5 size.
Also set Roboto Bold 10.5 for window heading.

Now open Google Chrome (the only one browser for Linux that doesn't suck), go to Settings and change fonts in it.
From up to down:
```
Tinos
Tinos
Arimo
Cousine
```
All at 16 size. OK. Close it.

Now we will fix the rendering.
Create two files: `~/.Xdefaults` and `~/.Xresources` with the same content:
```
Xft.antialias: 1
Xft.autohint:  0
Xft.hinting:   1
Xft.hintstyle: hintslight
Xft.lcdfilter: lcddefault
Xft.rgba:      rgb
```

Next create files: `~/.fonts.conf`, `~/.fonts/fonts.conf` and  `~/.config/fontconfig/fonts.conf` with the same content:
```
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
<!-- Set preferred serif, sans serif, and monospace fonts. -->
<alias>
<family>serif</family>
<prefer><family>Tinos</family></prefer>
</alias>
<alias>
<family>sans-serif</family>
<prefer><family>Arimo</family></prefer>
</alias>
<alias>
<family>sans</family>
<prefer><family>Arimo</family></prefer>
</alias>
<alias>
<family>monospace</family>
<prefer><family>Cousine</family></prefer>
</alias>
<!-- Aliases for commonly used MS fonts. -->
<match>
<test name="family"><string>Arial</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Arimo</string>
</edit>
</match>
<match>
<test name="family"><string>Helvetica</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Arimo</string>
</edit>
</match>
<match>
<test name="family"><string>Verdana</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Arimo</string>
</edit>
</match>
<match>
<test name="family"><string>Tahoma</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Arimo</string>
</edit>
</match>
<match>
<!-- Insert some stupid joke here -->
<test name="family"><string>Comic Sans MS</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Arimo</string>
</edit>
</match>
<match>
<test name="family"><string>Times New Roman</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Tinos</string>
</edit>
</match>
<match>
<test name="family"><string>Times</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Tinos</string>
</edit>
</match>
<match>
<test name="family"><string>Courier New</string></test>
<edit name="family" mode="assign" binding="strong">
<string>Cousine</string>
</edit>
</match>
<!-- Improve font rendering -->
<match target="font">
<edit name="antialias" mode="assign">
<bool>true</bool>
</edit>
<edit name="autohint" mode="assign">
<bool>false</bool>
</edit>
<edit name="embeddedbitmap" mode="assign">
<bool>false</bool>
</edit>
<edit name="hinting" mode="assign">
<bool>true</bool>
</edit>
<edit name="hintstyle" mode="assign">
<const>hintslight</const>
</edit>
<edit name="lcdfilter" mode="assign">
<const>lcddefault</const>
</edit>
<edit name="rgba" mode="assign">
<const>rgb</const>
</edit>
</match>
</fontconfig>
```
Ok. Save, exit and reboot.

Now check if everything works, run this:
```
for family in serif sans-serif monospace Arial Helvetica Verdana "Times New Roman" "Courier New"; do
    echo -n "$family: "
    fc-match "$family"
done
```
Read the output. If all is OK you should see Tinos, Arimo and Cousine fonts applied as default fonts.
That's it! Now we have beautiful crispy fonts!

## Set Google Material GTK theme, icons and cursor
As a developer I like darkness. Also I like Google Material color palette and I found beautiful GTK theme that uses it.
Also I found suitable flat icons pack that match this theme. Let's install them:
```
sudo add-apt-repository ppa:tista/adapta
sudo apt update
sudo apt -y install adapta-gtk-theme
wget -qO- https://raw.githubusercontent.com/PapirusDevelopmentTeam/papirus-icon-theme/master/install-papirus-root.sh | sh
```
Now go to Appearance settings in "Unity Tweak Tool" and set theme to "Adapta Nokto" and Icons to "Papirus Dark".
Also create file `~/.config/gtk-3.0/settings.ini` with this content:
```
[Settings]
gtk-application-prefer-dark-theme=1
```
Save, exit, reboot and now you get a fantastic Dark Google Material theme and perfectly matched flat icons in Unity.

Also to make Unity greeter match our theme create file `/usr/share/glib-2.0/schemas/10_unity-greeter.gschema.override` with this content:
```
[com.canonical.unity-greeter]
theme-name='Adapta-Nokto'
background-color='#455A64'
icon-theme-name='Papirus-Dark'
play-ready-sound=false
draw-grid=false
font-name='Roboto 10.5'
```
Save and exit. Now recompile schema:
```
sudo glib-compile-schemas /usr/share/glib-2.0/schemas/
```
This will not only make our greeting screen stylish but also it will disable that annoying drum sound at login.
OK. Let's go deeper.

One more thing is to set some new cursor. I like "Open Zone" cursor.
Dowload it and unzip - https://dl.opendesktop.org/api/files/download/id/1462316428/111343-OpenZone-1.2.5.tar.xz
Now choose which version of it you like and copy it to `/usr/share/icons` (I like black slim version).
Again, go to "Unity Tweak Tool" and set your new cursor.
Also adjust here Unity Panel color to match our dark theme.
Beautiful! Do you want more whistles? Ok!

Now we will fix Ubuntu notification bubbles. Run this:
```
sudo add-apt-repository ppa:leolik/leolik
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
```
It will update our `notify-osd`.
Now create file `~/.notify-osd` with this content:
```
slot-allocation = dynamic
bubble-expire-timeout = 10sec
bubble-vertical-gap = 70px
bubble-horizontal-gap = 16px
bubble-corner-radius = 18px
bubble-icon-size = 33px
bubble-gauge-size = 6px
bubble-width = 240px
bubble-background-color = 00bcd4
bubble-background-opacity = 100%
text-margin-size = 10px
text-title-size = 100%
text-title-weight = bold
text-title-color = ffffff
text-title-opacity = 100%
text-body-size = 90%
text-body-weight = normal
text-body-color = ffffff
text-body-opacity = 100%
text-shadow-opacity = 10%
location = 1
bubble-prevent-fade = 0
bubble-close-on-click = 1
bubble-as-desktop-bg = 0
```
This config will make our notifications beautifully match GTK theme and also
adds option to close notification bubble by mouse click.

Now restart notify-osd and check if it works by running these commands:
```
pkill notify-osd
notify-send Date "`date`"
```
Do you see the magic? That's it!

Also you can change icons in "Unity Tweak Tool", "LibreOffice", "Filezilla" e.t.c just search [Papirus GitHub page](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) for this.

And now we will install patched `sni-qt` to make icons on Unity panel nicer:
```
sudo add-apt-repository ppa:andreas-angerer89/sni-qt-patched
sudo apt update
sudo apt -y install sni-qt sni-qt:i386 hardcode-tray
hardcode-tray -ug
hardcode-tray --conversion-tool Inkscape
```
OK. Reboot now and enjoy.

Read more information about these tweaks here:  
[Adapta GTK theme GitHub](https://github.com/adapta-project/adapta-gtk-theme)  
[Papirus Icon set GitHub](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme )  
[Open Zone cursor theme](https://www.gnome-look.org/p/999999/)  
[Leolik's PPA with patched notify-osd](https://launchpad.net/~leolik/+archive/ubuntu/leolik)  
[Herdcode tray patch](https://github.com/bil-elmoussaoui/Hardcode-Tray)  

## Powerline for Gnome Terminal
Powerline is a statusline plug-in for vim, and provides statuslines and prompts for several other applications,
including zsh, bash, tmux, IPython, Awesome and Qtile.

Run this to install patched-for-powerline fonts:
```
git clone https://github.com/powerline/fonts.git
cd fonts/
sudo ./install.sh
```
Now set in "Unity Tweak Tool" Roboto Mono for Powerline font with size 10.5 for monospaced fonts.
Or set it in Gnome Terminal Profile settings.

Next run:
```
sudo apt -y install powerline
```

Add these lines to your `~/.bash.rc`:
```bash
force_color_prompt=yes

if [ "$TERM" == "xterm" ]; then
    export TERM=xterm-256color
fi

if [ -f `which powerline-daemon` ]; then
    powerline-daemon -q
    POWERLINE_BASH_CONTINUATION=1
    POWERLINE_BASH_SELECT=1
    . /usr/share/powerline/bindings/bash/powerline.sh
fi
```
And at last run this:
```
mkdir -p ~/.config/powerline
cat <<-'EOF' > ~/.config/powerline/config.json
{
    "ext": {
        "shell": {
            "theme": "default_leftonly"
        }
    }
}
EOF
powerline-daemon --replace
```
OK. Now it is properly set up.
Close and open again terminal and try to edit some git branch in terminal and you will see the magic!
More information on [Powerline GitHub page](https://github.com/powerline/powerline) and in [documentation](https://powerline.readthedocs.io/en/master/overview.html)
