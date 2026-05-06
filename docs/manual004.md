# Manual 004 — 첫 C 프로그램 네이티브 컴파일

> 시리즈: **임베디드 리눅스 실습 교과서**
> 버전: v0.7.0
> 작성일: 2026-05-04
> 선행 매뉴얼: [manual003.md](manual003.md) — 터미널·셸·기본 명령 워밍업
> 다음 매뉴얼: `manual005.md` — 크로스 컴파일러 설치 + ARM 바이너리 생성 (예정)

---

## 학습 목표

이 매뉴얼을 끝까지 따라 한 후 여러분은 다음을 할 수 있어야 합니다.

1. C 소스 파일이 실행 파일이 되기까지의 **4단계(전처리 → 컴파일 → 어셈블 → 링킹)** 를 설명할 수 있다.
2. `gcc` 옵션으로 각 단계를 **분리 실행**하고 중간 산출물(`.i`, `.s`, `.o`)을 확인할 수 있다.
3. `file`, `objdump`, `nm`, `size` 명령으로 바이너리를 **분석**할 수 있다.
4. 여러 소스 파일을 **Makefile**로 관리하고 증분 빌드(incremental build)가 작동하는 것을 직접 확인할 수 있다.
5. 컴파일 경고와 에러의 **차이**를 이해하고 주요 경고 옵션을 사용할 수 있다.

> **예상 소요 시간**: 90 ~ 120분

---

## 사전 지식 점검

- Ubuntu 환경에서 `gcc --version` 이 버전을 출력하는 상태.
- M003에서 Makefile 기본 구조(타겟·의존성·명령)를 익혔다.
- 이 매뉴얼의 모든 명령은 **Ubuntu 터미널** 안에서 실행합니다.

환경 확인:

```bash
gcc --version
make --version
```

둘 다 버전이 출력되면 OK.

---

## 1. 컴파일이란 무엇인가

### 1.1 사람이 읽는 코드 vs 기계가 읽는 코드

우리가 작성하는 C 코드는 사람이 읽고 이해할 수 있는 **텍스트**입니다. 그러나 CPU는 0과 1로 이루어진 **기계어(machine code)** 만 이해합니다. 이 둘 사이의 변환 과정 전체를 넓은 의미에서 **컴파일(compilation)** 이라고 부릅니다.

```text
hello.c (텍스트) ──컴파일──▶ hello (기계어 실행 파일)
```

### 1.2 컴파일의 4단계

실제로는 단번에 이루어지는 게 아니라 **4개의 단계**를 순서대로 거칩니다.

```text
┌─────────┐   전처리    ┌──────────┐   컴파일    ┌──────────┐
│ hello.c │ ──────────▶ │ hello.i  │ ──────────▶ │ hello.s  │
│ (C 소스)│ (cpp/gcc)  │ (전처리됨)│ (cc1)      │ (어셈블리)│
└─────────┘             └──────────┘             └──────────┘
                                                       │
                                                  어셈블
                                                       ▼
                         링킹      ┌──────────┐   ┌──────────┐
                hello ◀────────── │  libc.a  │   │ hello.o  │
              (실행 파일)          │ (라이브러리)│   │ (오브젝트)│
                                  └──────────┘   └──────────┘
```

| 단계 | 도구 | 입력 | 출력 | 하는 일 |
|------|------|------|------|---------|
| **전처리(preprocessing)** | cpp | `.c` | `.i` | `#include` 파일 삽입, `#define` 매크로 치환, 주석 제거 |
| **컴파일(compilation)** | cc1 | `.i` | `.s` | C 코드를 CPU 명령어인 어셈블리(텍스트)로 변환 |
| **어셈블(assembly)** | as | `.s` | `.o` | 어셈블리를 기계어(바이너리)로 변환 — 오브젝트 파일 |
| **링킹(linking)** | ld | `.o` + 라이브러리 | 실행 파일 | 여러 오브젝트 파일과 라이브러리를 하나로 합침 |

---

## 2. 실습 환경 준비

```bash
mkdir -p ~/embedded-linux/manual004
cd ~/embedded-linux/manual004
```

---

## 3. 소스 파일 작성

이 매뉴얼에서 사용할 C 소스 파일을 만듭니다.

```bash
nano hello.c
```

내용:

```c
#include <stdio.h>

#define GREETING "Hello, Embedded Linux!"

int main(void) {
    printf("%s\n", GREETING);
    return 0;
}
```

각 줄의 의미:

- `#include <stdio.h>` — 전처리 지시자. `stdio.h` 헤더 파일 전체를 이 자리에 삽입하라는 뜻. `printf`의 선언이 이 파일 안에 있음.
- `#define GREETING "Hello, Embedded Linux!"` — 매크로 정의. 이후 코드에서 `GREETING`이라는 글자를 보면 문자열로 치환.
- `int main(void)` — 프로그램의 진입점(entry point). OS가 프로그램을 실행하면 여기서 시작.
- `printf("%s\n", GREETING)` — 형식 문자열로 출력. `%s`는 문자열 자리표시자, `\n`은 줄바꿈.
- `return 0` — `main`의 반환값. `0`은 정상 종료를 의미. 셸에서 `echo $?`로 확인 가능.

---

## 4. 전처리 단계 — `.c` → `.i`

전처리만 단독으로 실행합니다. `-E` 옵션이 전처리에서 멈추라는 뜻입니다.

```bash
gcc -E hello.c -o hello.i
```

결과 파일을 확인합니다.

```bash
wc -l hello.c hello.i
```

출력 예:

```text
  7 hello.c
 842 hello.i
```

`hello.c`는 7줄인데 `hello.i`는 800줄이 넘습니다. `stdio.h`의 내용이 그대로 삽입되었기 때문입니다.

파일 끝부분을 봅니다 (우리가 작성한 코드 부분이 남아 있습니다).

```bash
tail -20 hello.i
```

출력 예:

```text
# 5 "hello.c"
int main(void) {
    printf("%s\n", "Hello, Embedded Linux!");
    return 0;
}
```

주목할 점:

- `#define GREETING`이 사라지고 `GREETING`이 있던 자리에 `"Hello, Embedded Linux!"`가 직접 들어갔습니다 — 매크로 치환이 이루어진 것입니다.
- `#include <stdio.h>` 한 줄이 수백 줄로 펼쳐졌습니다.
- 주석이 모두 사라졌습니다.

이것이 **전처리**가 하는 일입니다.

---

## 5. 컴파일 단계 — `.i` → `.s`

전처리된 파일을 어셈블리 코드로 변환합니다. `-S` 옵션이 어셈블 이전에 멈추라는 뜻입니다.

```bash
gcc -S hello.i -o hello.s
```

또는 소스 파일에서 바로 어셈블리까지:

```bash
gcc -S hello.c -o hello.s
```

어셈블리 파일을 봅니다.

```bash
cat hello.s
```

출력 예 (x86_64):

```asm
	.file	"hello.c"
	.text
	.section	.rodata
.LC0:
	.string	"Hello, Embedded Linux!"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	pushq	%rbp
	movq	%rsp, %rbp
	leaq	.LC0(%rip), %rax
	movq	%rax, %rdi
	call	puts@PLT
	movl	$0, %eax
	popq	%rbp
	ret
```

어셈블리를 이해할 필요는 없습니다. 다만 이 단계에서:

- C의 `printf(...)` 한 줄이 여러 줄의 CPU 명령어로 변환되었습니다.
- `"Hello, Embedded Linux!"` 문자열이 `.rodata`(읽기 전용 데이터) 섹션에 배치되었습니다.
- 이 어셈블리는 **x86_64 전용**입니다. ARM 타겟으로 컴파일하면 완전히 다른 명령어가 나옵니다 — M005에서 확인합니다.

---

## 6. 어셈블 단계 — `.s` → `.o`

어셈블리를 기계어(바이너리)로 변환합니다. `-c` 옵션이 링킹 이전에 멈추라는 뜻입니다.

```bash
gcc -c hello.s -o hello.o
```

또는 소스에서 바로:

```bash
gcc -c hello.c -o hello.o
```

오브젝트 파일을 확인합니다.

```bash
file hello.o
```

출력:

```text
hello.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

- `ELF` — Executable and Linkable Format. 리눅스가 사용하는 바이너리 형식입니다.
- `relocatable` — 아직 최종 주소가 결정되지 않았습니다. 링킹 후 확정됩니다.
- `not stripped` — 디버깅 심볼이 포함되어 있습니다.

오브젝트 파일의 심볼 목록을 봅니다.

```bash
nm hello.o
```

출력 예:

```text
                 U puts
0000000000000000 T main
```

- `T main` — `main` 함수가 코드(Text) 섹션에 정의되어 있습니다.
- `U puts` — `puts` 함수는 Undefined(미정의)입니다. 아직 어디서 가져오는지 모릅니다 — 링킹 단계에서 C 라이브러리에서 연결됩니다.

---

## 7. 링킹 단계 — `.o` → 실행 파일

오브젝트 파일과 라이브러리를 합쳐 최종 실행 파일을 만듭니다.

```bash
gcc hello.o -o hello
```

실행 파일을 확인합니다.

```bash
file hello
```

출력:

```text
hello: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, ...
```

- `executable` — 이제 실행 가능한 상태입니다.
- `dynamically linked` — 실행 시점에 공유 라이브러리(`libc`)를 불러옵니다.
- `interpreter /lib64/ld-linux-x86-64.so.2` — 동적 링커의 경로입니다.

실행합니다.

```bash
./hello
echo $?
```

출력:

```text
Hello, Embedded Linux!
0
```

`$?`가 `0`이므로 정상 종료했습니다.

---

## 8. 한 번에 실행하기 vs 단계별 실행

지금까지 4단계를 각각 분리했지만, 실제로는 `gcc` 한 줄로 전부 처리할 수 있습니다.

```bash
gcc hello.c -o hello
```

이 명령 하나가 내부적으로 전처리 → 컴파일 → 어셈블 → 링킹을 순서대로 수행합니다.

단계별로 나누는 이유:

- **디버깅**: 어느 단계에서 문제가 생겼는지 특정할 수 있습니다.
- **빌드 최적화**: 변경된 파일만 재컴파일(`.c` → `.o`)하고 링킹만 다시 하면 됩니다.
- **크로스 컴파일 이해**: M005에서 같은 흐름으로 ARM용 오브젝트 파일을 만듭니다.

---

## 9. 바이너리 분석 도구

### 9.1 `size` — 섹션별 크기 확인

```bash
size hello.o hello
```

출력 예:

```text
   text    data     bss     dec     hex filename
     67       0       0      67      43 hello.o
   1608     600       8    2216     8a8 hello
```

세 가지 섹션:

- **text** — 실행 코드 (CPU가 실행할 명령어).
- **data** — 초기값이 있는 전역 변수.
- **bss** — 초기값이 없는 전역 변수 (실제로는 파일에 저장되지 않고 실행 시 0으로 초기화됨).

임베디드에서 RAM 크기를 계산할 때 `text + data + bss` 합계를 기준으로 합니다.

### 9.2 `objdump` — 어셈블리 역어셈블

```bash
objdump -d hello
```

실행 파일의 코드 섹션을 어셈블리로 표시합니다. `main` 함수를 찾아봅니다.

```bash
objdump -d hello | grep -A 20 "<main>"
```

출력 예:

```text
0000000000001149 <main>:
    1149:	55                   	push   %rbp
    114a:	48 89 e5             	mov    %rsp,%rbp
    114d:	48 8d 05 b0 0e 00 00 	lea    0xeb0(%rip),%rax
    1154:	48 89 c7             	mov    %rax,%rdi
    1157:	e8 f4 fe ff ff       	call   1050 <puts@plt>
    115c:	b8 00 00 00 00       	mov    $0x0,%eax
    1161:	5d                   	pop    %rbp
    1162:	c3                   	ret
```

왼쪽부터: **주소 | 16진수 기계어 | 어셈블리 니모닉** 입니다.

### 9.3 `ldd` — 동적 라이브러리 의존성 확인

```bash
ldd hello
```

출력 예:

```text
linux-vdso.so.1 (0x00007ffe...)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f...)
/lib64/ld-linux-x86-64.so.2 (0x00007f...)
```

- `libc.so.6` — C 표준 라이브러리. `printf`, `malloc` 등이 여기 있습니다.

임베디드에서는 이 라이브러리가 타겟 보드에도 있어야 합니다. 없으면 실행이 안 됩니다 — 이것이 `rootfs`에 라이브러리를 포함시켜야 하는 이유입니다.

---

## 10. gcc 주요 옵션

### 10.1 경고 옵션

```bash
gcc -Wall -Wextra -Werror hello.c -o hello
```

| 옵션 | 의미 |
|------|------|
| `-Wall` | 일반적인 경고를 모두 활성화 (Wall = all warnings) |
| `-Wextra` | `-Wall`보다 더 많은 추가 경고 활성화 |
| `-Werror` | 경고를 에러로 취급 (경고가 하나라도 있으면 컴파일 실패) |
| `-Wpedantic` | 표준 C에서 벗어난 코드에 경고 |

경고 메시지를 직접 만들어 봅니다.

```bash
nano warn_test.c
```

내용:

```c
#include <stdio.h>

int main(void) {
    int x;                    /* 초기화 안 된 변수 */
    int unused_var = 42;      /* 사용하지 않는 변수 */
    printf("%d\n", x);        /* 초기화 안 된 값 사용 */
    return 0;
}
```

경고 없이 컴파일:

```bash
gcc warn_test.c -o warn_test
./warn_test    # 예측 불가능한 값이 출력됨
```

경고 활성화:

```bash
gcc -Wall -Wextra warn_test.c -o warn_test
```

출력 예:

```text
warn_test.c:5:9: warning: unused variable 'unused_var' [-Wunused-variable]
    5 |     int unused_var = 42;
      |         ^~~~~~~~~~
warn_test.c:6:20: warning: 'x' is used uninitialized [-Wuninitialized]
    6 |     printf("%d\n", x);
```

이 경고들이 임베디드에서 왜 위험한지:

- **초기화 안 된 변수**: RAM의 쓰레기 값이 들어가므로 하드웨어를 잘못 제어할 수 있습니다.
- **미사용 변수**: 코드 검토를 어렵게 하고, 의도하지 않은 로직이 숨어 있을 수 있습니다.

### 10.2 최적화 옵션

```bash
gcc -O0 hello.c -o hello_O0    # 최적화 없음 (디버깅에 적합)
gcc -O1 hello.c -o hello_O1    # 기본 최적화
gcc -O2 hello.c -o hello_O2    # 권장 릴리스 최적화
gcc -O3 hello.c -o hello_O3    # 적극적 최적화 (크기 증가 가능)
gcc -Os hello.c -o hello_Os    # 크기 최적화 (임베디드에서 유용)
gcc -Og hello.c -o hello_Og    # 디버깅에 적합한 수준의 최적화
```

크기를 비교합니다.

```bash
size hello_O0 hello_O2 hello_Os
```

출력 예:

```text
   text    data     bss     dec     hex filename
   1608     600       8    2216     8a8 hello_O0
   1432     600       8    2040     7f8 hello_O2
   1400     600       8    2008     7d8 hello_Os
```

임베디드에서 플래시 용량이 부족할 때 `-Os`를 사용합니다.

### 10.3 디버그 심볼 포함 — `-g`

```bash
gcc -g hello.c -o hello_debug
file hello_debug
```

출력에 `with debug_info`가 포함됩니다. `gdb` 디버거로 소스 레벨에서 실행을 추적할 수 있게 됩니다. 나중에 M012에서 사용합니다.

### 10.4 자주 쓰는 옵션 조합

| 목적 | 옵션 조합 |
|------|-----------|
| 개발 중 | `-Wall -Wextra -g -O0` |
| 릴리스 | `-Wall -Wextra -O2` |
| 임베디드 릴리스 | `-Wall -Wextra -Os` |
| 임베디드 + 디버그 | `-Wall -Wextra -g -Og` |

---

## 11. 여러 소스 파일 분리하기

실제 프로젝트는 파일 하나가 아닙니다. 역할에 따라 파일을 나누는 것이 좋은 습관입니다.

### 11.1 파일 구성

```bash
cd ~/embedded-linux/manual004
```

세 개의 파일로 나눕니다.

**`calc.h`** — 함수 선언 (헤더 파일)

```bash
nano calc.h
```

내용:

```c
#ifndef CALC_H
#define CALC_H

int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);

#endif /* CALC_H */
```

코드 설명:

- `#ifndef CALC_H` / `#define CALC_H` / `#endif` — **헤더 가드(include guard)**. 같은 헤더가 여러 번 포함되어 중복 선언 오류가 나는 것을 방지합니다.
- `int add(int a, int b);` — 함수 **선언(declaration)**. 세미콜론으로 끝나고 본문이 없습니다.

**`calc.c`** — 함수 구현 (소스 파일)

```bash
nano calc.c
```

내용:

```c
#include "calc.h"

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

int multiply(int a, int b) {
    return a * b;
}
```

코드 설명:

- `#include "calc.h"` — 따옴표(`"`)는 시스템 경로가 아닌 **현재 디렉토리**에서 먼저 찾습니다. `<>`는 시스템 헤더 경로.
- 각 함수가 본문(`{ ... }`)을 가집니다 — **정의(definition)**.

**`main.c`** — 진입점

```bash
nano main.c
```

내용:

```c
#include <stdio.h>
#include "calc.h"

int main(void) {
    int a = 10;
    int b = 3;

    printf("%d + %d = %d\n", a, b, add(a, b));
    printf("%d - %d = %d\n", a, b, subtract(a, b));
    printf("%d * %d = %d\n", a, b, multiply(a, b));

    return 0;
}
```

### 11.2 수동 컴파일

```bash
gcc -c calc.c -o calc.o
gcc -c main.c -o main.o
gcc calc.o main.o -o calculator
./calculator
```

출력:

```text
10 + 3 = 13
10 - 3 = 7
10 * 3 = 30
```

각 `.c` 파일을 독립적으로 `.o`로 만들고, 마지막에 링킹합니다.

### 11.3 Makefile로 자동화

```bash
nano Makefile
```

내용 (명령 앞은 반드시 Tab):

```makefile
# 컴파일러 및 옵션
CC      = gcc
CFLAGS  = -Wall -Wextra -g

# 타겟 및 소스
TARGET  = calculator
SRCS    = main.c calc.c
OBJS    = $(SRCS:.c=.o)      # .c를 .o로 치환: main.o calc.o

# 기본 타겟
all: $(TARGET)

# 실행 파일 생성: 모든 오브젝트 파일이 있어야 함
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^
	@echo "빌드 완료: $@"

# 오브젝트 파일 생성: %.o 는 %.c 에 의존
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 정리
clean:
	rm -f $(OBJS) $(TARGET)
	@echo "정리 완료"

# 정보 출력
info:
	@echo "타겟:  $(TARGET)"
	@echo "소스:  $(SRCS)"
	@echo "오브젝트: $(OBJS)"

.PHONY: all clean info
```

코드에서 새로 등장한 개념:

- `$(SRCS:.c=.o)` — 패턴 치환. `main.c calc.c` 에서 `.c`를 `.o`로 바꿔 `main.o calc.o`를 만듭니다.
- `%.o: %.c` — **패턴 규칙(pattern rule)**. `어떤이름.o`는 `어떤이름.c`에 의존한다는 규칙입니다. 파일마다 규칙을 쓸 필요가 없습니다.
- `$^` — 모든 의존성 파일 목록 (링킹 시 모든 `.o`를 넘길 때 사용).

### 11.4 증분 빌드 확인

처음 빌드:

```bash
make
```

```text
gcc -Wall -Wextra -g -c main.c -o main.o
gcc -Wall -Wextra -g -c calc.c -o calc.o
gcc -Wall -Wextra -g -o calculator main.o calc.o
빌드 완료: calculator
```

`calc.c`만 수정합니다 (아무 줄이나 주석 추가).

```bash
nano calc.c
```

`add` 함수 앞에 주석 한 줄 추가 후 저장.

다시 빌드:

```bash
make
```

```text
gcc -Wall -Wextra -g -c calc.c -o calc.o
gcc -Wall -Wextra -g -o calculator main.o calc.o
빌드 완료: calculator
```

`main.c`는 변경되지 않았으므로 `main.o`는 재컴파일하지 않았습니다. 파일이 수십 개라면 이 차이가 빌드 시간을 크게 단축합니다. 이것이 **증분 빌드(incremental build)** 입니다.

### 11.5 빌드 정보 확인

```bash
make info
./calculator
size calculator
ldd calculator
```

---

## 12. 정적 라이브러리 만들기 (맛보기)

임베디드에서는 라이브러리를 직접 만들어 링크할 일이 많습니다.

### 12.1 정적 라이브러리 생성

```bash
ar rcs libcalc.a calc.o
```

`ar`은 archive(묶음 파일)를 다루는 도구입니다.

- `r` — 파일을 아카이브에 추가(replace).
- `c` — 아카이브 파일이 없으면 생성(create).
- `s` — 심볼 인덱스 생성(symbol index).

생성된 라이브러리 확인:

```bash
file libcalc.a
ar t libcalc.a     # 라이브러리 안에 포함된 파일 목록
```

### 12.2 라이브러리 링크

```bash
gcc main.o -L. -lcalc -o calculator_static
./calculator_static
```

- `-L.` — 현재 디렉토리(`.`)에서 라이브러리를 찾습니다.
- `-lcalc` — `libcalc.a` 또는 `libcalc.so`를 링크합니다. (`lib` 접두어와 확장자는 자동으로 붙임)

`ldd calculator_static`으로 확인하면 `libcalc`가 동적 라이브러리로는 표시되지 않습니다 — 정적으로 실행 파일 안에 포함되었기 때문입니다.

---

## 13. 종합 실습 — 정리 및 전체 빌드

지금까지 만든 파일들을 한 번에 정리합니다.

```bash
ls -la ~/embedded-linux/manual004/
```

```text
hello.c   hello.i   hello.s   hello.o   hello
calc.c    calc.h    calc.o    libcalc.a
main.c    main.o    calculator  calculator_static
Makefile
warn_test.c  ...
```

`make clean` 으로 중간 산출물을 지우고 다시 빌드합니다.

```bash
make clean
ls -la
make
./calculator
```

---

## 14. 자주 발생하는 에러와 해결법

### 에러 1: `undefined reference to 'add'`

```text
main.o: In function `main':
main.c:(.text+0x1e): undefined reference to `add'
```

`calc.o`를 링킹에 포함하지 않았습니다. `gcc main.o calc.o -o ...` 처럼 모든 오브젝트를 포함합니다.

### 에러 2: `implicit declaration of function`

```text
main.c:7:5: warning: implicit declaration of function 'add' [-Wimplicit-function-declaration]
```

`calc.h`를 `#include`하지 않았습니다. `main.c` 상단에 `#include "calc.h"`를 추가합니다.

### 에러 3: `multiple definition of 'main'`

소스 파일 두 개에 `main` 함수가 있는 경우입니다. `main`은 전체 프로젝트에 **정확히 하나**여야 합니다.

### 에러 4: Makefile `*** missing separator`

명령 줄이 Tab이 아닌 스페이스로 시작합니다.

```bash
cat -A Makefile | grep "^    "    # 스페이스로 시작하는 줄 찾기
```

에디터에서 해당 줄의 들여쓰기를 스페이스 대신 Tab으로 교체합니다.

---

## 15. 정리 및 다음 단계

이 매뉴얼에서 한 일:

- 컴파일의 4단계(전처리→컴파일→어셈블→링킹)를 `-E`, `-S`, `-c` 옵션으로 분리 실행했다.
- `.i`, `.s`, `.o` 중간 산출물을 직접 확인했다.
- `file`, `nm`, `size`, `objdump`, `ldd` 로 바이너리를 분석했다.
- `-Wall`, `-Wextra`, `-O2`, `-Os`, `-g` 등 주요 gcc 옵션을 사용했다.
- 소스 파일을 헤더/구현/진입점으로 분리하고 Makefile 패턴 규칙(`%.o: %.c`)으로 자동화했다.
- 증분 빌드가 변경된 파일만 재컴파일하는 것을 직접 확인했다.
- 정적 라이브러리(`libcalc.a`)를 만들고 링크했다.

**지금까지의 흐름:**

```text
M001: 개념 이해
M002: Ubuntu 환경 구축
M003: 셸 명령, 스크립트, Makefile 기초
M004: C 컴파일 4단계, 바이너리 분석, 멀티파일 빌드  ← 지금 여기
M005: 같은 코드를 ARM용으로 크로스 컴파일 → 다음 단계
```

다음 매뉴얼 **M005 — 크로스 컴파일러 설치 + ARM 바이너리 생성**에서는 이 매뉴얼에서 만든 `hello.c`와 `calculator`를 **ARM 타겟용**으로 다시 빌드합니다. `gcc`를 `arm-linux-gnueabihf-gcc`로 바꾸는 것 외에 Makefile을 거의 수정하지 않아도 됩니다 — `CC` 변수 한 줄만 바꾸면 됩니다.

---

## 부록 A — gcc 옵션 빠른 참조

| 옵션 | 단계 | 의미 |
|------|------|------|
| `-E` | 전처리까지 | `.i` 파일 생성 |
| `-S` | 컴파일까지 | `.s` 어셈블리 파일 생성 |
| `-c` | 어셈블까지 | `.o` 오브젝트 파일 생성 |
| `-o <파일>` | 출력 파일 이름 지정 | |
| `-Wall` | 일반 경고 모두 활성화 | |
| `-Wextra` | 추가 경고 활성화 | |
| `-Werror` | 경고를 에러로 처리 | |
| `-g` | 디버그 심볼 포함 | gdb 사용 시 필요 |
| `-O0` | 최적화 없음 | |
| `-O2` | 표준 최적화 | |
| `-Os` | 크기 최적화 | 임베디드에 유용 |
| `-I<경로>` | 헤더 파일 검색 경로 추가 | |
| `-L<경로>` | 라이브러리 검색 경로 추가 | |
| `-l<이름>` | 라이브러리 링크 | `-lm` = libm.so |

## 부록 B — 바이너리 분석 도구

| 명령 | 용도 |
|------|------|
| `file <파일>` | 파일 타입 감지 (ELF, relocatable, executable 등) |
| `nm <파일>` | 심볼 목록 (T=코드, U=미정의, D=데이터) |
| `size <파일>` | text/data/bss 섹션별 크기 |
| `objdump -d <파일>` | 코드 섹션 역어셈블 |
| `objdump -h <파일>` | 섹션 헤더 목록 |
| `ldd <실행파일>` | 동적 라이브러리 의존성 |
| `strings <파일>` | 바이너리 안의 문자열 추출 |
| `hexdump -C <파일>` | 16진수+ASCII 덤프 |
| `readelf -a <파일>` | ELF 파일 전체 정보 |

## 부록 C — 이 매뉴얼에서 사용한 명령어 한 줄 풀이

| 명령 | 한 줄 풀이 |
|------|-----------|
| `gcc -E hello.c -o hello.i` | 전처리만 수행, 결과를 .i 파일로 저장 |
| `gcc -S hello.c -o hello.s` | 어셈블리까지 수행, .s 파일로 저장 |
| `gcc -c hello.c -o hello.o` | 오브젝트 파일까지 생성, 링킹 안 함 |
| `gcc hello.o -o hello` | 오브젝트 파일을 링크해 실행 파일 생성 |
| `gcc hello.c -o hello` | 전 단계를 한 번에 수행 |
| `file <파일>` | 파일 타입 감지 |
| `nm <파일>` | 심볼 목록 출력 |
| `size <파일>` | 섹션 크기 출력 |
| `objdump -d <파일>` | 역어셈블 |
| `ldd <실행파일>` | 동적 라이브러리 의존성 출력 |
| `ar rcs libcalc.a calc.o` | 정적 라이브러리 생성 |
| `ar t libcalc.a` | 라이브러리 내 파일 목록 |

---

*다음 매뉴얼: `manual005.md` — 크로스 컴파일러 설치 + ARM 바이너리 생성 (작성 예정)*
