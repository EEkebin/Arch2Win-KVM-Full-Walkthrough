# Arch2Win-KVM

**A full walkthrough of the commands and steps required to take to get Windows KVM installed for a Single GPU with GPU Patching.**
> This is to help out a lay person on getting their feet wet with Arch Linux and KVM technology. Although, I really recommend to start off with a simpler Operating System.

1. Download the latest Arch Linux .iso using your preferred method from the link below.
    https://archlinux.org/download/
2. Use Rufus to flash the .iso onto a USB (preferably 8 GB to 32 GB for seamless FAT32 formatting).
3. Use the following command to install Arch Linux.

```bash
archinstall
```

4. Please read and follow each step making sure to choose the options that pertain to your region, locale, and the drive to install the OS.
5. When choosing a bootloader, the best ones are GRUB and systemd-boot. If you are more comfortable getting your hands dirty for a little less overhead and lightweightedness, please use systemd-boot, however, either will work (hopefully)
6. After Installation, arch-chroot into the system.
7. Create a swapfile if wanted (recommended). The count argument is how much in MegaBytes. Please replace '16384' with your preferred amount.
> Think of a swapfile as a spot for hibernated apps, temporary or unused data to be stored in instead of being killed and lost when you run out of memory.
```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=16384 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```
