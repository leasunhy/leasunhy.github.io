---
title: How to Know if a Newly Bought USB Drive is Really New
date: 2015-07-15 22:46:46
tags:
  - Linux
---

You bought a new USB drive recently, but how do you know it **is** new, i.e. not used by any one after manufactured? Maybe someone just formats an old one, put it (that looks new) into the box and sells it to you, and in no way can you tell it from a new one by its appearance.

No, actually, we have a way :-)

<!-- more -->

## How?! ##
Let me show you :-)
Since I'm using `ArchLinux` (the best GNU/Linux distribution! joke ;) , I will show you the Linux way. If your computer is a Mac, these steps should work too.


### Obtain The Device Name ###
First, we need to know the device name that the kernel has given the drive.
Fire up your favorite terminal, and:

Run `lsblk -f`.
The output should look like this:

```nohighlight
NAME    FSTYPE  LABEL         UUID                                 MOUNTPOINT
sda                                                                
├─sda1  vfat                  7A5B-03A1                            /boot
├─sda2                                                             
[.... more about sda]
sdb                                                                
└─sdb1  vfat                  DD62-64BE 
```
So we have two block storage devices, `/dev/sda` and `/dev/sdb`.
Since `/dev/sda` has numerous partitions and the partitions are `ntfs`, `hfsplus`, `ext` filesystems, `/dev/sda` should be our hard drive.

Ok, then `/dev/sdb` is what we are looking for.

## Look for ascii strings in the device ##

Enter the following command in your terminal and pray for a while :-), and then hit enter. Press `Ctrl+C` after about 10 seconds to terminate (if not, it will scan the whole disk).
Remenber to change `/dev/sdX` to your real device name.

`sudo tail -c +512 /dev/sdX | strings -a`

Here, the option `-c +512` passed to `tail` is to skip the first 512 bytes of the drive, which is the Master Boot Record.

My output on a newly bought USB drive (yes I write this post just to share my experience:-):

```nohighlight
3.31.0C 5B 2014-12-31
06/06/15
05:00:09
1.2.84.0
.\MP2324615-027.A01LF2.ini
mkfs.fat
NO NAME    FAT32   
This is not a bootable disk.  Please insert a bootable floppy and
press any key to try again ... 
RRaA
rrAa
mkfs.fat
NO NAME    FAT32   
This is not a bootable disk.  Please insert a bootable floppy and
press any key to try again ... 
```

Oops, some strings are found in my drive :-(
Some of them seem to be things in PBR, but I have only one partition, so where do they come from? I don't know...
The strangest and most suspectable string is `.\MP2324615-027.A01LF2.ini`. It seems like a file name? I have never ever seen such a file before, not to mention stroring it in the new USB drive...

Ok, so after you bought a new USB drive, do you know what to do now? :-) 

## How does it work? ##
In fact, normally, when we *format* a partition, we just erase some metadata (depends on what filesystem you use) so that the os treats it as a new partition and write data to it. In other words, **the original data will not be lost** until new data come in and override them.

Therefore, in most cases, by simply inspecting what data are in the *new* USB drive, we can know whether the drive has been used by others!


## Conclusion ##
So, following the steps above, you may find your new drive not so new...

But don't mind...
After all, it can still be used for exchanging files, can't it? ;-)

