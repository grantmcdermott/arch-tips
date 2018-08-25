# Antergos tips

Customization log on my Antergos linux system

## Installation

### UEFI stuff

Follow the [Arch wiki](https://wiki.archlinux.org/index.php/Dell_XPS_15_9560#UEFI). Basically (1) switch SATA mode from RAID to ACHI, (2) disable secure boot, and (3) change fastboot to "Through" in "Post behaviour". Not sure that this last step is needed. I also enabled legacy BIOS drivers, since my system was having trouble reading the live USB.

## Touchpad

The KDE graphical touchpad settings (*System Settings > Input Devices > Touchpad*) didn't seem to last and kept reverting back to the default behaviour. So I [installed](https://wiki.archlinux.org/index.php/Libinput#Installation) `libinput` and then followed the final section of [this guide](https://www.dell.com/support/article/us/en/04/sln308258/precision-xps-ubuntu-general-touchpad-mouse-issue-fix?lang=en) (See Fig. 6) to get tapping, right-click two finger tap, etc. working.


## Data science setup

I followed (most of) the tips on Patrick Schratz' [exellent guide](https://github.com/pat-s/antergos_setup_guide). 
