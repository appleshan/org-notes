# Make your Arch fonts beautiful easily!

Read more in [Make your Arch fonts beautiful easily!](https://www.reddit.com/r/archlinux/comments/5r5ep8/make_your_arch_fonts_beautiful_easily/)

## Install some fonts, for example:

Install the fonts you will need: `sudo pacman -S ttf-dejavu noto-fonts-cjk noto-fonts`

If you want to install Emoji: `sudo pacman -S noto-fonts-emoji`

Or substitue with **Windows 10 fonts** by `copy` from a Windows machine or:
```bash
# Arch Linux
pacaur -S ttf-ms-fonts
# Debian
sudo apt install ttf-mscorefonts-installer
```

## Enable font presets by creating symbolic links:

On Arch Linux:

```bash
sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
sudo ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
sudo ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```

On Debian 9:

```bash
sudo ln -s /usr/share/fontconfig/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
sudo ln -s /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
sudo ln -s /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```

## Delete xfonts-75dpi and xfonts-100dpi

```bash
apt-get autoremove xfonts-75dpi xfonts-100dpi
```

## Uninstall freefonts and delete some man page in `/usr/share/man`

On Arch: `sudo pacman -Rns ttf-freefont`

On Debian: `sudo apt autoremove ttf-freefont fonts-freefont-ttf`

## Check font

`fc-match -s monospace` or `fc-list 'dejavu'` and delete any **pfc** font

## Update font cache

Use `fc-cache -fv`, if not working, `reboot` the system instead.

## Run this to check common fonts

```bash
for family in serif sans-serif monospace Arial Helvetica Verdana "Times New Roman" "Courier New"; do
  echo -n "$family: "
  fc-match "$family"
done
```

## On Debian 9 with sublime text 3

### Applications without fontconfig support

Some apps like URxvt and Emacs will ignore fontconfig settings. Using **~/.Xresoures**
```bash
Xft.antialias:  1
Xft.autohint:   0
Xft.dpi:        96
Xft.hinting:    0
Xft.hintstyle:  hintnone
Xft.lcdfilter:  lcddefault
Xft.rgba:       rgb
```

Make sure the settings are loaded properly when X starts with `xrdb -q`.

### In sublime text

In settings: "font_face": "Dejavu Sans Mono", "Consolas"

Remove QT config file `~/.config/Trolltech.conf` and add these line to it:

```bash
[Qt]
style=GTK+
```

> For font consistency, all applications should be set to use the serif, sans-serif,
> and monospace aliases, which are mapped to particular fonts by fontconfig.

Create [`~/.fonts.conf`](../config.d/fonts.conf) (or `/etc/fonts/local.conf`, or even add directly in `50-user.conf`)
