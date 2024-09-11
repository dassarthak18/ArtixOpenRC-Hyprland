# Connecting to the Internet

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

Now it's time to [partition the disk](install_2.md).
