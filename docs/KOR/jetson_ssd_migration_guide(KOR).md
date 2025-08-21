## Jetson Orin Nano SSD Migration Guide

Jetson Orin Nano에서 microSD 기반 Root 파일 시스템을 SSD로 옮기는 가이드입니다.
ext4 NVMe SSD를 기준으로 하며, rsync를 사용해 마이그레이션 합니다


### 1. 사전 준비

| 사용 모델 | JetPack 버전 | Source | Target |
| --- | --- | --- | --- |
| Jetson Orin Nano | 6.x | MicroSD 128GB / GPT | NVMe 1TB / GPT |

**필수 준비**
1. Disks에서 SSD 파티션 생성 & ext4 포맷
2. `lsblk -f`로 SSD UUID 메모


### 2. Rootfs로 복사 (rsync)

#### 2.1. SSD 마운트

```bash
sudo mkdir -p /mnt/ssdroot
sudo mount /dev/nvme0n1p1(SSD Device) /mnt/ssdroot
```
⚠️ 반드시 Disks에서 확인한 SSD 파티션을 사용하세요.


#### 2.2. 루트 복사

```bash
sudo mount /dev/nvme0n1p1 /mnt/ssdroot
sudo rsync -aAXHvx --numeric-ids \
  --exclude=/dev/ --exclude=/proc/ --exclude=/sys/ \
  --exclude=/run/ --exclude=/tmp/ --exclude=/mnt/ \
  --exclude=/media/ --exclude=/lost+found \
  / /mnt/ssdroot
```

#### 복사 검증
```bash
ls /mnt/ssdroot/bin/bash
ls /mnt/ssdroot/sbin/init
ls /mnt/ssdroot/lib/systemd/systemd
ls /mnt/ssdroot/etc/fstab
```

### 3. fstab 수정

#### 3.1. SSD UUID 확인 & fstab 수정

```bash
sudo apt install nano
# Check SSD UUID
lsblk -f
# Edit ssdroot/etc/fstab
sudo nano /mnt/ssdroot/etc/fstab
```

- /dev/root → UUID=<SSD_UUID>로 변경

#### 3.2. initramfs 재생성

```bash
echo -e "nvme/nnvme_core" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```

#### 3.3. extlinux.conf 수정

```bash
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.bak
sudo nano /boot/extlinux/extlinux.conf
```

- APPEND 라인에서 root=/dev/... → root=UUID=<SSD_UUID>로 변경

### 3.4. 재부팅 & 확인

```bash
sudo sync && sudo reboot
```

**부팅 후 확인**
```bash
mount | grep " / "
```
- 출력에 /dev/nvme . . . 이면 성공입니다.

### Troubleshooting


#### 1. rsync 무한 루프

- **원인** : /sys, /proc, /run 같은 가상 파일시스템이 복사 대상에 포함됨
- **해결** : --exclude 옵션으로 경로 제외


#### 2. code 23 (Partial transfer)

- **원인** : 일부 파일 소속 복사 실패 (캐시,로그,가상 FS 등)
- **해결** : 무시 가능 (필수 부팅 파일만 확인)

**확인 명령어**
```bash
stat /mnt/ssdroot/bin/bash
stat /mnt/ssdroot/lib/systemd/systemd
stat /mnt/ssdroot/etc/fstab
```

#### 3. fstab이 여전히 /dev/root 인 경우

- 부팅 디스크는 SSD지만 /dev/root로 표시될 수 있음 -> 정상 동작
  
**확인 명령어**
```bash
mount | grep " / "
sudo sed -n '1,120p' /etc/fstab
```


## License

이 문서는 [CC BY-NC-ND 4.0](./LICENSE) 라이선스를 따릅니다.

- 수정 불가 
- 비상업적 사용만 허용
- 원문을 그대로 유지해야 함
