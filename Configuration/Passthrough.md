# Proxmox-VE
## Инструкция по пробросу GPU

```
#--- Update the sources list:
nano /etc/apt/sources.list

#--- Add the following lines:
# non-free firmwares
deb http://deb.debian.org/debian bookworm non-free-firmware

# non-free drivers and components
deb http://deb.debian.org/debian bookworm non-free

#--- Install the drivers if IGPU:
apt update && apt install intel-media-va-driver-non-free intel-gpu-tools vainfo
```
### GRUB Parameters Configuration:
Before creating VM in Proxmox VE, we should configure kernel and boot parameters and variables.

1. First we change GRUB, GRand Unified Bootloader, parameters. It is a popular bootloader for Linux based systems and used for initializing OS and kernel — OS connectivity purposes.

2. Proceed to the Proxmox Host machine’s shell:

3. Enter this command `nano /etc/default/grub`
```
#--- Change this line starting with GRUB_CMDLINE_LINUX_DEFAULT to this; (if CPU is)
#--- Intel : 
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"

#---AMD : 
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```
4. Enter `update-grub` command in the shell

#### IOMMU Verification:
Enter the following command in the shell of your Proxmox system, so that you can see whether IOMMU is enabled or not:

```
dmesg | grep -e DMAR -e IOMMU
```
Description: Result of this command will be quite long, but everything is super okey as long as you see these followings inside of that text
```
[    0.020742] DMAR: IOMMU enabled
[    1.219719] AMD-Vi: Found IOMMU at 0000:00:00.2 cap 0x40
```

### Virtual Function I/O (VFIO) Configuration
1. Add vfio modules, using nano `nano /etc/modules`

```
#--- Add the following modules :
vfio
vfio_pci
vfio_virqfd
vfio_iommu_type1
#--- Modules required for Intel GVT |deprecated in new version of proxmox?
#kvmgt
#exngt
#Vfio-mdev
```
2. IOMMU Commands: Run the following commands in the shell (it basically add the quoted values into the *.conf files):
```
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```
3. Blacklisting the drivers: Enter following commands into the shell, which blacklist the drivers based on their vendor types.
```
#--- AMD GPUs
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
#--- NVIDIA GPUs
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidia*" >> /etc/modprobe.d/blacklist.conf 
#--- Intel GPUs
echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf
```
4. Update config :

``update-initramfs -u -k all``
5. GPU to the VFIO Settings: In this part, you will find several IDs of your GPU then configure those to the VFIO.

Use `lspci -nn` or `lspci -v` to find VGA adapter. It will be similar to this:
```
02:00.0 VGA compatible controller: NVIDIA Corporation GP106 [GeForce GTX 3060] (rev a1) (prog-if 00 [VGA controller])
02:00.1 Audio device: NVIDIA Corporation GP106 High Definition Audio Controller (rev a1)
00:02.0 VGA compatible controller [0300]: Intel Corporation Iris Plus Graphics 650 [8086:5927] (rev 06)
```
After noting the initial decimals, you will type `lspci -n -s XX:XX` from previous example my initial decimals were 02:00.0 and 02:000.1, we will use the first two decimal. This command will give you output like this:
```
lspci -n -s 00:02
00:02.0 0300: 8086:5927 (rev 06)
lspci -n -s 02:00
01:00.0 0000: 10de:1b81 (rev a1)
01:00.1 0000: 10de:10f0 (rev a1)
```
Take the vendor ID of both output, and then add these GPU Vendor ID to the VFIO.
```
# NVIDIA
echo "options vfio-pci ids=10de:13c2, 10de:0fbb" > /etc/modprobe.d/vfio.conf
# Intel
echo "options vfio-pci ids=8006:5927 disable_vga=1" > /etc/modprobe.d/vfio.conf
```
