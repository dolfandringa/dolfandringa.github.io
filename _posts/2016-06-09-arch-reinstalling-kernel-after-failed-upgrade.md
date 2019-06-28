---
layout: post
title: Reinstalling the kernel on Arch Linux after a failed upgrade.
date: "2016-06-09"
tags: 
    - ArchLinux
    - live
    - kernel
    - upgrade
    - modules
---
Since a few weeks my laptop battery is dead. I am still trying to work with Asus to get a new one as it is still under warranty (but sending it in for repair will apparently take 2 months while I already know that a new battery will solve the problem). So I am working of the net power only. The power sockets in the Philippines aren't great though, and Asus decided to integrate the plug into the transformer/regulator unit for my laptop charger, creating a big heavy plug that is really too heavy for vertical wall sockets. So as you can guess from the title of this blog post, yesterday I was working on my laptop and decided to install upgrades on my Arch Linux box. While I was doing this though, I accidentally touched the power cord of my laptop, which then fell out of the wall socket. This isn't the first time this happened, but unfortunately my computer was in the middle of a kernel upgrade.... And of course the kernel modules weren't fully installed, and the initramfs wasn't fully updated either. 

So when I started my laptop again, it dropped me into the emergency mode. After looking at the logs with ```journalctl -xe``` and going almost all the way to the top of the logs, I found these error messages:

{% highlight bash %}
systemd[1]: Starting Load Kernel Modules...
systemd[1]: systemd-modules-load.service: Main process exited, code=exited, status=1
systemd[1]: Failed to start Load Kernel Modules.
systemd[1]: systemd-modules-load.service: Unit entered failed state.
systemd[1]: systemd-modules-load.service: Failed with result 'exit-code'.
systemd-modules-load[173]: Failed to lookup alias 'loop': Function not implemented
systemd-modules-load[173]: Failed to lookup alias 'vboxdrv': Function not implemented
systemd-modules-load[173]: Failed to lookup alias 'vboxpci': Function not implemented
systemd-modules-load[173]: Failed to lookup alias 'vboxnetadp': Function not implemented
systemd-modules-load[173]: Failed to lookup alias 'vboxnetflt': Function not implemented
{% endhighlight %}

And a little further down:
{% highlight bash %}
systemd[1]: Mounting /boot/efi...
mount[336]: mount: unknown filesystem type 'vfat'
systemd[1]: boot-efi.mount: Mount process exited, code=exited status=32
systemd[1]: Failed to mount /boot/efi.
systemd[1]: Dependency failed for Local File Systems.
systemd[1]: local-fs.target: Job local-fs.target/start failed with result 'dependency'.
systemd[1]: local-fs.target: Triggering OnFailure= dependencies.
systemd[1]: boot-efi.mount: Unit entered failed state.
{% endhighlight %}


So there was clearly something wrong with my kernel modules, causing them not to load. Fortunately I still had my [custom persistent arch iso live USB stick]({{ site.baseurl }}{% link _posts/2016-02-04-creating_custom_persistent_arch_live_iso.md %}). With this, the solution was quite easy:

I booted my laptop from the usb stick. Once in the live cd, I [setup the network with netctl](https://wiki.archlinux.org/index.php/netctl#Wireless_.28WPA-PSK.29). Once I had network, I mounted my the partition with my OS's root filesystem with ```mount /dev/sda8 /mnt``` and chrooted into it with ```arch-chroot /mnt /bin/bash```. Now I was back in my normal OS. All I had to do now was reinstall the kernel and modules. I did ```pacman -S --force linux linux-headers mkinitcpio kmod```. The ```--force``` in there makes sure everything gets reinstalled even if files from the package already exist in the filesystem. With me this was necessary since the same kernel and modules were already partially installed. It also re-built the initramfs and installed it. After this was done and a reboot, my os was up and running again. Phew! Thank heavens for the live USB stick.

The last thing I needed to do is check which other packages might need to be reinstalled. If you have yaourt installed, you can show your installed packages, sorted by installation date with ```yaourt -Q --date```. The packages at the bottom are the ones that were installed last. I just guessed from there which were the packages that might be corrupt, and reinstalled them the same way as the linux/linux-headers/etc pacakges. I also got an error when installing them about nvidia modules (I am running bumblebee and bbswitch was one of the likely incompletely installed packages) and therefore decided to reinstal all those packages (nvidia, nvidia-utils, bumblebee, opencl-nvidia, lib32-nvidia-utils, bbswitch) as well. After this all errors were gone.
