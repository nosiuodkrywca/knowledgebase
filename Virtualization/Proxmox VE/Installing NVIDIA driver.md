# Proxmox VE NVIDIA driver installation

## Prerequisites
`apt install -y pve-headers pve-kernel-libc-dev linux-headers-$(uname -r)`

## Download the driver package
`mkdir /opt/nvidia`

`cd /opt/nvidia`

`wget https://download.nvidia.com/XFree86/Linux-x86_64/470.141.03/NVIDIA-Linux-x86_64-470.141.03.run`

`chmod +x NVIDIA-Linux-x86_64-470.141.03.run`

## Disable Nouveau
`./NVIDIA-Linux-x86_64-470.141.03.run --no-questions --ui=none --disable-nouveau`

Check if the file contents match:

`more /etc/modprobe.d/nvidia-installer-disable-nouveau.conf`
```conf
#generated by nvidia-installer
blacklist nouveau
options nouveau modeset=0
```
`reboot`

## Run the installer again
`/opt/nvidia/NVIDIA-Linux-x86_64-470.141.03.run --no-questions --ui=none`

## Check if nvidia-smi works properly
`nvidia-smi`

# Create/update modules.conf file
`nano /etc/modules-load.d/modules.conf`
```conf
# /etc/modules-load.d/modules.conf
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
nvidia
nvidia_uvm
```

## Generate new initramfs image with new configuration
`update-initramfs -u`

## Create rules to load proper drivers at boot
`nano /etc/udev/rules.d/70-nvidia.rules`
```conf
# /etc/udev/rules.d/70-nvidia.rules
# Create /nvidia0, /dev/nvidia1 … and /nvidiactl when nvidia module is loaded
KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L'"
#
# Create the CUDA node when nvidia_uvm CUDA module is loaded
KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u'"
```

## Install NVIDIA driver persistence daemon
`git clone https://github.com/NVIDIA/nvidia-persistenced.git`

`cd cd nvidia-persistenced/init`

`./install.sh`

## Check the service status
`systemctl status nvidia-persistenced`

## Reboot
`reboot`

## Instal patch for NVIDIA drivers
>Removes encoding sessions limit

`cd /opt/nvidia`

`git clone https://github.com/keylase/nvidia-patch.git`

`cd nvidia-patch`

`./patch.sh`

`reboot`

# Installing NVIDIA driver inside a container

## Verify that all NVIDIA devices are recognized and running
`ls -l /dev/nv*`
```s
crw-rw-rw- 1 root root 195,   0 Aug  2 14:32 /dev/nvidia0
crw-rw-rw- 1 root root 195, 255 Aug  2 14:32 /dev/nvidiactl
crw-rw-rw- 1 root root 195, 254 Aug  2 14:32 /dev/nvidia-modeset
crw-rw-rw- 1 root root 234,   0 Aug  2 14:32 /dev/nvidia-uvm
crw-rw-rw- 1 root root 234,   1 Aug  2 14:32 /dev/nvidia-uvm-tools
```
>Note the groups ID for NVIDIA devices (here: 195, 234)

## Edit configuration file (before boot)
`nano /etc/pve/lxc/<id>.conf`
```
lxc.cgroup.devices.allow: c 195:* rwm
lxc.cgroup.devices.allow: c 234:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```
>In some cases the group for -uvm and -uvm-tools will change on reboot. If this occurs, it is recommended to add all known occurences to the file.

## Start the LXC container and download the NVIDIA drivers
`mkdir /opt/nvidia`

`cd /opt/nvidia`

`wget https://download.nvidia.com/XFree86/Linux-x86_64/470.141.03/NVIDIA-Linux-x86_64-470.141.03.run`

`chmod +x NVIDIA-Linux-x86_64-470.141.03.run`

`./NVIDIA-Linux-x86_64-470.141.03.run --no-kernel-module`

## Run nvidia-smi to verify the drivers
`nvidia-smi`

`ls -l /dev/nv*`
