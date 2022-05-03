1.  Edit `/etc/default/grub`:
    
    Add the following parameter to the `GRUB_CMDLINE_LINUX_DEFAULT` variable:

        xhci_hcd.quirks=270336

2.  Run `update-grub` as root.
3.  Reboot.