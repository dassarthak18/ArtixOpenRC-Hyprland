# Artix-OpenRC + Hyprland
Installation guide and config files for my artix-openrc + Hyprland setup, for posterity's sake and in case I fuck up. As it turns out, I did fuck up quite a few times attempting to dual boot my existing install with [FreeBSD](https://www.freebsd.org/) + [Windowmaker](https://www.windowmaker.org/), and learned a couple of things the hard way:

1. Always install BSD first. BSD's swap system can and will interfere with Linux's, and besides, you don't have to manually configure grub post-install because Linux's grub will detect the existing BSD install anyway.
2. BSD is hard, and wildly different from Linux. There are no ```sudo``` privileges for you, you spoilt brat.

So naturally, I dropped the idea (rather, shelved it for some other day).

# Table of Contents

- [Introduction](#introduction)
- [Installation of Artix](#installation-of-artix)
  - [Connecting to the Internet](#connecting-to-the-internet)
  - [Partitioning the Disk](#partitioning-the-disk)
  - [Installing the System](#installing-the-system)
  - [Configuring the System](#configuring-the-system)
  - [Post-Installation](#post-installation)
- [Installation of Hyprland](#installation-of-hyprland)

# Introduction
This setup was installed on my Thinkpad X250 with Intel i7-5600U (4 cores @ 3.2 GHz) and Intel HD Graphics 5500, 8GB RAM and 256GB SSD.

Like every other Linux installation (unless you are a Gentoo guy) the process begins with visiting the [official website](https://artixlinux.org/download.php) and downloading the image file of your choice. In my case, it was ``artix-base-openrc-20240823-x86_64.iso``. The reason for choosing an Arch-based distro was that I wanted a minimal setup that I could customize to my heart's content and that would do justice to this dated hardware. I did try out Ubuntu server a long time ago as Debian-based distros are my cup of tea, but had no luck with it. The reason for going with the base install of Artix was that ``openrc`` as an init-system (the first process started during booting of an OS, and the last process to terminate before shutdown) is more lightweight and fast as compared to Arch's ``systemd`` - which has its [fair share of criticisms](https://www.youtube.com/watch?v=o_AIw9bGogo).

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

If you're using an ethernet connection, this step is a pass. You can move on to the next section.

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
ping archlinux.org
```

Now it's time to partition the disk.

## Partitioning the Disk

Artix provides the disk partitioning tool ```cfdisk``` (type ```cfdisk <disk_name>``` to start) out of the box for creating a GPT (GUID partition table, not the generative AI models), and I personally find it to be more intuitive than, say, [GParted](https://gparted.org/). The documentation on RHEL 8 is a good introduction to partitions in general, and is definitely a great read. In particular, there is a table that serves as a useful guide for [how much swap space to allocate](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices#recommended-system-swap-space_getting-started-with-swap):

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

My final partition table was a 300M FAT32 partition for EFI, an 16G Linux swap partition, and a 222.2G ext4 partition for the root.

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

## Configuring the System

Now it is time to configure our system. The first thing to do is set up the localtime and sync it with the system clock:

```bash script
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

Fill in the appropriate region and city in the command - in my case it is ```Asia/Kolkata```; in your case you can refer to [the tz database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) and set up your localtime accordingly.

Then, we must configure the locale (language and keyboard layout) of the system. In order to do that, we edit the ```locale.gen``` file in the ```/etc/``` directory and uncomment the suitable locale (```en_US.UTF-8 UTF-8``` in my case), and generate the locale accordingly. We also set the language in the ```locale.conf``` file in the same directory by writing ```LANG=en_US.UTF-8```, or whatever it is in your case:

```bash script
nano /etc/locale.gen
locale-gen
nano /etc/locale.conf
```

Next, we set the hostname of the system by writing the hostname of our choice (I set mine to 'artix') to ```/etc/hostname``` and configuring the ```/etc/hosts``` file:

```bash script
nano /etc/hostname
nano /etc/hosts
```

The configuration of the ```etc/hosts``` file is as follows:

	127.0.0.1  localhost
	::1        localhost
	127.0.1.1  <hostname>.localdomain  <hostname>

where ```<hostname>``` is the hostname of our device.

We can then set the password for the super user (root):

```bash script
passwd
```

Following this, we download the essential packages required for the system to function properly:

```bash script
pacman -S grub efibootmgr dialog os-prober dosfstools linux-headers
pacman -S networkmanager networkmanager-openrc network-manager-applet wpa_supplicant openssh openssh-openrc
pacman -S bluez bluez-openrc bluez-utils cups cups-openrc
pacman -S dbus dbus-openrc pipewire pipewire-openrc pipewire-alsa pipewire-pulse pipewire-jack wireplumber wireplumber-openrc
pacman -S mesa mesa-demos xf86-video-intel
```

The packages ```grub``` and ```efibootmgr``` serve as the bootloader for our system. ```dialog``` displays dialog boxes from shell scripts, while ```os-prober``` is needed for dual boot support. ```dosfstools``` provides tools to create, check and label file systems of the FAT family, while ```linux-headers``` provides headers for the Linux kernel.

The packages ```networkmanager```, ```networkmanager-openrc```, ```network-manager-applet```and ```wpa_supplicant``` provide the framework and interface for network connectivity, while the packages ```openssh``` and ```openssh-openrc``` provide support for remote login via the SSH protocol. The packages ```bluez```, ```bluez-openrc``` and ```bluez-utils``` provide bluetooth support, while the packages ```cups``` and ```cups-openrc``` provide support for printers and scanners.

The packages ```dbus``` and ```dbus-openrc``` provide the message bus system used for communication between applications and services. The packages ```pipewire```, ```pipewire-alsa```, ```pipewire-pulse``` and ```pipewire-jack``` provide the modern audio and multimedia framework. ```wireplumber``` is the session manager for PipeWire, handling policy and device management to ensure smooth audio routing. ``pipewire-openrc`` and ``wireplumber-openrc`` are the respective OpenRC daemons for ``pipewire`` and ``wireplumber``.

Finally, ```mesa``` and ```mesa-demos``` provides OpenGL/Vulkan rendering, while ```xf86-video-intel``` provides plug-and-play drivers for Intel GPU, which is the case for my device. If your device has an AMD GPU, install the package ```xf86-video-amdgpu``` instead. Alternatively, if your device has an Nvidia GPU, you will need the packages ```nvidia``` and ```nvidia-utils```.

Now that we have installed ```grub```, it is time to set up GRUB as our bootloader:

```bash script
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

We also add the network, SSH, bluetooth, printer and message bus services to our init-system to autostart on boot, that is, at the default runlevel. In ```systemd``` we use the command ```systemctl``` for configuring daemons (background processes; their names usually end with a _d_). But in our case we are using ```openrc```, so our command will be ```rc-update```. Note that other init-systems shipped with Artix (namely ```runit```, ```dinit``` and ```s6```) will vary in their commands for configuring daemons, although the rest of the installation and configuration process remains mostly the same. Also note that ```pipewire``` and ```wireplumber``` are not system-level services but user-level.

```bash script
rc-update add NetworkManager default
rc-update add sshd default
rc-update add bluetoothd default
rc-update add cupsd default
rc-update add dbus default
```

Upon first login post-install, we will make sure to run the following commands:

```bash script
rc-update add pipewire default --user
rc-update add wireplumber default --user
```

and reboot to make sure the audio services autostart on login.

We are done with out configuration. Now it only remains to add users to our system if we so choose. This is done using the ```useradd``` command. The flag ```m``` creates a home directory for the user while the flag ```G``` adds the user to a specified group which, in this case, is the ```wheel``` (the group of users with ```sudo``` privileges). We can also set a password for an user.

```bash script
useradd -mG wheel <username>
passwd <username>
```

Finally, we use the ```visudo``` command (I open it using ```nano``` as my text editor, but you can go with anything you like) and allow the users in the ```wheel``` group to execute all commands with a password prompt, by uncommenting the line ``` %wheel ALL=(ALL:ALL) ALL```:

```
EDITOR=nano visudo
```

Our installation is complete. Now we can exit the ```artix-chroot```, unmount the drive and reboot. Upon sending the reboot signal, we can safely remove the installation media (USB drive) and proceed with our system.

```bash script
exit
umount -R /mnt
reboot
```

Upon booting into the system for the first time, we can connect to a wifi network of our choice using the ```nmtui``` command, which is an intuitive TUI (terminal user interface) for the ```NetworkManager```. The system will remember our preference, and autoconnect to that wifi network on login.

## Post-Installation

We are done with the installation, so as a duty-bound citizen of the Linux community, I can now execute the following commands ([fastfetch](https://github.com/fastfetch-cli/fastfetch) replaces the now depreciated neofetch):

```bash script
sudo pacman -S fastfetch
fastfetch
lsblk
```

We can also verify if the audio services are in place using [SoX](https://github.com/chirlu/sox):

```bash script
sudo pacman -S sox
play -n synth 1 sine 440
```

If we hear a beep sound, we should be fine.

# Installation of Hyprland

## Setting Up Waybar

Before we install and configure Hyprland itself, we must install up Waybar. [Waybar](https://github.com/Alexays/Waybar) provides a highly customizable Wayland bar for compositors like Hyprland. To install Waybar, we run:

```bash script
sudo pacman -S alacritty waybar blueman pavucontrol nm-connection-editor htop hyprlock
```

This installs not only Waybar, but also

* ``alacritty`` which is our terminal emulator of choice,
* ``blueman``, ``pavucontrol`` and ``nm-connection-editor`` which are GUI for configuring bluetooth, audio and network connections respectively,
* ``htop`` which is our system monitor of choice, and
* ``hyprlock`` which is the screen locker for Hyprland.

My Waybar config file is provided in ``dotfiles/waybar/config.jsonc``. Clone the repository and copy the config file to ``~/.config/waybar/config.jsonc`` as follows:

```bash script
sudo pacman -S git
git clone https://github.com/dassarthak18/ArtixOpenRC-Hyprland.git
cp ArtixOpenRC-Hyprland/dotfiles/waybar/config.jsonc ~/.config/waybar/config.jsonc
```
