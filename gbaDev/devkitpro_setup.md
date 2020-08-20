# Setting Up DevKitPro on Ubuntu



https://devkitpro.org/



Steps:
1. Download devkitpro's custom package manager from https://github.com/devkitPro/pacman/releases/
1. Update the available packages for DevKitPro's package manager. On the command line type `sudo dkp-pacman -Sy`
1. To install all packages required for gba-dev `sudo dkp-pacman -S gba-dev`. Press enter when prompted which packages you want to install all
1. Set the DEVKITARM env var using `sudo nano ~/.profile`

```
# devkitpro
export DEVKITPRO=/opt/devkitpro
export DEVKITARM=/opt/devkitpro/devkitARM
export DEVKITPPC=/opt/devkitpro/devkitPPC
```

1. Reload env vars from profile `. ~/.profile`
1. Install Visual Boy Advance `sudo apt-get install visualboyadvance`


Debian Packages

https://github.com/devkitPro/pacman/releases/tag/v1.0.2


Docker Images

https://hub.docker.com/u/devkitpro