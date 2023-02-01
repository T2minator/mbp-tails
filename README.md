# T2Tails

This is a detailed guide (first release had a ready-made ISO, still downloadable as a demo) for how to have [Tails](https://tails.boum.org) work on T2 Apple devices (e.g. 2019 16-inch MacBook Pro) without needing an external keyboard and mouse.

**T2Tails:** *Tails Fast, Tails Furious*

## Introduction

Although Tails is more or less regular Debian (specifically, the Debian Live GNOME disk image), its use case is completely different to the other distros with guides / ISOs in the [t2linux wiki](https://wiki.t2linux.org). Tails focuses on anonymity, privacy, and security - not necessarily perfection of hardware experience - and there are unique safety considerations for people who use Tails for the reasons that Tails exists.

This particular guide doesn't show how to get *everything* in the T2 hardware to work (like the Wifi, which is [possible](https://wiki.t2linux.org/guides/wifi/#on-linux)). Personally, I don't care for the Touch Bar or the ambient light sensor. Currently I only am getting the internal keyboard and trackpad to work. However, the process is explained clearly enough for you to add any other T2 mods to get it persistently working in Tails.

This repo might also help someone trying to modify the kernel of any Debian Live ISO, or add drivers to it.

Eventually I aim to make this method emulate an untouched Tails environment as much as possible (see safety notes below), e.g. to adapt the [official Tails building instructions](https://tails.boum.org/contribute/build/) and insert the absolute minimal modifications to make a stock Tails ISO T2-compatible.

In the meantime, any existing or deprecated procedures held in this repo are very useful. Whenever I revise the instructions, I will keep older versions easily visible in the 'Archive' folder for public reference.

I'll do the best I can to advise best practices for both stability and safety, when considering the particular use case of Tails.

## Safety notes (for Tails users)

**This is not Tails:** The Tails project would probably prefer you to simply use other hardware instead of modifying the core Tails system (or wait for the Linux kernel to be updated directly for T2 compatibility, and just use an external keyboard + mouse for now). However, they openly document the [procedure](https://tails.boum.org/contribute/build/) for building Tails yourself, and dub any customisation of such a process ["a fork of Tails"](https://tails.boum.org/contribute/customize/). Therefore, this page is an openly documented example of that. This procedure will NOT result in you using 'Tails', but instead, a very close fork of it - T2Tails!

**Measure your risks:** I don't think this procedure introduces a significant risk for your anonymity when using Tails. For me, it brings a negligible decrease to my safety, due to my experience and knowledge of various risks. If Tails already allowed for an Internet-connected program to know what your hardware was (e.g. a program that can legally see `APPLE SSD` as the hard-coded name of your internal NVMe hard disk), or to deeply scan your filesystem to read such deanonymising information, then you would have bigger problems anyway, which would make some of the *extra* risks introduced by this procedure moot. Your T2-modding will only reveal in such a situation that you've slightly modified your system to make Tails work *better* with the hardware that it *already* can detect. Is that bad? You decide. It depends on the nature of your threat model.

**If you are paranoid:** An advanced risk I can identify is that an powerful global adversary might be able to link (your) prior Internet activity associated with following this repo's procedure - e.g. the Tor IP addresses or other elements of fingerprinting and advanced identification practised by online web services in your browser - together with subsequent local inspection of the slightly modified, read-only filesystem on your Tails stick (e.g. upon physical inspection of the disk), or, if it is possible, remote Internet-based detection of the kernel and driver modifications implemented here. If that is an issue for you, please do not follow this procedure. However, good OPSEC applied to how you obtain or build 'T2Tails' could make such a risk almost negligible, and/or the benefits somehow outweigh the cons.

**Think about kernel safety:** AFAIK, none of the modding below affects Tails' custom anonymisation code, scripts, design, etc. However, the kernel patches in these instructions are not officially vetted by Tails. I don't know exactly how unsafe it is to use the T2 kernel patches, which are not (yet) accepted by the mainline Linux kernel developers. To me, the trade-off is worth it because Tails is not the only thing I do to keep myself safe, and it is better to use Tails than not to. The full kernel replacement itself is quite safe because it is from upstream sources which Tails distributes third-hand. However, improvements ideally need to be made to this procedure to incorporate the *Debian-specific* Kernel, which e.g. disables some upstream Kernel features that the Debian maintainers don't think are safe or stable enough for Debian users. This means that currently, the T2Tails kernel could be less safe than the unmodified Debian/Tails kernel, for more than one reason.

**Firmware upgrading:** For safety and stability reasons, all things considered it is best to always update the firmware of your T2 Mac's hardware provided by Apple's latest macOS software updates. Apple regularly releases important firmware-level security fixes (e.g. for the Intel CPU microcode) through macOS system updates, which can affect security within Linux on your T2 Mac. [mrmacintosh.com](https://mrmacintosh.com) is a great blog reporting if there's a firmware update in any given macOS system update. If it is desirable within your threat model, you should definitely keep your firmware updated because it will generally only improve stability and/or performance. E.g. I observed that updating to Big Sur firmware over Catalina significantly improved the waiting time after hitting Alt before the Apple logo appears. NOTE: merely booting from a self-created bootable Big Sur (or later) installer USB *alone* will automatically upgrade most of your Mac's firmware, even if you don't *install* the newer macOS thereafter. But fully installing macOS (whether on the internal SSD or to an external SSD while connected to the Mac) will do the fullest amount of internal hardware firmware upgrades on your T2 Mac, with extra upgrades sometimes applied such as Bluetooth.

**Note: For Tails to boot, you now must update your machine's firmware to at least macOS Monterey. Otherwise, try the `BOOTX64.EFI` hack detailed in earlier (archived) versions of this README.**

Install and use 'T2Tails' at your own risk. I hope to refine the process over time to minimise any safety compromises added as a result of the process (in terms of security, privacy, and anonymity).

## Instructions - v2.4

I no longer produce a ready-made ISO. Instead, follow these steps to create a T2Tails yourself.

### You will need

This is merely a suggestion.

I use three storage disks:

1. DISK 1: A bootable Tails disk to be your building environment (Minimum size: whatever Tails recommends)

2. DISK 2: A large mass storage disk on which to compile your customised kernel (Minimum size: 32 GB, though 16 GB might be OK.) (This could potentially be the same storage as DISK 1 if it is large enough, e.g. create some Persistent Storage or a manual Ext4 partition on the free space on the Tails stick using GNOME `Disks`.)

3. DISK 3: A third disk to burn 'T2Tails' on, and boot from once it's created.

### Tips before you start

- You will be doing long, extensive file operations on the storage media you use. USB storage can be very slow and result in a poor experience when compiling the Linux kernel because of moving hundreds of thousands of files (involved in the moving, copying, and extracting of squashfs filesystems, or extracted kernel source folders). Use modern and fast storage technology and cables for DISK 2. I recommended a fast external (or internal) SSD with at least USB 3.0+ connection speed. Quality / correctly specced cabling also makes a difference in performance of such heavy file operations.

- If you want to compile the kernel on your T2 machine itself (in a distro like stock Tails), beware of iBridge kernel panics on some Linux ISOs (including untouched Tails). To make untouched Tails run stably, right after you boot into Tails do this in Tails Terminal: `echo high | tee /sys/bus/pci/drivers/amdgpu/0000:??:??.?/power_dpm_force_performance_level`.

- Although Tails releases updates every month, these steps don't need to change very often. The below iteration is for Tails 5.9.

### Other notes before you begin

- NEW: I'll investigate next time, but you could now potentially use some precompiled Debian kernel DEB installers at https://github.com/andersfugmann/T2-Debian-Kernel instead of having to patch, configure and compile the kernel yourself. However, you may suffer a minor OPSEC disadvantage since it's not using Tails' kernel config (with potential lockdowns Tails has added for safety reasons), so I may continue instructing on how to compile nonetheless. (Note for self: next time, try compiling the Tails-selected and Debian-specific kernel once more, it may work this time.)

- I don't have the elite knowledge of a Debian or Linux kernel developer. Some of the commands I instruct to do, or packages I say to install, might not necessarily be needed, and the instructions may be a little messy (leaving unneeded files copied or left over in the resulting T2Tails), but at least it works. Create a GitHub issue if there's a meaningful improvement you can contribute.

- Where I instruct to do multi-line commands, paste the whole code box selection into Terminal to execute it all at once.

### Steps

#### Create your building environment

First download a standalone ISO or IMG of the current Tails. (It cannot be an existing Tails that is upgraded - it must have only one .squashfs file in the disk structure which you will then copy and modify.)

Then create DISK 1 from your downloaded Tails image - a bootable Tails which you will also build T2Tails in. This can be done in macOS, however it has to burn to a non-read-only filesystem. The official Tails ISO/IMG burning process might create a `ISO9660` filesystem on the Tails disk, which can't be changed to be read/write (which is currently required for this procedure to work). First format DISK 1 as FAT, then place the Tails files extracted from the ISO or IMG. One way to do it is to boot into a normal Tails stick created officially, then while in that Tails, copying contents from `/lib/live/mount/medium` onto DISK 1. Then DISK 1 is a bootable and *also* editable Tails.

After creating your bootable DISK 1, *do not* download/transfer/mount/read/write ANY other files for this procedure in macOS. Do *everything* below in Tails. (It needs to be Ext4 due to symlinks and other file permission issues in the kernel source files. Paragon extFS ruins Ext4 files in macOS for this process.)

#### Download, patch, and recompile the Linux kernel

You should now be in Tails (booted inside DISK 1).

If DISK 2 is not Ext4 filesystem, reformat it as such. (It could be encrypted e.g. VeraCrypt or LUKS, or you could format it unencrypted but with the Linux (Ext4) filesystem e.g. by using GNOME 'Disks'.)

After you mount and open the new Ext4 filesystem on DISK 2, decide on a working directory for your creation process.

Then, open a new Terminal window at your working directory (e.g. `cd` to it).

For convenience to not enter your sudo password a million times from this point on, I suggest you change to root user. (I don't know of any resulting permissions or security problems from doing this.)

```
sudo su
```

Now make a copy of the contents of your live Tails stick you're booting from into your working folder and into a subdir, like so: 

```
mkdir -p tails-amd64-5.9 && \
cp -R /lib/live/mount/medium/* tails-amd64-5.9
```

Now install the packages required for the entirety of this procedure (some packages may already be pre-installed in Tails but they are listed here for reference):

```
sudo apt update && \
sudo apt install -t sid -y pahole dwarves libc-bin binutils binutils-common libbinutils binutils-x86-64-linux-gnu libzstd1 libctf0 libgprofng0 libctf-nobfd0 libjansson4 libbpf1 && \
sudo DEBIAN_FRONTEND=noninteractive apt install -y squashfs-tools libzstd1 libncurses5-dev libncurses-dev libssl-dev flex bison build-essential libelf-dev bc gcc zstd fakeroot kmod devscripts cpio ccache lz4 qtbase5-dev schedtool jfsutils reiserfsprogs xfsprogs btrfs-progs pcmciautils xz-utils bindgen
```

*Note about the packages: Almost all are typically needed for seamless kernel compiling. (For Tails 5.9 I've had a hell of a time trying to make sure I have all necessary packages, more than may really be necessary have been added to cover all possible bases. It's overkill but what the hell. I've even added dependencies needing a manual `sid` specification for whatever reason too. Over time it could change slightly in the Debian repos, you'll have to adapt, just make sure you have all the above packages installed, one way or another.) `squashfs-tools` (and `libzstd1`) is for the step to mod the Tails filesystem (to install the patched kernel and T2 device drivers).*

Now `exit` out of root user (as I haven't yet worked out how to do the below patch line as root user), to successfully execute the following script - modified by me, adapted from https://wiki.t2linux.org/guides/kernel/ - to download the kernel, patch it, and inject the T2 related drivers into the kernel source files:

```
sudo -u amnesia git clone --depth=1 https://github.com/t2linux/linux-t2-patches $PWD/patches
rm ./patches/1001*

pkgver=$(sudo -u amnesia torsocks curl -sL https://github.com/t2linux/T2-Ubuntu-Kernel/releases/latest/ | grep "<title>Release" | awk -F " " '{print $2}' | cut -d "v" -f 2 | cut -d "-" -f 1)
_srcname=linux-${pkgver}
sudo -u amnesia wget -c https://www.kernel.org/pub/linux/kernel/v${pkgver//.*}.x/linux-${pkgver}.tar.xz
tar xf $_srcname.tar.xz
cd $_srcname

sudo -u amnesia git clone --depth=1 https://github.com/kekrby/apple-bce $PWD/drivers/staging/apple-bce

for patch in ../patches/*.patch; do
    patch -Np1 < $patch
done
```

*Note: I've found that compiling a pristine upstream kernel source instead of the more precisely-matched Debian-specific kernel is what works better. (Last time I tested this was in Tails 4.19 - For some reason, the latter resulted in a less than ideal, generally crappy, and non-working experience in Tails - e.g. no Internet in Tails 4.18, various GNOME settings hangs, other things.) I hope to investigate because there are many reasons why, for advanced safety reasons, one should default to the *Debian-specific* Linux kernel, and not the pre-Debian one at kernel.org.*

Double check that in Terminal you are in the kernel source folder (the folder with `arch`, `block`, etc. directly underneath it).

Then for convenience, go back to being super user:

```
sudo su
```

Now copy the default Tails kernel configuration into your patched kernel source (to match Tails as much as possible):

```
cp /boot/config-$(uname -r) ./.config
```

Now do the following kernel configuring to eliminate certain compiling errors and make it smoother:

```
scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYS ""
scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYRING ""
scripts/config --set-str CONFIG_MODULE_SIG_KEY ""
scripts/config --set-val CONFIG_DEBUG_INFO n
scripts/config --set-val CONFIG_DEBUG_INFO_BTF n
scripts/config --set-val CONFIG_DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT n
scripts/config --set-val CONFIG_DEBUG_INFO_BTF_MODULES n
scripts/config --set-val CONFIG_VIDEO_CX18 n
scripts/config --set-val CONFIG_VIDEO_CX18_ALSA n
scripts/config --module CONFIG_BT_HCIBCM4377
scripts/config --module CONFIG_HID_APPLE_IBRIDGE
scripts/config --module CONFIG_HID_APPLE_TOUCHBAR
scripts/config --module CONFIG_HID_APPLE_MAGIC_BACKLIGHT
scripts/config --module CONFIG_APPLE_BCE
```

*We have now patched and mostly configured the kernel source code. Time to compile it!*

Do the following command to start compiling your patched Linux kernel, with full CPU usage (best to not use the system while compiling):

```
make deb-pkg -j`nproc`
```

In the Terminal output, you may now have some remaining config choices to make before it starts compiling - several of them (like 20 or 30). For this iteration of the procedure (tested with kernel 6.1.8), just answer 'n' to everything brought up. For both convenience and security, you should answer 'n' because Tails doesn't have such features enabled in its own similarly-versioned config anyway.

*Side note: if you want to increase your privacy and security from one angle, even at the risk of potentially lessening the anonymity inside your Tails system (at a deep/advanced fingerprinting level), you could go ahead and disable many kernel features and hardware support in the kernel that you don't need. An extreme way of doing this is the command `make localmodconfig` which will make it compile to support ONLY your current specific hardware connected inside or to your machine (which means you must compile it on the T2 Machine itself). Research needs to be done as to whether this would meaningfully harm your anonymity. Tails users already use different hardware to each other, and therefore may already have 'fingerprints' of what their potentially gleanable list of hardware is (which is both a privacy *and* anonymity concern). Is it already possible for JavaScript-executing websites, Internet-connected applications, or unpatched exploits (in malware) in Tails to fingerprint and undermine you by reading the hardware information of your physical Tails hardware? If so, locking down your Tails kernel (by way of recompiling and disabling support for 99% of hardware which you don't use, e.g. a chipset used by a random USB manufacturer that could be used by an evil maid) might only bring a net benefit. (Fingerprinting in the game of anonymity is a complex problem. The best choice for you depends on your threat model, or the balance of multiple threat models if applicable. I'll try to advise the best practice here.)*

Compiling the kernel will take quite a while depending on how powerful your CPU is, and how you configure it before compiling. For the above instructions, it could be 25 minutes, 45 minutes, 2 hours, or more.

Among the files it spits out, you will have 3 kernel DEB files ready for the next step.

#### Install the modded kernel + drivers in a 'chroot' that represents what will be your modified 'T2Tails' filesystem

*Explanation time: Debian Live - on which Tails is based - compresses its Linux filesystem (containing the folders like /home, /usr, /lib, etc.) into a single file called `filesystem.squashfs` on the actual disk. It loads and decompresses a *copy* of that file into your computer's live RAM for the current session (while leaving the original template filesystem file in your stick untouched, so that it is never modified by default). We can copy this file from an existing Tails disk, decompress it, edit the files inside (to install a T2-modded kernel and T2 drivers inside its file structure, etc.), recompress the files into a new `filesystem.squashfs`, then replace the original one on the Tails stick with that. Ta-daa!*

Now `cd` back up to your primary working directory, so that it is one level directly above the `tails-amd64-5.9` folder:

```
cd ..
```

Decompress the extracted Tails squashfs file into a folder called `squashfs-root` inside your working directory:

```
sudo unsquashfs -d squashfs-root tails-amd64-5.9/live/filesystem.squashfs
```

Copy your three kernel DEB files into the `squashfs-root` folder (which needs root or sudo):

```
sudo cp linux-*.deb squashfs-root
```

*Explanation time: To modify the extracted Tails filesystem in a smooth and successful way, we're going to use `chroot`. Chroot is a way to treat any random folder as if it is the root dir of its own virtual Linux operating system (and it makes you act as the `root` user of it - 'change root'). It enables you to do commands like `apt install something` and when you do, it installs it only in the file and folder structure inside the temporary 'chroot'. If you have enough of the required Linux system files inside the folder that you chroot into (which you now have in `squashfs-root`), you can do powerful commands to persistently update the folder structure as if it's a real live Debian OS. Very cool!*

*Some dirs and files are still needed to inject into the chroot to do everything we need to do, so we will bind-mount certain folders from your current live Tails system into it, before entering the chroot. (It's like temporary symlinking.)*

Do the following so we can smoothly update the chroot's kernel (for some reason some commands this time need to be executed separately):

```
sudo mkdir -p squashfs-root/run/live/medium 
```

```
sudo mount -o remount,rw /lib/live/mount/medium
```

```
sudo mount --bind /lib/live/mount/medium squashfs-root/run/live/medium
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

Mandatory step: In as low a voice as possible, say, "I am Chroot." Every time. (It won't work otherwise.)

Now install the three kernel files in the chroot:

```
dpkg -i linux-libc-*.deb && \
dpkg -i linux-headers-*.deb && \
dpkg -i linux-image-*.deb
```

(Wait for it to fully return to prompt. It may take a minute or two.)

This creates two installed kernel files `initrd*.img` and `vmlinuz*` and other necessary files in the main chroot filesystem.

Then do this command to make the internal (T2) keyboard + trackpad drivers automatically load at boot:

```
echo "apple-bce" > /etc/modules-load.d/t2.conf
```

For usability (at least if you want to use the discrete AMD GPU in Tails - if not just add `nomodeset` to kernel params), you now should do the following to prevent your Mac from potentially crashing due to iBridge panics in every session:

```
echo '#!/bin/sh -e
echo high | tee /sys/bus/pci/drivers/amdgpu/0000:??:??.?/power_dpm_force_performance_level
exit 0' > /etc/rc.local
```

Then do:

```
chmod +x /etc/rc.local
```

(Other methods to make this sudo command execute after every boot might be potentially sooner-executing after boot, like the new official systemd method, but I think it's extremely unlikely you'll ever get an iBridge crash before the above method executes, let's see.)

Now is the time when you may want to add some of your own other chroot modifications (e.g. install random Debian packages) that you may deem critical or useful depending on your usage scenario, if you don't consider doing so a compromise to your prospects for anonymity or privacy. (Some packages such as driver-related ones are not quite as convenient to work with unless they are pre-installed before the Tails session boots.)

Finally, clean up unnecessary junk that we added in the chroot:

```
apt clean && rm *.deb && rm -rf /var/cache/apt/* /var/cache/man/* /var/lib/apt/lists/* /tmp/* /home/amnesia/.bash_history
```

(Note: ideally we should investigate the full extent of exactly what should be cleaned up so that the modded Tails filesystem has *as few* extra files/packages added as possible. (This includes not having a second kernel version folder under `/lib/modules`.) I hope to improve this eventually.)

Now exit the chroot:

```
exit
```

Unmount the live kernel folder bind in the chroot, as well as the other 6 dirs we mounted into the chroot (ignore the warnings on the third command, the asterisk wildcard is just a hacky shortcut to make you do less input):

```
sudo umount squashfs-root/run/live/medium
```

Then:

```
sudo umount squashfs-root/etc/apt && \
sudo umount squashfs-root/dev/pts && \
sudo umount squashfs-root/*
```

More cleanup:

```
sudo rmdir squashfs-root/run/live/medium
```

Now delete the original Tails kernel and filesystem files from the extracted Tails contents:

```
sudo rm tails-amd64-5.9/live/filesystem.squashfs tails-amd64-5.9/live/initrd.img tails-amd64-5.9/live/vmlinuz
```

Then move the two T2-modded kernel files into the extracted contents:

```
sudo mv /lib/live/mount/medium/live/initrd.img /lib/live/mount/medium/live/vmlinuz tails-amd64-5.9/live
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
sudo mksquashfs squashfs-root tails-amd64-5.9/live/filesystem.squashfs -b 1024k -comp xz -Xbcj x86 -e boot
```

(It might take about 20 minutes or more, depending on your system. Why so long? This command has to pack about 150,000 files into one single file without a single mistake. That's a lot of files to process, even on current modern storage - no mean feat!)

You can then create a successful bootable T2Tails thusly:

Format DISK 3 as a big FAT partition, then simply copy over everything inside the `tails-amd64-5.9` folder directly onto the FAT disk.

(For creating an ISO, see earlier archived versions of this README.)

Congratulations, you now have a bootable **T2Tails** for your T2 Mac! The internal keyboard + trackpad will work out of the box.

## The T2Tails experience

Useful tips:

- To get Internet and Tor working in Tails (I use Ethernet), sometimes I have needed to do some or all of the following: 1. Connect Ethernet (via USB-C adapter) after Tails has loaded past the Welcome screen 2. Do `lspci` 3. Go to `Network` GNOME settings window (and if need be, disconnect 'Apple Ethernet'), then select the alternative Ethernet NIC to connect. Then Tor in Tails is able to connect as normal.

## CHANGELOG notes

v2.4 instructions:

- Instructions made a little simpler.
- New option referenced for potentially not needing to compile the kernel.
- New method necessary for preventing iBridge OS crashes during linux usage. Thank you [@Redecorating](https://github.com/Redecorating) for the tip!
- No more need for a `BOOTX64.EFI` file replacement hack.

v2.3 instructions:

- Added step to make sure the working Tails env has read-write-possible filesystem for the procedure to work.

v2.2 instructions:

- Fix for Tails not booting. Sounds like a good idea.

v2.1 instructions:

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
- Current method: it's a bit hacky how I have the `initrd.img` and `vmlinuz` replacement method (by hooking into the current Live Tails' live system). `/usr/sbin/update-initramfs` in the chroot indicates possible ways to make it more elegant. But it's complicated in how many mounts I have from live system dirs, so for now, keep it 'hacky' because it works.
- Current method: Check official Debian Live and Tails docs/build scripts for the ideal squashfs compression parameters, if different.
- Eventually, change method to build Tails ISOIMG from scratch using official instructions (but including the necessary kernel and driver modifications and insertions for T2) to produce an ISO/IMG. Potentially make everything automatic script, for producing the T2Tails ISO.

## Useful reference

- https://debian-handbook.info/browse/stable/sect.kernel-compilation.html
- https://wiki.t2linux.org

## Acknowledgements

- Everyone involved with t2linux.org - Thank you very much!
- [aunali1](https://github.com/aunali1)
- [Redecorating](https://github.com/Redecorating)
- AdityaGarg8
- Me. (Let's face it!...) ("Many brain cells died to bring us this information...") (WORTH IT.)
