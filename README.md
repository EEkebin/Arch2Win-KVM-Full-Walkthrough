# Arch2Win-KVM
> I realize most of this can be automated. PLEASE IF YOU ARE SKILLED IN THIS AREA, SEND ME AN EMAIL, MESSAGE ME ON DISCORD, AND/OR CONTRIBUTE TO THE PROJECT!!!

**A full walkthrough of the commands and steps required to take to get Windows KVM installed for a Single GPU with GPU Patching.**

> This is to help out a lay person on getting their feet wet with Arch Linux and KVM technology. Although, I really recommend to start off with a simpler Operating System.

1. Download the latest [Arch Linux](https://archlinux.org/download/) .iso using your preferred method from the link below.
2. Download the latest [Rufus](https://rufus.ie/) to flash the .iso onto a USB (preferably 8 GB to 32 GB for seamless FAT32 formatting).
3. Use the following command to install Arch Linux.

```bash
archinstall
```

4. Please read and follow each step making sure to choose the options that pertain to your region, locale, and the drive to install the OS.
5. When choosing a bootloader, the best ones are GRUB and systemd-boot. If you are more comfortable getting your hands dirty for a little less overhead and lightweightedness, please use systemd-boot, however, either will work (hopefully)
6. I recommend choosing a Desktop installation with the Desktop Environment being KDE Plasma or something your familiar with already.
7. Please use pulseaudio.
8. After Installation, arch-chroot into the system.
9. Create a swapfile if wanted (recommended). The count argument is how much in MegaBytes. Please replace '16384' with your preferred amount.
> Think of a swapfile as a spot for hibernated apps, temporary or unused data to be stored in instead of being killed and lost when you run out of memory.

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=16384 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```
10. Add the swapfile to the fstab.

```bash
sudo nano /etc/fstab
```
```txt
/swapfile none swap defaults 0 0
```

11. Install and enable recommended packages.
```bash
sudo pacman -S bash-completion
sudo pacman -S --needed git base-devel libappindicator-gtk3 nano
sudo systemctl enable fstrim.timer
sudo systemctl enable sshd
```

## If you are using systemd-boot:
12. Edit `/boot/loader/entries/arch.conf` to make sure all of the intel_iommu, iommu, video stuff is inputted correctly before quiet splash.
```bash
sudo nano /boot/loader/entries/arch.conf
```
```txt
options root=UUID=0a3407de-014b-458b-b5c1-848e92a327a3 rw intel_iommu=on iommu=pt iommu=1 intel_iommu=igfx_off video=efifb:off quiet splash
```

## If you are using Grub:
12. Edit `/etc/default/grub` to make sure the grub commands look identical or similar enough to your needs.
```txt
GRUB_DEFAULT=0
GRUB_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_FORCE_HIDDEN_MENU="true"
GRUB_DISTRIBUTOR="Arch Linux"
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on iommu=pt iommu=1 intel_iommu=igfx_off video=efifb:off quiet splash"
```
Run the following command to update grub.
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

13. Comment out the following line in `/etc/pulse/default.pa`
```bash
sudo nano /etc/pulse/default.pa
```
```txt
load-module module-suspend-on-idle
```

14. Uncomment the following lines in `/etc/pacman.conf`
```bash
sudo nano /etc/pacman.d/mirrorlist
```
```txt
[Multilib]
Include = /etc/pacman.d/mirrorlist
```

15. Enable set-ntp.
```bash
sudo timedatectl set-ntp true
```

16. Install your preferred or the recommended desktop apps and packages. Here's a couple of my favourites that are good.
```bash
sudo pacman -S bashtop neofetch filezilla okteta zip unzip spectacle ark ntfs-3g qbittorrent kalgebra discord steam celluloid
```

17. Install yay.
```bash
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si
```

18. Install your favourite webbrowser. Mine is Brave.
```bash
yay -S brave-bin
```

## You can stop at this point if you don't want VFIO, GPU Patching, Windows KVM, or any other advanced features.

19. If you are using an audio jack that is NOT connected to a separate audio device (USB or PCIe sound card) then you will need to install the following package to use scream network audio instead.
```bash
yay -S scream
```

20. Install nvflash.
```bash
yay -S nvflash
```

21. Launch your browser once or twice to make sure everything is working.
22. Disable KWallet and other unnecessary things in `/.config/kwalletrc`
```bash
sudo nano /.config/kwalletrc
```
Add in the following:
```txt
[Wallet]
First Use=false
Enabled=false
```

23. Create a directory for your vgabios.
```bash
sudo mkdir /usr/share/vgabios
```

24. Download the forked and slightly modified version of NVIDIA-vBIOS-VFIO-Patcher from my GitHub through the following commands.
```bash
wget https://raw.githubusercontent.com/EEkebin/NVIDIA-vBIOS-VFIO-Patcher/master/vbios_patcher.py
sudo chmod +x vbios_patcher.py
```

> I REALLY RECOMMEND YOU DO THIS NEXT BIT VIA SSH FROM A REMOTE MACHINE OR PHONE OR SOMETHING ELSE OTHER THAN THIS PC BECAUSE THE SCREEN WILL BE GOING BLANK WITH THE FOLLOWING COMMANDS.

25. Run the following commands to dump and patch the vbios.
```bash
sudo systemctl isolate multi-user.target
sudo rmmod nvidia_drm nvidia_modeset nvidia
sudo nvflash --save oldrom.rom
```

26. Reboot.
```bash
sudo reboot
```

27. Once booted back up, run the following commands to place it in the correct directory and give it the proper permissions.
```bash
sudo mv oldrom.rom /usr/share/vgabios/
sudo chmod +x /usr/share/vgabios/oldrom.rom
```
28. Delete old temporary files.

29. Install required packages for KVM.
```bash
sudo pacman -S qemu libvirt edk2-ovmf virt-manager dnsmasq ebtables
sudo systemctl enable --now libvirtd
sudo virsh net-start default
sudo virsh net-autostart default
sudo usermod -aG kvm,input,libvirt $(whoami)
```
30. Download the win-virtio drivers
```bash
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

31. Create Windows VM with the following settings:
```txt
Chipset: Q35
Firmware: UEFI x86_64: OVMF_CODE.secboot.fd
CPU: host-passthrough, fix CPU Topology
VirtIO Disk, add CDROM SATA Windows.iso and virtio-win.iso
VirtIO NIC.
```
During initial boot of Windows you may or may not have to force-shutdown and turn back on the VM if you experience black screens"

32. Boot the Windows VM and enable Hyper-V by going searching for "Turn Windows features on or off" and checking on "Hyper-V".
33. Install the virtio-win drivers.
34. Shutdown the Windows VM.
35. Make sure to follow the following instructions to setup the correct settings for the VM.
```txt
Remove Channel Spice, Display Spice, Video XQL, Sound ich*, and other unnecessary devices.
Add Nvidia Devices, VGA and HD Audio Drivers, in PCI Host.
Add the Entire USB Controller and USB Hubs.
```
36. Make sure to change the settings to your own liking and pertaining to your system in the XML file of the settings of the VM.
Here's mine:
```xml
<vcpu placement="static">6</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="2"/>
  <vcpupin vcpu="1" cpuset="3"/>
  <vcpupin vcpu="2" cpuset="4"/>
  <vcpupin vcpu="3" cpuset="5"/>
  <vcpupin vcpu="4" cpuset="6"/>
  <vcpupin vcpu="5" cpuset="7"/>
</cputune>
<features>
  <acpi/>
  <apic/>
  <hyperv>
    <relaxed state="on"/>
    <vapic state="on"/>
    <spinlocks state="on" retries="8191"/>
    <vpindex state="on"/>
    <runtime state="on"/>
    <synic state="on"/>
    <stimer state="on"/>
    <reset state="on"/>
    <vendor_id state="on" value="randomid"/>
    <frequencies state="on"/>
    <reenlightenment state="on"/>
    <tlbflush state="on"/>
    <evmcs state="off"/>
  </hyperv>
  <kvm>
    <hidden state="on"/>
  </kvm>
  <vmport state="off"/>
  <smm state="on"/>
  <ioapic driver="kvm"/>
</features>
<cpu mode="host-passthrough" check="none" migratable="on">
  <topology sockets="1" dies="1" cores="6" threads="1"/>
  <cache mode="passthrough"/>
</cpu>
<clock offset="localtime">
  <timer name="hpet" present="yes"/>
  <timer name="hypervclock" present="yes"/>
  <timer name="tsc" present="yes" mode="native"/>
</clock>
```

37. In the GPU VGA section, add the following line:
```xml
<hostdev mode="subsystem" type="pci" managed="yes">
  ...
  <rom file="/usr/share/vgabios/patched.rom"/>
  ...
</hostdev>
```

38. Uncomment and modify the following lines in `/etc/libvirtd.conf`. Make sure to get replace the () placeholders with your information.
```txt
#unix_socket_group = "libvirt"
#unix_socket_ro_perms = "0770"
#user = "(logname)"
#group = "(hostname)"
```

39. Run the following commands to restart libvirtd.
```bash
sudo usermod -aG kvm,input,libvirt $(whoami)
sudo systemctl restart libvirtd
```

## Installing Hooks (Required)
40. Run the following commands to install hooks and restart the computer.
```bash
git clone https://gitlab.com/risingprismtv/single-gpu-passthrough.git
cd single-gpu-passthrough
sudo chmod +x install_hooks.sh
sudo sh install_hooks.sh
cd ..
sudo rm -R single-gpu-passthrough
sudo reboot
```

## Fix up some last minute Hooks settings that are required to run the KVM smoothly without your host OS getting in the way.
41. Edit the following file: `/etc/libvirt/hooks/qemu`
```bash
sudo nano /etc/libvirt/hooks/qemu
```
> Obviously, yours may look different, but the important thing to note is the `AllowedCPUs` parameters. In the prepare operation, it is only allowing your host OS to utilize cores 0 and 1, atleast for me in my configuration. And in the release, it will reset the allowed cores to the maximum that mine is allowed, 0-7. (Since my processor is a total of 8 cores and 8 threads.)
> If you have a hyperthreading or multithread capable processor, please use corresponding logical cores. Please do not split them up for different workloads. For example: If you have a dual core processor, you can use logical cores 0 and 1 for workloads which make up the first processors primary/physical core.
```txt
...
case "$OPERATION" in
    "prepare")
    systemctl set-property --runtime -- user.slice AllowedCPUs=0,1
    systemctl set-property --runtime -- system.slice AllowedCPUs=0,1
    systemctl set-property --runtime -- init.scope AllowedCPUs=0,1
    ...
    ;;
    "release")
    systemctl set-property --runtime -- user.slice AllowedCPUs=0-7
    systemctl set-property --runtime -- system.slice AllowedCPUs=0-7
    systemctl set-property --runtime -- init.scope AllowedCPUs=0-7
    ...
    ;;
esac
...
```
42. Reboot the computer and happy KVMing!
```
sudo reboot
```