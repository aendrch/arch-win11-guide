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
- DEACTIVATE UEFI SECURE BOOT!!!
