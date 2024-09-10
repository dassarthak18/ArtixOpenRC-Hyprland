# Partitioning the Disk

Artix provides the disk partitioning tool ```cfdisk``` out of the box for creating a GPT partition table, and I personally find it to be more intuitive than, say, [GParted](https://gparted.org/). The documentation on RHEL 8 is a good introduction to partitions in general, and is definitely a great read. In particular, there is a table that serves as a useful guide for [how much swap space to allocate](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices#recommended-system-swap-space_getting-started-with-swap):

| Amount of RAM | Recommended swap | Recommended swap if allowing for hibernation |
| -------- | ------- | ------- |
| $$\leq$$ 2GB | RAM*2 | RAM*3 |
| 2-8GB | RAM*1 | RAM*2 |
| 8-64GB | $$\geq$$ 4GB | RAM*1.5 |
| > 64GB | $$\geq$$ 4GB | Hibernation not recommended |

For me, it was a 200M partition of EFI system (the Extensible Firmware Interface, the partition used for UEFI boot, contains the bootloaders and device drivers for booting the OS) as recommended by Artix (which turns out to be pretty low for higher-end Linux distros such as my daily driver [Pop!_OS](https://pop.system76.com/); so for dual booting with such systems I would recommend 2-4G), 16G of Linux swap (supplements physical RAM, enabling the system to continue functioning when there is no more free physical RAM) because hell yeah I want hibernation, and the rest of the disk space as Linux filesystem for our root user.

Once the partition table is created, it then remains to format the partitions and mount the drives to proceed with the actual installation process. Assuming the EFI, swap and root partitions are labelled ```sda1```, ```sda2``` and ```sda3``` respectively, we proceed with the following commands (make sure to check out your partition labels using ```lsblk``` to prevent messing up):

```bash script
mkfs.fat -F32 /dev/sda1
swapon /dev/sda2
mkfs.ext4 /dev/sda3
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

Once done, we verify the final partition table using ```lsblk``` and proceed to [install the system](install_3.md).

My final partition table was a 200M FAT32 partition for EFI, an 8G Linux swap partition, and a 207.4G ext4 partition for the root.
