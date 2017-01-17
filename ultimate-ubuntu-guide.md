## Index
- [Introduction](#why-i-wrote-this-another-things-to-do-guide) 
- Basic OS installation
- Edit `/etc/fstab` mount options
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
> - *Do not buy current Thinkpads because they are crap*:shit:  
> - *Do not mess with damned Lenovo who ruin IBM's Thinkpad line*:cry:  

Also I ripped out that f#ckin useless x240 touchpad and replaced it with touchpad from x250 that have those lovely real
buttons for using with trackpoint. So I can't tell you how to configure x240 touchpad with those virtual buttons.
I was trying different configs and even patched `libinput` for it. But no matter what it is impossible to use it
without pain and tears. That's why I ripped it out. And I recommend you to do the same.

OK. Here are the steps to reproduce:  

1. Download Ubuntu 16.04 image and create bootable USB stick.
2. Go to BIOS and **disable** "Secure Boot" and "Security Chip",**enable** "USB UEFI BIOS Support" and all in "I/O Port Access", also **set** "Legacy Only" boot option.
3. Boot in "Live Mode" by choosing your bootable media in BIOS.
4. Now when you are in "Live Mode" run "Gnome Disks utility" or "GParted" to format your SSD with zero-rewriting.
5. Run Ubuntu installer and when it will ask you about your target disk for installation choose "Manual Partitioning" option and create the **only one** partition with BTRFS file system and root mount-point (`/`). It will automatically create BTRFS subvolumes `@/` and `@home` which is useful.
6. Next set your username, pass, e.t.c and just wait until installation process will finish.
7. Now boot to your fresh installed Ubuntu and check "Software updates" options and enable "Partner repository". Also enable (:heavy_check_mark:) `intel-microcode` on "Drivers" tab.
8. Then update Ubuntu to current state by running this command:
```
sudo apt update && sudo apt -y upgrade && sudo apt -y dist-upgrade
```
9. Clean up and Reboot after command finishes:
```
sudo apt -y autoremove && sudo fstrim --all && sudo reboot now
```
Thats it. Now you have installed and updated Ubuntu ready for configuration and tweaks.
