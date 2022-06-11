# No Software RAID

Remove the software RAID to improve performances.

Here an example of imageinstall config file:

```
DRIVE1 /dev/sda
DRIVE2 /dev/sdb
SWRAID 0
SWRAIDLEVEL 0
BOOTLOADER grub
HOSTNAME dmshift.eu
PART /boot ext3 512M
PART lvm vg0 all
LV vg0 root / ext4 80G
LV vg0 swap swap swap 12G
LV vg0 libvirt /var/lib/libvirt/images xfs all
IMAGE /root/images/CentOS-80-stream-amd64-base.tar.gz
```

Run `imageinstall`.

When the OS is ready, you will notice that the second disk is not used, the following steps, will add `/dev/sdb` to the volume group:

1. Set the partition type to Linux LVM, 0x8e, using `fdisk /dev/sdb`.
   - create 1 partition with all the space
   - then:

   ```
   Type t to select the partition:

   Command (m for help): t
   Partition number (1-4): 1
   Set partition type as 8e which for Linux LVM.

   Hex code (type L to list codes): 8e
   ```
2. Add the disk to the volume group and expand it:

   ```sh
   partprobe
   pvcreate /dev/sdb1
   vgextend vg0 /dev/sdb1
   lvresize -l +100%FREE /dev/vg0/libvirt
   ```
3. Issue the `reboot` command

4. At the restart, use this command to resize the image partition:

   ```sh
   xfs_growfs /var/lib/libvirt/images
   ```
      
