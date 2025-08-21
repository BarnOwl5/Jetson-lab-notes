## Jetson Orin Nano SSD Migration Guide (English Version)

### 1. Environment

| Device | JetPack | Source | Target |
| --- | --- | --- | --- |
| Jetson Orin Nano | 6.x | MicroSD 128GB (GPT) | NVMe 1TB (GPT) |


### 2. Prepare the SSD
1. Use **Disks** to create a GPT partition and format as `ext4`.
2. Check the SSD UUID with:
```bash
lsblk -f
```

### 3. Copy Root Filesystem

#### 3.1. Mount SSD

```bash
sudo mkdir -p /mnt/ssdroot
sudo mount /dev/nvme0n1p1(SSD Device) /mnt/ssdroot
```

#### 3.2. Copy with rsync

```bash
sudo mount /dev/nvme0n1p1 /mnt/ssdroot
sudo rsync -aAXHvx --numeric-ids \
  --exclude=/dev/ --exclude=/proc/ --exclude=/sys/ \
  --exclude=/run/ --exclude=/tmp/ --exclude=/mnt/ \
  --exclude=/media/ --exclude=/lost+found \
  / /mnt/ssdroot
```

Verify essential files:

```bash
ls /mnt/ssdroot/bin/bash
ls /mnt/ssdroot/sbin/init
ls /mnt/ssdroot/lib/systemd/systemd
ls /mnt/ssdroot/etc/fstab
```

### 4. Update fstab
Check UUID and edit fstab:

```bash
lsblk -f
sudo nano /mnt/ssdroot/etc/fstab
```
Change `/dev/root` -> `UUId=<SSD_UUID>`


### 5. Rebuild initramfs
```bash
echo -e "nvme/nnvme_core" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```

### 6. Configure extlinux.conf
Backup and edit:
```bash
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.bak
sudo nano /boot/extlinux/extlinux.conf
```
Change APPED line to:
`root=/dev/...` -> `root=UUID=<SSD_UUID>`

### 7. Reboot
```bash
sudo sync && sudo reboot
```

Check: 
```bash
mount | grep " / "
```
If output shows `/dev/nvme. . .` -> Migration successful!

## Troubleshooting

### 1. rsync infinite loop

- **Cause** : Copying virtual filesystem (/sys, /proc, /run).
- **Fix** : exclude them usingg `--exclude`


### 2. code 23 (Partial transfer)

- **Cause** : failed to copy non-critical files (cache, logs).
- **Fix** : safe to ignore if essential files are present.

Verify with:
```bash
stat /mnt/ssdroot/bin/bash
stat /mnt/ssdroot/lib/systemd/systemd
stat /mnt/ssdroot/etc/fstab
```

### 3. fstab still shows /dev/root

Even if `/etc/fstab/` shows `/dev/root`, the system can still boot from SSD.
Verify with:
```bash
mount | grep " / "
sudo sed -n '1,120p' /etc/fstab
```

## License

This document is licensed under [CC BY-NC-ND 4.0](./LICENSE).
- Non-commercial use only
- Attribution required
- No modifications
