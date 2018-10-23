# Antergos tips

Change and customization log on my Antergos linux system

## Installation

### UEFI stuff

Follow the [Arch wiki](https://wiki.archlinux.org/index.php/Dell_XPS_15_9560#UEFI){:target="_blank"}. Basically (1) switch SATA mode from RAID to ACHI, (2) disable secure boot, and (3) change fastboot to "Through" in "Post behaviour". Not sure that this last step is needed. I also enabled legacy BIOS drivers, since my system was having trouble reading the live USB.

## Touchpad

The KDE graphical touchpad settings (*System Settings > Input Devices > Touchpad*) didn't seem to last and kept reverting back to the default behaviour. So I [installed](https://wiki.archlinux.org/index.php/Libinput#Installation){:target="_blank"} `libinput` and then followed the final section of [this guide](https://www.dell.com/support/article/us/en/04/sln308258/precision-xps-ubuntu-general-touchpad-mouse-issue-fix?lang=en){:target="_blank"} (See Fig. 7) to get tapping, right-click two finger tap, etc. working.


## Data science setup

I followed (most of) the tips on Patrick Schratz' [exellent guide](https://github.com/pat-s/antergos_setup_guide){:target="_blank"}. I also made the following changes in addition to that.

### Set common *R* library path

Adapting [this](https://stackoverflow.com/questions/44861967/r-3-4-1-single-candle-personal-library-path-error-unable-to-create-na/44903158#44903158){:target="_blank"} SO post answer, I set a system wide library path as follows:
```
sudo groupadd rusers
sudo gpasswd -a grant rusers
cd /usr/lib/R ## Get location by typing ".libPaths()" in your R console
sudo chown grant:rusers -R R/
sudo chmod -R 775 R/
```
Once that's done, tell *R* to make this shared library path the default for your user, by adding the following line to your `~/.Renviron` file:
```
R_LIBS_USER=/usr/lib/R/library ## Or whatever location you get by typing ".libPaths()" in your R console
```

## conda

I [installed](https://jakevdp.github.io/PythonDataScienceHandbook/00.00-preface.html#Installation-Considerations){:target="_blank"} Miniconda3 using bash before I switched over to the zsh shell. As a result (or perhaps it's required no matter when you switch over to zsh), I had to [add the Miniconda directory to the zsh PATH environment variable](https://stackoverflow.com/a/35246794){:target="_blank"}

```
> echo 'export PATH="/home/grant/miniconda3/bin:$PATH"' >> .zshrc
> source ~/.zshrc ## Or you can just close and reopen the shell
```

## GPU / NVIDIA CUDA

My laptop (Dell Precision 9570) comes with a hybrid graphics system. I initially tried to get CUDA support going by installing the nvida package from the Arch repositories... Which turned out to be a mistake!The system would boot up fine, but I was subsequently presented with a blank screen once I got passed the GRUB menu. 

**Solution:** Boot directly into the shell (i.e. TTY) and uninstall the nvidia package: Press "Ctr-Alt-F2" at the grub menu and then hit "e" to edit the selection. Look for the line starting with "linux" and add "3" (without the quotation marks) to the end of that line. F10 to exit and then you will be presented with the shell upon booting up. Enter your username, followed by your password. Finally, uninstall the nvidia package by typing `sudo pacman -Rs nvidia` and 
reboot as normal ("CTR-ALT-DEL").

## Miscellaneous

### Printing

My home printer had been found automatically at first. However, I couldn't find it after a while for some reason. Adding it manually was a pain, because I didn't have the correct permissions in CUPS. (Adding myself to the "cups" user group didn't work either.) I solved the problem by following [these instructions](https://kernelmastery.com/enable-regular-users-to-add-printers-to-cups/).

### Wi-fi from Shell/TTY

Relevant to cases where you need to log into the Shell/TTY to fix some hanging/freeze problem (e.g. CUDA/NVIDIA above). The easiest way I've found is to use `nmcli` (command line version of network manger). To see the available SSIDs type
```
nmcli dev wifi
```
Then connect with
```
nmcli dev wifi connect SSID_NAME password SSID_PASSWORD
```

### Starting Gnome session from Shell/TTY

Similar rational to the above:
```
XDG_SESSION_TYPE=wayland dbus-run-session gnome-session
```


### HiDPI

The Arch wiki has the goods here. One thing I'll add explicitly here is how to change the default Linux Console font that appears when booting up (or when booting into TTY). First download the terminus fonts family:
```
pac install terminus-font
```
You can then see the set of available fonts by typing `ls /usr/share/kbd/console`. To temporarily test out a larger font, type
```
setfont ter-132n
```
(Type `showconsolefont` if you want to see a table of the font's glyphs and letters).

To set this font permanently, open `/etc/vconsole.conf` with nano and add
```
FONT=ter-132n
```

