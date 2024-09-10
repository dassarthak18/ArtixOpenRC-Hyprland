# Installing the System

Artix, like Arch, is a rolling release distribution, which means that there is a single, constantly updated version unlike point releases such as Debian or Ubuntu. So, before anything else, we need to update the system to make sure we are up to date with the current code base. Despite using a different init-system than Arch, Artix still uses the ```pacman``` package manager, so the command for updating the system remains pretty much the same:

```bash script
pacman -Syy
```

Next, we install our system to the ```/mnt``` directory using the ```basestrap``` command:

```bash script
basestrap /mnt base base-devel openrc elogind-openrc linux linux-firmware nano intel-ucode
```

The packages ```base``` and ```base-devel``` provide the core of the tools and utilities Artix uses, while ```openrc``` and ```elogind``` make up the init-system of the install. ```linux``` and ```linux-firmware``` provide the Linux kernel, while ```nano``` is my text editor of choice which will come in handy in the next step when I [configure the system](install_4.md). You can use ```vim``` instead if you prefer, but I find it unnecessarily complicated.

Last but not the least, ```intel-ucode``` is a microcode that provides security and bug fixes for the CPU, ensuring system stability and security. If your device uses an AMD processor, you use ```amd-ucode``` instead.

Finally, we generate the file system table for our install to seal the deal:

```bash script
fstabgen -U /mnt >> /mnt/etc/fstab
```

Once this is done, we enter the following command to exit the live system and begin [configuring our install](install_4.md):

```bash script
artix-chroot /mnt
```
