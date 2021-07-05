# T2Tails

This is a detailed guide (not downloadable ISO) for how to get [Tails](https://tails.boum.org) fully working on T2 Apple devices (e.g. 2019 16-inch MacBook Pro) without needing an external keyboard and mouse.

**T2Tails:** *Tails Fast Tails Furious*

---

## Introduction

Although Tails is basically regular Debian (specifically, the Live GNOME ISO), its use case is completely different to the other distros with guides and ISOs in the [t2linux wiki](https://wiki.t2linux.org). Tails focuses on anonymity, privacy, and security - not necessarily hardware nirvana - and there are unique safety considerations for people who use Tails for the reasons it's created.

As such, I will not prioritise making a ready-made 'T2Tails' ISO, and this guide doesn't include steps to get *everything* in the T2 hardware working (like the Wifi, which is now [actually possible](https://wiki.t2linux.org/guides/wifi/#on-linux)).

Personally I don't care for the Touch Bar or the ambient light sensor. I only currently provide steps to get the keyboard and trackpad working. However, this procedure is explained clearly enough for you to add the other T2 mods to get them all working your T2 machine. I've done the hard bit already.

Also, this guide could help people out there trying to modify the kernel of a Debian Live disk (or install new drivers to it). The current instructions are a little hacky but they work, and the procedure is quite efficient too (compared to making an ISO).

Nonetheless, over time I will aim to make the method both more elegant and safer (see safety notes below), e.g. to adapt the [official Tails building instructions](https://tails.boum.org/contribute/build/) and insert the absolute minimal additional steps to make the stock Tails ISO T2 compatible, where you produce your own T2Tails ISO/IMG.

However, the current instructions are very useful. (It took a long time to trawl the Internet and put the steps together, along with endless errors to troubleshoot through and finally make it smooth.) So if I move to a more elegant method later, I will keep this current guide in the repo for reference.

I do indicate that I have plenty more to learn myself, but I'll do the best I can to advise best practice for both stability and safety (considering the particular use case of Tails).

## Safety notes (for Tails users)

**This is not Tails:** The Tails project probably prefers you to just use other hardware instead of modifying the core Tails code. However, they do openly document the [specific process](https://tails.boum.org/contribute/build/) for building Tails yourself, and approvingly deem the customisation of such a process as ["a fork of Tails"](https://tails.boum.org/contribute/customize/). Therefore, this page is an openly documented example of that. This procedure will NOT result in you using 'Tails', but instead a kind of fork of Tails - T2Tails. UNDERSTAND that, and use at your own risk.

**Measure your risks:** I don't think this procedure adds a serious problem for your anonymity when using Tails. For me, it brings a negligible change to my safety due to my own experienced knowledge of the risks. If or where Tails already allows for an Internet-connected program to know what your hardware is (e.g. the "APPLE SSD" hard-coded name of your internal hard disk) - e.g. if it can deeply scan your filesystem to read such things - then you have bigger problems regardless of whether you follow this procedure. All this hack would reveal is that you've slightly modified your system to make Tails work better with the hardware already revealed. Is that bad? You decide. It may depend on your situation.

**If you are paranoid:** One advanced risk I *can* identify is that an advanced persistent threat might be able to link (your) prior Internet activity associated with following this procedure (e.g. the Tor IP addresses or other aspects of fingerprinting as collected online by Internet services) together with subsequent local inspections of your slightly modified, read-only filesystem on the Tails stick (e.g. physical inspection of the disk), or, if it is possible, remote Internet-based detection of the kernel and driver modifications implemented here. If that is an issue for you, please do not follow this procedure. However, good OpSec (about how you obtain or build T2Tails online) could make that risk almost negligible.

**Think about kernel safety:** None of the modding below affects the core anonymisation code in Tails. However, the kernel patches in the instructions are not vetted by Tails. Be knowledgeable of that. I don't know precisely how unsafe it is to use these T2 kernel patches created by aunali1 which are not (yet) accepted by the mainline Linux kernel developers. To me, the trade-off is worth it because Tails is not the only thing that I do to keep me safe. The full kernel replacement itself is safe because it is from official sources which is a codebase Tails uses and trusts in the first place.

**Firmware upgrade trade-offs:** It's a good idea to update the firmware of your T2 Mac's hardware because Apple often issues important security updates (e.g. for Intel microcode) that affect security on Linux as much as macOS. HOWEVER, there is a bad *hardware bug* (not just software byg) with Apple's T2 chip - the T2 kernel crashes. Updating to the latest firmware provided by Apple (in Big Sur) seems a mixed bag, it can both reduce and [increase](https://piunikaweb.com/2021/05/06/macos-big-sur-crashing-kernel-panic-issues-persist-after-11-3-1-update) kernel panic frequencies. If you are not already updated to recent Big Sur firmware, carefully decide whether to introduce new potential kernel panic problems. BEWARE, successfully booting from a self-created bootable Big Sur installer USB will itself upgrade most of your firmware, even if you don't install Big Sur after that. But fully installing Big Sur (whether on the internal SSD or to an external SSD while connected to the Mac) will do the fullest amount of internal hardware firmware upgrades in your T2 Mac, with extra upgrades observed such as Bluetooth. One silver lining: after Big Sur firmware updates, the waiting time after hitting Alt before the Apple logo is *significantly* less. Maybe it's worth it, once you have your patched Tails. See notes down the bottom for more tips on living with the T2 kernel panics.

Install and use 'T2Tails' at your own risk. NOT suggested for users who are new to anonymity OpSec or Linux newbies. Nonetheless, I will aim to refine the procedure over time for maximum safety (security, privacy, and anonymity).

## Instructions

### You will need

Three USB sticks:

1. A base Tails USB to be your T2Tails building environment (minimum size: whatever Tails recommends)

2. A stick to be a large storage disk to build the customised kernel on (minimum size: 32GB, but 16GB might be OK, have not tested.)

3. A third stick that you will turn into 'T2Tails' to boot from after process finished.

### Tips before you start

- Use a FAST USB drive for USB2 (i.e. one that reviews measure a high speed test). Copying or moving tens of thousands of Linux kernel source files takes a long time if it's not a very fast USB 3.0 drive. [Find one](https://www.rightisbest.com/fastest-usb-3-0-flash-drive-on-amazon-2018.html).

- Due to kernel panics on the T2 hardware, it may be better to create your T2Tails stick on a non-T2 device such as an older Mac or another computer that boots Tails.

- Do not attempt this process unless you are already comfortable with Linux and Terminal. This is not for newbies. E.g. if you don't know what the instruction `cd` already means, either don't do this or be willing to search things online to learn as you go. I try to make the steps clear enough to follow, but you could encounter errors that affect your own attempt to follow the steps. That can be time-consuming to troubleshoot and overcome.

- I will update the steps for each new Tails as it comes out. The below process is for Tails 4.19rc1.

### Other notes before you begin

- I am not at the knowledge level of a Linux kernel or Debian developer. Some of the commands I say to do or packages I say to install might not necessarily be needed, but at least it works. It was so difficult to get this working in the first place. Create a GitHub issue if there's a meaningful improvement to contribute and we can trim down the steps to make it more elegant.

### Steps

#### Create your building environment

First, burn current stable Tails version onto your USB3 (which will be your build environment Tails stick). (This can be done from within macOS.)

Then, burn current stable Tails version onto your USB3 (which will become your 'T2Tails' stick). Don't clone it from the base Tails (USB1). Burn it from the original ISO/IMG downloaded from Tails website.

From this point on, *do not* download/transfer/edit/mount/read any working files in macOS. Do *everything* only in Tails. (It needs to be ext4 due to symlinks inside the kernel source, and other file permission issues. Paragon extFS ruins ext4 files in macOS for this process. I tried.)

Create an ext4 filesystem on USB2 with at least 64GB of space. (This could potentially be the same USB as USB1 if it is large enough, e.g. create Persistent Storage). One way is to create a VeraCrypt volume from within Tails (using the downloaded VeraCrypt binary), or format it unencrypted but with Linux (ext4) filesystem (e.g. via Tails 'Disks' program).

#### Download, patch, and recompile the Linux kernel version used by Tails

You are now in Tails. Open Terminal and make sure prompt is at the default `~/`.

Install packages required for this procedure: 

```
sudo apt update
sudo apt install -y libncurses5-dev libncurses-dev libssl-dev flex bison build-essential libelf-dev bc gcc squashfs-tools
```

(Then Ctrl-C to return to prompt.)

*Note about the packages: `libncurses5-dev libncurses-dev libssl-dev flex bison build-essential libelf-dev bc gcc` needed for the compiling. `squashfs-tools` is for the step to mod the Tails filesystem (to install the patched kernel and T2 device drivers).*

Now download the source code of the Linux kernel version chosen by Tails for the current version.

Currently for Tails 4.19rc1, do this:

```
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.10.24.tar.gz
```

*Note: For now, compiling a pristine kernel source instead of an exactly-matched Debian-specific kernel works best. (For some reason, the latter resulted in a less than ideal, crappy, or even non-working experience inside Tails - things such as no Internet back in Tails 4.18, GNOME settings hanging, and more.) I will investigate because there are many reasons why, for advanced safety reasons, we should default to *exactly* the *Debian-specific* Linux kernel and not the non-Debian once at kernel.org. These instructions will evolve.*

Then move that file from `~/` to your USB2 working folder, and extract it.

In Terminal, `cd` to your extracted (untouched) kernel source folder. (Make sure it is the actual folder kernel source folder with `arch`, `block`, etc directly underneath it.)

Now download the Linux kernel patches for T2 hardware by [aunali1](https://github.com/aunali1). We should use aunali1's closest matching [version](https://github.com/aunali1/linux-mbp-arch/releases/) to the current Tails kernel version (gathered by `uname -a`) but which is also a higher version than it (not lower). For Tails 4.19rc1 let's choose `v5.10.37-1`. Download and extract that, e.g. do:

```
wget -O ../T2patch.tar.gz https://github.com/aunali1/linux-mbp-arch/archive/refs/tags/v5.10.37-1.tar.gz
tar -xvf ../T2patch.tar.gz -C ../
```

Now apply aunali1's T2 patches to the kernel source, e.g.:

```
for i in ../linux-mbp-arch-5.10.37-1/*.patch; do patch -p1 --verbose < $i; done
```

(Wait for Terminal to return to the prompt before continuing. It may take a minute to do all the patching.)

*We just patched the kernel. Time to compile it.*

Copy the default Tails kernel configuration into your patched kernel source (we should match Tails as much as possible):

```
cp /boot/config-$(uname -r) ./.config
```

Do this to prevent a common error in kernel compilation:

```
scripts/config --set-str SYSTEM_TRUSTED_KEYS ""
``` 

Do this to disable compiling useless debugging info with the kernel. It's not included in Tails by default (AFAIK) and saves a huge deal of time and storage space during compilation (e.g. 30-40 GB). A large ~1GB `-dbg` DEB file will be produced if you don't do this, and it's not needed at all:

```
scripts/config --disable DEBUG_INFO
```

Now do the command to start compiling the patched Linux kernel with full CPU usage (leave the computer alone while compiling):

```
make deb-pkg -j`nproc`
```

You might now have a few final kernel configuration choices to make for the compiling of the kernel. When compiling a Linux kernel, there are a million choices you can make for whether to enable certain features or to enable support for certain hardware. We've already copied 99% of these choices from the existing Tails default config, but inevitably, there are a few left since we're patching / recompiling. Currently, these are my choices, and the resultant T2Tails works: **unprivileged user namespace support: n (if 'y' I got a compile error), sensors n, NVIDIA stuff yes, Anonymous Shared Memory Subsystem n, android n, ntfs ones no**. Because advanced anonymity (and security) considerations are on my mind, I will investigate why extra choices are popping up despite copying the Tails existing config, and what best to do to reduce your T2Tails from "sticking out" (pun unintended) from the rest, if and where that matters. Safety is of paramount importance for all Tails users.

*Side note: if you wanted to potentially lessen the anonymity of the Tails system (at a deep/advanced fingerprinting level), but increase your privacy and security, you could go ahead and disable a plethora of features and hardware support in the kernel which you don't need. E.g. the command `localmodconfig` will make it compile to support ONLY your current specific hardware connected inside or to your machine (therefore for that to work you would compile on the T2 Machine itself). Research needs to be done as to whether this would meaningfully reduce your anonymity. E.g. Tails users already use different hardware to each other, therefore may have different 'fingerprints' of what their potentially readable list of hardware is (which is both a privacy *and* anonymity concern). Can a JavaScript-executing website, an Internet-connected program, or an unpatched exploit (malware) in Tails *already* fingerprint or deanonymise you, by reading the hardware information of your Tails system already? If so, locking down your Tails kernel (by way of recompiling and disabling support for hardware you don't use, e.g. a malicious USB stick of random chipset by an attacker) might only be a net positive, on the balance. (Fingerprinting in anonymity is a nightmare. The best choice for you depends on your threat model or the balance of multiple threat models that you're looking to defend against. I'll try to advise the best practice here. DON'T PANIC, and ALWAYS remember where you towel is.)*

Compiling the kernel will take quite a while, depending on how powerful your CPU is. Could be 25 minutes, 45 minutes, 2 hours, or more.

Among the files it spits out, you will have 3 DEB files, ready to install.

#### Install the modded kernel + drivers in a 'chroot' representing what will become your 'T2Tails' filesystem

*Explanation time: Debian Live (on which Tails is based) compresses its Linux filesystem (containing folders like /home, /usr, /lib, etc.) into a single file called `filesystem.squashfs` on the live disk. It loads and decompresses a *copy* of that file into your computer's live RAM for the current session (leaving the original template filesystem file untouched, so that it is never modifiable by default). That's how the magic of Tails amnesia works.*

*We can copy this file from an existing Tails disk, decompress it, edit the files inside (e.g. install a T2-modded kernel and T2 drivers within its file structure), recompress them to a new `filesystem.squashfs`, then replace the original file on the Tails stick with that. Neat!*

Insert your USB3 (with Tails already burned to it).

Open `Disks` program and mount that USB's `Partition 1: Tails` FAT32 partition by clicking the play button under that partition in the program. It will mount to `/media/amnesia/Tails`.

From there it is easy to modify any of the core Tails files (if not currently booting from the stick itself), and yet not destroy its ability to boot thereafter. (If this is a scary discovery to you, then this is why read-only media - at the *physical* level - is recommended for increased/maximum Tails security.)

Change your working directory to the Amnesia home folder:

```
cd ~/
```

Decompress the base Tails stick's filesystem into a folder called `squashfs-root` in amnesia Home:

```
sudo unsquashfs -d /home/amnesia/squashfs-root /lib/live/mount/medium/live/filesystem.squashfs
```

Copy your three kernel DEB files into the `squashfs-root` folder (it needs root permission), e.g. do the pattern `sudo cp '/path/to/my/external/USB/working/folder/linux-libc-dev_5.10.13-1_amd64.deb' squashfs-root` (or use a Root Terminal-launched Nautilus).

Download the latest T2 device driver files:

```
git clone https://github.com/t2linux/apple-bce-drv /home/amnesia/apple-bce-drv
```

Create and edit a `dkms.conf` file in the downloaded folder and place the below contents into it. One way is:

```
nano apple-bce-drv/dkms.conf
```

Then paste the following contents into it, then do Ctrl-O, ENTER, Ctrl-X, and the file is saved.

```
PACKAGE_NAME="apple-bce"
PACKAGE_VERSION="drv"
MAKE[0]="make KVERSION=$kernelver"
CLEAN="make clean"
BUILT_MODULE_NAME[0]="apple-bce"
DEST_MODULE_LOCATION[0]="/kernel/drivers/misc"
AUTOINSTALL="yes"
```

(Instruction is based on https://wiki.t2linux.org/guides/dkms)

Now copy that folder into the `squashfs-root` location:

```
sudo cp -R apple-bce-drv squashfs-root/usr/src
```

*Explanation time: To modify the extracted Tails filesystem in a smooth and working way, we're going to use `chroot`. Chroot is a way to treat any folder as if it is the root dir of its own virtual Linux operating system (and it changes you to be the `root` user of the chroot). It enables you to suddenly do commands like `apt install something` and when you do, it only does it in the file and folder structure inside this temporary 'chroot'. If you have enough normal Linux system files inside the folder which happen to make up the Linux file system (which you now have at `/home/amnesia/squashfs-root`), you can do powerful commands to completely update the folder structure as if it's a real live Debian OS. It's very cool! (It's also a useful security feature for app isolation, and a method to repair broken Linux systems whose boot partitions have gone haywire, where you can just mount the broken system's disk as an external mount in a second OS and do commands to repair it in the Terminal.)*

*Instead of copying large amounts of necessary system files into the chroot for what we need to do, we can bind-mount certain folders from your current live Tails system into it. (It's like temporary symlinking.)*

Do these three commands to make it possible to update the chroot's kernel:

```
sudo mount -o remount,rw /lib/live/mount/medium
sudo mkdir squashfs-root/lib/live/mount/medium
sudo mount --bind /lib/live/mount/medium squashfs-root/lib/live/mount/medium
```

Backup the current USB1's kernel files so that we don't unnecessarily destroy them during the chroot's kernel replacement process:

```
sudo mv /lib/live/mount/medium/live/initrd.img /lib/live/mount/medium/live/initrd.img.bak
sudo mv /lib/live/mount/medium/live/vmlinuz /lib/live/mount/medium/live/vmlinuz.bak
```

Now do the following six commands to bind various system folders to the chroot, to make kernel modding and driver installation completely smooth inside it:

```
sudo mount --bind /dev squashfs-root/dev
sudo mount --bind /sys squashfs-root/sys
sudo mount --bind /proc squashfs-root/proc
sudo mount --bind /run squashfs-root/run
sudo mount -t devpts none squashfs-root/dev/pts
sudo mount --bind /etc/apt squashfs-root/etc/apt
```

Now chroot into the folder:

```
sudo chroot squashfs-root
```

Mandatory step: In as low a voice as possible, say "I am Chroot." Always. Every time.

Do this to help make the commands in chroot perform smoothly:

```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Now install the three kernel files in the chroot with these three commands (you are now root user inside this chroot):

```
dpkg -i linux-libc-*.deb
dpkg -i linux-headers-*.deb
dpkg -i linux-image-*.deb
```

(Wait for it to return to prompt. May take a minute.)

This creates two installed kernel files `initrd.img` and `vmlinuz` and other necessary files in the main chroot filesystem that work together to enable automatic T2 driver loading at boot.

Install `dkms` in the chroot (the package to install the T2 driver):

```
apt update
apt install -y dkms
```

(Then Ctrl-C to return to prompt.)

Install the T2 driver in the chroot, and in a way that will work with our newly added T2Tails-modded kernel:

```
dkms install -m apple-bce -v drv -k 5.10.24
```

Now do this command to make the T2 internal keyboard + trackpad automatically work at boot:

```
echo "apple-bce" >> /etc/modules-load.d/t2.conf
```

Finally clean up unnecessary junk that we added in the chroot:

```
apt clean
rm *.deb
rm -rf /var/cache/apt/*
rm -rf /var/cache/man/*
rm -rf /var/lib/apt/lists/*
rm -rf /tmp/* ~/.bash_history
```

(Note: I have to investigate a lot more regarding the full extent of what should be cleaned up so that the replacement Tails filesystem is as close to nothing added as humanly possible. (This includes not having a second / two kernel version folders in `lib/modules`.) I'll improve this and please help via a ticket if you know ways to improve it already.)

Now exit the chroot:

```
exit
```

Unmount the live kernel folder bind in the chroot:

```
sudo umount squashfs-root/lib/live/mount/medium
```

Also unmount the other 6 dirs we mounted:

```
sudo umount squashfs-root/etc/apt
sudo umount squashfs-root/dev/pts
sudo umount squashfs-root/run
sudo umount squashfs-root/proc
sudo umount squashfs-root/sys
sudo umount squashfs-root/dev
```

*(Note to self: These six lines are exhausting to input (both times). If not scripted, can we at least combine them those a one-liner? What can I say? "I'll be back.")*

More cleanup (I will make this more comprehensive later):

```
sudo rmdir squashfs-root/lib/live/mount/medium
```

Delete the original Tails kernel and filesystem files on USB3 (which we will replace with T2-modded ones):

```
rm /media/amnesia/Tails/live/filesystem.squashfs
rm /media/amnesia/Tails/live/initrd.img
rm /media/amnesia/Tails/live/vmlinuz
```
Move your two new T2-modded kernel files to the USB3:

```
sudo mv /lib/live/mount/medium/live/initrd.img /lib/live/mount/medium/live/vmlinuz /media/amnesia/Tails/live
```

(Ignore the ownership permission error. It works.)

Restore USB1's original kernel files because they got modified during the chroot's kernel install: 

```
sudo mv /lib/live/mount/medium/live/initrd.img.bak /lib/live/mount/medium/live/initrd.img
sudo mv /lib/live/mount/medium/live/vmlinuz.bak /lib/live/mount/medium/live/vmlinuz
```

Then, as a safety recommendation, do this to make your current live Tails stick be read-only again:

```
sudo mount -o remount,ro /lib/live/mount/medium
```

Now, compress your modded Tails filesystem and write this file to the USB3:

```
sudo mksquashfs squashfs-root /media/amnesia/Tails/live/filesystem.squashfs -b 1024k -comp xz -Xbcj x86 -e boot
```

(It might take about 20 minutes or more, depending on your system.)

One final step is needed to make Tor successfully connect in Tails 4.19 onwards. You must add the kernel boot parameter `modprobe.blacklist=thunderbolt`. To make it convenient for you, add it permanently to the Tails stick's grub config. 

*(For most users, this shouldn't have a deanonymising effect. You already are known to use Tails with your specific hardware at the local physical level. But consider the possibility for rogue apps to to read your `/proc/cmdline` file as an advanced deanonymising technique.)*

Do:

```
nano /media/amnesia/Tails/EFI/debian/grub.cfg
```

Then: add `modprobe.blacklist=thunderbolt` to the list of parameters (with a space before and after) in the (default) `live` entry parameters list, and save changes to file.

(TODO: make it a `sed` find/replace one-liner.)

TA-DA!! Done. Boot your new **T2Tails** (the USB3) on your T2 Mac and the internal keyboard + trackpad will automatically work.

## The T2Tails experience

Observations and tips:

- To get Internet and Tor working (I use Ethernet), currently I: 1. Connect Ethernet (via USB-C adapter) after Tails has loaded past the Welcome screen 2. Do `lspci` 3. Go to `Network` GNOME settings and (if need be disconnect 'Apple Ethernet' then) select `Realtek` to connect. Then the Tor Connection assistant will pop up and you can connect you to Tor.

- Kernel panics may be an ongoing problem. I will experiment and update here with any tricks that mitigate Apple's hardware-caused kernel panics.
	- So far, the main tip: have as little USB-C peripherals connected as possible at all times.
	- I'm currently on the latest Big Sur-installed firmware, because I think it's more important to receive firmware updates than not (even though it may actually introduce extra kernel panics). Instead I'm exploring other kernel panic mitigations listed here.
	- Adding the kernel boot parameter `modprobe.blacklist=thunderbolt` to Tails might help for this issue. (Because connecting and disconnecting things on the ports seems to be a trigger and crash errors say "thunderbolt".)
	- I am experimenting by physically disconnecting internal hardware in the MacBookPro16,1 to see if it helps eliminate or reduce kernel panics (due to having as little T2-connected hardware as possible). Following iFixit's [excellent guide](https://www.ifixit.com/Guide/MacBook+Pro+16-Inch+2019+Battery+Replacement/135185), I have so far disconnected the internal cables for: lid angle sensor and Touch Bar 'digitizer' (Steps 32-34); Touch Bar display (Steps 35, 36); speakers (Steps 44-49); microphone array (Steps 59, 60); and wireless antennae (Steps 61, 62). Observation: no effect on reduction of kernel crashing in Tails whatsoever (at least not before permanently introducing `modprobe.blacklist=thunderbolt`). They remain disconnected at present.

## TODO (important)

(These are either for elegance and/or to increase safety, including advanced anonymity considerations)

- Chroot cleanup: we should uninstall dkms package? And linux-headers-* package? Determine in a giant chroot filesystem diff comparison to an untouched Tails `filesystem.squashfs`.
- Chroot cleanup: Do we need to retain `usr/src/apple-bce*` dir (once you've installed the driver)? I don't think so. Test.
- Investigate kernel configuration choices left over for the user. Explore `make oldconfig`. Test compiling an exactly matched debian-specific kernel with Tails with just Debian-specific patches and *without* aunali1 patches. What is ONLY asked about in that case? Compare to when aunali1 patch is ALSO added to that. If there are some configs only asked about due to adding aunali1 patches, consider decision based on that. If something's not needed for the T2 hardware and Tails doesn't have an already-chosen configuration option for it, the default should be 'n' to disable it.
- `/boot` in the chroot. Creates extra files, decide what to clean up (e.g. original kernel version now replaced anyway), inspect config differences, eventually try to make it the same exact kernel (in code and in name, as if installed from debian apt) anyway, etc. AKA make it so that `uname -a` / `uname -r` is identical to the stock Tails (for each release). Find a working process via Debian snapshot repositories?
- Also `/home/amnesia/squashfs-root/lib/modules/5.10.0-0.bpo.3-amd64/updates/dkms` indicates more work is needed to fully replace / merge the modded kernel into the stock Tails one. Let's not remove any Tails functionality, only add it as surgically cleanly as possible.
- Clean up: find a way to only have one linux-headers in the system (or none at all once you have T2Tails).
- Examine what changes are made inside filesystem.squashfs and minimise them as much as humanly possible.
- Current method: it's a bit hacky how I have the `initrd.img` and `vmlinuz` replacement method (by hooking into the current Live Tails' live system). `/usr/sbin/update-initramfs` in the chroot indicates possible ways to make it more elegant. But it's complicated in how many mounts I have from live system dirs, so for now, keep it 'hacky' because it 'just works'.
- Current method: Check official Debian Live and Tails docs/build scripts for the ideal squashfs compression parameters, if different.
- Eventually, change method to build Tails ISO from scratch using official instructions (but including the necessary kernel and driver modifications and insertions for T2) to produce an ISO. Potentially make everything automatic script, and and/or release a T2Tails ISO auto-built by the script.

## TODO (research)

- Is `config` file in aunali1's patches repo superior (for the T2 user experience) in any way? Must balance that with Tails anonymity considerations, with the latter having precedence. Do a diff comparison of Tails config vs. aunali1 file.

## Useful reference

- https://debian-handbook.info/browse/stable/sect.kernel-compilation.html
- https://wiki.t2linux.org
- Will add more later

## Acknowledgements

- Everyone involved with t2linux.org - Thank you very much!
- The amazing aunali1 in particular
- Me (LOL.) (Let's face it.) ("Many brain cells died to bring us this information...")
