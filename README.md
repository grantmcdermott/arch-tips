# Arch Linux tips

Change and customization log on my Arch Linux system

<!-- TOC depthFrom:2 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Installation](#installation)
	- [UEFI stuff](#uefi-stuff)
	- [Home folder on separate partition](#home-folder-on-separate-partition)
- [Backup](#backup)
- [Data science setup](#data-science-setup)
	- [R](#r)
	- [conda (Python)](#conda-python)
	- [Julia](#julia)
	- [GPU / NVIDIA CUDA](#gpu-nvidia-cuda)
- [Removing Antergos](#removing-antergos)
- [Miscellaneous](#miscellaneous)
	- [Printing](#printing)
	- [Wi-fi from Shell/TTY](#wi-fi-from-shelltty)
	- [Starting Gnome session from Shell/TTY](#starting-gnome-session-from-shelltty)
	- [HiDPI](#hidpi)
	- [Manually compile a package from source with edited PKGBUILD (Julia example)](#manually-compile-a-package-from-source-with-edited-pkgbuild-julia-example)
	- [Touchpad](#touchpad)

<!-- /TOC -->

## Installation

### UEFI stuff

Follow the [Arch wiki](https://wiki.archlinux.org/index.php/Dell_XPS_15_9560#UEFI). Basically (1) switch SATA mode from RAID to ACHI, (2) disable secure boot, and (3) change fastboot to "Through" in "Post behaviour". Not sure that this last step is needed. I also enabled legacy BIOS drivers, since my system was having trouble reading the live USB.

### Home folder on separate partition

I think this was an option on the original install media, but I somehow missed it. At any rate, creating this after the fact was relatively easy. I first created a GParted Live USB (download the ISO image [here](https://gparted.org/liveusb.php) and flash with Etcher). This was a lot quicker than creating a live USB of an entire distro and I only needed to resize some partitions anyway. From here, there are various guides (e.g. [here](https://help.ubuntu.com/community/Partitioning/Home/Moving) and [here](https://www.maketecheasier.com/move-home-folder-ubuntu/)) and I just followed along. FWIW, keeping your home directory on a separate partition is probably safer and also makes [distro hopping](https://www.maketecheasier.com/switch-between-linux-distros-without-losing-data/) easier.

## Backup

Very easy with rsync. See [this video](https://www.youtube.com/watch?v=oS5uH0mzMTg).

```sh
$ bash ## zsh doesn't work for some reason
$ sudo rsync -aAXv --delete --dry-run --exclude=/dev/* --exclude=/proc/* --exclude=/sys/* --exclude=/tmp/* --exclude=/run/* --exclude=/mnt/* --exclude=/media/* --exclude="swapfile" --exclude="lost+found" --exclude=".cache" --exclude=".VirtualBoxVMs" --exclude=".ecryptfs" / /run/media/grant/PrecisionBackup
```

## Data science setup

I followed (most of) the tips on Patrick Schratz' [outstanding guide](https://pjs-web.de/post/arch-install-guide-for-r/). His setup is tailored to R and spatial analysis, which covers my major needs. However, I also made the following changes in addition to his suggestions (including notes on some other lanaguages).

### R

#### Set common *R* library path

Adapting [this](https://stackoverflow.com/questions/44861967/r-3-4-1-single-candle-personal-library-path-error-unable-to-create-na/44903158#44903158) SO post answer, I set a system wide library path as follows:

```sh
$ sudo groupadd rusers
$ sudo gpasswd -a grant rusers
$ cd /usr/lib/R ## Get location by typing ".libPaths()" in your R console
$ sudo chown grant:rusers -R R/
$ sudo chmod -R 775 R/
```

Once that's done, tell *R* to make this shared library path the default for your user, by adding it to your `~/.Renviron` file:

```sh
$ echo 'R_LIBS_USER=/usr/lib/R/library' >>  ~/.Renviron
```

#### Compile R packages in parallel

Reinstallation of R packages is already much quicker thanks to ccache (again, see Patrick's [guide](https://pjs-web.de/post/arch-install-guide-for-r/#ccache)). However, I also enabled parallel compilation of R packages to speed up first-time installation, as well as any further compiling that needs to be done.

```sh
echo 'options(Ncpus=parallel::detectCores())' >> ~/.Rprofile
```

### conda (Python)

Following [Jake Vanderplas](https://jakevdp.github.io/PythonDataScienceHandbook/00.00-preface.html#Installation-Considerations), I opted for Miniconda instead of the full-blown Anaconda install.

#### Add conda to path (if changing shells after installation)

In most cases, your conda environment should automatically get added to your PATH. However I installed Miniconda3 using bash before switching over to the zsh shell. As a result, I had to add the Miniconda directory to the zsh PATH environment variable (see [here](https://stackoverflow.com/a/35246794)).

```sh
$ echo 'export PATH="/home/grant/miniconda3/bin:$PATH"' >> .zshrc
$ source ~/.zshrc ## Or you can just close and reopen the shell
```

#### conda environments

To activate conda environments, one first need to initiliase this capability for your particular shell (zsh, fish, etc.). However, this has the undesirable effect of automatically activating the previous conda env whenever you open the shell, regardless of whether you want to use conda or not. Luckily there's a pretty simple [solution](https://stackoverflow.com/a/54560785/4115816):

```sh
$ conda init zsh ## Initialise. Then log out and then back in.
$ conda config --set auto_activate_base false ## Stop automatic activation.
```

Once that's done, it's easy to activate and switch between environments

```sh
$ conda info --envs ## list all environments
$ conda activate tf_gpu ## activate the "tf_gpu" environment
$ conda deactivate
```

#### Add channels

Necessary, for example, when I wanted to add the Apache Arrow C++ module from Conda Forge

```sh
$ conda config --add channels conda-forge
$ conda config --set channel_priority strict
```

### Julia

There's a distributed version of Julia via the Arch community repos, but I eventually switched to using the official Julia binaries instead as is [recommended](https://julialang.org/downloads/platform.html). To take the pain out managing subsquent versions and updates, I used the handy [JILL](https://github.com/abelsiqueira/jill) script:

```sh
$ sudo bash -ci "$(curl -fsSL https://raw.githubusercontent.com/abelsiqueira/jill/master/jill.sh)"
```

### GPU / NVIDIA CUDA

My Dell Precision 9570 laptop comes with a hybrid graphics system comprised of two card: 1) an integrated Intel GPU (UHD 630) and 2) an NVIDIA Quadro P2000. After various steps and misteps trying to install the NVIDIA drivers, I finally got everything working thanks to [this outstanding guide](https://wiki.archlinux.org/index.php/Dell_XPS_15_9570#Manually_loading/unloading_NVIDIA_module) on the Arch wiki. (Which, in turn, is based on this [this community thread](https://bbs.archlinux.org/viewtopic.php?pid=1826641#p1826641).). Some high level remarks:

- The NVIDIA GPU and drivers only appear to work well on the Xorg session. Wayland performance is still shaky, or not even supported AFAIK.
   - A side problem with this is that scaling on my HiDPI screen is better on Wayland. However, I managed to resolve this &mdash; more or less &mdash; [here.](https://github.com/grantmcdermott/arch-tips#hidpi))
- The way the setup works is that the discrete NVIDIA GPU is switched off by default when the computer boots up to save battery life, etc.
- To turn it on, I just need to run a simple shell script that I've saved to my home directory.

```sh
$ cd ~ ## Just emphasising the location
$ sudo bash enablegpu.sh ## 'sudo bash disablegpu.sh' to turn off. Or, just shut down.
$ nvidia-smi ## Optional: confirm that the NVIDIA card has been enabled
```

As noted above, my working NVIDIA setup only came after various misteps. And the "solution" to these misteps was basically just to keep the discrete GPU switched off at all times. Thankfully, the real solution was easy enough thanks to the Arch wiki guide. Still, for posterity and since I learned a lot from working through these steps, the old changelog is under the fold...

<details>
  <summary>Old NIVIA changelog </summary>
   I initially tried to get CUDA support going by installing the `nvida` package from the Arch repositories... Which turned out to be a mistake! The system would boot up fine, but I was subsequently presented with a blank screen once I got passed the GRUB menu.

   **Solution:** Boot directly into the shell (i.e. TTY) and uninstall the nvidia package: Press "Ctr-Alt-F2" at the grub menu and then hit "e" to edit the selection. Look for the line starting with "linux" and add "3" (without the quotation marks) to the end of that line. F10 to exit and then you will be presented with the shell upon booting up. Enter your username, followed by your password. Finally, uninstall the nvidia package by typing `sudo pacman -Rs nvidia` and
   reboot as normal ("CTR-ALT-DEL").

   **Update:** After some package and system updates, I'm back to the post-login blank screen! Weirdly, starting GDM from TTY1 (see above) works fine, so this is my current workaround. Have left a question on the [Antergos forum](https://forum.antergos.com/topic/11077/blank-screen-after-log-in-nvidia-issue) about this.

   **Update 2:** Added "nouveau.modeset=0" to the [kernel boot parameters](https://wiki.archlinux.org/index.php/Kernel_parameters#GRUB) as per various online suggestions:
   ```sh
   $ sudo nano /etc/default/grub
   ```
   Add "nouveau.modeset=0" to the GRUB\_CMDLINE\_LINUX\_DEFAULT variable. Then CTL+X and "y" to save. Re-generate the grub.cfg file:
   ```sh
   $ sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```

   This solves the log-in and hibernate problem... but only for Xorg. In other words, now my Wayland session(s) have disappeared!

   **Update 3:** Have tried various fixes in the interim, including removing KDE/Plasma entirely in case there were some sytem conflicts with Gnome. I also tried removing the folder `~/.config/gnome-session` as per [this thread](https://bbs.archlinux.org/viewtopic.php?pid=1708172#p1708172). Didn't work. In fact, it turns out that the issue of GDM not being able to recognize Wayland sessions is common. Here are some relevant threads: [1](https://bbs.archlinux.org/viewtopic.php?id=225477), [2](https://www.reddit.com/r/archlinux/comments/823ye9/wayland_with_gnome/), [3](https://www.reddit.com/r/archlinux/comments/89vkwq/gnomegdm_issue_no_wayland_session/), [4](https://www.reddit.com/r/archlinux/comments/9wycf1/wayland_option_gone_from_login_screen_gnomegdm/). Will read through these various threads and try different options.

   **Update 4:** Solved! Thanks to [this](https://www.reddit.com/r/archlinux/comments/9wycf1/wayland_option_gone_from_login_screen_gnomegdm/) suggestion, I needed to enable early KMS start. (Basically, my laptop is too fast for its own good.) The full solution then is to disable modesetting for the Nouveau driver (see **Update 2** above) and enable early KMS start for the integrated Intel GPU, by adding following Intel modules to `/etc/mkinitcpio.conf`:
   ```
   /etc/mkinitcpio.conf
   ---
   MODULES=(intel_agp i915)
   ```

   Once that's done, regenerate initramfs:
   ```sh
   $ sudo mkinitcpio -p linux
   ```

   Reboot and I can now log directly into Gnome Wayland from GDM.

</details>

## Removing Antergos

Following the [resolution of the Antergos Project](https://antergos.com/blog/antergos-linux-project-ends/), I removed all (or, at least, most) of the residual Antergos libraries following [these](https://forum.antergos.com/topic/11878/antefree-gnome) [guides](https://forum.antergos.com/topic/11887/antefree-gnome-cleaning-from-aur). This leaves a pure Arch system.

In related news, [Endeavour OS](https://endeavouros.com/) has picked up where Antergos left off and looks really cool.


## Miscellaneous

### Printing

My home printer had been found automatically at first. However, I couldn't find it after a while for some reason. Adding it manually was a pain, because I didn't have the correct permissions in CUPS. (Adding myself to the "cups" user group didn't work either.) I solved the problem by following [these instructions](https://kernelmastery.com/enable-regular-users-to-add-printers-to-cups/).

### Wi-fi from Shell/TTY

Relevant to cases where you need to log into the Shell/TTY to fix some hanging/freeze problem (e.g. CUDA/NVIDIA above). The easiest way I've found is to use `nmcli` (command line version of network manger). To see the available SSIDs type
```sh
$ nmcli dev wifi
```
Then connect with
```sh
$ nmcli dev wifi connect SSID_NAME password SSID_PASSWORD
```

### Starting Gnome session from Shell/TTY

Similar rationale to the above:
```sh
$ XDG_SESSION_TYPE=wayland dbus-run-session gnome-session
```

Alternatively, launch via GDM:
```sh
$ sudo systemctl start gdm
```

### HiDPI

After installing the NVIDIA drivers and CUDA directly on Arch (see below), I also installed the C
Having successfully connected to my NVIDIA card, I then proceeded to install CUDA directly on my system via the Arch repos. I also installed a version (multiple versions, in fact) through conda, as detailed above.
For R-Julia interoperability, I use the [RCall](http://juliainterop.github.io/RCall.jl/stable/index.html) and [JuliaCall](https://non-contradiction.github.io/JuliaCall/index.html) packages, respectively. However, I ran into a (documented) error related to libstdc.so being outdated (libstdc++6.so is required by Rcpp among other libraries). A [temporary solution ](https://github.com/Non-Contradiction/JuliaCall/issues/1) is to direct Julia to the newer library path with `export` in each new session, but a more permanent solution is to add them to my `/etc/environment` file.
```
# GM: Adding manually for R-Julia interoperability
# See: https://github.com/Non-Contradiction/JuliaCall/issues/110
JLIB=/opt/julias/julia-1.2.0/lib
LD_LIBRARY_PATH=$JLIB
R_LD_LIBRARY_PATH="$(R RHOME)/lib:$JLIB"
```
You need to log out and back in again for the settings to take hold.

The Arch wiki has the goods here. One thing I'll add explicitly here is how to change the default Linux Console font that appears when booting up (or when booting into TTY). First download the terminus fonts family:
```sh
$ pac install terminus-font
```
You can then see the set of available fonts by typing `ls /usr/share/kbd/console`. To temporarily test out a larger font, type
```sh
$ setfont ter-132n
```
(Type `showconsolefont` if you want to see a table of the font's glyphs and letters).

To set this font permanently, open `/etc/vconsole.conf` with nano and add
```
FONT=ter-132n
```

### Manually compile a package from source with edited PKGBUILD (Julia example)

I initally installed [Julia](https://julialang.org/) via the Arch community repos. However, a build problem cropped up when upgrading to Julia 1.2.0; see [bug report](https://bugs.archlinux.org/task/63536?project=5&string=julia) and [GitHub issue](https://github.com/JuliaLang/julia/issues/33038). I ultimately took this as a sign to install the official Julia binaries instead. (See [above](#julia).) However, this also seemed like a good chance to practice compiling a package from the community repos with an edited PKGBUILD. There's suprisingly little guidance about this online, although [this Manjaro forum thread](https://forum.manjaro.org/t/how-to-install-pkgbuild-file-downloaded-manually/69365/2) proved very helpful. My full steps as follows:

1. Create some folder to download the relevant files to.  I used `~/julia` but I don't think the location really matters.
```sh
$ mkdir ~/julia
$ cd ~/julia
```
2. Download/create the [PKBUILD and ancilliary files](https://git.archlinux.org/svntogit/community.git/tree/trunk?h=packages/julia) to this location. There may be a smart way to to this automatically, but I just created the files manually using nano+copy+paste.
3. Edit the PKGBUILD (and any other files) as needed.
4. Build the package. This will take a while. Note: Using regular `$ makepkg -si` gave me errors. So instead I used:
```sh
$ makepkg -g >> PKGBUILD
#$ makepkg ## Gives GPG verification error. Fix that below first:
$ gpg --recv-key 66E3C7DC03D6E495 ## Add GPG key
$ makepkg ## NB: Don't use sudo!
```
5. Install the **pkg.tar** file. Be careful not to confuse with any of the other .tar files lying around in the same directory.
```sh
$ sudo pacman -U julia-2:1.2.0-1-x86_64.pkg.tar
```
6. The updated version of Julia was now ready to go:
```sh
$ julia
```

### Touchpad

_Note: Only relevant for the Plasma DE, which I'm no longer using._ The KDE graphical touchpad settings (*System Settings > Input Devices > Touchpad*) didn't seem to last and kept reverting back to the default behaviour. So I [installed](https://wiki.archlinux.org/index.php/Libinput#Installation) `libinput` and then followed the final section of [this guide](https://www.dell.com/support/article/us/en/04/sln308258/precision-xps-ubuntu-general-touchpad-mouse-issue-fix?lang=en) (See Fig. 7) to get tapping, right-click two finger tap, etc. working.
