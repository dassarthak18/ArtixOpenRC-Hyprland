# Artix-OpenRC + Hyprland
Installation guide and config files for my artix-openrc + hyprland setup, for posterity's sake and in case I fuck up.

# Introduction
This setup was installed on my Thinkpad X250 with Intel i7-5600U (4 cores @ 3.2 GHz) and Intel HD Graphics 5500, 8GB RAM and 256GB SSD.

Like every other Linux installation (unless you are a Gentoo guy) the process begins with visiting the [official website](https://artixlinux.org/download.php) and downloading the image file of your choice. In my case, it was ``artix-base-openrc-20240823-x86_64.iso``. The reason for choosing an Arch-based distro was that I wanted a minimal setup that I could customize to my heart's extent and that would do justice to this dated hardware. The reason for going with the base install of Artix was that ``openrc`` as an init-system (the first process started during booting of an OS, and the last process to terminate before shutdown) is more lightweight and fast as compared to Arch's ``systemd`` - which has its [fair share of criticisms](https://www.youtube.com/watch?v=o_AIw9bGogo).

{% include youtube.html id="o_AIw9bGogo" %}

The next logical step then is to create a bootable USB tool using any tool you get your hands on - Rufus and [balenaEtcher](https://etcher.balena.io/) are popular options. I used to be a balenaEtcher user, but have eventually shifted to [Ventoy](https://www.ventoy.net/en/index.html). Ventoy offers several advantages - first, it creates a multi-bootable USB (that is, you can flash several disk images of your choice in a single USB and boot into the one you want to install), and secondly, adding a new ISO to the mix is as easy as copying it into the USB drive. No need for re-doing the process over and over again.

Anyway, then you boot into the system by accessing the boot menu in the BIOS - in my case, I had to press the F12 key, select my USB drive as the boot option, and then select the Artix iso to boot into. Make sure to have secure boot option disabled, and UEFI mode enabled. Once you're inside the live system, the real fun begins.
