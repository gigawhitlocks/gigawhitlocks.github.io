---
title: Cloning Arch Linux to New Hardware
layout: post
---

Two days ago I cloned my Arch install off of my laptop onto my desktop. This was a fun task that illuminated some of the fundamental strengths of Arch's design, and this post talks about how I went about doing this.

###Background
Anyone who's done an Arch Linux installation knows that it's a tedious affair. There's no automated installer, and instead the install disk is a live Arch system with a couple of special Arch-specific commandline tools and the rest of the toolbox one needs for installing an operating system. This, combined with the wonderful documentation on the Arch wiki are the installer.

This is fine when you only want to do one or two Arch installs. The first time is fun on its own as a learning experience, but after that it starts to be a real chore, especially getting all of your software and your environment all set up. This becomes especially apparent if you generally set things up the same way on all of your machines.

I've spent a lot of time on my environment, especially on my laptop where I spend most of my time. This weekend I finally decided that I wanted to copy the setup that I have on my laptop on my desktop, which has had a hosed Arch install on it for almost a year now, that I haven't had the time (due to Arch's tedious install process) to fix.

###Guide
Forewarning: if you want to use this is as a guide, it's not entirely complete. You should be comfortable with Linux and the command line already so that you can fill in the blanks and don't need to copy and paste most commands. Considering this guide involves already having installed one Arch system, if you're here you're probably fine.

First, I download an Arch live CD from archlinux.org and booted it. Since I had a hosed Arch install, I was able to skip the formatting steps in the [installation guide](https://wiki.archlinux.org/index.php/Installation_guide). If you're following along, go ahead and prepare your disk normally.

Get your networking working and install `rsync` with `pacman`.

Mount your new `/` directory at `/mnt` like the install guide tells you, along with any other system directories that you've given their own partitions (like `/boot`). I wanted to preserve the data in `/home` from my hosed install, so I didn't mount `/home`. This process will be even more complete if you do mount your `/home` in `/mnt/home` and copy over your user data too, but I skipped this since the only user data I wanted is already stored in my GitHub dotfiles repository.

Start up an ssh server on your working machine and make sure that `root` login is enabled. It's enabled by default on most systems, but you might need to modify your `/etc/sshd/sshd_config` file to include `PermitRootLogin: Yes` and restart the server. 

I had data in `/mnt` at this point, from the hosed install, which I removed with the very-scary `rm -rf /mnt/*`, but you should hopefully not need to do this.

Now here's the fun part. Transfer over your old install to your new PC with:

{% highlight bash %}
# rsync -aAXv root@remote-server:/* /mnt/ --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/media/*,/lost+found\}
{% endhighlight %}

I added `/home` to the `--exclude` list to avoid my user data, but you should do as necessary here. This transfers all non-special system files to your new install. The flags for `rsync` are taken from [the Arch wiki guide on backing up a full system with rsync](https://wiki.archlinux.org/index.php/Full_System_Backup_with_rsync). They preserve file ownership and permissions across the transfer, and of course if `rsync` fails due to a network outage or something, call it again and it'll start where it left off.

Once this is done, the system is mostly installed. You can skip the `pacstrap` step in the installation guide and move onto [Configure the system](https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system). You'll probably want to set your hostname in `/etc/hostname` as though this is a fresh install, and you definitely still want to run the `genfstab` and `mkinitcpio` commands listed in the install guide. Finish up by [installing a bootloader](https://wiki.archlinux.org/index.php/Boot_Loaders) and then reboot and enjoy your system that is now exactly the same as the previous one.

###TL;DR:
You can clone an Arch install by running the above `rsync` command instead of `pacstrap` during an installation. This eliminates the need for all (or most) post-install setup, and will leave you with a clone of the original system. Pretty cool!