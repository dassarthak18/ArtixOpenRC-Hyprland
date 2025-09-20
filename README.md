# Artix-OpenRC + Hyprland
Installation guide and config files for my artix-openrc + hyprland setup, for posterity's sake and in case I fuck up. As it turns out, I did fuck up quite a few times attempting to dual boot my existing install with [FreeBSD](https://www.freebsd.org/) + [Windowmaker](https://www.windowmaker.org/), and learned a couple of things the hard way:

1. Always install BSD first. BSD's swap system can and will interfere with Linux's, and besides, you don't have to manually configure grub post-install because Linux's grub will detect the existing BSD install anyway.
2. BSD is hard, and wildly different from Linux. There are no ```sudo``` privileges for you, you spoilt brat.

So naturally, I dropped the idea (rather, shelved it for some other day).

# Introduction
This setup was installed on my Thinkpad X250 with Intel i7-5600U (4 cores @ 3.2 GHz) and Intel HD Graphics 5500, 8GB RAM and 256GB SSD.

Like every other Linux installation (unless you are a Gentoo guy) the process begins with visiting the [official website](https://artixlinux.org/download.php) and downloading the image file of your choice. In my case, it was ``artix-base-openrc-20240823-x86_64.iso``. The reason for choosing an Arch-based distro was that I wanted a minimal setup that I could customize to my heart's content and that would do justice to this dated hardware. I did try out Ubuntu server a long time ago as Debian-based distros are my cup of tea, but had no luck with it. The reason for going with the base install of Artix was that ``openrc`` as an init-system (the first process started during booting of an OS, and the last process to terminate before shutdown) is more lightweight and fast as compared to Arch's ``systemd`` - which has its [fair share of criticisms](https://www.youtube.com/watch?v=o_AIw9bGogo).

<div align="center">
  <a href="https://www.youtube.com/watch?v=o_AIw9bGogo"><img src="https://img.youtube.com/vi/o_AIw9bGogo/0.jpg" alt="Benno Rice - The Tragedy of systemd"></a>
</div>

The next logical step then is to create a bootable USB installer using any tool you get your hands on - Rufus and [balenaEtcher](https://etcher.balena.io/) are popular options. I used to be a balenaEtcher user, but have eventually shifted to [Ventoy](https://www.ventoy.net/en/index.html). Ventoy offers several advantages - first, it creates a multi-bootable USB (that is, you can flash several disk images of your choice in a single USB drive and boot into the one you want to install), and secondly, adding a new ISO (or removing an existing one!) to the mix is as easy as copying (or deleting) it into the USB drive. No need for re-doing the process over and over again.

Anyway, then you boot into the system by accessing the boot menu in the BIOS - in my case, I had to press the F12 key, select my USB drive as the boot option, and then select the Artix ISO to boot into. Make sure to have secure boot option disabled, and UEFI mode enabled. Once you're inside the live system, the real fun begins. Login as the root - the default password is 'artix'. If the font is too small to be legible, the size can be doubled by using the command

```bash
setfont -d
```

This can be undone by a simple

```bash
setfont
```

With that out of the way, let's get into the nitty-gritty of the installation process. In the rest of the tutorial, for the most part, I use the terms 'I' and 'my' - this is because these steps were jotted down more as a journal to self than for the purpose of dissemination, but here we are.

# Installation of Artix

## Connecting to the Internet

If you're using an ethernet connection, this step is a pass. You can move on to the [next section](install_2.md).

If you're like me, and don't have access to an ethernet cable, you need to use wifi. This part deals with exactly how to do that. Before that however, check whether your wifi card is supported, or your device has a wifi card at all. If not, I'm afraid I can't help you much. Get a graphical installation image instead of a base ISO. As the [Artix wiki](https://wiki.artixlinux.org/Main/Installation) puts it,

> unless you are very experienced or doing a headless server installation, there is little reason to use the base images.

To be frank, a headless installation is exactly what I'm looking for, so I'll connect the wifi now. First, I set up my wifi connection. I get my device name using the command:

```bash script
ip a
```

which turns out to be ```wlan0``` in my case. So, accordingly, I enter:

```bash script
rfkill unblock wifi
ip link set wlan0 up
```

Next, I enter the ```connmanctl``` utility to scan and find to the wifi network of my choice.

```bash script
connmanctl
```
	> scan wifi
	> agent on
	> services

The ```services``` command shows the list of wifi networks available to connect to. I choose my wifi name and enter the service id in the following command. The service id may be long and confusing, so I use the autocomplete feature by pressing TAB. I enter the passphrase (of course there is a passphrase) and ```connmanctl``` confirms that I'm connected. Now, I can exit the utility.

	> connect <service id>
	> quit

I can enter the following command again (or simply run a ```ping```) to verify that I'm connected to the internet:

```bash script
ip a
```
```bash script
ping https://archlinux.org/
```

Now it's time to partition the disk.

## Partitioning the Disk

Artix provides the disk partitioning tool ```cfdisk``` out of the box for creating a GPT (GUID partition table, not the generative AI models), and I personally find it to be more intuitive than, say, [GParted](https://gparted.org/). The documentation on RHEL 8 is a good introduction to partitions in general, and is definitely a great read. In particular, there is a table that serves as a useful guide for [how much swap space to allocate](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices#recommended-system-swap-space_getting-started-with-swap):

| Amount of RAM | Recommended swap | Recommended swap if allowing for hibernation |
| -------- | ------- | ------- |
| $$\leq$$ 2GB | RAM*2 | RAM*3 |
| 2-8GB | RAM*1 | RAM*2 |
| 8-64GB | $$\geq$$ 4GB | RAM*1.5 |
| > 64GB | $$\geq$$ 4GB | Hibernation not recommended |

For me, it was a 200M partition of EFI system (the Extensible Firmware Interface, the partition used for UEFI boot, contains the bootloaders and device drivers for booting the OS) as recommended by Artix (which turns out to be pretty low for higher-end Linux distros such as my daily driver [Pop!_OS](https://pop.system76.com/); so for dual booting with such systems I would recommend 2-4G), 16G of Linux swap (supplements physical RAM, enabling the system to continue functioning when there is no more free physical RAM) because hell yeah I want hibernation, and the rest of the disk space as Linux filesystem for our root user.

In case the new partition table does not show up immediately, run the following commands to refresh and optionally turn off the swap before further configuration (assuming ```sda``` is indeed your disk; verify this first):

```bash script
swapoff -a
partx -d /dev/sda
partx -a /dev/sda
```

Once the partition table is created, it then remains to format the partitions and mount the drives to proceed with the actual installation process. Assuming the EFI, swap and root partitions are labelled ```sda1```, ```sda2``` and ```sda3``` respectively, we proceed with the following commands (make sure to check out your partition labels using ```lsblk``` to prevent messing up):

```bash script
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

Once done, we verify the final partition table using ```lsblk``` and proceed to install the system.

My final partition table was a 200M FAT32 partition for EFI, an 8G Linux swap partition (after all that talk), and a 207.4G ext4 partition for the root.

## Installing the System

Artix, like Arch, is a rolling release distribution, which means that there is a single, constantly updated version unlike point releases such as Debian or Ubuntu. So, before anything else, we need to update the system to make sure we are up to date with the current code base. Despite using a different init-system than Arch, Artix still uses the ```pacman``` package manager, so the command for updating the system remains pretty much the same:

```bash script
pacman -Syy
```

Next, we install our system to the ```/mnt``` directory using the ```basestrap``` command:

```bash script
basestrap /mnt base base-devel openrc elogind-openrc linux linux-firmware nano intel-ucode
```

The packages ```base``` and ```base-devel``` provide the core of the tools and utilities Artix uses, while ```openrc``` and ```elogind``` make up the init-system of the install. ```linux``` and ```linux-firmware``` provide the Linux kernel (the core program that manages the fundamentals of the OS) and firmware (programs providing instructions for the device's basic operations), while ```nano``` is my text editor of choice which will come in handy in the next step when I configure the system. You can use ```vim``` instead if you prefer, but I find it unnecessarily complicated.

Last but not the least, ```intel-ucode``` is a microcode that provides security and bug fixes for the CPU, ensuring system stability and security. If your device uses an AMD processor, you use ```amd-ucode``` instead.

Finally, we generate the file system table for our install to seal the deal:

```bash script
fstabgen -U /mnt >> /mnt/etc/fstab
```

Once this is done, we enter the following command to exit the live system and begin configuring our install:

```bash script
artix-chroot /mnt
```

## [Configuring the System](tutorials/install_4.md)

## Post-Installation

We are done with the installation, so as a duty-bound citizen of the Linux community, I can now execute the following commands:

```bash script
pacman -S neofetch
neofetch
lsblk
```

A screenshot where a screenshot is due:

![neofetch](screenshots/IMG_8387.jpeg)

As you can see, the system is taking up only 158MiB of RAM. This is the part where I gleefully say: I use Artix, btw.
