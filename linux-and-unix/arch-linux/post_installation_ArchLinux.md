# Post installation note for Arch Linux and Debian Stretch (9)

https://wiki.archlinux.org/index.php/Improving_performance#Watchdogs

## avahi-daemon[519]: chroot.c: open() failed: No such file or directory

**References**:

- https://bugzilla.redhat.com/show_bug.cgi?id=1356304#c23 (not work)

## Disable ipv6

> Ref https://wiki.archlinux.org/index.php/IPv6#Disable_functionality)

Adding to `/etc/sysctl.conf` file:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

If disabling IPv6 by sysctl,
you should comment out the IPv6 hosts in `/etc/hosts`:
```
#<ip-address> <hostname.domain.org> <hostname>
127.0.0.1 localhost.localdomain localhost
#::1 localhost.localdomain localhost
```

Otherwise there could be some connection errors because hosts are resolved to
their IPv6 address which is not reachable.

And https://github.com/zidarko/scrolls/wiki/Exim4-Port-problem

## Parallel downloading

```bash
apt install aria2c
```

## No hardware encryption ath9k

```bash
echo "options ath9k nohwcrypt=1" | sudo tee /etc/modprobe.d/ath9k.conf
```

https://askubuntu.com/questions/673156/atheros-ar9485-wifi-disconnects-randomly

## Blank screen after lock/sleep

**To debug**:
- Install `accountsservice` and `xserver-xephyr`, where:

  + `accountsservice` for *Enhanced user accounts handling*
  + `xserver-xephyr` for *LightDM test mode*

Then run LightDM as an X application for debugging: `$ lightdm --test-mode --debug`.

- LightDM's log file is `/var/log/lightdm/lightdm.log`, you will need root privilege to see it.
- Or see output of `dmesg` with `sudo dmesg`.

**List of workaround methods**:
- Try to suspend and resume again

  Press <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F1</kbd>,
  then login and type `systemctl suspend`.
  Press power button to resume and switch to `tty7` by <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F7</kbd>.
  In my case, I did 2 times to escape the blank screen.

- Use DRI2 instead of DRI3 may solve the problem: https://wiki.archlinux.org/index.php/intel_graphics#DRI3_issues
- Disable `at-spi-dbus-bus.desktop` (may NOT work)

  ```bash
  sudo mv -v -i /etc/xdg/autostart/at-spi-dbus-bus.desktop /etc/xdg/autostart/at-spi-dbus-bus.desktop.disabled
  ```

- Replace `light-locker` with `xscreensaver`, then reboot.

**References**:
- https://wiki.archlinux.org/index.php/LightDM#Testing
- https://bbs.archlinux.org/viewtopic.php?pid=1671675#p1671675
- https://bugs.launchpad.net/ubuntu/+source/light-locker/+bug/1320989 with comment #17
- https://bugs.launchpad.net/ubuntu/+source/unity/+bug/1617471 with comment #25

## Reduce writing to SSD

```bash
$ cat /etc/fstab
# <device>             <dir>         <type>    <options>             <dump> <fsck>
/dev/sda1              /             ext4      defaults,noatime      0      1
/dev/sda2              none          swap      defaults              0      0
/dev/sda3              /home         ext4      defaults,noatime      0      2
```

## Enable user list LightDM

By default, `LightDM` is configured so that the user should enter login name
and password. Login name is considered sensitive information.

To enable user list, place the following settings into `/usr/share/lightdm/lightdm.conf.d/01_my.conf`:

```bash
[Seat:*]
greeter-hide-users=false
```

## Extracting fonts from a Windows ISO

The fonts can also be found in a Windows ISO file.

Extract the `sources/install.esd` or the `sources/install.wim` file in the ISO and
look for a `Windows/Fonts` directory within this file.
It can be extracted with p7zip:

```bash
sudo apt install libxml2-utils p7zip-full
7z e Win10_1809_English_x64.iso sources/install.wim
7z e install.wim '[1].xml'
xmllint --format '[1].xml' | grep -E 'INDEX|EDITIONID|<NAME>|DESCRIPTION|DISPLAYNAME|DISPLAYDESCRIPTION'
7z e install.wim 10/Windows/{Fonts/"*".{ttf,ttc},System32/Licenses/neutral/"*"/"*"/license.rtf} -ofonts/
```

## Polkit requesting root password to suspend

### Using Polkit

- If PolKit version >= 0.106

  You can check version of Polkit by: `pkaction --version`

  If PolKit version < 0.106,
  there are **NO** `.rules` files but only old `.pkla` and `.conf` files.
  Because Polkit versions < 0.106 doesn't have the Javascript interpreter.

  Just adding a file `/etc/polkit-1/rules.d/85-suspend.rules` with:

  ```js
  polkit.addRule(function(action, subject) {
      if (action.id == "org.freedesktop.login1.suspend" &&
          subject.isInGroup("users")) {
          return polkit.Result.YES;
      }
  });
  ```

  And:

  ```bash
  sudo chmod 755 /etc/polkit-1/rules.d
  sudo chmod 644 /etc/polkit-1/rules.d/85-suspend.rules
  ```

- If PolKit version < 0.106

  In this case, adding a file `/var/lib/polkit-1/localauthority/50-local.d/50-enable-suspend-on-lockscreen.pkla` with:

  ```bash
  [Allow suspending in lockscreen]
  Identity=unix-group:users;unix-user:user
  Action=org.freedesktop.login1.suspend
  ResultAny=yes
  ResultInactive=yes
  ResultActive=yes
  ```

  Replace `user` in `unix-user:user` with your own username.

  And: `sudo chmod 644 /var/lib/polkit-1/localauthority/50-local.d/50-enable-suspend-on-lockscreen.pkla`

Read [More about pklocalauthority][freedesktop_pklocalauthority]

[freedesktop_pklocalauthority]: https://www.freedesktop.org/software/polkit/docs/0.105/pklocalauthority.8.html

### Using Power Manager settings (not sure if it works)

In XFCE Power Manager under the `Security` tab set: `Automatically lock the session` to *Never*

Check `Lock the screen when the system is going for sleep`

Under the `Display` tab I still blank the screen after 15 minutes. Put to *Sleep*
and *Switch off* times appear to be disabled (greyed out).

Under the `System` tab I still have system sleep mode going to *Suspend* after an hour.

### Else edit policy file

In `usr/share/polkit-1/actions/org.freedesktop.login1.policy`, near line:

```xml
<action id="org.freedesktop.login1.suspend">
```

check these (but `before` that make a backup file)

```xml
<defaults>
<allow_any>yes</allow_any>
<allow_inactive>yes</allow_inactive>
<allow_active>yes</allow_active>
</defaults>
```

**References:**

+ [stintel's blog][tintel_polkit]
+ [bugs.launchpad.net][launchpad_polkit]

[tintel_polkit]: https://stijn.tintel.eu/blog/2015/09/11/polkit-requesting-root-password-to-suspend-after-updating-version-0112-to-0113
[launchpad_polkit]: https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1605189/comments/7

## Change default UMASK in `/etc/login.defs`

Both Debian and Ubuntu ship with **pam_umask**.
This allows you to configure umask in `/etc/login.defs` and have them apply system-wide,
regardless of how a user logs in.

To **enable** it, add a line to `/etc/pam.d/common-session`

```bash
# enable configure umask in /etc/login.defs
session optional pam_umask.so
```

or it may already be enabled. Then edit `/etc/login.defs` and change the `UMASK` line to

```bash
UMASK           077
```

, which means new file will be set to `rw-------` (the default is 022).

Note that users may still override umask in their own `~/.profile` or `~/.bashrc` or similar,
but (at least on new Debian and Ubuntu installations) there shouldn't be any overriding of
umask in `/etc/profile` or `/etc/bash.bashrc`. (If there are, just remove them.)

In Arch Linux, change it in `/etc/profile` or in the default shell configuration
files, e.g. `/etc/bash.bashrc`

## Adjust grub time

In `/etc/default/grub` change **GRUB_TIMEOUT** from 5 to `2`

Update grub by run:

```bash
# on Debian
sudo update-grub
# on Arch Linux
sudo grub-mkconfig -o /boot/grub/grub.cfg
# on Fedora 27 BIOS
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# Fedora 27 UEFI
grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

## Additional completion definitions for Zsh

https://github.com/zsh-users/zsh-completions

## Install compton to avoid screen tearing

See more: [Use compton for a tear free in Xfce][manjaro_compton]
and [How to switch to compton for beautiful tear free compositing in XFCE][duncan_xfce]

[Manpage][man_compton]

[man_compton]: https://github.com/chjj/compton/blob/master/man/compton.1.asciido
[manjaro_compton]: https://wiki.manjaro.org/index.php?title=Using_Compton_for_a_tear-free_experience_in_Xfce
[duncan_xfce]: http://duncanlock.net/blog/2013/06/07/how-to-switch-to-compton-for-beautiful-tear-free-compositing-in-xfce/

**Note**: Start `compton` with `compton --daemon` to fork it in the background

## Password protection of GRUB menu (AND won't let people boot)

If you want to secure GRUB so it is not possible for anyone to change boot
parameters or use the command line, use `grub-mkpasswd-pbkdf2`, type
password you want twice, it will output something like:

```bash
Enter password:
Reenter password:
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.C8ABD3E...
```

Then, add the following to `/etc/grub.d/40_custom` (see more at [Password protection of GRUB edit and console options only][arch_pass_grub]):

```bash
set superusers="username"
password_pbkdf2 <username> <password>
```

where **`<password>`** is the string generated by `grub-mkpasswd-pbkdf2`.

If you want people still boot (see more at [Secure the grub boot loader][daniel_secure_grub]),
the `CLASS` variable in the beginning of `/etc/grub.d/10_linux` can be modified.

```bash
CLASS="--class gnu-linux --class gnu --class os --unrestricted"
```

Make a backup of this file `/etc/grub.d/10_linux` as it will be overwritten by grub updates.
Remember to make these backup file no eXcutable with `chmod 644`.

Finally, update grub by `update-grub`.

### If you are using Arch Linux

Create `/etc/pacman.d/hooks/grub.hook` and add this:

```bash
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = grub
[Action]
Description = Unrestricting GRUB's linux boot entries...
Depends = sed
When = PostTransaction
Exec = /usr/bin/sed -i -e 's/--class os/--class os --unrestricted/g' /etc/grub.d/10_linux
```

[arch_pass_grub]: https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks#Password_protection_of_GRUB_edit_and_console_options_only
[daniel_secure_grub]: https://daniel-lange.com/archives/75-Securing-the-grub-boot-loader.html

## Disable hibernation for [SSD](https://wiki.debian.org/Suspend)

`sudo systemctl mask hibernate.target hybrid-sleep.target`

## Keep application state in RAM

Some older kernels making machine become unresponsive when dealing with slower storage, such as USB drives or SD cards:

```bash
sudo tee -a /etc/sysctl.d/99-sysctl.conf <<-EOF
vm.swappiness=1
vm.vfs_cache_pressure=50
vm.dirty_background_bytes=16777216
vm.dirty_bytes=50331648
EOF
```

---

## For Debian

## Before you start install any packages

`sudo apt install curl wget apt-transport-https dirmngr`

## Disable downloading translations

Create a file named `/etc/apt/apt.conf.d/99translations` and put the following in it:

```bash
Acquire::Languages "none"; # no newline
```

Remove any existing **i18n** files from `/var/lib/apt/lists/`, list and choose
what to delete: `ls *i18n*`

Type: `man apt.conf` for more info.

## Install driver HD 8790M and avoid over heating

```bash
apt install firmware-linux-nonfree firmware-amd-graphics
apt install linux-cpupower cpufrequtils
```

---

## For Arch Linux

## How to manually install AUR package
```bash
mkdir ~/.aur/<pkg_name>
git clone https://aur.archlinux.org/<pkg_name>.git
cd <pkg_name>
makepkg -sr PKGBUILD
pacman -U $pkg.tar.gz
```

## How to use zip to create archive
`zip <file.zip> -0 <file list>`

## Decompile python from .pyc
`pip2 install --user uncompyle6`

## Update man page and whatis database after installing new package
```
mandb
```

## Install eigen lib for matrix operations OR install python2-numpy
```bash
pacman -Ss eigen # for C++
```

## Install necessary package
```bash
ibus-unikey audacious jre7-openjdk pngcheck binwalk spek
tcpdump gvfs checksec foremost
gst-plugins-base gst-plugins-base-libs gst-plugins-good
gst-plugins-bad gst-plugins-ugly gst-libav gstreamer-vaapi
```

## Install pwntools
```bash
pacaur -S python2-pwntools
python2
>>> from pwn import *
```

## Install peda
```bash
pacman -S peda
ln -s /usr/share/peda/peda.py ~/
echo 'source ~/peda.py' >> ~/.gdbinit
```
