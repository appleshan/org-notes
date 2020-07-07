# General recommendations to maintain your Arch Linux system

## Check new keyring

`sudo pacman-key --populate archlinux`

## Broken symlinks

Old, broken symbolic links might be sitting around your system; you should remove them. Examples on achieving this can be found here and here.

To quickly list all the broken symlinks of your system, use:

```bash
sudo find /etc/alternatives -xtype l -exec rm {} +    # With Debian 9 Stretch
sudo find / -not -path '/proc/*' -not -path '/run/*' -xtype l -print 2>/dev/null
```

Then inspect and remove unnecessary entries from this list.

## Removing unused packages (orphans)

For recursively removing orphans and their configuration files:

```bash
sudo pacman -Rns $(pacman -Qtdq)
```

If no orphans were found, pacman errors with error: no targets specified. This is expected as no arguments were passed to `pacman -Rns`.
