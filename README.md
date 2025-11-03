# [젯슨 오린 나노 gs-usb 드라이버 설치 방법]
### 1.필수 설치 파일
```
$ sudo apt update
$ sudo apt install -y build-essential dkms git
```

- 드라이버를 설치하기 위해 필요한 설치파일을 설치하세요.

### 2.버전 확인
```
$ cat /etc/nv_tegra_release
```

### 3.드라이버 폴더 다운로드
```
$ wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/sources/public_sources.tbz2
```

### 4.드라이버 폴더 압축 풀기
```
$ tar -xvjf public_sources.tbz2
```

### 5.gs-usb 드라이버 폴더 압축 풀기
```
$ cd ~/Linux_for_Tegra/source
$ sudo tar -xvjf kernel_src.tbz2
```
Linux-for-Tegra 폴더가 home에 생성된 것을 확인하고, 그안에 source 폴더에 들어가고,
kernel_src.tbz2 압축폴더가 있는지 확인 후, 압축풀기를 진행하세요.

### 6.커넬 관련 폴더에 들어감
```
$ cd kernel/kernel-jammy-src
```

### 7.커넬 설치 여부를 확인 
```
$ uname -r
```
만약 커넬 버전이 나오질 않는다면, 설치가 안된것이니, 다시 진행해주세요.

### 8.현재 커널 설정을 config 파일로 저장
```
$ sudo -s
$ cd ~/Linux_for_Tegra/source/kernel/kernel-jammy-src
$ zcat /proc/config.gz > .config
$ exit
```
