# Manual 005 — 크로스 컴파일러 설치 + ARM 바이너리 생성

> 시리즈: **임베디드 리눅스 실습 교과서**
> 버전: v0.8.0
> 작성일: 2026-05-06
> 선행 매뉴얼: [manual004.md](manual004.md) — 첫 C 프로그램 네이티브 컴파일
> 다음 매뉴얼: `manual006.md` — QEMU로 ARM 바이너리 실행 (예정)

---

## 학습 목표

이 매뉴얼을 끝까지 따라 한 후 여러분은 다음을 할 수 있어야 합니다.

1. **크로스 컴파일**이 왜 필요한지, 어떤 원리로 동작하는지 설명할 수 있다.
2. ARM용 크로스 컴파일러(`arm-linux-gnueabihf-gcc`)를 Ubuntu에 설치할 수 있다.
3. `hello.c`를 ARM 바이너리로 빌드하고 `file` 명령으로 아키텍처를 확인할 수 있다.
4. M004의 Makefile에서 `CC` 변수 **한 줄만 바꿔** ARM 빌드로 전환할 수 있다.
5. **툴체인 접두어(triple)** 의 의미(`arm-linux-gnueabihf-`)를 구성 요소별로 설명할 수 있다.
6. ABI 차이(`gnueabi` vs `gnueabihf`)를 이해하고 올바른 툴체인을 선택할 수 있다.

> **예상 소요 시간**: 60 ~ 90분

---

## 사전 지식 점검

- M004에서 `gcc`로 네이티브 컴파일 4단계를 완료했다.
- `gcc -c`, `gcc -o`, `make` 를 자신 있게 사용할 수 있다.
- Ubuntu 환경에서 `sudo apt install`을 사용할 수 있다.

환경 확인:

```bash
uname -m
```

`x86_64` 또는 `aarch64`(M1/M2 Mac의 Multipass)가 출력됩니다. 이것이 **호스트 아키텍처**입니다. 우리가 만들 **타겟 아키텍처**는 ARM(32비트)입니다.

---

## 1. 크로스 컴파일이란 무엇인가

### 1.1 문제: 아키텍처가 다르다

우리가 만드는 임베디드 보드(예: Raspberry Pi, BeagleBone)의 CPU는 대부분 **ARM** 계열입니다. 그런데 개발용 PC는 **x86_64** 또는 **aarch64** CPU를 사용합니다.

CPU 아키텍처가 다르면 기계어가 다릅니다. x86_64용 `gcc`로 만든 바이너리는 ARM CPU 위에서 실행할 수 없습니다.

```text
[ 호스트 PC ]              [ 타겟 보드 ]
  x86_64 CPU                 ARM Cortex-A CPU
  Ubuntu                     임베디드 리눅스

  ./hello    ─────────▶  실행 불가 ✗
  (x86_64용)             (ARM 기계어가 아님)
```

### 1.2 해결: 크로스 컴파일러

**크로스 컴파일러(cross-compiler)** 는 호스트 위에서 실행되지만 타겟 아키텍처용 바이너리를 만드는 컴파일러입니다.

```text
[ 호스트 PC ]                          [ 타겟 보드 ]
  x86_64 CPU

  arm-linux-gnueabihf-gcc hello.c
           │
           ▼
       hello (ARM 바이너리) ──────────▶  실행 가능 ✓
```

크로스 컴파일러는 호스트에서 실행되는 x86_64 프로그램이지만, 생성하는 출력물은 ARM 기계어입니다.

### 1.3 툴체인 접두어(triple) 읽기

크로스 컴파일러의 이름에는 타겟 정보가 담긴 **접두어(prefix)** 가 붙습니다.

```text
arm  -  linux  -  gnueabihf  -  gcc
 │         │          │           └── 도구 이름
 │         │          └────────────── ABI / C 라이브러리 정보
 │         └───────────────────────── 운영체제
 └─────────────────────────────────── CPU 아키텍처
```

이 네 부분을 합친 것을 **타겟 트리플(target triple)** 이라고 부릅니다. 각 부분의 의미:

| 부분 | 값 | 의미 |
|------|-----|------|
| 아키텍처 | `arm` | 32비트 ARM (ARMv7 등) |
| OS | `linux` | 리눅스 커널 위에서 실행 |
| ABI/libc | `gnueabihf` | GNU C 라이브러리, EABI, 하드웨어 부동소수점 |
| 도구 | `gcc`, `ld`, `as`... | 실제 도구 이름 |

### 1.4 `gnueabi` vs `gnueabihf` — ABI 차이

ABI 부분에서 자주 혼동이 옵니다.

| 툴체인 | 접미어 | 부동소수점 처리 | 대상 보드 |
|--------|--------|----------------|-----------|
| `arm-linux-gnueabi-gcc` | `eabi` | **소프트웨어** 에뮬레이션 | FPU 없는 구형 ARM |
| `arm-linux-gnueabihf-gcc` | `eabihf` | **하드웨어** FPU 사용 | Cortex-A (RPi 등) 대부분 |

`hf`는 **Hard Float**의 약자입니다. FPU(Floating Point Unit) 하드웨어가 있는 보드는 `hf`를 써야 성능이 제대로 나옵니다.

이 매뉴얼에서는 Raspberry Pi / BeagleBone 계열과 호환되는 **`gnueabihf`** 를 사용합니다.

---

## 2. 크로스 컴파일러 설치

### 2.1 패키지 설치

```bash
sudo apt update
sudo apt install -y gcc-arm-linux-gnueabihf
```

설치에 수 분이 걸릴 수 있습니다.

완료 후 확인:

```bash
arm-linux-gnueabihf-gcc --version
```

출력 예:

```text
arm-linux-gnueabihf-gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
Copyright (C) 2023 Free Software Foundation, Inc.
```

버전 앞에 `arm-linux-gnueabihf-`가 붙어 있으면 크로스 컴파일러입니다.

### 2.2 함께 설치된 도구 확인

크로스 컴파일러 패키지에는 `gcc` 외에도 여러 도구가 함께 설치됩니다.

```bash
ls /usr/bin/arm-linux-gnueabihf-*
```

출력 예:

```text
/usr/bin/arm-linux-gnueabihf-addr2line
/usr/bin/arm-linux-gnueabihf-ar
/usr/bin/arm-linux-gnueabihf-as
/usr/bin/arm-linux-gnueabihf-gcc
/usr/bin/arm-linux-gnueabihf-gcc-13
/usr/bin/arm-linux-gnueabihf-ld
/usr/bin/arm-linux-gnueabihf-nm
/usr/bin/arm-linux-gnueabihf-objdump
/usr/bin/arm-linux-gnueabihf-size
/usr/bin/arm-linux-gnueabihf-strip
```

M004에서 사용한 도구(`nm`, `objdump`, `size`)가 모두 ARM용으로도 있습니다. 앞으로 ARM 바이너리를 분석할 때 이 버전을 사용합니다.

### 2.3 (선택) AArch64(64비트 ARM) 툴체인 추가

Raspberry Pi 4, 5 등 64비트 ARM 보드를 위한 툴체인도 함께 설치해 둡니다.

```bash
sudo apt install -y gcc-aarch64-linux-gnu
aarch64-linux-gnu-gcc --version
```

이 매뉴얼 실습은 32비트 `arm-linux-gnueabihf-`로 진행하지만, 64비트 보드를 쓸 때를 위해 알아두면 좋습니다.

---

## 3. 첫 ARM 바이너리 만들기

### 3.1 작업 디렉토리 준비

```bash
mkdir -p ~/embedded-linux/manual005
cd ~/embedded-linux/manual005
```

M004에서 만든 `hello.c`를 복사합니다.

```bash
cp ~/embedded-linux/manual004/hello.c .
cat hello.c
```

```c
#include <stdio.h>

#define GREETING "Hello, Embedded Linux!"

int main(void) {
    printf("%s\n", GREETING);
    return 0;
}
```

### 3.2 네이티브 빌드 (x86_64)

먼저 기존 `gcc`로 네이티브 빌드합니다.

```bash
gcc hello.c -o hello_x86
file hello_x86
```

출력:

```text
hello_x86: ELF 64-bit LSB pie executable, x86-64, ...
```

`x86-64`로 표시됩니다.

### 3.3 크로스 빌드 (ARM)

이제 ARM용으로 빌드합니다. `gcc` 대신 `arm-linux-gnueabihf-gcc`를 사용합니다.

```bash
arm-linux-gnueabihf-gcc hello.c -o hello_arm
file hello_arm
```

출력:

```text
hello_arm: ELF 32-bit LSB pie executable, ARM, EABI5 version 1 (SYSV), dynamically linked, ...
```

같은 소스 파일 `hello.c`인데 컴파일러만 바꿨더니 `ARM`용 바이너리가 만들어졌습니다.

### 3.4 두 바이너리 비교

```bash
file hello_x86 hello_arm
```

```text
hello_x86: ELF 64-bit LSB pie executable, x86-64, ...
hello_arm: ELF 32-bit LSB pie executable, ARM, EABI5 ...
```

크기 비교:

```bash
size hello_x86 hello_arm
```

```text
   text    data     bss     dec     hex filename
   1608     600       8    2216     8a8 hello_x86
   1244     268       8    1520     5f0 hello_arm
```

ARM 바이너리가 더 작습니다. 임베디드에서 중요한 이점입니다.

어셈블리 비교 — x86_64:

```bash
arm-linux-gnueabihf-objdump -d hello_arm | grep -A 15 "<main>"
```

출력 예:

```text
00010415 <main>:
   10415:	b580      	push	{r7, lr}
   10417:	af00      	add	r7, sp, #0
   10419:	4b03      	ldr	r3, [pc, #12]
   1041b:	4618      	mov	r0, r3
   1041d:	f7ff ef8a 	blx	103d4 <puts@plt>
   10421:	2300      	movs	r3, #0
   10423:	4618      	mov	r0, r3
   10425:	bd80      	pop	{r7, pc}
```

x86_64의 `push %rbp`, `movq` 대신 ARM의 `push {r7, lr}`, `ldr`, `blx` 명령이 나옵니다. 같은 C 코드이지만 CPU 아키텍처가 다르면 기계어도 완전히 다릅니다.

### 3.5 호스트에서 ARM 바이너리 실행 시도

```bash
./hello_arm
```

출력:

```text
bash: ./hello_arm: cannot execute binary file: Exec format error
```

예상대로 실행이 안 됩니다. 호스트 CPU(x86_64)가 ARM 명령어를 이해하지 못하기 때문입니다. 실제 ARM 보드나 에뮬레이터(QEMU)가 필요합니다 — M006에서 다룹니다.

---

## 4. Makefile에서 크로스 컴파일러 적용

### 4.1 M004 파일 복사

```bash
cp ~/embedded-linux/manual004/calc.h .
cp ~/embedded-linux/manual004/calc.c .
cp ~/embedded-linux/manual004/main.c .
cp ~/embedded-linux/manual004/Makefile .
```

기존 Makefile 확인:

```bash
cat Makefile
```

```makefile
CC      = gcc
CFLAGS  = -Wall -Wextra -g
TARGET  = calculator
SRCS    = main.c calc.c
OBJS    = $(SRCS:.c=.o)

all: $(TARGET)
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
clean:
	rm -f $(OBJS) $(TARGET)
.PHONY: all clean
```

### 4.2 `CC` 변수 한 줄만 바꾸기

```bash
nano Makefile
```

첫 번째 줄만 수정합니다.

```makefile
CC      = arm-linux-gnueabihf-gcc
```

저장 후:

```bash
make clean
make
```

출력:

```text
arm-linux-gnueabihf-gcc -Wall -Wextra -g -c main.c -o main.o
arm-linux-gnueabihf-gcc -Wall -Wextra -g -c calc.c -o calc.o
arm-linux-gnueabihf-gcc -Wall -Wextra -g -o calculator main.o calc.o
빌드 완료: calculator
```

```bash
file calculator
```

```text
calculator: ELF 32-bit LSB pie executable, ARM, EABI5 ...
```

**Makefile 구조가 잘 설계되어 있으면, `CC` 변수 한 줄 변경으로 네이티브 빌드 ↔ 크로스 빌드를 전환할 수 있습니다.** 이것이 임베디드 Makefile을 `CC` 변수로 추상화하는 이유입니다.

### 4.3 Makefile에 빌드 타겟 두 개 추가

명령행에서 `make native` 또는 `make arm`으로 선택할 수 있게 합니다.

```bash
nano Makefile
```

전체 내용:

```makefile
# 크로스 컴파일러 설정
CROSS_COMPILE = arm-linux-gnueabihf-
CC_NATIVE     = gcc
CC_CROSS      = $(CROSS_COMPILE)gcc

# 기본 컴파일러는 크로스
CC      = $(CC_CROSS)
CFLAGS  = -Wall -Wextra -g

TARGET  = calculator
SRCS    = main.c calc.c
OBJS    = $(SRCS:.c=.o)

# 기본: ARM 크로스 빌드
all: $(TARGET)

# 명시적 타겟
native:
	$(MAKE) CC=$(CC_NATIVE) TARGET=calculator_native

arm:
	$(MAKE) CC=$(CC_CROSS) TARGET=calculator_arm

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^
	@echo "빌드 완료: $@ (CC=$(CC))"

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) calculator calculator_native calculator_arm
	@echo "정리 완료"

info:
	@echo "CC:            $(CC)"
	@echo "CROSS_COMPILE: $(CROSS_COMPILE)"
	@echo "CFLAGS:        $(CFLAGS)"
	@echo "SRCS:          $(SRCS)"

.PHONY: all native arm clean info
```

코드 설명:

- `CROSS_COMPILE = arm-linux-gnueabihf-` — 접두어만 변수로 추출. 이 변수 하나만 바꾸면 다른 아키텍처로 전환됩니다.
- `$(MAKE)` — 현재 실행 중인 `make`를 재귀 호출합니다. `make` 명령을 하드코딩하지 않는 관습입니다.
- `CC=$(CC_NATIVE)` — 재귀 호출 시 `CC` 변수를 덮어씁니다.

실행:

```bash
make clean
make native    # x86_64 빌드
make arm       # ARM 빌드
file calculator_native calculator_arm
```

출력:

```text
calculator_native: ELF 64-bit LSB pie executable, x86-64, ...
calculator_arm:    ELF 32-bit LSB pie executable, ARM, EABI5 ...
```

---

## 5. 컴파일 옵션 심화 — 타겟 CPU 지정

### 5.1 ARM 세부 아키텍처 지정

ARM에도 다양한 버전이 있습니다. 특정 CPU를 명시하면 해당 CPU에 최적화된 코드가 나옵니다.

```bash
# Cortex-A7 (Raspberry Pi 2 등)
arm-linux-gnueabihf-gcc -mcpu=cortex-a7 hello.c -o hello_a7

# Cortex-A53 (Raspberry Pi 3 등, 64비트지만 32비트 모드로)
arm-linux-gnueabihf-gcc -mcpu=cortex-a53 hello.c -o hello_a53

# ARMv7-A 일반 (호환성 높음)
arm-linux-gnueabihf-gcc -march=armv7-a -mfpu=neon hello.c -o hello_armv7
```

주요 옵션:

| 옵션 | 의미 |
|------|------|
| `-mcpu=cortex-a7` | 특정 CPU 코어 지정 (아키텍처 + 기능 세트 자동 설정) |
| `-march=armv7-a` | ARM 아키텍처 버전만 지정 |
| `-mfpu=neon` | NEON SIMD 부동소수점 유닛 활성화 |
| `-mfloat-abi=hard` | 하드웨어 FPU 레지스터 사용 (gnueabihf와 일치) |
| `-mthumb` | Thumb 명령어 세트 사용 (코드 크기 감소) |

### 5.2 Thumb 명령어 세트

ARM은 32비트 명령어(ARM 모드)와 16비트 명령어(Thumb 모드) 두 가지를 지원합니다.

```bash
arm-linux-gnueabihf-gcc hello.c -o hello_arm_mode
arm-linux-gnueabihf-gcc -mthumb hello.c -o hello_thumb
size hello_arm_mode hello_thumb
```

출력 예:

```text
   text    data     bss     dec     hex filename
   1244     268       8    1520     5f0 hello_arm_mode
   1192     268       8    1468     5bc hello_thumb
```

Thumb 모드가 코드 크기가 약간 작습니다. 플래시 용량이 부족한 임베디드에서 유용합니다.

---

## 6. 툴체인 구성 요소 살펴보기

크로스 툴체인에는 `gcc` 외에도 여러 도구가 있습니다. 각각 언제 사용하는지 알아둡니다.

### 6.1 binutils — 바이너리 도구 모음

```bash
# ARM용 objdump
arm-linux-gnueabihf-objdump -d hello_arm | head -40

# ARM용 nm — 심볼 목록
arm-linux-gnueabihf-nm hello_arm

# ARM용 size — 섹션 크기
arm-linux-gnueabihf-size hello_arm

# ARM용 strip — 디버그 심볼 제거 (배포용 이미지 크기 축소)
cp hello_arm hello_arm_stripped
arm-linux-gnueabihf-strip hello_arm_stripped
ls -la hello_arm hello_arm_stripped
size hello_arm hello_arm_stripped
```

`strip`은 디버그 심볼을 제거해 파일 크기를 줄입니다. 임베디드 rootfs에 올릴 때 용량을 아끼기 위해 사용합니다.

### 6.2 `readelf` — ELF 파일 상세 정보

```bash
arm-linux-gnueabihf-readelf -h hello_arm
```

출력에서 중요한 필드:

```text
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 ...
  Class:                             ELF32
  Data:                              2's complement, little endian
  Machine:                           ARM
  Entry point address:               0x103e5
  ...
```

- `ELF32` — 32비트 ELF.
- `little endian` — ARM은 리틀 엔디안. 멀티바이트 정수를 낮은 주소에 하위 바이트부터 저장.
- `Entry point address` — 프로그램 실행 시 CPU가 처음 점프하는 주소.

섹션 목록:

```bash
arm-linux-gnueabihf-readelf -S hello_arm
```

동적 라이브러리 의존성:

```bash
arm-linux-gnueabihf-readelf -d hello_arm | grep NEEDED
```

출력:

```text
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
```

ARM 실행 파일도 `libc.so.6`가 필요합니다. 타겟 보드의 rootfs에 ARM용 libc가 있어야 실행됩니다.

---

## 7. 정적 링크 빌드 — rootfs 없이 실행 가능한 바이너리

타겟 보드에 라이브러리 파일이 없을 때를 위해 **정적 링크(static linking)** 바이너리를 만들 수 있습니다.

```bash
arm-linux-gnueabihf-gcc -static hello.c -o hello_arm_static
file hello_arm_static
```

출력:

```text
hello_arm_static: ELF 32-bit LSB executable, ARM, EABI5 ..., statically linked, ...
```

`statically linked` — 라이브러리가 바이너리 안에 포함되어 있습니다.

크기 비교:

```bash
ls -lh hello_arm hello_arm_static
```

출력 예:

```text
-rwxr-xr-x 1 ubuntu ubuntu  8.1K hello_arm
-rwxr-xr-x 1 ubuntu ubuntu 520K  hello_arm_static
```

정적 링크 바이너리가 훨씬 큽니다. libc 전체가 포함되었기 때문입니다.

트레이드오프:

| | 동적 링크 | 정적 링크 |
|--|-----------|-----------|
| 파일 크기 | 작음 | 큼 |
| 타겟 의존성 | libc.so 필요 | 불필요 |
| 메모리 공유 | 여러 프로세스가 libc 공유 | 프로세스마다 libc 복사 |
| 사용 시점 | rootfs가 갖춰진 시스템 | rootfs 없는 초기 부팅, 임시 도구 |

---

## 8. 종합 실습 — 빌드 비교 스크립트

지금까지 만든 바이너리들을 한눈에 비교합니다.

```bash
nano ~/embedded-linux/manual005/build_compare.sh
```

내용:

```bash
#!/bin/bash
# 네이티브 vs 크로스 빌드 비교 스크립트

set -e

NATIVE_CC="gcc"
CROSS_CC="arm-linux-gnueabihf-gcc"
SRC="hello.c"

echo "================================================"
echo " 빌드 비교: 네이티브 vs ARM 크로스"
echo "================================================"
echo ""

# 빌드
echo "[빌드 중...]"
$NATIVE_CC "$SRC" -o hello_native 2>/dev/null
$NATIVE_CC -Os "$SRC" -o hello_native_Os 2>/dev/null
$CROSS_CC "$SRC" -o hello_arm 2>/dev/null
$CROSS_CC -Os "$SRC" -o hello_arm_Os 2>/dev/null
$CROSS_CC -mthumb -Os "$SRC" -o hello_arm_thumb 2>/dev/null
$CROSS_CC -static "$SRC" -o hello_arm_static 2>/dev/null
echo ""

# 결과 표
printf "%-25s %-12s %-10s %s\n" "파일" "아키텍처" "크기" "링크 방식"
printf "%s\n" "------------------------------------------------------------"
for f in hello_native hello_native_Os hello_arm hello_arm_Os hello_arm_thumb hello_arm_static; do
    ARCH=$(file "$f" | grep -oE "x86-64|ARM" | head -1)
    SIZE=$(ls -lh "$f" | awk '{print $5}')
    LINK=$(file "$f" | grep -oE "dynamically|statically" | head -1)
    printf "%-25s %-12s %-10s %s\n" "$f" "${ARCH:-unknown}" "$SIZE" "${LINK:-unknown}"
done
echo ""

echo "[size 명령으로 섹션 크기 비교]"
size hello_native hello_arm hello_arm_Os hello_arm_static
echo ""
echo "================================================"
echo " 정리 완료"
echo "================================================"
```

```bash
chmod +x ~/embedded-linux/manual005/build_compare.sh
cd ~/embedded-linux/manual005
~/embedded-linux/manual005/build_compare.sh
```

출력 예:

```text
================================================
 빌드 비교: 네이티브 vs ARM 크로스
================================================

[빌드 중...]

파일                      아키텍처     크기       링크 방식
------------------------------------------------------------
hello_native              x86-64       8.2K       dynamically
hello_native_Os           x86-64       8.0K       dynamically
hello_arm                 ARM          8.1K       dynamically
hello_arm_Os              ARM          7.9K       dynamically
hello_arm_thumb           ARM          7.8K       dynamically
hello_arm_static          ARM          520K       statically

[size 명령으로 섹션 크기 비교]
   text    data     bss     dec     hex filename
   1608     600       8    2216     8a8 hello_native
   1244     268       8    1520     5f0 hello_arm
   1192     268       8    1468     5bc hello_arm_Os
 432924    2648    5700  441272   6ba78 hello_arm_static
```

---

## 9. 자주 발생하는 문제와 해결법

### 문제 1: `arm-linux-gnueabihf-gcc: command not found`

```bash
sudo apt install -y gcc-arm-linux-gnueabihf
```

### 문제 2: `cannot execute binary file: Exec format error`

ARM 바이너리를 호스트(x86_64)에서 직접 실행한 경우입니다. M006에서 QEMU로 해결합니다.

### 문제 3: `/lib/ld-linux-armhf.so.3: No such file or directory`

동적 링크 ARM 바이너리를 QEMU user-mode로 실행할 때 ARM용 libc 런타임이 없는 경우입니다. M006에서 상세히 다룹니다.

### 문제 4: 컴파일은 되지만 타겟에서 실행 오류

`-mcpu` 옵션에 지정한 CPU와 실제 보드 CPU가 다른 경우입니다. 예: `-mcpu=cortex-a72`로 컴파일 후 Cortex-A7 보드에서 실행 → Illegal instruction 오류. 타겟 보드의 CPU를 정확히 확인하고 맞춰야 합니다.

---

## 10. 정리 및 다음 단계

이 매뉴얼에서 한 일:

- 크로스 컴파일의 필요성과 원리를 이해했다.
- 툴체인 접두어(`arm-linux-gnueabihf-`)의 네 구성 요소를 분해해서 읽었다.
- `gnueabi` vs `gnueabihf`(소프트/하드 부동소수점) 차이를 이해했다.
- `arm-linux-gnueabihf-gcc`를 설치하고 `hello.c`를 ARM 바이너리로 빌드했다.
- `file` 명령으로 `x86-64` vs `ARM` 아키텍처를 확인했다.
- M004 Makefile의 `CC` 변수 한 줄만 바꿔 ARM 빌드로 전환했다.
- `CROSS_COMPILE` 변수 패턴으로 유연한 Makefile을 만들었다.
- `-mcpu`, `-mthumb`, `-static` 옵션의 의미와 트레이드오프를 이해했다.
- `strip`, `readelf` 로 ARM 바이너리를 분석했다.

**현재까지의 흐름 정리:**

```text
M001 개념 이해
M002 Ubuntu 환경 구축
M003 셸, 스크립트, Makefile 기초
M004 C 컴파일 4단계, 바이너리 분석        ← 네이티브(x86_64)
M005 크로스 컴파일러, ARM 바이너리 생성   ← 타겟(ARM) 빌드
M006 QEMU로 ARM 바이너리 실행            ← 다음: 호스트에서 ARM 실행
```

다음 매뉴얼 **M006 — QEMU로 ARM 바이너리 실행**에서는 실제 ARM 보드 없이 PC에서 ARM 프로그램을 실행합니다. QEMU user-mode 에뮬레이터를 설치하고, 이 매뉴얼에서 만든 `hello_arm`을 호스트 위에서 바로 실행합니다.

---

## 부록 A — 주요 ARM 크로스 컴파일러 종류

| 접두어 | 대상 | 설치 패키지 |
|--------|------|-------------|
| `arm-linux-gnueabi-` | ARM 32비트, 소프트 FPU | `gcc-arm-linux-gnueabi` |
| `arm-linux-gnueabihf-` | ARM 32비트, 하드 FPU | `gcc-arm-linux-gnueabihf` |
| `aarch64-linux-gnu-` | ARM 64비트 (AArch64) | `gcc-aarch64-linux-gnu` |
| `arm-none-eabi-` | ARM, OS 없음 (베어메탈) | `gcc-arm-none-eabi` |

- `none` — OS 없이 하드웨어 직접 제어 (마이크로컨트롤러, 부트로더).
- `linux` — 리눅스 커널 위에서 실행.

## 부록 B — ARM 컴파일 옵션 빠른 참조

| 옵션 | 의미 |
|------|------|
| `-mcpu=cortex-a7` | 특정 Cortex-A 코어 지정 |
| `-march=armv7-a` | ARMv7-A 아키텍처 지정 |
| `-mfpu=neon` | NEON SIMD 부동소수점 활성화 |
| `-mfloat-abi=hard` | 하드웨어 FPU 레지스터 사용 |
| `-mthumb` | Thumb(16비트) 명령어 세트 사용 |
| `-static` | 모든 라이브러리를 정적으로 포함 |
| `-shared` | 공유 라이브러리(.so) 생성 |

## 부록 C — 이 매뉴얼에서 사용한 명령어 한 줄 풀이

| 명령 | 한 줄 풀이 |
|------|-----------|
| `arm-linux-gnueabihf-gcc hello.c -o hello_arm` | ARM용 크로스 컴파일 |
| `arm-linux-gnueabihf-gcc -static hello.c -o hello_static` | 정적 링크 ARM 바이너리 생성 |
| `arm-linux-gnueabihf-gcc -mthumb -Os hello.c -o hello_thumb` | Thumb 모드 + 크기 최적화 |
| `arm-linux-gnueabihf-objdump -d hello_arm` | ARM 바이너리 역어셈블 |
| `arm-linux-gnueabihf-nm hello_arm` | ARM 바이너리 심볼 목록 |
| `arm-linux-gnueabihf-size hello_arm` | ARM 바이너리 섹션 크기 |
| `arm-linux-gnueabihf-strip hello_arm` | 디버그 심볼 제거 |
| `arm-linux-gnueabihf-readelf -h hello_arm` | ELF 헤더 상세 정보 |
| `arm-linux-gnueabihf-readelf -d hello_arm` | 동적 섹션 (라이브러리 의존성 등) |
| `ls /usr/bin/arm-linux-gnueabihf-*` | 설치된 크로스 툴 목록 확인 |

---

*다음 매뉴얼: `manual006.md` — QEMU로 ARM 바이너리 실행 (작성 예정)*
