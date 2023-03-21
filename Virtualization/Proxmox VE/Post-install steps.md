# Proxmox VE Post-install steps


## Add pve-no-subscription repository
`echo "deb http://download.proxmox.com/debian stretch pve-no-subscription" >> /etc/apt/sources.list`

## Remove enterprise repository
`/etc/apt/sources.list.d/pve-enterprise.list`

Comment out `# deb https://enterprise.proxmox.com/debian stretch pve-enterprise`

## Remove ProxMox nag message
`sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service`

## Update, upgrade, reboot
`apt update && apt dist-upgrade -y`

`reboot`

## Install sudo, git, make
`apt-get install sudo git gcc make pve-headers-$(uname -r)`

## Resize root partition
> Do so only if you don't plan on storing any VM/container data on the boot drive

Delete any storage pool (default local-lvm) from the GUI, then run:
```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```