# 🧩 [Jetson Orin Nano] gs-usb 드라이버 설치 방법

이 문서는 NVIDIA **Jetson Orin Nano** 환경에서 **USB-to-CAN 어댑터(CANable, Candlelight 등)** 를 사용하기 위한  
`gs_usb` 드라이버를 직접 빌드하고 설치하는 방법을 단계별로 설명합니다.

---

### 1. 필수 설치 파일
```
sudo apt update
sudo apt install -y build-essential dkms git
```

- 커널 모듈을 빌드하기 위해 필요한 도구들을 설치합니다.
  - `build-essential`: gcc, make 등 기본 빌드 도구
  - `dkms`: 커널 모듈 자동 관리 도구
  - `git`: 소스 코드 버전 관리 도구
---

### 2. 버전 확인
```
cat /etc/nv_tegra_release
```
- Jetson의 L4T (Linux for Tegra) 버전을 확인합니다.
- 예시 출력: `# R36 (release), REVISION: 3.0` → R36.3.0 소스 다운로드 필요

---

### 3. 드라이버 폴더 다운로드
```
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/sources/public_sources.tbz2
```

- NVIDIA 공식 L4T 공개 소스를 다운로드합니다.  
- JetPack 버전에 따라 URL이 다를 수 있으므로, NVIDIA 공식 사이트에서 확인하세요.
---

### 4. 드라이버 폴더 압축 풀기
```
tar -xvjf public_sources.tbz2
```

- 압축을 해제하면 `Linux_for_Tegra/source` 폴더가 생성됩니다.
---

### 5. gs-usb 드라이버 폴더 압축 풀기
```
cd ~/Linux_for_Tegra/source
sudo tar -xvjf kernel_src.tbz2
```
- `kernel_src.tbz2`가 존재하는지 확인 후 압축을 해제합니다.  
- 압축 해제 후 `kernel/kernel-jammy-src` 폴더가 생깁니다.

---

### 6. 커널 관련 폴더에 들어감
```
cd ~/Linux_for_Tegra/source/kernel/kernel-jammy-src
```
- 커널 빌드 관련 주요 파일들이 위치한 경로입니다.

---

### 7. 커널 설치 여부 확인
```
uname -r
```

- 현재 커널 버전을 확인합니다.
- 출력값이 없다면 커널이 설치되지 않은 상태이므로, JetPack을 재설치하거나 헤더를 재설정해야 합니다.

---

### 8. 현재 커널 설정을 config 파일로 저장
```
sudo -s
cd ~/Linux_for_Tegra/source/kernel/kernel-jammy-src
zcat /proc/config.gz > .config
exit
```

- 현재 실행 중인 커널의 설정을 `.config`로 저장합니다.  
- 커널과 동일한 설정으로 외부 모듈을 빌드하기 위함입니다.

---

### 9. OpenSSL 라이브러리 개발용 패키지 설치
```
sudo apt update
sudo apt install -y libssl-dev
```
- 커널 빌드 과정에서 암호화 관련 헤더가 필요하므로 해당 패키지를 설치합니다.

---

### 10. 리눅스 커널 소스 트리 준비
```
sudo make modules_prepare
```
- 외부 모듈(ko 파일)을 빌드할 수 있도록 사전 준비합니다.  
- 이 명령은 커널 헤더를 정리해 빌드 환경을 맞추는 과정입니다.

---

### 11. gs_usb.c를 모듈로 빌드하도록 Makefile에 설정 추가
```
cd ~/Linux_for_Tegra/source/kernel/kernel-jammy-src/drivers/net/can/usb
echo "obj-m += gs_usb.o" | sudo tee -a Makefile
```

- `obj-m`은 커널 외부 모듈로 빌드할 대상을 지정하는 구문입니다.  
- Makefile에 `gs_usb.o` 빌드 항목을 추가합니다.

---

### 12. 드라이버 모듈 빌드
```
sudo make -C ~/Linux_for_Tegra/source/kernel/kernel-jammy-src M=$(pwd) modules
```
- 실제로 `gs_usb.ko` 파일을 빌드합니다.
- 빌드 완료 후 현재 폴더에 `gs_usb.ko` 파일이 생성됩니다.

---

### 13. 커널 모듈을 커널에 삽입
```
sudo insmod gs_usb.ko
```
- 빌드된 모듈을 커널에 직접 삽입합니다.
- 정상적으로 삽입되면 다음으로 확인 가능합니다:
```
lsmod | grep gs_usb
dmesg | grep -i can
```
---

### 14. (선택) 재부팅 후 자동 로드 설정
```
echo "gs_usb" | sudo tee -a /etc/modules
```
- `/etc/modules` 파일에 `gs_usb`를 추가하면, 부팅 시 자동으로 로드됩니다.

---

### ✅ 추가 확인 명령어
```
lsusb | grep -i can
sudo ip link set can0 up type can bitrate 1000000
candump can0
```
- USB-CAN 장치 인식 및 통신 테스트를 위한 명령어입니다.

---

### 🧠 참고
- 커널이 업데이트되면 `gs_usb.ko`를 다시 빌드해야 합니다.
- DKMS를 등록하면 커널 업데이트 시 자동 재빌드가 가능합니다.

---

**작성자:** 오현석  
**환경:** Jetson Orin Nano (Ubuntu 22.04 / JetPack 6.x / L4T R36.3.0)
