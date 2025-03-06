## Wifibox Made Easy

I notice the FreeBSD manpage on Wifibox was somewhat incomplete, which prompted me to make this guide on setting up Wifibox.

Mainly sourced and presented as a combination of: [Jrgsystems](https://jrgsystems.com/posts/2022-04-20-802.11ac-on-freebsd-with-wifibox/) and the [FreeBSD manpage](https://man.freebsd.org/cgi/man.cgi?query=wifibox&apropos=0&sektion=8&manpath=freebsd-ports&format=html)

First, we install Wifibox (duh): ```pkg install wifibox```

Run both of these:
```pciconf -lv | grep -B3 -i wireless```
```pciconf -lv | grep -B3 -i ax```
if one of these gives you an output, like 
```
ppt0@pci0:37:0:0:	class=0x028000 rev=0x1a hdr=0x00 vendor=0x8086 device=0x2725 subvendor=0x8086 subdevice=0x0024
    vendor     = 'Intel Corporation'
    device     = 'Wi-Fi 6E(802.11ax) AX210/AX1675* 2x2 [Typhoon Peak]'
    class      = network
```
then great. We can move on. If not, then find it manually with ```pciconf -lv```

If you're using an AMD CPU, then put ```hw.vmm.amdvi.enable=1``` in ```/boot/loader.conf```. Reboot.

Open ```/usr/local/etc/wifibox/bhyve.conf``` to point to our wifi card. Find ```passthru=``` and modify it to point to your wifi card. If it shows ```ppt0@pci0:37:0:0``` in pciconf, then it would be ```37/0/0```. 

Now we need to edit ```wpa_supplicant.conf```. Run ```rm -rf /usr/local/etc/wifibox/wpa_supplicant/wpa_supplicant.conf && cp /etc/wpa_supplicant.conf /usr/local/etc/wifibox/wpa_supplicant/wpa_supplicant.conf``` to give the bhyve vm the right wifi info. 

We finally got all of that out of the way, so let's do rc.conf tweaking now.
```
sysrc wifibox_enable="YES"
devmatch_enable="YES"
devmatch_blocklist="if_iwm if_iwlwifi"
service devmatch start
kldunload if_iwlwifi
kldunload if_iwm
service wifibox start
sysrc ifconfig_wifibox0="SYNCDHCP"
sysrc background_dhclient_wifibox0="YES"
defaultroute_delay="0"
service netif start wifibox0
service routing restart
service devd restart
```
This entire readme is an insanely brief abbreviation of all the steps. Read [this](https://man.freebsd.org/cgi/man.cgi?query=wifibox&apropos=0&sektion=8&manpath=freebsd-ports&format=html) know what these commands do.

Hopefully, a reboot will result in a working wifibox configuration for you. Hope this helps.
