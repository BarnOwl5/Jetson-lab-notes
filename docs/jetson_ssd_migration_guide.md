# Jetson Orin Nano SSD Migration Guide

Jetson의 MicroSD 기반 Root 파일 시스템을 SSD로 옮기는 가이드로, ext4 NVMe SSD 기준이며, rsync로 마이그레이션 합니다.  





## 1. 사전 준비

| 사용 모델 | JetPack 버전 | Source | Target |
| --- | --- | --- | --- |
| Jetson Orin Nano | 6.x | MicroSD 128GB / GPT | NVMe 1TB /GPT |
1. Disks에서 SSD 파티션 생성 & ext4 포맷
2. `lsblk -f`로 SSD UUID 메모


## 2. Rootfs로 복사 (rsync)

### 2.1. SSD 마운트

```bash
sudo mkdir -p /mnt/ssdroot
sudo mount /dev/nvme0n1p1(SSD Device) /mnt/ssdroot
```

> ⚠️ 이 때, Disks에서 확실히 마운트 됐는지 확인합니다.
> 

### 2.2. 루트 복사

```bash
sudo mount /dev/nvme0n1p1 /mnt/ssdroot
sudo rsync -aAXHvx --numeric-ids \
  --exclude=/dev/ --exclude=/proc/ --exclude=/sys/ \
  --exclude=/run/ --exclude=/tmp/ --exclude=/mnt/ \
  --exclude=/media/ --exclude=/lost+found \
  / /mnt/ssdroot
```

> SD 카드에서 SSD로 복사하기까지 시간이 꽤 오래 걸립니다. 성공적으로 끝날 경우 다음의 명령어로 필수 부팅 파일이 생성되었는지 확인합니다.
> 
> 
> ```bash
> ls /mnt/ssdroot/bin/bash
> ls /mnt/ssdroot/sbin/init || ls /mnt/ssdroot/lib/systemd/systemd
> ls /mnt/ssdroot/etc/fstab
> ```
> 

## 3. fstab 수정

### 3.1. SSD UUID 확인 & fstab 수정

```bash
sudo apt install nano
# Check SSD UUID
lsblk -f
# Edit ssdroot/etc/fstab
sudo nano /mnt/ssdroot/etc/fstab
```

- /dev/root → UUID=<yourSSD>로 변경

### 3.2. initramfs 재생성

```bash
echo -e "nvme/nnvme_core" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```

### 3.3. extlinux.conf 백업 & 수정

```bash
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.bak
sudo nano /boot/extlinux/extlinux.conf
```

- APPEND 라인에서 root=/dev/<yourSD> → root=UUID=<yourSSD>로 변경

### 3.4. 재부팅 & 확인

```bash
sudo sync && sudo reboot
```

- 부팅 후 mount | grep " / " → /dev/nvme . . . 이면 성공입니다.

## Troubleshooting


### 1. rsync 무한 루프

- **원인**

/sys, /proc, /run 같은 가상 파일시스템이 복사 대상에 포함되어 커널이 계속 업데이트 → 새 파일로 인식

- **해결**
1. -x 또는 --one-file-system 옵션 사용
2. --exclude=/sys/ 등 경로 제외
- **명령어 예시**

```bash
sudo rsync -aAXHvx --numeric-ids \
  --exclude=/dev/ --exclude=/proc/ --exclude=/sys/ \
  --exclude=/run/ --exclude=/tmp/ --exclude=/mnt/ \
  --exclude=/media/ --exclude=/lost+found \
  / /mnt/ssdroot
```
---
### 2. code 23 (Partial transfer) Warning

- **의미**

일부 파일이나 속성을 복사하지 못했을 때 표시됩니다. 보통 캐시, 로그, 가상 파일시스템에서 발생하며 부팅에 필수적인 파일이 아니라면 무시 가능합니다.

- **무시 가능 조건**
1. /home/<user>/.cache, /var/log, /snap 등 비필수 경로에서만 발생
2. /bin, /lib, /etc, /usr 같은 핵심 경로는 정상 복사됨
- **확인 명령어 예시**

```bash
stat /mnt/ssdroot/bin/bash
stat /mnt/ssdroot/lib/systemd/systemd
stat /mnt/ssdroot/etc/fstab
```
---

### 3. 부팅 후 fstab이 여전히 /dev/root 일 경우

부팅 디스크는 SSD지만, 커널 부팅 인자에서 초기 이름이 /dev/root로 표시될 수 있습니다. 정상 동작에 문제 없습니다.

- **명령어 예시**

해당 명령어로 ssd UUID가 나오면 정상

```bash
mount | grep " / "
sudo sed -n '1,120p' /etc/fstab
```

---

이 문서는 학습 목적으로 Jetson Orin Nano 환경에서 SSD로 마이그레이션하는 절차를 설명합니다.  

## License

이 문서는 [CC BY-NC-ND 4.0](./LICENSE) 라이선스 하에 배포됩니다.  
비상업적 사용만 허용되며, 원문을 그대로 유지해야 합니다.