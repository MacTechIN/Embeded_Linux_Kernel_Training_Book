# Manual 002 — 호스트 개발 환경 구축: 리눅스 환경 마련하기

> 시리즈: **임베디드 리눅스 실습 교과서**
> 버전: v0.4.0
> 작성일: 2026-05-04
> 선행 매뉴얼: [manual001.md](manual001.md) — 임베디드 리눅스 첫걸음
> 다음 매뉴얼: `manual003.md` — 터미널·셸·기본 명령 워밍업 (예정)

---

## 학습 목표

이 매뉴얼을 끝까지 따라 한 후 여러분은 다음을 할 수 있어야 합니다.

1. 왜 임베디드 리눅스 개발에 리눅스 호스트 환경이 필요한지 설명할 수 있다.
2. 자신의 PC 환경(macOS / Windows / Linux)에 맞는 방법으로 리눅스 개발 환경을 마련한다.
3. 마련된 환경에서 터미널을 열고 `uname -a`로 Ubuntu 커널을 확인할 수 있다.
4. 임베디드 리눅스 개발에 필수적인 기본 패키지 목록을 설치한다.

> **예상 소요 시간**: 30 ~ 60분 (네트워크 속도에 따라 설치 시간이 달라짐)

---

## 사전 지식 점검

- [manual001.md](manual001.md) 을 완료했다 (호스트 OS·아키텍처·셸을 알고 있다).
- M001 실습에서 `uname -m` 결과가 `x86_64` 또는 `arm64`임을 확인했다.
- 이미 **Ubuntu/Debian 계열 리눅스 PC**를 호스트로 쓰고 있다면 **섹션 2~4는 건너뛰고** 섹션 5(필수 패키지 설치)부터 시작해도 됩니다.

---

## 1. 왜 리눅스 환경이 필요한가

### 1.1 크로스 컴파일러는 리눅스에서 태어난다

임베디드 리눅스 개발에서 가장 먼저 필요한 도구는 **크로스 컴파일러**입니다. ARM 타겟을 위한 컴파일러, 커널 빌드 도구, Buildroot, Yocto 등 대부분의 핵심 도구가 **리눅스 호스트 환경을 전제**로 만들어졌습니다.

macOS에서도 ARM 크로스 컴파일러를 설치할 수 있지만, 빌드 스크립트가 리눅스 전용 경로나 명령에 의존하는 경우가 빈번합니다. Windows는 더 어렵습니다.

따라서 이 시리즈에서는 **호스트 환경을 Ubuntu(Linux)로 통일**합니다. 본인의 PC가 macOS나 Windows라도, 그 위에 Ubuntu를 올려서 그 안에서 작업하는 방식으로 진행합니다.

### 1.2 세 가지 선택지

본인 환경에 맞는 방법을 하나 고릅니다.

| 방법          | 대상 OS              | 장점                              | 단점                        |
|---------------|----------------------|-----------------------------------|-----------------------------|
| **Multipass** | macOS / Windows      | 가볍고 빠른 설치, 명령 한 줄     | GUI 없음 (터미널 전용)      |
| **WSL2**      | Windows 10/11        | Windows와 파일 공유 쉬움         | Windows 전용                |
| **VirtualBox**| macOS / Windows      | 데스크톱 GUI 포함                | 무겁고 느림, 설정 복잡      |
| **Docker**    | macOS / Windows      | 빠른 시작, 재현성 높음           | 파일시스템 구조가 달라 혼란 |
| **Native**    | Ubuntu/Debian PC     | 가장 빠름, 설정 최소              | 리눅스 PC가 필요            |

이 매뉴얼에서는 **macOS → Multipass**, **Windows → WSL2**를 기본 안내합니다. 두 방법 모두 빠르고 설정이 단순합니다. VirtualBox를 원한다면 섹션 4를 참고하세요.

---

## 2. macOS 사용자 — Multipass로 Ubuntu 설치

Multipass는 Canonical(Ubuntu 개발사)이 만든 경량 VM 관리 도구입니다. 명령 한 줄로 Ubuntu 인스턴스를 만들고 지울 수 있습니다.

### 2.1 Multipass 설치

**방법 A — Homebrew 사용 (권장)**

Homebrew가 이미 설치되어 있다면:

```bash
brew install --cask multipass
```

Homebrew가 없다면 아래에서 설치 후 위 명령을 실행합니다.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**방법 B — 인스톨러 직접 다운로드**

<https://multipass.run/install> 에서 macOS용 `.pkg` 파일을 받아 실행합니다.

설치 완료 확인:

```bash
multipass --version
```

출력 예:

```text
multipass  1.14.0+mac
multipassd 1.14.0+mac
```

버전 숫자는 달라도 됩니다. 명령이 인식되면 OK.

### 2.2 Ubuntu 인스턴스 생성

```bash
multipass launch --name embedded-dev --cpus 2 --memory 4G --disk 20G
```

각 옵션의 의미:

- `launch` — 새 인스턴스를 만들고 시작한다. 뒤에 버전을 명시하지 않으면 최신 Ubuntu LTS가 설치된다.
- `--name embedded-dev` — 인스턴스 이름. 나중에 이 이름으로 접속한다.
- `--cpus 2` — 가상 CPU 코어 수. 커널 빌드는 CPU를 많이 사용하므로 최소 2개.
- `--memory 4G` — RAM 할당. 커널 빌드에는 최소 2 GB, 여유 있게 4 GB.
- `--disk 20G` — 가상 디스크 크기. 커널 소스·빌드 결과물이 수 GB를 차지하므로 20 GB 이상.

Ubuntu 이미지 다운로드와 인스턴스 초기화에 수 분이 걸릴 수 있습니다. 완료 후:

```text
Launched: embedded-dev
```

### 2.3 인스턴스 목록 확인

```bash
multipass list
```

출력 예:

```text
Name                    State             IPv4             Image
embedded-dev            Running           192.168.64.3     Ubuntu 24.04 LTS
```

`State`가 `Running`이면 동작 중입니다.

### 2.4 인스턴스에 접속

```bash
multipass shell embedded-dev
```

접속에 성공하면 프롬프트가 다음처럼 바뀝니다.

```text
ubuntu@embedded-dev:~$
```

이제부터 이 터미널 안에서 입력하는 모든 명령은 **Ubuntu 인스턴스 안에서** 실행됩니다. 접속을 종료하려면:

```bash
exit
```

다시 접속하려면 `multipass shell embedded-dev`를 반복하면 됩니다.

### 2.5 자주 쓰는 Multipass 명령

나중에 참고할 수 있도록 정리합니다.

```bash
multipass start embedded-dev    # 정지된 인스턴스 시작
multipass stop embedded-dev     # 인스턴스 정지 (삭제 아님)
multipass restart embedded-dev  # 재시작
multipass shell embedded-dev    # 접속
multipass info embedded-dev     # 인스턴스 상세 정보
multipass delete embedded-dev   # 인스턴스 삭제 (복구 가능)
multipass purge                 # 삭제된 인스턴스 완전히 제거
```

**섹션 2 완료** — macOS 사용자는 섹션 5(필수 패키지 설치)로 이동합니다.

---

## 3. Windows 사용자 — WSL2로 Ubuntu 설치

WSL2(Windows Subsystem for Linux 2)는 Windows 안에서 리눅스를 실행하는 Microsoft 공식 기능입니다. 별도의 VM 소프트웨어 없이 Windows와 리눅스를 나란히 사용할 수 있습니다.

### 3.1 WSL2 활성화 (Windows 10 2004 이상 / Windows 11)

**PowerShell을 관리자 권한으로 실행합니다.**

`Win` 키 → "PowerShell" 검색 → **마우스 오른쪽 클릭** → **관리자 권한으로 실행**.

다음 명령 한 줄로 WSL2 + Ubuntu를 동시에 설치합니다.

```powershell
wsl --install
```

이 명령은:

1. WSL2 기능을 Windows에 활성화한다.
2. 기본 배포판(Ubuntu)을 자동으로 설치한다.

설치 중 재부팅을 요구할 수 있습니다. 재부팅 후 Ubuntu 설정 창이 자동으로 열립니다.

### 3.2 Ubuntu 초기 설정 (재부팅 후)

Ubuntu 창이 열리면 사용자 이름과 비밀번호를 설정하라고 요청합니다.

```text
Enter new UNIX username: embedded
New password:
Retype new password:
```

- 사용자 이름: 소문자와 숫자만 사용 (예: `embedded`).
- 비밀번호: 입력할 때 화면에 아무것도 표시되지 않지만 입력되고 있습니다. 당황하지 마세요.
- 비밀번호는 `sudo`(관리자 권한 명령) 실행 시 사용되므로 기억해야 합니다.

### 3.3 WSL2 버전 확인

PowerShell에서 (관리자 권한 불필요):

```powershell
wsl --list --verbose
```

출력 예:

```text
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

`VERSION`이 `2`여야 WSL2입니다. `1`이라면 다음으로 업그레이드합니다.

```powershell
wsl --set-version Ubuntu 2
```

### 3.4 Ubuntu 터미널 열기

이후부터는 **Ubuntu 창**(또는 Windows Terminal에서 Ubuntu 탭)에서 작업합니다.

`Win` → "Ubuntu" 검색 → 클릭. 프롬프트 예:

```text
embedded@DESKTOP-XXXXX:~$
```

이제부터 이 안에서 입력하는 명령은 모두 Ubuntu 위에서 실행됩니다.

**섹션 3 완료** — Windows 사용자는 섹션 5(필수 패키지 설치)로 이동합니다.

---

## 4. VirtualBox 사용자 — Ubuntu 설치 (선택)

VirtualBox는 무료 가상화 소프트웨어로 풀 데스크톱 GUI를 제공합니다. Multipass/WSL2보다 무겁지만 리눅스 데스크톱 환경을 시각적으로 익히고 싶다면 좋은 선택입니다.

### 4.1 VirtualBox 설치

<https://www.virtualbox.org/wiki/Downloads> 에서 본인 OS용 인스톨러를 받아 실행합니다.

### 4.2 Ubuntu ISO 다운로드

<https://ubuntu.com/download/desktop> 에서 최신 Ubuntu LTS(Long Term Support) Desktop ISO 파일을 받습니다. 파일 크기가 약 4 GB이므로 시간이 걸립니다.

### 4.3 새 VM 생성

VirtualBox 관리자 창 → **새로 만들기** 버튼.

설정값:

- 이름: `embedded-dev`
- 종류: `Linux`
- 버전: `Ubuntu (64-bit)`
- 메모리(RAM): `4096 MB` 이상
- 가상 하드 디스크: `VDI` 형식, `20 GB` 이상 (동적 할당 권장)

VM을 선택 후 **시작** → ISO 파일을 선택 → Ubuntu 설치 과정을 따라갑니다. "Ubuntu 설치" 화면에서 **최소 설치**를 선택하면 시간이 단축됩니다.

**섹션 4 완료** — VirtualBox 사용자는 섹션 5(필수 패키지 설치)로 이동합니다.

---

## 5. 필수 패키지 설치 (모든 환경 공통)

이제부터는 **Ubuntu 터미널 안에서** 실행합니다. Multipass라면 `multipass shell embedded-dev` 후, WSL2라면 Ubuntu 창에서, VirtualBox라면 Ubuntu 데스크톱의 터미널에서 진행합니다.

### 5.1 환경 확인

먼저 Ubuntu임을 확인합니다.

```bash
uname -a
```

출력에 `Linux`와 `Ubuntu`(또는 `ubuntu`)가 포함되어 있으면 OK.

```bash
cat /etc/os-release
```

출력 예:

```text
NAME="Ubuntu"
VERSION="24.04.2 LTS (Noble Numbat)"
...
```

### 5.2 패키지 목록 업데이트

패키지를 설치하기 전에 항상 목록을 먼저 갱신합니다.

```bash
sudo apt update
```

`sudo`는 "superuser do"의 약자입니다. 뒤에 오는 명령을 관리자(root) 권한으로 실행합니다. 비밀번호를 처음 묻는 경우, Ubuntu 설정 시 정한 비밀번호를 입력합니다.

`apt`는 Ubuntu/Debian의 패키지 관리자입니다. `update`는 설치 가능한 패키지의 최신 목록을 서버에서 가져옵니다 (실제 업그레이드는 아닙니다).

출력 중 마지막 줄이 아래와 비슷하면 정상입니다.

```text
Reading package lists... Done
```

### 5.3 필수 빌드 도구 설치

임베디드 리눅스 개발에 필요한 핵심 패키지를 한 번에 설치합니다.

```bash
sudo apt install -y \
    build-essential \
    gcc \
    g++ \
    make \
    git \
    wget \
    curl \
    file \
    bc \
    bison \
    flex \
    libssl-dev \
    libncurses-dev \
    libelf-dev \
    python3 \
    python3-pip \
    rsync \
    unzip \
    cpio \
    xxd
```

각 패키지의 역할:

| 패키지              | 역할                                                  |
|---------------------|-------------------------------------------------------|
| `build-essential`   | gcc, g++, make, libc 헤더 등 기본 빌드 도구 묶음     |
| `gcc` / `g++`       | C / C++ 컴파일러                                     |
| `make`              | Makefile 기반 빌드 자동화 도구                       |
| `git`               | 버전 관리 (커널 소스 clone에도 사용)                 |
| `wget` / `curl`     | 파일 다운로드 도구                                   |
| `file`              | 파일의 타입을 감지 (ELF 바이너리 확인에 사용)        |
| `bc`                | 커널 빌드 스크립트에서 사용하는 계산기               |
| `bison` / `flex`    | 커널 빌드에 필요한 파서 생성 도구                    |
| `libssl-dev`        | 커널 모듈 서명에 사용하는 SSL 라이브러리 헤더        |
| `libncurses-dev`    | `make menuconfig` (커널 메뉴 설정 UI)에 필요         |
| `libelf-dev`        | ELF 파일(리눅스 바이너리 형식) 조작 라이브러리       |
| `python3`           | 커널 빌드 스크립트, Buildroot 등에서 사용            |
| `rsync`             | 파일 동기화 도구 (rootfs 생성 시 사용)               |
| `unzip` / `cpio`    | 압축 해제 도구 (initramfs 생성에 cpio 사용)          |
| `xxd`               | 16진수 뷰어 (바이너리 파일 검사 시 사용)             |

`-y` 옵션은 "설치를 묻는 확인 질문에 모두 yes로 자동 응답"하라는 뜻입니다. 이 옵션 없이 실행하면 중간에 `Do you want to continue? [Y/n]` 질문이 나올 때 직접 `Y`를 입력해야 합니다.

설치에 수 분이 걸립니다. 완료 후 마지막 줄이 아래와 비슷하면 정상입니다.

```text
Processing triggers for man-db ...
```

### 5.4 설치 확인

주요 도구가 제대로 설치되었는지 한 번씩 확인합니다.

```bash
gcc --version
```

```text
gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
...
```

```bash
make --version
```

```text
GNU Make 4.3
...
```

```bash
git --version
```

```text
git version 2.43.0
```

```bash
python3 --version
```

```text
Python 3.12.3
```

버전 숫자는 달라도 괜찮습니다. 명령이 인식되어 버전이 출력되면 OK.

### 5.5 작업 디렉토리 생성

Ubuntu 환경 안에 임베디드 개발 전용 작업 공간을 만듭니다.

```bash
mkdir -p ~/embedded-linux
cd ~/embedded-linux
pwd
```

출력:

```text
/home/ubuntu/embedded-linux
```

(사용자 이름에 따라 `/home/<이름>/embedded-linux` 형태)

이 디렉토리가 앞으로 모든 실습의 기준 디렉토리가 됩니다.

---

## 6. 실습 — 환경 확인 한 번에 점검하기

모든 설치가 완료된 Ubuntu 환경에서 다음 명령들을 순서대로 실행해 결과를 확인합니다.

### 6.1 시스템 정보 종합 확인

```bash
uname -a && echo "---" && cat /etc/os-release | grep -E "^(NAME|VERSION)=" && echo "---" && nproc && free -h
```

이 명령은 `&&`로 여러 명령을 연결합니다. 하나가 성공해야 다음이 실행됩니다.

- `uname -a` — 커널 정보
- `cat /etc/os-release | grep ...` — OS 이름과 버전만 필터링
- `nproc` — 사용 가능한 CPU 코어 수 출력 (커널 빌드 시 `-j<n>` 옵션에 사용)
- `free -h` — 메모리 사용량을 사람이 읽기 좋은 단위(h = human-readable)로 표시

출력 예:

```text
Linux embedded-dev 6.8.0-51-generic #52-Ubuntu SMP ... x86_64 GNU/Linux
---
NAME="Ubuntu"
VERSION="24.04.2 LTS (Noble Numbat)"
---
2
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       500Mi       2.9Gi        10Mi       400Mi       3.2Gi
Swap:          1.0Gi          0B       1.0Gi
```

- CPU 코어 수: `2` (Multipass에서 `--cpus 2`로 지정했기 때문)
- 총 메모리: `3.8Gi` (Multipass에서 `--memory 4G`로 지정, 일부는 시스템이 사용)

### 6.2 설치된 도구 버전 일괄 확인

```bash
for tool in gcc make git wget curl python3 bc flex bison; do
    echo -n "$tool: "
    $tool --version 2>&1 | head -1
done
```

이 코드는 **셸 반복문(for loop)** 입니다.

- `for tool in ...` — `tool` 변수에 목록의 단어를 하나씩 대입하면서 반복.
- `do ... done` — 반복할 내용.
- `echo -n "$tool: "` — 줄바꿈 없이(`-n`) 도구 이름을 출력. `$tool`은 변수 참조.
- `$tool --version 2>&1 | head -1` — 버전 첫 줄만 출력. `2>&1`은 에러 출력도 함께 받겠다는 뜻.

출력 예:

```text
gcc: gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
make: GNU Make 4.3
git: git version 2.43.0
wget: GNU Wget 1.21.4 built on linux-gnu.
curl: curl 8.5.0 (x86_64-pc-linux-gnu) ...
python3: Python 3.12.3
bc: bc 1.07.1
flex: flex 2.6.4 Apple(flex-34)
bison: bison (GNU Bison) 3.8.2
```

모든 도구가 출력되면 환경 준비 완료입니다.

---

## 7. (선택) macOS ↔ Multipass 파일 공유

Multipass 인스턴스와 macOS 사이에서 파일을 주고받을 때 유용합니다. 지금 당장 필요하지 않으면 건너뛰어도 됩니다.

### 7.1 macOS 디렉토리 마운트

macOS 터미널에서 (Multipass shell 밖에서):

```bash
multipass mount ~/embedded-linux-study embedded-dev:/mnt/host
```

이렇게 하면 macOS의 `~/embedded-linux-study`가 Multipass 인스턴스 안의 `/mnt/host`로 연결됩니다. 양쪽에서 같은 파일에 접근할 수 있습니다.

마운트 해제:

```bash
multipass umount embedded-dev
```

### 7.2 파일 직접 복사

마운트 대신 파일만 복사하려면:

```bash
multipass transfer ~/some-file.txt embedded-dev:/home/ubuntu/
```

---

## 8. 자주 발생하는 문제와 해결법

### 문제 1: `multipass` 명령을 찾을 수 없다 (macOS)

```text
zsh: command not found: multipass
```

**해결**: Multipass 설치 후 새 터미널 창을 열어야 PATH가 업데이트됩니다.

```bash
source ~/.zshrc    # 또는 ~/.bashrc
```

여전히 안 된다면 재부팅 후 시도합니다.

### 문제 2: `wsl --install` 실행 시 오류 (Windows)

"WSL 2 requires an update to its kernel component" 같은 오류가 나오면:

```powershell
wsl --update
```

실행 후 재부팅합니다.

### 문제 3: `sudo apt update` 시 "Unable to fetch" 오류

인터넷 연결 또는 DNS 문제입니다.

```bash
ping 8.8.8.8
```

응답이 없으면 VM/WSL의 네트워크 어댑터 설정을 확인합니다. Multipass는 재시작으로 해결되는 경우가 많습니다.

```bash
exit                                 # Multipass 빠져나오기
multipass stop embedded-dev
multipass start embedded-dev
multipass shell embedded-dev
sudo apt update                      # 재시도
```

### 문제 4: 디스크 공간 부족

`df -h`로 남은 용량을 확인합니다.

```bash
df -h /
```

Multipass 인스턴스를 처음 만들 때 `--disk`를 20 GB 미만으로 지정했다면, 인스턴스를 삭제하고 더 크게 다시 만드는 것이 가장 간단합니다.

```bash
# macOS 터미널에서
multipass delete embedded-dev
multipass purge
multipass launch --name embedded-dev --cpus 2 --memory 4G --disk 30G
```

---

## 9. 정리 및 다음 단계

이 매뉴얼에서 한 일:

- 왜 Ubuntu 환경이 임베디드 리눅스 개발에 필요한지 이해했다.
- macOS → Multipass, Windows → WSL2, 또는 VirtualBox로 Ubuntu 환경을 마련했다.
- 임베디드 개발에 필요한 기본 패키지 (`gcc`, `make`, `git`, `bison`, `flex` 등)를 설치했다.
- 설치된 도구들을 셸 반복문으로 일괄 확인했다.

지금 상태를 정리하면:

```text
[ 호스트 PC (macOS/Windows) ]
        |
        | Multipass shell / WSL2 / VirtualBox
        v
[ Ubuntu 인스턴스 ]
  ~/embedded-linux/      ← 앞으로의 모든 실습 기준 디렉토리
  gcc, make, git 등 설치 완료
```

다음 매뉴얼 **M003 — 터미널·셸·기본 명령 워밍업**에서는 Ubuntu 환경 안에서 자주 쓰는 셸 명령을 집중적으로 익힙니다. 파일·디렉토리 조작, 텍스트 파이프라인, 환경 변수, Makefile 기초 등 이후 매뉴얼을 따라가기 위한 최소한의 셸 사용법을 다룹니다.

---

## 부록 A — Multipass 치트시트

```bash
# 인스턴스 관리
multipass launch --name <이름> --cpus <n> --memory <xG> --disk <xG>
multipass list
multipass start <이름>
multipass stop <이름>
multipass restart <이름>
multipass delete <이름> && multipass purge

# 접속 및 파일
multipass shell <이름>
multipass transfer <로컬파일> <이름>:<원격경로>
multipass mount <로컬경로> <이름>:<원격경로>
multipass umount <이름>

# 정보 조회
multipass info <이름>
multipass exec <이름> -- uname -a    # shell 없이 명령 하나만 실행
```

## 부록 B — WSL2 치트시트

```powershell
# PowerShell에서
wsl --install                   # 최초 설치
wsl --list --verbose            # 배포판 목록 + 버전 확인
wsl --set-version Ubuntu 2      # WSL2로 업그레이드
wsl --shutdown                  # 모든 WSL 인스턴스 종료
wsl --update                    # WSL 커널 업데이트
```

```bash
# WSL Ubuntu 터미널에서
explorer.exe .                  # 현재 리눅스 디렉토리를 Windows 탐색기로 열기
cd /mnt/c/Users/<이름>/         # Windows C 드라이브 접근
```

## 부록 C — 이 매뉴얼에서 사용한 명령어 한 줄 풀이

| 명령                                  | 한 줄 풀이                                                     |
|---------------------------------------|----------------------------------------------------------------|
| `sudo apt update`                     | 패키지 목록을 서버에서 최신으로 갱신                           |
| `sudo apt install -y <패키지...>`     | 패키지를 확인 질문 없이 설치                                   |
| `cat /etc/os-release`                 | OS 릴리스 정보 파일 출력                                       |
| `nproc`                               | 사용 가능한 CPU 코어 수 출력                                   |
| `free -h`                             | 메모리 사용량을 사람이 읽기 쉬운 단위로 출력                   |
| `df -h /`                             | 루트 파티션의 디스크 사용량 출력                               |
| `for x in ...; do ...; done`          | 셸 반복문                                                      |
| `echo -n`                             | 줄바꿈 없이 출력                                               |
| `cmd 2>&1`                            | 표준 에러(stderr)를 표준 출력(stdout)으로 합침                 |
| `cmd \| head -1`                      | 파이프: 앞 명령의 출력 중 첫 줄만 통과                         |

---

*다음 매뉴얼: `manual003.md` — 터미널·셸·기본 명령 워밍업 (작성 예정)*
