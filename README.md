# arch-win11-guide
Steps to setup an dualboot Arch/Win11 environment 

### Info sources:
- https://wiki.archlinux.org/

### Important information before installation (!!!)
#### Windows UEFI vs BIOS Limitations

Microsoft imposes limitations on which firmware boot mode and partitioning style can be supported based on the version of Windows used:

- **Windows 8/8.1** and **10** **x86_64** versions support booting in x86_64 UEFI mode from GPT disk only, OR in BIOS mode from MBR disk only. They do not support IA32 UEFI boot, x86_64 UEFI boot from MBR disk, or BIOS boot from GPT disk.
- **Windows 11** only supports **x86_64** and a boot in UEFI mode from GPT disk.

##### Bootloader UEFI

[Windows Setup](https://en.wikipedia.org/wiki/Windows_Setup "wikipedia:Windows Setup") creates a 100 MiB [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition "EFI system partition") (except for [Advanced Format](https://wiki.archlinux.org/title/Advanced_Format "Advanced Format") 4K native drives where it creates a 300 MiB ESP), so multiple [kernel](https://wiki.archlinux.org/title/Kernel "Kernel") usage is limited. Workarounds include:

- Mount ESP to `/efi` and use a [boot loader](https://wiki.archlinux.org/title/Boot_loader "Boot loader") that has file system drivers and is capable of launching kernels that reside on other partitions.
- Expand the EFI system partition, typically either by decreasing the Recovery partition size or moving the Windows partition (**UUIDs will change**).
- Backup and delete unneeded language directories in `_esp_/EFI/Microsoft/Boot/` (e.g. to only keep `en-US`).
- Use a higher, but slower, [compression for the initramfs images](https://wiki.archlinux.org/title/Mkinitcpio#COMPRESSION "Mkinitcpio"). E.g. `COMPRESSION="xz"` with `COMPRESSION_OPTIONS=(-9e)`.
- **DEACTIVATE UEFI SECURE BOOT!!!**

##### UEFI systems
Only if you already have Win 10 or Win 11, it will already have created some partitions on a GPT-formatted disk:

If you already have Windows installed, it will already have created some partitions on a [GPT](https://wiki.archlinux.org/title/GPT "GPT")-formatted disk:

- a [Windows Recovery Environment](https://en.wikipedia.org/wiki/Windows_Recovery_Environment "wikipedia:Windows Recovery Environment") partition, generally of size 499 MiB, containing the files required to boot Windows (i.e. the equivalent of Linux's `/boot`),
- an [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition "EFI system partition") with a [FAT32](https://wiki.archlinux.org/title/FAT32 "FAT32") filesystem,
- a [Microsoft Reserved Partition](https://en.wikipedia.org/wiki/Microsoft_Reserved_Partition "wikipedia:Microsoft Reserved Partition"), generally of size 128 MiB,
- a Microsoft basic data partition with a NTFS filesystem, which corresponds to `C:`,
- potentially system recovery and backup partitions and/or secondary data partitions (corresponding often to `D:` and above).

Using the Disk Management utility in Windows, check how the partitions are labelled and which type gets reported. This will help you understand which partitions are essential to Windows, and which others you might repurpose. The Windows Disk Management utility can also be used to shrink Windows (NTFS) partitions to free up disk space for additional partitions for Linux.

> Tip:
> - [rEFInd](https://wiki.archlinux.org/title/REFInd "REFInd") and [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot "Systemd-boot") will autodetect _Windows Boot Manager_ (`\EFI\Microsoft\Boot\bootmgfw.efi`) and show it in their boot menu automatically. For [GRUB](https://wiki.archlinux.org/title/GRUB "GRUB") follow either [GRUB#Windows installed in UEFI/GPT mode](https://wiki.archlinux.org/title/GRUB#Windows_installed_in_UEFI/GPT_mode "GRUB") to add boot menu entry manually or [GRUB#Detecting other operating systems](https://wiki.archlinux.org/title/GRUB#Detecting_other_operating_systems "GRUB") for a generated configuration file.
- To save space on the EFI system partition, especially for multiple kernels, [increase the initramfs compression](https://wiki.archlinux.org/title/Mkinitcpio#COMPRESSION "Mkinitcpio").
- Computers that come with newer versions of Windows often have [Secure Boot](https://wiki.archlinux.org/title/Secure_Boot "Secure Boot") enabled. You will need to take extra steps to either disable Secure Boot or to make your installation media compatible with secure boot (see above and in the linked page).

##### UEFI firmware
Windows will use the already existing [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition "EFI system partition"). In contrast to what was [stated earlier](https://wiki.archlinux.org/title/Dual_boot_with_Windows#UEFI_systems), it is unclear if a single partition for Windows, without the Windows Recovery Environment and without Microsoft Reserved Partition, will not do.

Follows an outline, assuming [Secure Boot](https://wiki.archlinux.org/title/Secure_Boot "Secure Boot") is disabled in the firmware.

1. Boot into windows installation. Watch to let it use only the intended partition, but otherwise let it do its work as if there is no Linux installation.
2. Follow the [#Fast Startup and hibernation](https://wiki.archlinux.org/title/Dual_boot_with_Windows#Fast_Startup_and_hibernation) section.
3. Fix the ability to load Linux at start up, perhaps by following [#Cannot boot Linux after installing Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows#Cannot_boot_Linux_after_installing_Windows). It was already mentioned in [#UEFI systems](https://wiki.archlinux.org/title/Dual_boot_with_Windows#UEFI_systems) that some Linux boot managers will autodetect _Windows Boot Manager_. Even though newer Windows installations have an advanced restart option, from which you can boot into Linux, it is advised to have other means to boot into Linux, such as an arch installation media or a live CD.

##### Windows 10 with GRUB
The following assumes [GRUB](https://wiki.archlinux.org/title/GRUB "GRUB") is used as a boot loader (although the process is likely similar for other boot loaders) and that Windows 10 will be installed on a GPT block device with an existing EFI system partition (see the "System partition" section in the [Microsoft documentation](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions) for more information).

Create with program `gdisk` on the block device the following three new partitions. See [[5]](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions) for more precise partition sizes.

|Min size|Code|Name|File system|
|---|---|---|---|
|16 MB|0C01|Microsoft reserved|N/A|
|~40 GB|0700|Microsoft basic data|NTFS|
|300 MB|2700|Windows RE|NTFS|

Create NTFS file systems on the new Microsoft basic data and Windows RE (recovery) partitions using the _mkntfs_ program from package [ntfs-3g](https://archlinux.org/packages/?name=ntfs-3g).

Reboot the system into a Windows 10 installation media. When prompted to install select the custom install option and install Windows on the Microsoft basic data partition created earlier. This should also install Microsoft EFI files in the [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition "EFI system partition").

After installation (set up of and logging into Windows not required), reboot into Linux and [generate a GRUB configuration](https://wiki.archlinux.org/title/GRUB#Generate_the_main_configuration_file "GRUB") for the Windows boot manager to be available in the GRUB menu on next boot.

##### Time standard
- Recommended: Set both Arch Linux and Windows to use UTC, following [System time#UTC in Microsoft Windows](https://wiki.archlinux.org/title/System_time#UTC_in_Microsoft_Windows "System time"). Some versions of Windows revert the hardware clock back to _localtime_ if they are set to synchronize the time online. This issue appears to be fixed in Windows 10.
- Not recommended: Set Arch Linux to _localtime_ and disable all [time synchronization daemons](https://wiki.archlinux.org/title/System_time#Time_synchronization "System time"). This will let Windows take care of hardware clock corrections and you will need to remember to boot into Windows at least two times a year (in Spring and Autumn) when [DST](https://en.wikipedia.org/wiki/Daylight_saving_time "wikipedia:Daylight saving time") kicks in. So please do not ask on the forums why the clock is one hour behind or ahead if you usually go for days or weeks without booting into Windows.

### Commands to run in Arch

```shell

# Pre-installation
loadkeys la-latin1

# Post-installation
setxkbmap -layout latam,latam

iwctl

device list

station CUSTOM_DEVICENAME scan  

station CUSTOM_DEVICENAME get-networks

station CUSTOM_DEVICENAME connect ROUTER_NAME

exit

ping archlinux.org

timedatectl set-ntp true
```