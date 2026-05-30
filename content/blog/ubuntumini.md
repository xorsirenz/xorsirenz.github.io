---
title: 'Ubuntumini'
---
<article class="main">
    <h2 class="main-headers">11101011a&f- 'ᵇˡᵒᵍ</h2>
    <div class="main-content">

## UbuntuMini.iso Install Guide:

By default the Ubuntu minimal .iso will only boot with a legacy BIOS. So im going to show you how you can get a UbuntuMini.iso on a UEFI system so we can install a minimal Ubuntu setup for REMnux. For this to happen we must build our own ISO using the Ubuntu minimal ISO, and a Ubuntu Server ISO.<br> Through my research on this, I came across a script on <a target="_blank" href="https://noobient.com/2019/06/25/ubuntu-18-04-uefi-network-installer/">Noobient</a>

First lets create a temp directory for our work:<br>
`$ mkdir /tmp/remnux`

Next we will create a file for our script: (You can call this whatever you'd like, I personally just named it customiso.sh)<br>
`$ touch customiso.sh`

Lets copy the script into the customiso file, then make it executable. You can use whatever text editor you would like. I will be using vim in this example.<br>
`$ vim customiso.sh`

```sh
#!/bin/bash

set -eu

server_iso='ubuntu-20.04.1-legacy-server-amd64.iso'
mini_iso='mini.iso'
dist_dir='ubuntu-20.04-netinstall'

if [ ! -e ${server_iso} ]
then
    wget "https://cdimage.ubuntu.com/ubuntu-legacy-server/releases/focal/release/${server_iso}"
fi

if [ ! -e ${mini_iso} ]
then
    wget "http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/current/legacy-images/netboot/${mini_iso}"
fi

rm -rf ${dist_dir}*
7z x ${server_iso} -o${dist_dir}-tmp install/hwe-netboot/ubuntu-installer/amd64/linux
7z x ${server_iso} -o${dist_dir}-tmp install/hwe-netboot/ubuntu-installer/amd64/initrd.gz
7z x ${server_iso} -o${dist_dir} EFI
7z x ${mini_iso} -o${dist_dir}
mv ${dist_dir}-tmp/install/hwe-netboot/ubuntu-installer/amd64/linux ${dist_dir}/linux
mv ${dist_dir}-tmp/install/hwe-netboot/ubuntu-installer/amd64/initrd.gz ${dist_dir}/initrd.gz
zip -r ${dist_dir}.zip ${dist_dir}
```
`$ chmod +x customiso.sh`

Run the script:<br>
`$ ./customish.sh`

Now if you do a `ls` you will see three new files and a two new directories. The two important files to verify are `ubuntu-20.04.1-legacy-server-amd64.iso` `mini.iso` and a the new directory called `ubuntu-20.04-netinstall`

Time to verify the sha of your .iso files, to do this run the `sha256sum` command on both files:<br>
```sh
$ sha256sum mini.iso
0e79e00bf844929d40825b1f0e8634415cda195ba23bae0b041911fde4dfe018  mini.iso
$ sha256sum ubuntu-20.04.1-legacy-server-amd64.iso 
f11bda2f2caed8f420802b59f382c25160b114ccc665dbac9c5046e7fceaced2  ubuntu-20.04.1-legacy-server-amd64.iso
```

Make a custom ISO using the `mkisofs` command. If you do not have the command. You can download `cdrtools` through your package manager.
What I did was run<br>
`$ mkisofs -o ubuntu-20.04-netinstall.iso \ `

This should give you a prompt to run the rest of the commands:
```sh
> -b ubuntu-20.04-netinstall/isolinux.bin \
> -c ubuntu-20.04-netinstall/boot.cat \
> -no-emul-boot \
> -boot-load-size 4 \
> -boot-info-table -J -R -V \
> UbuntuMinimal .
```

After completing the previous step, you should now see another file in your directory called `ubuntu-20.04-netinstall.iso`

We now need to burn this iso onto our USB drive. Lets verify the name of our USB stick first with:
```sh
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 931.5G  0 disk 
├─sda1   8:1    0     1G  0 part /boot
├─sda2   8:2    0    30G  0 part /
└─sda3   8:3    0 900.5G  0 part /home
sdb      8:16   1  14.6G  0 disk
```

Be careful to make sure you recognize the proper block device name for your USB. In the picture above my USB is listed as `sdb` If you are unsure what the block device name of your USB stick is called, you can run the `lsblk` command without the USB inserted in your PC and compare the differences to see what NAME is added.<br>
Lets actually burn the ISO onto our USB now using the command `dd`.

<strong>WARNING: YOU CAN DELETE YOUR OS IF YOU TYPE IN THE INCORRECT BLOCK DEVICE NAME</strong>

`$ dd bs=4M if=ubuntu-20.04-netinstall.iso of=/dev/sdb status=progress`

Once this is completed it's now time to boot your USB stick. I am going to assume you are installing REMnux onto another device. So lets plug in the USB stick into a USB port on the device you'd like to install Ubuntu on.

<br><b>DISCLAIMER: I am going to assume that you have the knowledge about setting your boot order, and GRUB, so a few steps here will be skipped.</b>

When I personally plugged in my USB I was getting an error that my PC could not find a bootable device when my USB stick was set to #1 in my boot order. I seen that this was not an issue with other people trying to boot a minimal Ubuntu iso with UEFI. If you are having the same issue as me, and already have grub installed, go back into your BIOS and set GRUB as #1 in your boot order.

Once you see Grub pop up, depending on what is on your screen you will need to do a few steps differently
                       
Personally I previously had Arch installed on this device so I type in `c` to get to the grub terminal. If this is the case. We will need to type in a few commands to get the Ubuntu Installer to run. First lets list our devices.
`grub> ls`

Depending on your previous setup and partitions this could look a bit differnt. In my case my output of `ls` looked like this:<br>
`(hd0) (hd1) (hd1,gpt3) (hd1,gpt2) (hd1,gpt1) (hd2)`

If you are not positive on which device is your USB, once again you can unplug your usb device. Load up the grub terminal and perform a `ls` and see what device is missing. Pay close attention to the devices listed with `,gptX` in them as they can give you a hint. For example if I remove my USB and perform a `ls` my output would look like this:
`(hd0) (hd0,gpt3) (hd0,gpt2) (hd0,gpt1) (hd1)`

As you can see the `hdX` number changes based on the order its listed and not the actual device itself. I can see that when my USB stick in inserted it is listed as `(hd0)` since the `hdX,gptX)` devices change numbers, and the trailing (hdX) also changes numbers. So now lets boot into our Ubuntu install:
```sh
grub> set root=(hd0)
grub> linux /ubuntu-20.04-netinstall/linux
grub> initrd /ubuntu-20.04-netinstall/initrd.gz
grub> boot
```

This will bring up the Ubuntu Installer. To finish the remnux install I will direct you to the <a target="_blank" href="http://Remnux.org">Remnux.org</a> website as they already have a detailed writeup on the correct steps to take.<br>
<a target="_blank" href="https://docs.remnux.org/install-distro/install-from-scratch">Remnux Docs</a>
</p>

##### xorsirenz &copy; 2021
