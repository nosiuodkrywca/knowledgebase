# Single GPU passthrough to a VM

## Prerequisites

1. The GPU is not in use by any other VM, container, or program.
2. The VM is going to use the physical output of the video card being passed through. There will be no console output in the web interface.
3. There will be no output from the Proxmox itself if the GPU is the only video card installed.

## Kernel commandline modifications

### Enabling IOMMU, PCI passthrough

> **Note:** This section is created based on the official documentation which can be found [here](https://pve.proxmox.com/wiki/Pci_passthrough). This snippet is to be used with Intel CPUs and GRUB+systemd environment only.

> Login as `root`

1. Edit the GRUB commandline:

    `nano /etc/default/grub`

    ```
    GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on video=vesafb:off video=efifb:off video=simplefb:off rdblaclist=nouveau"
    ```

2. Reconfigure the kernel modules

    `nano /etc/modules`
    
    ```
    vfio
    vfio_iommu_type1
    vfio_pci
    vfio_virqfd
    ```
    > Comment out any modules regarding the display driver, eg. `nvidia` or `nvidia_uvm`.

3. Edit/create modprobe config files

    ```
    echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
    ```

    ```
    echo "options vfio-pci ids=10de:1d01,10de:0fb8 disable_vga=1" > /etc/modprobe.d/vfio.conf
    ```

    > Replace `10de:1d01,10de:0fb8` with IDs of your specific PCIe device. To obtain these IDs, you'll have to use the command `lspci` to locate your card, and `lspci -n -s 01:00` (replace with your `lspci` output) to get the device and vendor ID.

    ```
    echo "options kvm ignore_msrs=1 report_ignored_msrs=0" > /etc/modprobe.d/kvm.conf
    ```

    ```
    cat <EOF >> /etc/modprobe.d/pve-blacklist.conf       
    blacklist nvidiafb
    blacklist nvidia
    blacklist nouveau
    blacklist radeon
    blacklist snd_hda_intel
    EOF
    ```

4. Apply these changes

    `update-grub`

    `update-initramfs -u`

## Fixing minor issues (blinking cursor, constant warnings)

1. Create startup script

    `nano /root/fix_gpu_pass.sh`

    ```
    #!/bin/bash

    echo 1 > /sys/bus/pci/devices/0000\:01\:00.0/remove
    echo 1 > /sys/bus/pci/devices/0000\:01\:00.1/remove
    echo 1 > /sys/bus/pci/rescan

    echo 0 > /sys/devices/virtual/vtconsole/vtcon0/bind
    echo efi-framebuffer.0 > /sys/devices/platform/efi-framebuffer.0/driver/unbind
    ```

    > Replace `00.0`, `00.1` with your `lspci` IDs.

    > `vtcon0` might be `vtcon1` depending on your environment.

2. Edit crontab

    `crontab -e`

    ```
    @reboot /root/fix_gpu_pass.sh
    ```

## Creating the VM

1. Set the BIOS type to OVMF (UEFI).

    You may be able to use SeaBIOS for some guests, but OVMF offers more compatibility with passthrough GPUs.

2. Set the machine type to q35 for faster graphic cards.

    Enabling PCI-Express is available only with a q35 machine.

3. Add an EFI Disk.

    Required for Windows installations and Ubuntu with Secure Boot.

4. Add the PCI Device.

    Select 'All Functions' to pass through sound output (eg. via HDMI). Select PCI-Express for better performance. Select 'Primary GPU` and disable virtual display.

> When creating a Windows VM, if the video output crashes during installation, it might be due to the driver not updating the view properly. The solution here is to enable virtualized display and temporarily disable 'Primary GPU' in the VM's config. After installing the OS and GPU drivers, you can go back to the physical GPU.