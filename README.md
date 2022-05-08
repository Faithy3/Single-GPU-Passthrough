
# Single GPU Passthrough on Linux: Abbreviated
This guide is a modified version of the original, meant to be more of a quick-reference for myself.

```sudo apt install libvirt-daemon qemu-kvm virt-manager ovmf```

```yay -S virt-manager ovmf qemu```

Kernel params are at /etc/kernelstub/configuration for pop, add intel_iommu=on. remember to change it on the global and not just the default!
On Arch: `sudo nano /boot/loader/entries/arch.conf` or whatever it's named, add the kernel param

```sudo mkdir -p /etc/libvirt/hooks```

https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu

Put that at /etc/libvirt/hooks/qemu

Make all folders that you need

```# Before a VM is started, before resources are allocated:
/etc/libvirt/hooks/qemu.d/$vmname/prepare/begin/*

# Before a VM is started, after resources are allocated:
/etc/libvirt/hooks/qemu.d/$vmname/start/begin/*

# After a VM has started up:
/etc/libvirt/hooks/qemu.d/$vmname/started/begin/*

# After a VM has shut down, before releasing its resources:
/etc/libvirt/hooks/qemu.d/$vmname/stopped/end/*

# After a VM has shut down, after resources are released:
/etc/libvirt/hooks/qemu.d/$vmname/release/end/*
```

The original scripts, without forbidden commands.

Starting script

```#!/bin/bash
# Helpful to read output when debugging
set -x

# Stop display manager
systemctl stop display-manager.service
## Uncomment the following line if you use GDM
#killall gdm-x-session

# Unbind VTconsoles
echo 0 > /sys/class/vtconsole/vtcon0/bind
echo 0 > /sys/class/vtconsole/vtcon1/bind

# Unbind EFI-Framebuffer
echo efi-framebuffer.0 > /sys/bus/platform/drivers/efi-framebuffer/unbind

# Avoid a Race condition by waiting 2 seconds. This can be calibrated to be shorter or longer if required for your system
sleep 2

# Load VFIO Kernel Module  
modprobe vfio-pci  
```

Ending script

```
#!/bin/bash
set -x

# Reload nvidia modules
modprobe nvidia
modprobe nvidia_modeset
modprobe nvidia_uvm
modprobe nvidia_drm

# Rebind VT consoles
echo 1 > /sys/class/vtconsole/vtcon0/bind
# Some machines might have more than 1 virtual console. Add a line for each corresponding VTConsole
#echo 1 > /sys/class/vtconsole/vtcon1/bind

nvidia-xconfig --query-gpu-info > /dev/null 2>&1
echo "efi-framebuffer.0" > /sys/bus/platform/drivers/efi-framebuffer/bind

# Restart Display Manager
systemctl start display-manager.service

```

Logs can be found under /var/log/libvirt/qemu/[VM name].log
