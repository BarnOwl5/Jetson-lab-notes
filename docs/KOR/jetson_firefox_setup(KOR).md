## Jetson Firefox Setup


### 1. Snap 기반 Firefox 제거
미리 설치된 Snap 버전의 Firefox를 완전히 삭제합니다. (의존성 포함)
```bash
sudo snap remove firefox
sudo apt purge firefox
sudo apt autoremove
```

### 2. Mozillateam PPA 추가
공식 Mozillateam PPA를 활성화합니다. 이 저장소는 정기적으로 업데이트된 `.deb` 패키지를 제공합니다.
```bash
sudo add-apt-repository ppa:mozillateam/ppa
sudo apt update
```

### 3. Snap 대신 PPA 버전 고정
APT가 Snap 대신 Mozillateam PPA에서 Firefox를 설치하도록 고정합니다. 
```bash
sudo tee /etc/apt/preferences.d/mozilla-firefox > /dev/null <<EOF
Package: firefox*
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001
EOF
```

### 4. Firefox 설치 (Deb 버전)
APT를 통해 PPA 버전의 Firefox를 설치합니다. 해당 버전은 apt upgrade를 통해 업데이트됩니다.
```bash
sudo apt update
sudo apt install firefox -y
```

### 5. 바이너리 경로 확인
실행 파일이 `/usr/bin/firefox`(PPA 버전)인지 확인합니다. (`/snap/bin/firefox`경로가 아니어야 함)
```bash
which firefox
>> /usr/bin/firefox

hash -r
firefox --version
>> Mozilla Firefox 142.0
```

### 6. (선택) 실행 편의를 위한 Ailas 추가
Firefox 명령어가 항상 PPA 버전을 실행하도록 Alias를 설정합니다.
```bash
echo "alias firefox='usr/bin/firefox'" >> ~/.bashrc
source ~/.bashrc
```

## 📌참고
💡 이 설정은 Firefox가 Mozillateam PPA를 통해 설치되고, `apt upgrade`를 통해 업데이트 해야 합니다.

💡 보안 업데이트를 자동으로 적용하려면 unattended-upgrades를 활성화할 수 있습니다.

💡 만약 snap이 재설치되는 경우, apt 환경설정이 제대로 PPA를 가리키는지 확인해야 합니다.

## License
이 문서는 [CC BY-NC-ND 4.0](./LICENSE) 라이선스를 따릅니다.

- 비상업적 사용만 허용
- 출처 명시 필수
- 수정 불가
