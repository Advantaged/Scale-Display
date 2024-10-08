# Scale-Display
Set physical-DPI system-wide
## Get-monitor-Dimensions-n-scale-appearances.md

### 1. Recognize all aspect of your GPU & Monitor

* **Merit to:** [Ask Ubuntu](https://askubuntu.com/questions/736113/how-can-i-get-my-laptops-monitor-size)

*   To get dimension of your display:

    * First line is the command, second line is the output

```
    xrandr | grep ' connected'

    DisplayPort-2 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 607mm x 345mm
```

*   To get your bus IDs (in hexadecimal):


    * First line is the command, second line is the output

```
    lspci | grep -e VGA -e 3D

    43:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 XL/XT [Radeon RX Vega 56/64] (rev c1)
```

* To get complete Screen-Resolution, **merit to:** [Vinareo](https://winaero.com/find-change-screen-dpi-linux/)


    * First line is the command, other lines are the output

```
    xdpyinfo | grep -B 2 resolution

    screen #0:
      dimensions:    3840x2160 pixels (1016x571 millimeters)
      resolution:    96x96 dots per inch

```

### 2. Install prerequisite

```
paru -S --needed --noconfirm xsettingsd edid-decode-git

```
* Collect additional information through EDID **merit to** [AdamDesk](https://www.adamsdesk.com/posts/learn-to-read-edid-displayid-metadata-using-linux/):

```
    xrandr --prop | edid-decode

```

* From monitor-manufacturer we know now how big is our monitor in reality:

    `Maximum image size: 61 cm x 35 cm`

* Real-Monitor-Dimensions & DPI:

    * Set your on-board-calculator to scientific and excecute the calculation.

    * **Very important Note:** The hardware-pixels are not homogenouse, that mean... one of the given dimension is wrong, the whole trick is... don't excede the maximum values given by manufaturer. In fact if i take (in mycase) the height 350mm divide by 9 & multiply by 16. i get 622.22 mm that the monitor (following the manufacturer) cannot disply. But, if i take the width as reference (610/16*9)... i get 343,125 mm that the monitor can display. i wasted a lot of time trying to find the right Dpi for my LCS-Display, don't make the same misstake!

    * 16:9 ➡️ 610x343 mm
> 610/25,4=/3840=1/x --> 160 dpi (approx./ca./rounded up) 
>
> 343/25,4=/2160=1/x --> 160 dpi (approx./ca./rounded up)



### 3. Settings

* **Note:** DON'T make all changes at ones, set e.g. first the `xorg` with the right dimensions of you monitor/disply & reboot. In the most of the cases, are the results always perfect or satifying for daily use.

1. Settings "Xorg"

* Create this file: `/etc/X11/xorg.conf.d/90-monitor.conf` and populate the file as follow:

```
# Will set your DPI to 160x160 if the screen is 2160p & ca. 28 inches diagonal
Section "Monitor"
    Identifier   "<default monitor>"
    DisplaySize  610 343    # In millimeters
EndSection

```


2. Settings "Sddm"

* KDE/Plasma haven't any problem to set up in `systemsettings` and hold or distribute the settings to all application. Following [GrimBandito](https://bbs.archlinux.org/viewtopic.php?id=220525#:~:text=17%3A45%3A20-,GrimBandito,-Member) [in this thread](https://bbs.archlinux.org/viewtopic.php?id=220525)...:

> A better solution would be to ensure Xorg is configured correctly, specifically in the monitor section you need to make sure either your dpi is specified there or that the correct dimensions (mm) of your monitor are specified. Xorg can then correctly calculate your DPI.

* **Ergo:** With step "1." above we solve also the problem having to small fonts & no scaling by booting with "SDDM" 🟰 Problem solved ❗️

* Two things can be done here, first: "imput handling" = export `~/.config/kcminputrc` to get right 'mouse-acceleration' & 'cursor-size', second: "monitor & display handling" = export `~/.config/kwinoutputconfig.json` to get 'resolution, scaling, frequenz...' like in the [Arch-Wiki](https://wiki.archlinux.org/title/SDDM) explained.
    * Assure first you have admin-/root-rights with `sudo su` or `su`, than execute following two commands to...

> To enable correct input handling in SDDM (tap-to-click, touchscreen mapping,...), you can copy the appropriate configuration file from your home directory to the one of SDDM:

```
    cp ~/.config/kcminputrc /var/lib/sddm/.config/

    chown sddm:sddm /var/lib/sddm/.config/kcminputrc

```

* **Note:** Also after you executed the step above reboot & look the results in the login-manager "SDDM", if everything is good... stop modifyng further things.

> To enable correct display & monitor handling in SDDM (scaling, monitor resolution, hz,...), you can copy or modify the appropriate configuration file from your home directory to the one of SDDM:

```
    cp ~/.config/kwinoutputconfig.json /var/lib/sddm/.config/

    chown sddm:sddm /var/lib/sddm/.config/kwinoutputconfig.json

```

* **Note:** Leave/escape "admin-mode" in terminal (`konsole`) with [CTRL+D] 😉

* **Numlock=on** For the people thy "need"...:
    * Edit file `/etc/sddm.conf.d/kde_settings.conf` & add under `[General]` one line with `Numlock=on` like shown below:

```
[General]
HaltCommand=/usr/bin/systemctl poweroff
RebootCommand=/usr/bin/systemctl reboot
Numlock=on

```


3. Settings "Gtk"

* If you are not satify whith the "job" of `xorg`, you can modify further your config.
* Here we use the installed app `xsettingsd`, to create the base of configuration, use command:

    `xsettingsd -c` to create/make the standard config-file:

    `~/.config/xsettingsd.conf`

* All i have changed here are the following two things:

    `Gdk/UnscaledDPI 160`  
    
    `Gdk/WindowScalingFactor 2`

    * here the complete file of ArcolinuxD-Plasma

```
Gtk/ToolbarStyle 2
Net/ThemeName "Breeze"
Gtk/CursorThemeName "breeze_cursors"
Net/IconThemeName "breeze"
Gtk/CursorThemeSize 36
Gtk/FontName "Noto Sans,  11"
Gdk/UnscaledDPI 160
Gdk/WindowScalingFactor 2
Xft/Hinting 1
Xft/HintStyle "hintsfull"
Xft/Antialias 1
Xft/RGBA "rgb"

```

* i have found also the following settings, but, first: the Gtk apps are scaled properly through `xorg` alone & second: honestly i don't know where to set those "options".
    * Presumably those "options" going in to `~/.bashrc` & in to `/etc/bash.bashrc`



```
export GDK_SCALE=2
export GDK_DPI_SCALE=0.8

```

* **Note/Explanation:** Here is the global scale ("Applicationstyle" & "Window Decoration") `2` = 200(%), the Dpi is `160` ➡️ the Dpi-scale is 🟰 [160/200] = 0,8 🟰 in computer-language `0.8`   .
    * Fractional "global-scaling" is in "Gtk" not possible, hence use integer like `1`, `2` or `3` etc..
    * Calucalate your `GDK_DPI_SCALE` according to your own global scale `GDK_SCALE` & round the result up. e.g. [125/200] = 0,625, here use either `0.6` or `0.7`   .

4. Settings "Wayland"

* In case you want use or use Wayland, i discourage to make any settings of "global-scaling" or "Dpi" in "SDDM/Wayland", as i made some settings (in the past) under "SDDM/Wayland" (with my Dpi)... everything was scaled double. I don't know how works Wayland & don't know  where Wayland take the numbers to scale "Gtk" properly.
* As i understood, the majority use `xwayland`. This mean... Wayland take the information from `xorg` = a middle way between `wayland` & `xorg`.


✅ **Done & Enjoy**❗️



