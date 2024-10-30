# Proxmox-VE
## После установки:

```
#--- URL
https://tteck.github.io/Proxmox/#proxmox-ve-post-install

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

```
#--- URL
https://tteck.github.io/Proxmox/#proxmox-ve-processor-microcode

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/microcode.sh)"
```

## Подготовка проброса Intel GPU


```
#--- Update the sources list:
nano /etc/apt/sources.list

#--- Add the following lines:
# non-free firmwares
deb http://deb.debian.org/debian bookworm non-free-firmware

# non-free drivers and components
deb http://deb.debian.org/debian bookworm non-free

#--- Install the drivers:
apt update && apt install intel-media-va-driver-non-free intel-gpu-tools vainfo