---
layout: post
title: Creating a custom persistent Arch Linux live cd for UEFI and BIOS systems
category: blog
date: "2016-02-04"
tags: 
    - ArchLinux
    - UEFI
    - live
    - remaster
    - iso
    - persistent
    - cow
---
I am running Arch Linux for a while now and am quite happy with it. Even though it is not the easiest n00b distro, in the end the documentation is so good that if you're new to Linux, you can probably find out how to do most stuff if you're inquisitive. One thing I had trouble with though, is getting a custom Live CD that could boot from my USB drive, has a persistence layer so changes are saved when booting, and booted both from UEFI only systems (my laptop) and preferably also BIOS only systems. It took me a while to figure this out, but I got it in the end. So I'll document it for posterity.

The three main sources of documentation that helped me setting this up is the [Arch Linux documentation on Remastering the Install ISO](https://wiki.archlinux.org/index.php/Remastering_the_Install_ISO), the [Boot Parameter documentation of ArchISO](https://github.com/djgera/archiso/blob/master/docs/README.bootparams) and [a blog post by Joshua Thijssen on creating partitioned disk images](https://adayinthelifeof.nl/2011/10/11/creating-partitioned-virtual-disk-images/).

The Arch documentation on remastering the iso tells you most you should know. Download the dual installation iso from the Arch Linux website and read through the docs on how to extract the airootfs.sfs, mount it, chroot to it and modify it. You can install packages after the step that says
{% highlight bash %}
mkinitcpio -p linux
{% endhighlight %}
in there and do other stuff to the Live OS. Then you exit the chroot again, recreate the airootfs.sfs image and copy the new kernel and initrd image to the right places. I didn't delete the squashfs-root folder though. I copied it to my homedir (outside the ~/customiso tree) so I can copy it back later if I need to instead of having to unsquashfs the airootfs.sfs file all the time. Also do the part with "modifying the EFI boot image". 

Up to there it was quite easy. The first problem I encountered was creating the ISO. Don't use the genisoimage command in the docs. It won't work for UEFI systems. The documentation below it on xorriso is better, if you combine it with the eltorito stuff lower down. In the end, this is the xorriso command I used to create the iso:
{% highlight bash %}
iso_label="ARCH_201601"
xorriso \
    -as mkisofs \
    -iso-level 3 \
    -full-iso9660-filenames \
    -volid "${iso_label}" \
    -appid "Arch Linux Live/Rescue CD" \
    -eltorito-boot isolinux/isolinux.bin \
    -eltorito-catalog isolinux/boot.cat \
    -no-emul-boot -boot-load-size 4 -boot-info-table \
    -isohybrid-mbr ~/customiso/isolinux/isohdpfx.bin \
    -eltorito-alt-boot \
    -e EFI/archiso/efiboot.img \
    -no-emul-boot \
    -isohybrid-gpt-basdat \
    -output ~/arch-custom.iso \
    ~/customiso
{% endhighlight %} 
Make sure the iso_label is correct. It needs to stay the same as the original iso file. If you run
{% highlight bash %}
file <original_iso>.iso
{% endhighlight %}
it will tell you the label in the output.

If you run isohybrid --uefi on the resulting iso file, you have a working, remastered disk image for both BIOS and UEFI systems.

Now for the persistent bit. It turns out that [archiso has boot parameters](https://github.com/djgera/archiso/blob/master/docs/README.bootparams) that allow you to modify the behavior of the live os. It took me a while to figure out how this works with mkinitcpio and stuff. But in essence there are initcpio hooks that get run on startup of the live cd if you specify them in the HOOKS part of /etc/mkinitcpio.conf. One of those hooks is archiso. That one sets-up the filesystem and accepts parameters for a persistence layer with a cow (copy-on-write) filesystem. Normally that cow filesystem is a tmpfs and gets deleted after shutdown. But if you specify other options, all starting with cow_, then you can change this behaviour and put the cow filesystem on a persistent medium.

What took me a while to figure out is that these options need to be passed from te bootloader at startup to the kernel for the initrd image. THat is all that is needed. So what I do at startup, when you get to the bootmenu of the livecd/usb, is specify cow_device=/dev/sdb3 at the end of the kernel line. You can edit the line by pressing "e" in the bootmenu. The archiso hook of initrd will check that device for a folder called "persistent_${iso_label}" and if it doesn't exist, create it. All changes you make to the cd will be stored there.

The last bit that I needed to figure out is how to make space on my USB drive for this persistent partition. My iso image is 1.2Gb and my USB drive is 8GB, so there is plenty of space for the persistent data. But if you just write the image to the usb drive with dd, the extra 6.8Gb isn't used. If I run gparted on the usb drive afterwards, it makes a mess of things. So I needed to modify the iso image before writing it to USB. That is where the [blogpost by Joshua](https://adayinthelifeof.nl/2011/10/11/creating-partitioned-virtual-disk-images/) came in. Basically I treated the iso file as a harddrive with some tricks. I first made it as big as my usb key, then created an extra partition in it, and formatted that partition with ext3. After that I wrote the iso file to the usb drive with dd the regular way, and lo-and-behold, there was a third partition on the usb drive with an ext3 filesystem (called /dev/sdb3 in my computer) that I could specify as cow_device in the boot menu at startup. This is how I managed it.


First calculate how much space the persistent partiton should have do this by with REMAINING=(usb_drive_size_in_Gb-iso_image_size_in_Gb)\*1024\*1024\*2. This will calculate the amount of blocks (each 512 bytes in size) to add to the iso. Make sure you leave some wriggle room for rounding errors and [difference between GiB and GB](https://en.wikipedia.org/wiki/Gibibyte). Instead of using the complete 7.1Gb of my usb drive, I made the image 7.0Gb. 
If you need to find out the actual size of your usb drive, use 
{% highlight bash %}
$ lsblk /dev/sdb
{% endhighlight %}
Once you know the size, grow the iso image by that size. Replace $REMAINING below by the calculated number of blocks above.
{% highlight bash %}
$ dd status=progress if=/dev/zero bs=512 count=$REMAINING >> arch-custom-hybrid.iso
{% endhighlight %}
After that you need to create a partition in the file. You do this with
{% highlight bash %}
$ fdisk arch-custom.iso
Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3): 
First sector (14608384-26142719, default 14608384): 
Last sector, +sectors or +size{K,M,G,T,P} (14608384-26142719, default 26142719): 

Created a new partition 3 of type 'Linux' and of size 5.5 GiB.

Command (m for help): p
Disk arch-custom.iso: 12.5 GiB, 13385072640 bytes, 26142720 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x78abe923

Device                  Boot    Start      End  Sectors  Size Id Type
arch-custom.iso1 *           0 14608383 14608384    7G  0 Empty
arch-custom.iso2           172      172        0    0B ef EFI (FAT-12/16/
arch-custom.iso3      14608384 26142719 11534336  5.5G 83 Linux

Command (m for help): w
The partition table has been altered.
Syncing disks.

{% endhighlight %}
For most of those questions in fdisk (the ones ending with a : without input) I just hit enter, choosing the default option.

At this point you're ready to create an ext3 (or other) filesystem in there. This is tricky as the file isn't an actual block device. The blogpost tells you how to do it. You create a loopback device that points to the partition inside the iso image. That loopback device is available as any harddrive and you can create a filesystem in it like you would do with a normal partition. But to create the loopback device, you need to tell it exactly where in the image file the partition starts and ends. THIS IS VERY IMPORTANT. If you do this wrong, you'll screw up the image file. So first you need to show exactly where the partition starts and ends. You can do this with
{% highlight bash %}
$ fdisk -l arch-custom.iso
Disk arch-custom.iso: 12.5 GiB, 13385072640 bytes, 26142720 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x78abe923

Device                  Boot    Start      End  Sectors  Size Id Type
arch-custom.iso1 *           0 14608383 14608384    7G  0 Empty
arch-custom.iso2           172      172        0    0B ef EFI (FAT-12/16/
arch-custom.iso3      14608384 26142719 11534336  5.5G 83 Linux
{% endhighlight %}
This tells you the Start and End of the arch-custom.iso3 partition.
By multiplying both these numbers with 512, you get resp the $OFFSET and $SIZELIMIT for the loopback device.
Setup the loopback device and format it with ext3 with
{% highlight bash %}
$ losetup -o $OFFSET --sizelimit $SIZELIMIT /dev/loop1 archiso-custom.iso
$ mkfs -t ext3 -L cow /dev/loop1
$ losetup -d /dev/loop1
{% endhighlight %}

At this point your image is ready (finally). You can now write it to the usb drive with (*WARNING!!!!! Make sure /dev/sdb is actually your usb drive. If it is any other drive YOU WILL ERASE IT COMPLETELY!!!*)
{% highlight bash %}
dd if=arch-custom.iso of=/dev/sdb
{% endhighlight %}

At this point you can reboot and start from the usb drive (you might need to do something to the UEFI/BIOS to make it start from USB). If you are in the archlinux live boot menu, pres "e" to edit the default menu, and add cow_device=/dev/sdb3 to make it use /dev/sdb3 as the persistent cow device. 
