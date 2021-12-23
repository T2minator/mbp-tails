# T2Tails

This is a detailed guide (sometimes with a ready-made ISO) for how to have [Tails](https://tails.boum.org) work on T2 Apple devices (e.g. 2019 16-inch MacBook Pro) without needing an external keyboard and mouse.

**T2Tails:** *Tails Fast, Tails Furious*

## Introduction

Although Tails is more or less regular Debian (specifically, the Debian Live GNOME disk image), its use case is completely different to the other distros with guides / ISOs in the [t2linux wiki](https://wiki.t2linux.org). Tails focuses on anonymity, privacy, and security - not necessarily perfection of hardware experience - and there are unique safety considerations for people who use Tails for the reasons it's created.

This particular guide / ISO doesn't tune *everything* in the T2 hardware to work (like the Wifi, which is [now possible](https://wiki.t2linux.org/guides/wifi/#on-linux)). Personally I don't care for the Touch Bar or the ambient light sensor. Currently I only am getting the internal keyboard and trackpad to work. However, the process is explained clearly enough for you to add the other T2 mods to get it everything persistently working in Tails.

This repo might also help someone trying to modify the kernel of any Debian Live ISO, or add drivers to it.

Over time I hope to make the method both more elegant and safer (see safety notes below), e.g. to adapt the [official Tails building instructions](https://tails.boum.org/contribute/build/) and insert the absolute minimal modifications to make a stock Tails ISO T2-compatible.

However, any existing or deprecated procedures held in this repo are very useful. Each time I move to a more elegant method, I will keep older versions easily visible in 'Archive' folder for reference on the Internet.

I'll do the best I can to advise best practices for both stability and safety, when considering the particular use case of Tails.

## Safety notes (for Tails users)

**This is not Tails:** The Tails project would probably prefer you to simply use other hardware instead of modifying the core Tails system (or wait for the Linux kernel to be updated directly for T2 compatibility and just use an external keyboard + mouse for now). However, they openly document the [procedure](https://tails.boum.org/contribute/build/) for building Tails yourself, and approvingly dub any customisation of such a process as ["a fork of Tails"](https://tails.boum.org/contribute/customize/). Therefore, this page is an openly documented example of that. This procedure will NOT result in you using 'Tails', but instead, a pretty close fork of it - T2Tails. Know that, and use at your own risk.

**Measure your risks:** I don't think this process introduces a serious problem for your anonymity when using Tails. For me, it brings a negligible decrease to my safety, due to my own experience and knowledge of the risks. If Tails already allowed for an Internet-connected program to know what your hardware was (e.g. a program that can legally see `APPLE SSD` as the hard-coded name of your internal NVMe hard disk), or to deeply scan your filesystem to read such deanonymising information, then you would have bigger problems, which would make some of the added risks of this procedure moot. This T2-modding will only reveal to such a layer that you've slightly modified your system to make Tails work *better* with the hardware that it already can read. Is that bad? You decide. It depends on the minuteae of your threat modelling.

**If you are paranoid:** An advanced risk I can identify is that an powerful global adversary might be able to link (your) prior Internet activity associated with following this procedure on this repo - e.g. the Tor IP addresses or other aspects of fingerprinting and advanced identification practised by online web services in your browser - together with subsequent local inspection of the slightly modified, read-only filesystem on your Tails stick (e.g. upon physical inspection of the disk), or, if it is possible, remote Internet-based detection of the kernel and driver modifications implemented here. If that is an issue for you, please do not follow this procedure. However, good OPSEC around how you obtain or build 'T2Tails' could make that risk almost negligible, and the benefits somehow outweigh the cons.

**Think about kernel safety:** AFAIK, none of the modding below affects the custom anonymising code, scripts, or design of Tails. However, the kernel patches in the instructions are not vetted by Tails. I don't know exactly how unsafe it is to use the T2 kernel patches created by aunali1 which are not (yet) accepted by the mainline Linux kernel developers. To me, the trade-off is worth it because Tails is not the only thing I do to keep myself safe, and it is better to use Tails than not. The full kernel replacement itself is quite safe because it is from upstream sources that Tails inherits third-hand. However, improvements should be made to this process to incorporate the *Debian-specific* Kernel, which e.g. disables certain upstream Kernel features that the Debian maintainers don't think are stable or safe enough for Debian users. This means that currently, the T2Tails kernel might not be as safe as using the unmodified Debian/Tails kernel.

**Firmware upgrading:** For safety and stability reasons, all things considered it is best to always update the firmware of your T2 Mac's hardware provided by Apple's latest macOS software updates. Apple regularly releases important, firmware-level security updates (e.g. for the Intel CPU microcode) contained in macOS system updates, which affect security on Linux as much as macOS. [mrmacintosh.com](https://mrmacintosh.com) is a great blog reporting if there's a firmware update in any given macOS system update. You should definitely keep your firmware updated as it generally will only improve stability and/or performance (at least for t2linux). E.g. I observed that updating to Big Sur firmware over Catalina significantly improved the waiting time after hitting Alt before the Apple logo appears. NOTE: merely booting from a self-created bootable Big Sur (or later) installer USB *alone* will automatically upgrade most of your Mac's firmware, even if you don't *install* the newer macOS thereafter. But fully installing Big Sur (whether on the internal SSD or to an external SSD while connected to the Mac) will do the fullest amount of internal hardware firmware upgrades on your T2 Mac with extra upgrades sometimes applied such as Bluetooth.

Install and use 'T2Tails' at your own risk. I hope to refine the process over time for maximum safety (in security, privacy, and anonymity).

## Instructions - v2.3

I'm not producing an ISO for more recent versions of Tails, so follow the steps to make an up-to-date T2Tails ISO yourself.

**Currently I am not providing a ready-made ISO. I'm busy and it's inconvenient to upload large files to Github.**

### You will need

This is just a suggestion.

I use three storage disks:

1. DISK1: A bootable Tails disk to be the building environment (Minimum size: whatever Tails recommends.)

2. DISK2: A large storage area in which to compile your customised kernel (Minimum size: 32GB, but 16GB might be OK.) (This could potentially be the same storage as DISK1 if it is large enough, e.g. create some Persistent Storage or a manual Ext4 partition in the free space on the Tails stick using GNOME `Disks`.)

3. DISK3: A third stick to burn 'T2Tails' on and boot from after the process finished.

### Tips before you start

- You will be doing extensive file operations on the storage media you use. USB sticks can be very slow and result in a bad experience when compiling the Linux kernel, due to moving hundreds of thousands of files at once (involved in the moving, copying, and extracting of squashfs filesystems, or extracted kernel source dirs). Use modern, fast storage technology and cables for DISK2. I recommended a fast external (or internal) SSD with minimum USB 3.0 connection. Correct and quality cables also make a big difference in performance of heavy file operations.

- If you want to compile the kernel on your T2 machine itself (in a distro like stock Tails), beware of kernel panics on some Linux ISOs (e.g. untouched Tails). E.g. to make stock Tails iBridge-stable, add to Tails the boot parameter `amdgpu.dpm=0` (and if still receive panics, others have said to try `intel_iommu=on` and `modprobe.blacklist=thunderbolt`).

- I will update the steps for each new Tails as it comes out (and may sometimes provide an ISO for convenience). The below process is for Tails 4.25.

### Other notes before you begin

- I am not at the elite skill level of a Debian or Linux kernel developer. Some of the commands I say to do or packages I say to install might not necessarily be needed, but at least it works. Create a GitHub issue if there's a meaningful improvement to contribute and we can make the steps more elegant.

- Where I instruct to do multi-line commands, just paste whole code box selection and perform at once in Terminal. Saves time.

### Steps

#### Create your building environment

First download the current stable Tails ISO or IMG, and place the file onto DISK2 inside any Linux-compatible filesystem such as exFAT or FAT32. (You may want to use the ISO in later steps.)

Then create DISK1 - a bootable Tails which you will build T2Tails in. You can create it from the standard Tails ISO or IMG. This might be done in macOS, however it has to burn to a non-read-only filesystem. The official Tails ISO/IMG burning process might create a `ISO9660` filesystem on the Tails disk, which can't be changed to be read/write (which is temporarily required for this procedure to work). First format DISK1 as FAT, then place the Tails stick files extracted from the ISO or IMG. One way to do it is to boot into a normal Tails stick created officially, then while in that Tails, copying contents from `/lib/live/mount/medium` onto DISK1, then DISK1 is bootable but editable Tails.

After creating your bootable DISK1, *do not* download/transfer/mount/read/write ANY other files for this procedure in macOS. Do *everything* below in Tails. (It needs to be Ext4 due to symlinks and other file permission issues in the kernel source files. Paragon extFS ruins Ext4 files in macOS for this process.)

#### Download, patch, and recompile the Linux kernel

You should now be in Tails (booted on DISK1).

If DISK2 is not Ext4 filesystem, reformat it as such. (It could encrypted e.g. VeraCrypt, or format unencrypted but with the Linux (Ext4) filesystem e.g. by using GNOME 'Disks'.)

If necessary, copy (or re-download) `tails-amd64-4.25.iso` to the temporary Home folder at `/home/amnesia`. Then after you mount and open the new Ext4 filesystem on DISK2, decide on a working directory for your creation process, and copy the Tails ISO `tails-amd64-4.25.iso` into your working directory.

Extract `tails-amd64-4.25.iso` (right-click in Nautilus and 'Extract Here'), making sure the ISO contents are directly contained in a `tails-amd64-4.25` dir directly below your working directory. Alternatively, copy the contents of your live Tails stick you're booting from i.e. copy from `/lib/live/mount/medium` into your extracted folder. 

Now, for your convenience, I suggest you change Terminal to root user, due to doing so many sudo commands for the rest of the process. (No permissions or security issues are observed by me.) You can also open `Root Terminal` in Tails, then `cd` to working directory. Otherwise, you'll have to enter admin password for all `sudo` commands.

```
sudo su
```

First - and this is needed since Sep 2021 for some reason (will probably be resolved in Tails 5.0) - remove the forcing of `sid` repo for `squashfs-tools` package so that we can install it without issue.

```
sudo sed -i 's/squashfs-tools/temp-replacement-string/g' /etc/apt/preferences
```

Now open Terminal and install the packages required for this procedure: 

```
sudo apt update && \
sudo apt install -t buster-backports libzstd1 && \
sudo apt install -y libncurses5-dev libncurses-dev libssl-dev flex bison build-essential libelf-dev bc gcc squashfs-tools xorriso zstd
```

*Note about the packages: `libncurses5-dev libncurses-dev libssl-dev flex bison build-essential libelf-dev bc gcc` needed for the compiling. `squashfs-tools` (and `libzstd1`) is for the step to mod the Tails filesystem (to install the patched kernel and T2 device drivers). `xorriso` needed to re-make our Tails ISO. `zstd` is because depending on what kernel version I instruct you to use for a given Tails version, it may be needed.*

Now `cd` to your primary working directory on DISK2.

Download the source code of the Linux kernel version that I suggest to use for the current Tails version. (Ordinarily I'd opt to choose the same or close to that chosen by Tails for the current version, but since Tails 4.20 I am choosing a newer kernel version as it resolves a major issue in Tor Connection Assistant otherwise not working.)

So currently for Tails 4.25, do this:

```
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.12.19.tar.gz
```

*Note: For now, compiling a pristine kernel source instead of the more precisely-matched Debian-specific kernel is what works better. (Last time I tested the difference was for Tails 4.19 - For some reason, the latter resulted in a less than ideal / generally crappy / non-working experience inside Tails - things like no Internet in Tails 4.18, various GNOME settings hanging, other things.) I hope to investigate because there are many reasons why, for advanced safety reasons, one should default to the *Debian-specific* Linux kernel, and not the pre-Debian one at kernel.org. These instructions should ideally evolve.*

Now extract the archive in your working folder.

In Terminal, `cd` to the extracted (and thus far untouched) kernel source folder (and make sure it is the actual folder kernel source folder with `arch`, `block`, etc directly underneath it, it should be one level below your working folder):

```
cd linux-5.12.19
```

Now download and extract the Linux kernel patches for T2 hardware from [aunali1](https://github.com/aunali1), e.g. do:

```
wget -O ../T2patch.tar.gz https://github.com/aunali1/linux-mbp-arch/archive/refs/tags/v5.12.19-2.tar.gz && \
tar -xvf ../T2patch.tar.gz -C ../
```

Now apply aunali1's T2 patches to the kernel source, e.g.:

```
for i in ../linux-mbp-arch-5.12.19-2/*.patch; do patch -p1 --verbose < $i; done
```

(Wait for Terminal to return to the prompt before continuing. It may take a minute to do all the patching.)

*We've just patched the kernel source code. Time to compile it.*

Copy the default Tails kernel configuration into your patched kernel source (we should match Tails as much as possible):

```
cp /boot/config-$(uname -r) ./.config
```

Do this to prevent a common error preventing kernel compilation from completing:

```
scripts/config --set-str SYSTEM_TRUSTED_KEYS ""
```

Do this to disable compiling useless debugging info along with your kernel. It's not included in Tails by default (AFAIK), and saves a huge deal of time and storage space during compilation (e.g. 30-40 GB). A large ~1GB `-dbg` DEB file will be produced if you don't do this, and it's not needed at all:

```
scripts/config --disable DEBUG_INFO
```

Now do the command to start compiling the patched Linux kernel, with full CPU usage (best to leave the computer alone while compiling):

```
make deb-pkg -j`nproc`
```

In the Terminal output, you might now have a few (or several, around 30) kernel configuration choices to make for the compiling of the kernel. When doing something so hardcore as compiling the Linux kernel, there are many choices you can make as to whether to enable certain features, or to enable support for certain hardware. We've already copied 99% of those choices from the existing Tails default config, but there may be some choices left for the recompiling. Currently, these are my choices ('no' for almost all), and the resultant T2Tails works: **unprivileged user namespace support: 'n' (in earlier procedures if 'y' I got a compile error, but with 5.12.x I think 'y' is acceptable), sensors 'n', NVIDIA stuff yes, Anonymous Shared Memory Subsystem 'n', android 'n', NTFS ones 'n', and everything else 'n'**. Because advanced anonymity (and security) considerations are on my mind, I hope to investigate why extra choices are popping up despite copying the Tails existing config, and what best to do to prevent your T2Tails from "sticking out" from the rest (pun unintended), if that matters. Safety is the bigger concern for most Tails users, not performance.

*Side note: if you want to potentially lessen the anonymity inside your Tails system (at the deep/advanced fingerprinting level), but increase your privacy and security, you could go ahead and disable a plethora of kernel features and hardware support in the kernel that you don't need. E.g. the command `make localmodconfig` will then make it compile to support ONLY your current specific hardware connected inside or to your machine (and therefore for that to work you would compile on the T2 Machine itself). Research needs to be done as to whether this would meaningfully harm your anonymity. Tails users already use different hardware to each other, and therefore may already have 'fingerprints' of what their potentially gleanable list of hardware is (which is both a privacy *and* anonymity concern). Is it already possible for JavaScript-executing websites, Internet-connected programs, or unpatched exploits (i.e. malware) in Tails to fingerprint and undermine you by reading the hardware information of your physical Tails hardware? If so, locking down your Tails kernel (by way of recompiling and disabling support for 99% of hardware which you don't use, e.g. a malicious USB stick of a random chipset used by an evil maid) might only bring a net benefit. (Fingerprinting in anonymity is a complex art. The best choice for you depends on your threat model, or the balance of multiple threat models, that you're looking to defend against. I'll try to advise the best practice here. DON'T PANIC, and ALWAYS remember where you towel is.)*

Compiling the kernel will take quite a while depending on how powerful your CPU is (and how you configure it before compiling). For the above default steps, it could be 25 minutes, 45 minutes, 2 hours, or more.

Among the files it spits out, you will have 3 kernel DEB files ready to install.

#### Install the modded kernel + drivers in a 'chroot' representing what will become your 'T2Tails' filesystem

*Explanation time: Debian Live (on which Tails is based) compresses its Linux filesystem (containing the folders like /home, /usr, /lib, etc.) into a single file called `filesystem.squashfs` on the actual disk. It loads and decompresses a *copy* of that file into your computer's live RAM for the current session (while leaving the original template filesystem file in your stick untouched, so that it is never modified by default). That's how the magic of Tails amnesia works!*

*We can copy this file from an existing Tails disk, decompress it, edit the files inside (to e.g. install a T2-modded kernel and T2 drivers inside its file structure), recompress the files into a replacement `filesystem.squashfs`, then replace the original file on the Tails stick with that. Neat!*

Now `cd` back up to your primary working directory, one level directly above the extracted `tails-amd64-4.25` folder:

```
cd ..
```

Decompress the extracted Tails squashfs file into a folder called `squashfs-root` inside your working folder:

```
sudo unsquashfs -d squashfs-root tails-amd64-4.25/live/filesystem.squashfs
```

Now copy your three kernel DEB files into the `squashfs-root` folder (which needs root or sudo):

```
sudo cp linux-*.deb squashfs-root
```

Download the latest T2 device driver files:

```
sudo -u amnesia git clone https://github.com/t2linux/apple-bce-drv apple-bce-drv
```

Create and edit a `dkms.conf` file in the downloaded folder and place the below contents into it. One way is:

```
nano apple-bce-drv/dkms.conf
```

Then paste the following contents into it:

```
PACKAGE_NAME="apple-bce"
PACKAGE_VERSION="drv"
MAKE[0]="make KVERSION=$kernelver"
CLEAN="make clean"
BUILT_MODULE_NAME[0]="apple-bce"
DEST_MODULE_LOCATION[0]="/kernel/drivers/misc"
AUTOINSTALL="yes"
```

Then do Ctrl-O, ENTER, Ctrl-X, and the file is saved. (Instruction based on https://wiki.t2linux.org/guides/dkms)

Now copy that folder into the `squashfs-root` location:

```
sudo cp -R apple-bce-drv squashfs-root/usr/src
```

*Explanation time: To modify the extracted Tails filesystem in a smooth and successful way, we're going to use `chroot`. Chroot is a way to treat any folder as if it is the root dir of its own virtual Linux operating system (and it changes you to be the `root` user of it). It enables you to do commands like `apt install something` and when you do, it only installs it in the file and folder structure inside this temporary 'chroot'. If you have enough normal Linux system files inside the folder which happen to make up the Linux file system (which you now have at `squashfs-root`), you can do powerful commands to completely update the folder structure as if it's a real live Debian OS. (It's also a useful security feature for app isolation, and a method to repair broken Linux systems whose boot partitions have gone haywire, where you can just mount the broken system's disk as an external mount in a second OS and do commands to repair it in the Terminal.) It's very cool!*

*Instead of copying large amounts of required system files into the chroot for modifying what we need to do inside it, we can bind-mount certain folders from your current live Tails system into it. (It's like temporary symlinking.)*

Do the following so we can smoothly update the chroot's kernel:

```
sudo mount -o remount,rw /lib/live/mount/medium && \
sudo mkdir -p squashfs-root/lib/live/mount/medium && \
sudo mount --bind /lib/live/mount/medium squashfs-root/lib/live/mount/medium
```

Now backup the current build environment's kernel files so that we don't unnecessarily destroy them during the in-chroot kernel replacement process:

```
sudo mv /lib/live/mount/medium/live/initrd.img /lib/live/mount/medium/live/initrd.img.bak && \
sudo mv /lib/live/mount/medium/live/vmlinuz /lib/live/mount/medium/live/vmlinuz.bak
```

Now do the following to bind various system folders to the chroot, as a clean way to make kernel modding and driver installation easy and smooth inside it:

```
sudo mount --bind /tmp squashfs-root/tmp && \
sudo mount --bind /dev squashfs-root/dev && \
sudo mount --bind /sys squashfs-root/sys && \
sudo mount --bind /proc squashfs-root/proc && \
sudo mount --bind /run squashfs-root/run && \
sudo mount --bind /dev/pts squashfs-root/dev/pts && \
sudo mount --bind /etc/apt squashfs-root/etc/apt
```

Now chroot into the folder:

```
sudo chroot squashfs-root
```

Mandatory step: In as low a voice as possible, say, "I am Chroot." Every time. (It gives you good luck!)

Now install the three kernel files in the chroot with these three commands (you are now root user inside this chroot):

```
dpkg -i linux-libc-*.deb && \
dpkg -i linux-headers-*.deb && \
dpkg -i linux-image-*.deb
```

(Wait for it to return to prompt. It may take a minute.)

This creates two installed kernel files `initrd*.img` and `vmlinuz*` and other necessary files in the main chroot filesystem that work together to enable automatic T2 driver loading at boot.

Now install `dkms` (the package to install the T2 driver) in the chroot:

```
apt update && apt install -y dkms
```

Install the T2 driver in the chroot, linked to our newly added T2Tails-modded kernel:

```
dkms install -m apple-bce -v drv -k 5.12.19
```

Now do this command to makes the internal (T2) keyboard + trackpad automatically work at boot:

```
echo "apple-bce" > /etc/modules-load.d/t2.conf
```

Now is where you may want to add some of your own personal chroot modifications (e.g. install other packages) that you may deem critical (and if you don't deem doing so a compromise to the OPSEC of your anonymity). E.g. some packages, like any driver-related ones, are not easy to make take effect unless they are installed before a Tails session boots.

Finally, clean up unnecessary junk that we added in the chroot:

```
apt clean && rm *.deb && rm -rf /var/cache/apt/* /var/cache/man/* /var/lib/apt/lists/* /tmp/* /home/amnesia/.bash_history
```

(Note: ideally one should investigate the full extent of exactly what should be cleaned up, so that the modded Tails filesystem is as close to *as little* extra files/packages added as possible. (This includes not having a second kernel version folder under `/lib/modules`.) I'll improve this, and please help via a ticket if you know ways to improve it yourself.)

Now exit the chroot:

```
exit
```

Unmount the live kernel folder bind in the chroot, as well as the other 6 dirs we mounted into the chroot (ignore the warnings on the third command, the asterisk wildcard is just a hacky shortcut to make us do less input):

```
sudo umount squashfs-root/lib/live/mount/medium && \
sudo umount squashfs-root/etc/apt && \
sudo umount squashfs-root/dev/pts && \
sudo umount squashfs-root/*
```

More cleanup (I hope to make cleanup more comprehensive later):

```
sudo rmdir squashfs-root/lib/live/mount/medium
```

Now delete the original Tails kernel and filesystem files from the extracted Tails contents:

```
sudo rm tails-amd64-4.25/live/filesystem.squashfs tails-amd64-4.25/live/initrd.img tails-amd64-4.25/live/vmlinuz
```

Then move the two T2-modded kernel files into the extracted contents:

```
sudo mv /lib/live/mount/medium/live/initrd.img /lib/live/mount/medium/live/vmlinuz tails-amd64-4.25/live
```

Restore your current build environment's original kernel files, since they got modified during the chroot's kernel package installation: 

```
sudo mv /lib/live/mount/medium/live/initrd.img.bak /lib/live/mount/medium/live/initrd.img && \
sudo mv /lib/live/mount/medium/live/vmlinuz.bak /lib/live/mount/medium/live/vmlinuz
```

Then, as a safety recommendation, return your current mounted live Tails stick to be read-only state again:

```
sudo mount -o remount,ro /lib/live/mount/medium
```

Now compress your modded Tails filesystem, outputting the file into the extracted contents:

```
sudo mksquashfs squashfs-root tails-amd64-4.25/live/filesystem.squashfs -b 1024k -comp xz -Xbcj x86 -e boot
```

(It could take about 20 minutes or more, depending on your system. Why so long: This command has to pack about 150,000 files into one single file without a single mistake - that's a lot of files to process, even on modern storage - no mean feat!)

One step below may be required on your T2 model for iBridge panics to not occur in Tails: You must add the kernel boot parameter `amdgpu.dpm=0`. To make it convenient, add it permanently to the Tails's grub config.

Do:

```
sudo nano tails-amd64-4.25/EFI/debian/grub.cfg
```

Scroll down to find the `live` grub entry parameters list, and add the text `amdgpu.dpm=0` anywhere among that list of parameters (with a space before and after it). Then save changes to file. You may want to add or remove other kernel parameters while you're here, depending on what peripherals, features, or tweaks you want to persistently take advantage of. I found that `modprobe.blacklist=thunderbolt` makes some USB devices not be detected as often, probably a clue is that `lspci` shows `Thunderbolt 3 USB controller`.

*(For most users, this parameter addition shouldn't have a deanonymising effect. At the local/physical level, you're already observed to use Tails with your specific hardware. But do consider the possibility for malicious apps in Tails to to read your `/proc/cmdline` file as an advanced deanonymising technique. Obscurity (e.g. using unusual hardware or especially locked down configurations) can be good for privacy, but is an enemy of anonymity.)*

One more thing: since recent versions of Tails, there's a bug preventing Tails from booting on some hardware including T2 Mac hardware. The bug is not fixed yet. Workaround is to copy an older version of `BOOTX64.EFI` (the 1.3 MB version) into your T2Tails stick files then it boots. I am not sure is there is a security drawback of doing that, but I think it's unlikely.

Ways to get the older `BOOTX64.EFI` file:

- Download an older version of Tails (e.g. 4.18) from linuxtracker.org
- If you know git well, probably can build an older version of Tails ISO using their complicated build process.
- Find any Debian live ISO from Debian archives that is older than mid-2020, which contains a `bootx64.efi` file of the 1.3 MB size. Capitalise to `BOOTX64.EFI` if need be, it works.

Then Overwrite the 900X KB file at `tails-amd64-4.25/EFI/BOOT/BOOTX64.EFI` with the older 1.3 MB one.

Now optionally, create a T2Tails ISO from your extracted and T2-modded Tails stick contents:

```
xorriso -as mkisofs \
   -r -V 'T2Tails 4.25 v2.3' \
   -o T2Tails-4.25-v2.3.iso \
   -b isolinux/isolinux.bin \
   -c isolinux/boot.cat \
   -boot-load-size 4 -boot-info-table -no-emul-boot \
   -eltorito-alt-boot \
   -e EFI/BOOT/BOOTX64.EFI \
   -no-emul-boot -isohybrid-gpt-basdat -isohybrid-apm-hfsplus \
   tails-amd64-4.25
```

The T2Tails ISO will be created in your working folder.

You can make a successful bootable T2Tails stick in two ways:

- Burn the ISO to DISK3 with `Tails Installer` in Tails (and not e.g. Etcher).

- Format DISK3 as a big FAT partition then just copy over everything in the pre-ISO `tails-amd64-4.25` folder onto the FAT stick, or extract files from T2Tails ISO then place them on the stick.

Congratulations, you now have **T2Tails** bootable on your T2 Mac! The internal keyboard + trackpad will automatically work.

## The T2Tails experience

Worthy tips:

- To get Internet and Tor working (I use Ethernet), sometimes I need to do some or all of the following: 1. Connect Ethernet (via USB-C adapter) after Tails has loaded past the Welcome screen 2. Do `lspci` 3. Go to `Network` GNOME settings window (and if need be, disconnect 'Apple Ethernet'), then select the alternative Ethernet NIC to connect. Then Tor in Tails is able to connect as normal.

## CHANGELOG notes

Since v2.3 instructions:

- Added step to make sure the working Tails env has read-write-possible filesystem for the procedure to work.

Since v2.2 instructions:

- Fix for Tails not booting. Sounds like a good idea.

Since v2.1 instructions:

- iBridge kernel panics are no longer a problem. Solved by booting with `amdgpu.dpm=0` parameter. Thank you [@Redecorating](https://github.com/Redecorating) for the tip!
- Using a newer kernel than Tails makes the new Tor Connection Assistant finally work. That is what was delaying me releasing a method for Tails 4.20 and 4.21. Glad I solved that.

## TODO (important)

(These TODO are either for elegance and/or to increase safety, including advanced anonymity considerations)

- Chroot cleanup: we should uninstall dkms package? And linux-headers-* package? Determine in a giant chroot filesystem diff comparison to an untouched Tails `filesystem.squashfs`. And/or installed apt package list comparison.
- T2Tails removes /boot (it shouldn't, to emulate original Tails properly). Investigate and fix.
- Chroot cleanup: Do we need to retain `usr/src/apple-bce*` dir (once you've installed the driver)? I don't think so. Test.
- Investigate kernel configuration choices left over for the user. Explore `make oldconfig`. Test compiling an exactly matched debian-specific kernel with Tails with just Debian-specific patches and *without* aunali1 patches. What is ONLY asked about in that case? Compare to when aunali1 patch is ALSO added to that. If there are some configs only asked about due to adding aunali1 patches, consider decision based on that. If something's not needed for the T2 hardware and Tails doesn't have an already-chosen configuration option for it, the default should be 'n' to disable it.
- `/boot` in the chroot. Creates extra files, decide what to clean up (e.g. original kernel version now replaced anyway), inspect config differences, eventually try to make it the same exact kernel (in code and in name, as if installed from debian apt) anyway, etc. AKA make it so that `uname -a` / `uname -r` is identical to the stock Tails (for each release). Find a working process via Debian snapshot repositories?
- Also `squashfs-root/lib/modules/5.10.0-0.bpo.3-amd64/updates/dkms` indicates more work is needed to fully replace / merge the modded kernel into the stock Tails one. Let's not remove any Tails functionality, only add it as surgically cleanly as possible.
- Clean up: find a way to only have one linux-headers in the system (or none at all once you have T2Tails). Emulate untouched Tails.
- Clean up: remove the old kernel modules and packages in chroot.
- Examine what changes are made inside filesystem.squashfs and minimise them as much as possible.
- Current method: it's a bit hacky how I have the `initrd.img` and `vmlinuz` replacement method (by hooking into the current Live Tails' live system). `/usr/sbin/update-initramfs` in the chroot indicates possible ways to make it more elegant. But it's complicated in how many mounts I have from live system dirs, so for now, keep it 'hacky' because it 'just works'.
- Current method: Check official Debian Live and Tails docs/build scripts for the ideal squashfs compression parameters, if different.
- Eventually, change method to build Tails ISO from scratch using official instructions (but including the necessary kernel and driver modifications and insertions for T2) to produce an ISO. Potentially make everything automatic script, for producing the T2Tails ISO.

## Useful reference

- https://debian-handbook.info/browse/stable/sect.kernel-compilation.html
- https://wiki.t2linux.org

## Acknowledgements

- Everyone involved with t2linux.org - Thank you very much!
- [aunali1](https://github.com/aunali1)
- [Redecorating](https://github.com/Redecorating)
- Me. (Let's face it...) ("Many brain cells died to bring us this information...") (WORTH IT!)
