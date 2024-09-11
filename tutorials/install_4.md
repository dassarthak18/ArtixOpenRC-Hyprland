# Configuring the System

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
pacman -S xf86-video-intel
```

The packages ```grub``` and ```efibootmgr``` serve as the bootloader for our system. ```dialog``` displays dialog boxes from shell scripts, while ```os-prober``` is needed for dual boot support. ```dosfstools``` provides tools to create, check and label file systems of the FAT family, while ```linux-headers``` provides headers for the Linux kernel.

The packages ```networkmanager```, ```networkmanager-openrc```, ```network-manager-applet```and ```wpa_supplicant``` provide the framework and interface for network connectivity, while the packages ```openssh``` and ```openssh-openrc``` provide support for remote login via the SSH protocol. The packages ```bluez```, ```bluez-openrc``` and ```bluez-utils``` provide bluetooth support, while the packages ```cups``` and ```cups-openrc``` provide support for printers and scanners.

Finally, ```xf86-video-intel``` provides plug-and-play drivers for Intel GPU, which is the case for my device. If your device has an AMD GPU, install the package ```xf86-video-amdgpu``` instead. Alternatively, if your device has an Nvidia GPU, you will need the packages ```nvidia``` and ```nvidia-utils```.

Now that we have installed ```grub```, it is time to set up GRUB as our bootloader:

```bash script
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

We also add the network, SSH, bluetooth and printer services to our init-system to autostart on boot, that is, at the default runlevel. In ```systemd``` we use the command ```systemctl``` for configuring daemons (background processes; their names usually end with a _d_). But in our case we are using ```openrc```, so our command will be ```rc-update```. Note that other init-systems shipped with Artix (namely ```runit```, ```dinit``` and ```s6```) will vary in their commands for configuring daemons, although the rest of the install process remains mostly the same.

```bash script
rc-update add NetworkManager default
rc-update add sshd default
rc-update add bluetoothd default
rc-update add cupsd default
```

We are done with out configuration. Now it only remains to add users to our system if we so choose. This is done using the ```useradd``` command. The flag ```m``` creates a home directory for the user while the flag ```G``` adds the user to a specified group which, in this case, is the ```wheel``` (the group of users with sudo privileges). We can also set a password for an user.

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
