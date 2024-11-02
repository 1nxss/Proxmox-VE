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
```
### GRUB Parameters Configuration: 
Before creating VM in Proxmox VE, we should configure kernel and boot parameters and variables.

1. First we change GRUB, GRand Unified Bootloader, parameters. It is a popular bootloader for Linux based systems and used for initializing OS and kernel — OS connectivity purposes.

2. Proceed to the Proxmox Host machine’s shell:

3. Enter this command `nano /etc/default/grub`
```
##--- Change this line starting with GRUB_CMDLINE_LINUX_DEFAULT to this; (if CPU is)
##--- Intel : 
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"

##---AMD : GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```
4. Enter `update-grub` command in the shell

### Add vfio modules, using nano :

``nano /etc/modules``

```
##--- Add the following modules :
vfio
vfio_pci
vfio_virqfd
vfio_iommu_type1
# Modules required for Intel GVT
kvmgt
exngt
Vfio-mdev
```
Update config :

``update-initramfs -u -k all``