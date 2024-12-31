
32b arm openwrt zImage-initramfs download:  
https://archive.openwrt.org/releases/22.03.6/targets/armvirt/32/


```bash
qemu-img create -f qcow2 openwrt-armvirt.qcow2 20G

```


```bash
qemu-system-arm -nographic -M virt -m 64 -kernel openwrt-22.03.6-armvirt-32-zImage-initramfs -drive file=openwrt-armvirt.qcow2,format=qcow2,if=virtio -append "root=/dev/vda"
```


