# Tutorial installing Linux on Asus T100HA

This is tutorial of installing Linux for Asus T100HA.
Known issues: wifi rarely breaks, can't change volume level (sound works), touchscreen doesn't work(on Manjaro, will probably work on Ubuntu).
Thanks @gmelchett for kernel patches and kerneel config. Original post here: https://bbs.archlinux.org/viewtopic.php?pid=1700679#p1700679

Requirements - USB Wi-Fi dongle

## 1. LiveUSB setup

Download Manjaro x64 XFCE .iso and make bootable pendrive (I recommend RUFUS for this task).


## 2. Setup BIOS

Reboot notebook, while booting press `F2`. Go to boot menu and disable secure boot. Reboot, press `escape` button while booting and choose boot from USB.

## 3.Install Manjaro

On boot menu press 'e' key and add kernel parameter `fbcon=rogate:3` (it's not mandatory but I recommend to do this, on other case you will have portrait mode).
Install Manjaro, choose 'erase disk' option.


## 4.Prepare Manjaro for custom kernel

Firstly you have to rotate screen (in settings) and install some programs. You do it by command `sudo pacman -S packagename` .
List of programs you have to install:
- bluez
- bluez-utils
- lzop
- wget

### Install RC.local
```
sudo rm /etc/rc.local
git clone https://aur.archlinux.org/rc-local.git
cd rc-local
makepkg -s
makepkg -i (may be sudo required)
```
Enable RC.local service (command shows after installation)
`sudo nano /etc/rc.local`
Paste this code:
```
#!/bin/sh
btattach -B /dev/ttyS1 -P bcm &
modprobe usbhid
(sleep 2; modprobe i915) &

echo '0' > /proc/sys/kernel/nmi_watchdog
echo '1500' > /proc/sys/vm/dirty_writeback_centisecs
echo 'auto' > /sys/bus/pci/devices/0000:00:14.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:00.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:03.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:1a.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:0b.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:02.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:0a.0/power/control
echo 'auto' > /sys/bus/pci/devices/0000:00:1f.0/power/control

systemctl restart acpid
```

### Update grub

`sudo nano /etc/default/grub`
in 'GRUB-CMDLINE-LINUX-DEFAULT' add following code: `modprobe.blacklist=i915 video=LVDS-1:d fbcon=rotate:3`
Then type command: `sudo update-grub`

### Add i915 and usbhid to blacklist

You have to add i915 to blacklist (on other case brightness adjustment won't work) and usbhid (for better touchpad)
`sudo nano /etc/modprobe.d/blacklist.conf`
Type:
```
blacklist i915
blacklist usbhid
```


### Disable GUI on startup

For brightness adjustment you must start GUI manually after login (startx). 
Type `sudo systemctl set-default multi-user.target`


### Add wi-fi support (It will be available after use new kernel)

```
wget https://android.googlesource.com/platform/hardware/broadcom/wlan/+archive/master/bcmdhd/firmware/bcm43341.tar.gz
tar xf bcm43341.tar.gz 
sudo mkdir -p /lib/firmware/brcm/ 
sudo cp fw_bcm43341.bin /lib/firmware/brcm/brcmfmac43340-sdio.bin
sudo mount -t efivarfs efivarfs /sys/firmware/efi/efivars
sudo cp /sys/firmware/efi/efivars/nvram-74b00bd9-805a-4d61-b51f-43268123d113 /lib/firmware/brcm/brcmfmac43340-sdio.txt
```

## 7.Compile kernel

Download this git repo and compile kernel(it will take several hours)
```
cd t100ha-kernel
makepkg -s --skipinteg
```
Then after compiling
'makepkg -i'


## 8. Reboot and choose t100ha-kernel on grub. Login and launch GUI by startx command.
