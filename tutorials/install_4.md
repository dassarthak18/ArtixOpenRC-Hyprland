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

pacman -S grub efibootmgr networkmanager networkmanager-openrc network-manager-applet wpa_supplicant dialog os-prober dosfstools linux-headers bluez bluez-openrc bluez-utils cups cups-openrc openssh openssh-openrc xf86-video-intel (or, xf86-video-amdgpu/ nvidia nvidia-utils)
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
rc-update add NetworkManager default
rc-update add bluetoothd default
rc-update add cupsd default
rc-update add sshd default

useradd -mG wheel username
passwd username
EDITOR=nano visudo (Uncomment %wheel ALL=(ALL:ALL) ALL)

exit
umount -R /mnt
reboot
