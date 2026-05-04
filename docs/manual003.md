# Manual 003 — 터미널·셸·기본 명령 워밍업

> 시리즈: **임베디드 리눅스 실습 교과서**
> 버전: v0.6.0
> 작성일: 2026-05-04
> 선행 매뉴얼: [manual002.md](manual002.md) — 호스트 개발 환경 구축
> 다음 매뉴얼: `manual004.md` — 첫 C 프로그램 네이티브 컴파일 (예정)

---

## 학습 목표

이 매뉴얼을 끝까지 따라 한 후 여러분은 다음을 할 수 있어야 합니다.

1. 파일과 디렉토리를 터미널에서 자유롭게 만들고, 복사하고, 이동하고, 삭제할 수 있다.
2. `cat`, `less`, `grep`, `find` 로 파일 내용을 조회·검색할 수 있다.
3. 파이프(`|`)와 리디렉션(`>`, `>>`)의 차이를 설명하고 직접 사용할 수 있다.
4. 환경 변수를 설정하고 셸 스크립트를 작성·실행할 수 있다.
5. `Makefile`의 기본 구조를 읽고 직접 작성할 수 있다 (빌드 자동화 기초).
6. 파일 권한(`rwx`) 표기를 읽고 `chmod`로 변경할 수 있다.

> **예상 소요 시간**: 90 ~ 120분 (실습 중심)

---

## 사전 지식 점검

- [manual002.md](manual002.md) 의 Ubuntu 환경이 준비되어 있고 필수 패키지가 설치된 상태.
- 이 매뉴얼의 **모든 명령은 Ubuntu 터미널 안에서** 실행합니다.
  - Multipass 사용자: `multipass shell embedded-dev` 로 접속 후 진행.
  - WSL2 사용자: Ubuntu 창에서 진행.
  - Native Ubuntu 사용자: 터미널을 열면 바로 시작.

접속 확인:

```bash
uname -a
```

`Linux`로 시작하는 출력이 나오면 Ubuntu 안에 있는 것입니다.

---

## 1. 셸이란 무엇인가

### 1.1 사용자와 운영체제 사이의 통역사

터미널에 `ls`를 입력하면 무슨 일이 일어날까요? 여러분이 친 글자는 **셸(shell)** 이라는 프로그램이 먼저 받습니다. 셸은 이것이 `ls`라는 명령임을 알아채고, 운영체제 커널에게 "ls 프로그램을 실행해 달라"고 요청합니다. 커널은 실행 결과를 다시 셸에게 돌려주고, 셸이 화면에 출력합니다.

```text
사용자 입력 → [ 셸 ] → 커널 → 하드웨어
              결과 ←       ←
```

셸은 이렇게 사람과 운영체제 사이의 **통역사** 역할을 합니다. 이 매뉴얼에서 사용할 셸은 `bash`(Bourne Again Shell)입니다.

### 1.2 프롬프트 읽기

Ubuntu의 기본 프롬프트:

```text
ubuntu@embedded-dev:~$
```

각 부분의 의미:

- `ubuntu` — 현재 로그인한 **사용자 이름**.
- `embedded-dev` — 이 컴퓨터(인스턴스)의 **이름**.
- `~` — 현재 작업 디렉토리. `~`은 홈 디렉토리의 약식 표기.
- `$` — 일반 사용자를 뜻하는 기호. `#`이면 루트(최고 관리자) 사용자.

디렉토리를 이동하면 `~` 부분이 바뀌는 것을 확인할 수 있습니다.

```bash
cd /etc
```

프롬프트가 다음처럼 바뀝니다.

```text
ubuntu@embedded-dev:/etc$
```

다시 홈으로 돌아갑니다.

```bash
cd ~
```

---

## 2. 파일과 디렉토리 조작

### 2.1 실습 작업 공간 만들기

이 매뉴얼의 실습은 전용 디렉토리 안에서 진행합니다. 실수로 다른 파일을 건드리지 않기 위해서입니다.

```bash
mkdir -p ~/embedded-linux/manual003
cd ~/embedded-linux/manual003
pwd
```

출력:

```text
/home/ubuntu/embedded-linux/manual003
```

이제 이 경로가 실습 기준 위치입니다.

### 2.2 디렉토리 목록 보기 — `ls`

```bash
ls
```

아직 아무것도 없어서 출력이 비어 있습니다. 옵션을 붙여 더 자세히 봅니다.

```bash
ls -l
```

`-l`은 long format의 약자입니다. 파일이 없어도 현재 위치 정보가 표시됩니다.

```bash
ls -la
```

`-a`는 all의 약자로, 이름이 `.`으로 시작하는 **숨김 파일까지 포함**합니다. `.`과 `..` 두 항목이 보입니다.

```text
total 0
drwxr-xr-x 2 ubuntu ubuntu  40 May  4 03:00 .
drwxr-xr-x 3 ubuntu ubuntu  60 May  4 03:00 ..
```

- `.` — 현재 디렉토리 자기 자신.
- `..` — 상위 디렉토리.

유용한 `ls` 옵션 조합:

```bash
ls -lh       # 파일 크기를 K/M/G 단위로 표시 (-h: human-readable)
ls -lhS      # 크기 큰 순으로 정렬 (-S: Size)
ls -lht      # 최신 순으로 정렬 (-t: time)
ls -lha      # 숨김 파일 포함, 자세히, 읽기 좋은 크기 단위
```

옵션들은 자유롭게 조합할 수 있습니다. `ls -l -h -a`와 `ls -lha`는 완전히 같습니다.

### 2.3 파일 만들기

**방법 1 — `touch`**

```bash
touch hello.txt
ls -la
```

`touch`는 파일이 없으면 **빈 파일**을 만들고, 이미 있으면 **최종 수정 시각만 현재로 갱신**합니다. 크기가 0인 것을 확인할 수 있습니다.

**방법 2 — 리디렉션 `>`**

```bash
echo "안녕하세요, 임베디드 리눅스!" > greeting.txt
cat greeting.txt
```

`echo "텍스트"` 는 텍스트를 화면에 출력합니다. `>`는 **출력을 파일로 돌리는(redirect)** 연산자입니다.

> **주의**: `>`는 파일을 **덮어씁니다**. 기존 내용이 있으면 모두 사라집니다.

**방법 3 — 리디렉션 `>>` (이어 쓰기)**

```bash
echo "첫 번째 줄" > lines.txt
echo "두 번째 줄" >> lines.txt
echo "세 번째 줄" >> lines.txt
cat lines.txt
```

출력:

```text
첫 번째 줄
두 번째 줄
세 번째 줄
```

`>>`는 파일이 없으면 새로 만들고, 있으면 **끝에 이어 붙입니다**. `docs/km.md`의 append-only(추가 전용) 방식이 이 원리입니다.

### 2.4 파일 복사 — `cp`

```bash
cp greeting.txt greeting_backup.txt
ls -la
```

`cp <원본> <사본>` 형식입니다. 원본은 그대로 남고 사본이 새로 생깁니다.

디렉토리째로 복사하려면 `-r`(recursive) 옵션이 필요합니다.

```bash
mkdir -p subdir_a
cp -r subdir_a subdir_b
ls -la
```

### 2.5 파일 이동·이름 변경 — `mv`

```bash
mv greeting_backup.txt greeting_copy.txt
ls -la
```

같은 디렉토리 내에서 `mv`를 쓰면 **이름 변경**이 됩니다.

```bash
mkdir -p archive
mv greeting_copy.txt archive/
ls -la archive/
```

다른 디렉토리로 이동할 때는 목적지 경로를 지정합니다.

### 2.6 파일 삭제 — `rm`

```bash
rm hello.txt
ls -la
```

파일이 사라졌습니다. 유닉스에서 `rm`은 **휴지통이 없습니다** — 삭제하면 복구가 거의 불가능합니다.

디렉토리 삭제:

```bash
rmdir archive    # 빈 디렉토리만 삭제 가능
```

비어 있지 않아서 오류가 납니다.

```text
rmdir: failed to remove 'archive': Directory not empty
```

안에 든 파일까지 함께 지우려면:

```bash
rm -r archive
ls -la
```

`-r`은 recursive(재귀)의 약자입니다. 디렉토리 안을 모두 삭제합니다.

> `-f`(force)를 함께 쓰면 확인 질문 없이 강제 삭제합니다. 경로를 잘못 지정하면 복구가 불가능하므로 신중하게 사용합니다.

### 2.7 경로 이해하기 — 절대 경로 vs 상대 경로

**절대 경로(absolute path)**: 항상 `/`(루트)에서 시작합니다.

```bash
ls /home/ubuntu/embedded-linux/manual003
```

어디서 실행하든 같은 위치를 가리킵니다.

**상대 경로(relative path)**: 현재 위치(`pwd`)를 기준으로 합니다.

```bash
cd ~/embedded-linux/manual003    # 기준 위치로 이동
ls .                             # 현재 디렉토리
ls ..                            # 상위 디렉토리
ls ../..                         # 두 단계 위
```

특수 경로 표기:

| 표기 | 의미 |
|------|------|
| `.` | 현재 디렉토리 |
| `..` | 상위 디렉토리 |
| `~` | 현재 사용자의 홈 디렉토리 |
| `/` | 최상위 루트 디렉토리 |

직접 이동하며 확인합니다.

```bash
cd ~/embedded-linux/manual003
pwd

cd ..
pwd

cd ./manual003
pwd

cd ~
pwd

cd -          # 직전 작업 디렉토리로 토글
pwd
```

`cd -`는 이전 위치와 현재 위치 사이를 왔다갔다할 때 매우 편리합니다.

---

## 3. 파일 내용 조회

실습용 파일을 준비합니다.

```bash
cd ~/embedded-linux/manual003
```

### 3.1 `cat` — 전체 내용 출력

```bash
cat lines.txt
```

파일 전체를 한 번에 출력합니다. 파일이 짧을 때 적합합니다.

줄 번호와 함께 보려면:

```bash
cat -n lines.txt
```

출력:

```text
     1	첫 번째 줄
     2	두 번째 줄
     3	세 번째 줄
```

### 3.2 `less` — 스크롤 가능한 뷰어

긴 파일을 볼 때는 `less`를 사용합니다.

```bash
less /etc/os-release
```

`less` 안에서 사용하는 키:

| 키 | 동작 |
|----|------|
| `↓` / `j` | 한 줄 아래 |
| `↑` / `k` | 한 줄 위 |
| `Space` / `f` | 한 화면 아래 |
| `b` | 한 화면 위 |
| `G` | 파일 끝으로 |
| `g` | 파일 처음으로 |
| `/검색어` + `Enter` | 아래 방향으로 검색 |
| `n` | 다음 검색 결과 |
| `N` | 이전 검색 결과 |
| `q` | 종료 |

### 3.3 `head`와 `tail` — 일부만 보기

```bash
head -5 /etc/os-release       # 처음 5줄
tail -5 /etc/os-release       # 마지막 5줄
```

로그 파일을 실시간으로 따라가려면:

```bash
tail -f /var/log/syslog
```

`-f`는 follow의 약자입니다. 파일에 새 줄이 추가되면 바로 화면에 나타납니다. `Ctrl+C`로 종료합니다.

### 3.4 `grep` — 패턴 검색

`grep`은 파일에서 특정 패턴이 포함된 줄만 골라냅니다.

```bash
grep "NAME" /etc/os-release
```

출력 예:

```text
NAME="Ubuntu"
PRETTY_NAME="Ubuntu 24.04.2 LTS (Noble Numbat)"
VERSION_CODENAME=noble
```

자주 쓰는 옵션:

```bash
grep -i "ubuntu" /etc/os-release      # -i: 대소문자 구분 없이
grep -n "NAME" /etc/os-release        # -n: 줄 번호 표시
grep -v "NAME" /etc/os-release        # -v: 해당 패턴 제외(invert)
grep -c "NAME" /etc/os-release        # -c: 매칭 줄 수만 출력
grep -r "ubuntu" /etc/default/        # -r: 디렉토리 재귀 검색
```

패턴에 `^`(줄의 시작)과 `$`(줄의 끝)을 쓸 수 있습니다.

```bash
grep "^NAME=" /etc/os-release         # NAME= 으로 시작하는 줄만
grep "LTS$" /etc/os-release           # LTS 로 끝나는 줄만 (없을 수 있음)
```

### 3.5 `find` — 파일 검색

`find`는 디렉토리 안에서 조건에 맞는 파일을 찾습니다.

```bash
find ~/embedded-linux -name "*.txt"       # 확장자로 검색
find ~/embedded-linux -type d             # 디렉토리만
find ~/embedded-linux -type f             # 일반 파일만
find ~/embedded-linux -type f -size +1k  # 1KB 초과 파일
find ~/embedded-linux -name "*.txt" -type f    # 확장자 + 파일 타입 조합
```

형식: `find <검색 시작 경로> <조건...>`

---

## 4. 파이프 — 명령을 연결하는 `|`

### 4.1 파이프의 개념

파이프(`|`)는 **앞 명령의 출력을 뒤 명령의 입력으로 전달**합니다.

```text
명령A | 명령B | 명령C
```

마치 수도관처럼 데이터가 한 방향으로 흐릅니다.

### 4.2 파이프 실습 1 — 필터링

```bash
ls -la /etc | grep "conf"
```

`ls -la /etc`의 출력 중 `conf`가 포함된 줄만 통과시킵니다.

### 4.3 파이프 실습 2 — 줄 수 세기

```bash
ls /etc | wc -l
```

`wc`는 word count의 약자입니다. `-l`은 줄(line) 수만 세는 옵션입니다. `/etc` 안의 항목 개수를 셉니다.

### 4.4 파이프 실습 3 — 단계별 조합

OS 이름만 깔끔하게 추출해 보겠습니다. 한 단계씩 확인하면서 진행합니다.

```bash
# 1단계: 전체 파일 출력
cat /etc/os-release
```

```bash
# 2단계: NAME= 으로 시작하는 줄만
cat /etc/os-release | grep "^NAME="
```

출력:

```text
NAME="Ubuntu"
```

```bash
# 3단계: = 뒤의 값만 잘라내기
cat /etc/os-release | grep "^NAME=" | cut -d= -f2
```

출력:

```text
"Ubuntu"
```

`cut -d= -f2`는 `=`를 구분자(`-d`)로 삼아 두 번째(`-f2`) 필드를 추출합니다.

```bash
# 4단계: 따옴표 제거
cat /etc/os-release | grep "^NAME=" | cut -d= -f2 | tr -d '"'
```

출력:

```text
Ubuntu
```

`tr -d '"'`는 `"` 문자를 모두 제거합니다. `tr`은 translate의 약자입니다.

이처럼 파이프는 간단한 도구들을 연결해 복잡한 처리를 만들어냅니다. 임베디드 빌드 스크립트에서도 이 패턴이 매우 자주 등장합니다.

---

## 5. 환경 변수

### 5.1 환경 변수란

**환경 변수(environment variable)** 는 셸 안에 저장된 이름=값 쌍입니다. 프로그램은 이 값을 읽어 동작 방식을 결정합니다.

```bash
echo $PATH
```

출력 예:

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

`PATH`는 명령 실행 파일을 어디서 찾을지 알려주는 변수입니다. 셸은 `ls`를 입력받으면 `PATH`에 적힌 디렉토리를 순서대로 뒤져 실행 파일을 찾습니다.

```bash
which ls        # ls 실행 파일의 실제 위치
which gcc
which python3
```

### 5.2 변수 설정

```bash
MY_NAME="Embedded Engineer"
echo $MY_NAME
```

이렇게 설정한 변수는 **현재 셸 세션에서만** 유효합니다. 터미널을 닫으면 사라집니다.

자식 프로세스(새로 실행하는 명령)에도 전달하려면 `export`가 필요합니다.

```bash
export WORK_DIR=~/embedded-linux
echo $WORK_DIR

# 새 bash 프로세스를 띄워서 변수가 전달됐는지 확인
bash -c 'echo "자식 프로세스에서: $WORK_DIR"'
```

`export` 없이 설정하면:

```bash
NO_EXPORT="테스트"
bash -c 'echo $NO_EXPORT'    # 비어 있음
```

### 5.3 영구 설정 — `.bashrc`

터미널을 새로 열어도 변수가 남아 있게 하려면 셸 초기화 파일에 추가합니다.

```bash
echo 'export EMBEDDED_STUDY=~/embedded-linux' >> ~/.bashrc
source ~/.bashrc
echo $EMBEDDED_STUDY
```

- `~/.bashrc` — bash 셸이 시작될 때마다 자동 실행되는 스크립트.
- `source <파일>` — 파일을 **현재 셸에서** 실행합니다 (새 프로세스 생성 없음). 변경 사항을 재시작 없이 즉시 적용할 때 사용합니다.

### 5.4 주요 내장 환경 변수

```bash
echo $HOME      # 홈 디렉토리 경로
echo $USER      # 현재 사용자 이름
echo $PWD       # 현재 작업 디렉토리
echo $SHELL     # 현재 셸 경로
echo $PATH      # 명령 검색 경로
echo $?         # 직전 명령의 종료 코드 (0=성공, 그 외=오류)
```

`$?`는 임베디드 빌드 스크립트에서 명령 성공 여부를 확인할 때 자주 쓰입니다.

```bash
ls /etc/os-release    # 파일이 있으므로 성공
echo $?               # 0 출력

ls /not/exist         # 없는 경로라서 실패
echo $?               # 2 출력 (0이 아닌 값 = 오류)
```

---

## 6. 셸 스크립트 기초

반복 작업이나 여러 명령의 묶음을 파일로 저장해두면 한 번에 실행할 수 있습니다.

### 6.1 첫 셸 스크립트 작성

```bash
cd ~/embedded-linux/manual003
```

텍스트 에디터로 `system_info.sh`를 만듭니다. (nano는 Ubuntu에 기본 설치되어 있습니다.)

```bash
nano system_info.sh
```

다음 내용을 입력합니다. (`Ctrl+O` → `Enter`로 저장, `Ctrl+X`로 종료)

```bash
#!/bin/bash
# 시스템 정보를 출력하는 스크립트

echo "=== 시스템 정보 ==="
echo "사용자:  $USER"
echo "호스트:  $(hostname)"
echo "커널:    $(uname -r)"
echo "아키텍처: $(uname -m)"
echo "날짜:    $(date '+%Y-%m-%d %H:%M:%S')"
echo ""
echo "=== 디스크 사용량 ==="
df -h /
echo ""
echo "=== 메모리 ==="
free -h
```

코드 설명:

- `#!/bin/bash` — **셔뱅(shebang)**: 이 파일을 `bash`로 실행하라는 선언. 반드시 첫 번째 줄에.
- `# 주석` — `#`으로 시작하는 줄은 주석입니다. 실행되지 않습니다.
- `$(명령)` — **명령 치환(command substitution)**: 괄호 안 명령을 실행하고 그 출력으로 대체합니다.
- `date '+%Y-%m-%d %H:%M:%S'` — 날짜 형식을 지정합니다. `+` 뒤의 문자열이 형식입니다.

### 6.2 실행 권한 부여

작성된 파일을 확인합니다.

```bash
ls -la system_info.sh
```

```text
-rw-r--r-- 1 ubuntu ubuntu 280 May  4 03:30 system_info.sh
```

권한 부분 `-rw-r--r--`에 실행(`x`) 권한이 없습니다. 부여합니다.

```bash
chmod +x system_info.sh
ls -la system_info.sh
```

```text
-rwxr-xr-x 1 ubuntu ubuntu 280 May  4 03:30 system_info.sh
```

`x`가 추가되었습니다.

### 6.3 스크립트 실행

```bash
./system_info.sh
```

출력 예:

```text
=== 시스템 정보 ===
사용자:  ubuntu
호스트:  embedded-dev
커널:    6.8.0-51-generic
아키텍처: x86_64
날짜:    2026-05-04 03:30:00

=== 디스크 사용량 ===
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.2G   16G  17% /

=== 메모리 ===
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       512Mi       2.9Gi       ...
```

`./`는 "현재 디렉토리의"를 의미합니다. `PATH`에 현재 디렉토리(`.`)가 없으므로 위치를 명시해야 합니다.

### 6.4 변수와 조건문

```bash
nano ~/embedded-linux/manual003/check_space.sh
```

내용:

```bash
#!/bin/bash
# 디스크 여유 공간을 확인하고 경고를 출력하는 스크립트

THRESHOLD=80    # 경고 임계값 (%)
PARTITION="/"

# df 명령에서 사용률(Use%)만 추출
USAGE=$(df "$PARTITION" | tail -1 | awk '{print $5}' | tr -d '%')

echo "파티션 $PARTITION 사용률: ${USAGE}%"

if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "경고: 디스크 사용률이 ${THRESHOLD}% 이상입니다!"
    echo "불필요한 파일을 정리하세요."
else
    echo "정상: 디스크 여유 공간이 충분합니다."
fi
```

코드 설명:

- `THRESHOLD=80` — 변수 선언. 등호 양쪽에 공백이 없어야 합니다.
- `$(df ...)` — 명령 치환으로 df 출력을 변수에 저장.
- `tail -1` — 마지막 줄(파티션 정보 줄)만.
- `awk '{print $5}'` — 다섯 번째 필드(Use%)를 추출. `awk`는 텍스트 처리 도구입니다.
- `tr -d '%'` — `%` 문자 제거.
- `if [ 조건 ]; then ... else ... fi` — 조건문. `fi`는 `if`를 거꾸로 쓴 것으로 블록의 끝을 나타냅니다.
- `-ge` — greater than or equal to (이상).

```bash
chmod +x ~/embedded-linux/manual003/check_space.sh
~/embedded-linux/manual003/check_space.sh
```

### 6.5 반복문

```bash
nano ~/embedded-linux/manual003/check_tools.sh
```

내용:

```bash
#!/bin/bash
# 필수 도구가 설치되어 있는지 확인하는 스크립트

TOOLS="gcc make git wget curl python3 bc bison flex"
ALL_OK=true

echo "=== 필수 도구 확인 ==="
for tool in $TOOLS; do
    if command -v "$tool" > /dev/null 2>&1; then
        VERSION=$("$tool" --version 2>&1 | head -1)
        printf "  [OK]      %-10s %s\n" "$tool" "$VERSION"
    else
        printf "  [없음]    %-10s 설치 필요\n" "$tool"
        ALL_OK=false
    fi
done

echo ""
if $ALL_OK; then
    echo "모든 도구가 준비되었습니다."
else
    echo "누락된 도구가 있습니다. sudo apt install 로 설치하세요."
fi
```

코드 설명:

- `for tool in $TOOLS; do ... done` — `TOOLS` 변수의 값을 공백으로 나눠 하나씩 반복.
- `command -v "$tool"` — 명령이 존재하는지 확인. 존재하면 경로를 출력하고 종료 코드 0.
- `> /dev/null 2>&1` — 표준 출력과 에러를 모두 버림. `/dev/null`은 출력을 버리는 특수 파일.
- `printf "  %-10s %s\n" ...` — 형식 지정 출력. `%-10s`는 왼쪽 정렬 10자리 문자열.

```bash
chmod +x ~/embedded-linux/manual003/check_tools.sh
~/embedded-linux/manual003/check_tools.sh
```

---

## 7. Makefile 기초

### 7.1 Makefile이 필요한 이유

C 파일 10개를 매번 손으로 컴파일할 수는 없습니다. 파일 하나를 고쳤을 때 변경된 파일만 재컴파일하면 좋겠습니다. **Makefile**은 이런 **빌드 자동화**를 담당합니다. 리눅스 커널 자체도 수천 줄의 Makefile로 빌드됩니다.

### 7.2 Makefile 기본 구조

```makefile
타겟: 의존성
	명령
```

세 가지 구성 요소:

- **타겟(target)**: 만들어낼 파일 이름, 또는 수행할 작업 이름.
- **의존성(dependency)**: 타겟을 만들기 전에 먼저 있어야 하는 파일.
- **명령(recipe)**: 타겟을 만드는 셸 명령. **반드시 Tab 문자로 들여씁니다** (스페이스 불가).

### 7.3 첫 Makefile 작성

```bash
cd ~/embedded-linux/manual003
nano Makefile
```

내용 (명령 앞의 들여쓰기는 반드시 Tab 키 사용):

```makefile
# 첫 번째 Makefile

# 변수 정의
CC      = gcc
CFLAGS  = -Wall -Wextra
TARGET  = hello
SRCS    = hello.c

# 기본 타겟: all
all: $(TARGET)

# hello 바이너리를 hello.c로 만들기
$(TARGET): $(SRCS)
	$(CC) $(CFLAGS) -o $(TARGET) $(SRCS)
	@echo "빌드 완료: $(TARGET)"

# 빌드 결과물 삭제
clean:
	rm -f $(TARGET)
	@echo "정리 완료"

# 빌드 결과물 삭제 후 재빌드
rebuild: clean all

# 이 타겟들은 실제 파일이 아님을 선언
.PHONY: all clean rebuild
```

코드 설명:

- `CC = gcc` — 컴파일러를 변수에 저장. 나중에 크로스 컴파일러로 바꾸려면 이 한 줄만 수정.
- `CFLAGS = -Wall -Wextra` — 컴파일 옵션. `-Wall`은 일반 경고, `-Wextra`는 추가 경고를 모두 활성화.
- `$(변수명)` — 변수 참조.
- `@echo ...` — `@`를 붙이면 명령 자체를 화면에 출력하지 않고 결과만 표시합니다.
- `.PHONY` — 파일이 아닌 "작업 이름" 타겟을 선언합니다. 같은 이름의 파일이 있어도 무시하고 항상 실행합니다.

### 7.4 hello.c 작성

Makefile이 참조하는 소스 파일을 만듭니다.

```bash
nano hello.c
```

내용:

```c
#include <stdio.h>

int main(void) {
    printf("Hello, Embedded Linux!\n");
    return 0;
}
```

### 7.5 Makefile 실행

```bash
make              # 기본 타겟(all) 실행
```

출력:

```text
gcc -Wall -Wextra -o hello hello.c
빌드 완료: hello
```

```bash
./hello           # 생성된 바이너리 실행
```

```text
Hello, Embedded Linux!
```

같은 `make`를 다시 실행하면:

```bash
make
```

```text
make: 'hello' is up to date.
```

`hello.c`가 변경되지 않았으므로 재컴파일하지 않습니다. **의존성 추적**이 작동하는 것입니다.

```bash
make clean        # 빌드 결과물 삭제
ls -la
make              # 다시 빌드
make rebuild      # clean 후 재빌드
```

### 7.6 Makefile 주요 자동 변수

Makefile을 읽다 보면 자주 나오는 특수 변수들입니다.

| 변수 | 의미 |
|------|------|
| `$@` | 현재 타겟 이름 |
| `$<` | 첫 번째 의존성 파일 이름 |
| `$^` | 모든 의존성 파일 목록 |
| `$$` | Makefile 안에서 `$` 문자 그 자체 |

예시:

```makefile
hello: hello.c
	$(CC) -o $@ $<    # $@ = hello, $< = hello.c
```

---

## 8. 파일 권한(Permission) 이해

임베디드 rootfs(루트 파일시스템)를 만들 때 권한 설정이 중요합니다. 지금 기초를 잡아둡니다.

### 8.1 권한 표기 읽기

```bash
ls -la ~/embedded-linux/manual003/
```

출력의 첫 열:

```text
-rwxr-xr-x 1 ubuntu ubuntu  280 May  4 03:30 system_info.sh
-rw-r--r-- 1 ubuntu ubuntu  100 May  4 03:00 lines.txt
drwxr-xr-x 2 ubuntu ubuntu   40 May  4 03:00 subdir_b
```

10자리 권한 표기 분해:

```text
- r w x   r - x   r - x
│ │ │ │   │ │ │   │ │ └── 기타(other): 실행(x), 쓰기(-), 읽기(r)
│ │ │ │   │ │ │   └────── 기타(other): 실행(x)
│ │ │ │   └─┴─┴────────── 그룹(group): r-x
│ └─┴─┴────────────────── 소유자(owner/user): rwx
└──────────────────────── 파일 타입: -=파일, d=디렉토리, l=링크
```

권한 기호:

- `r` — read (읽기)
- `w` — write (쓰기)
- `x` — execute (실행, 디렉토리의 경우 진입 가능)
- `-` — 해당 권한 없음

### 8.2 권한 변경 — `chmod`

**기호 방식:**

```bash
chmod +x system_info.sh     # 실행 권한 추가 (모든 대상)
chmod -w lines.txt          # 쓰기 권한 제거 (모든 대상)
chmod u+w lines.txt         # 소유자(u)에게만 쓰기 추가
chmod go-x system_info.sh  # 그룹(g)과 기타(o)의 실행 제거
```

**숫자 방식:**

각 권한에 숫자를 대응시켜 더합니다.

| 숫자 | 권한 | 의미 |
|------|------|------|
| 4 | r | 읽기 |
| 2 | w | 쓰기 |
| 1 | x | 실행 |
| 0 | - | 없음 |

소유자/그룹/기타 순서로 세 자리를 씁니다.

```bash
chmod 755 system_info.sh    # rwxr-xr-x: 소유자 모두, 그룹·기타 읽기+실행
chmod 644 lines.txt         # rw-r--r--: 소유자 읽기+쓰기, 그룹·기타 읽기만
chmod 600 secret.txt        # rw-------: 소유자 읽기+쓰기만, 나머지 없음
```

자주 쓰이는 권한:

| 숫자 | 기호 표기 | 용도 |
|------|-----------|------|
| `755` | `rwxr-xr-x` | 실행 파일, 디렉토리 |
| `644` | `rw-r--r--` | 일반 파일 (문서, 설정) |
| `600` | `rw-------` | 개인 파일 (키, 비밀번호) |
| `777` | `rwxrwxrwx` | 모두 허용 (보안에 취약, 지양) |

---

## 9. 종합 실습 — 환경 점검 스크립트

이 매뉴얼에서 배운 내용을 모두 활용해 임베디드 리눅스 학습 환경을 자동으로 점검하는 스크립트를 완성합니다.

```bash
nano ~/embedded-linux/setup_check.sh
```

내용:

```bash
#!/bin/bash
# 임베디드 리눅스 학습 환경 종합 점검 스크립트
# 사용법: ./setup_check.sh

set -e    # 명령이 실패하면 즉시 종료

STUDY_DIR="$HOME/embedded-linux"
REQUIRED_TOOLS="gcc make git wget curl python3 bc bison flex"
MIN_DISK_GB=5
ALL_OK=true

# 구분선 출력 함수
print_sep() {
    echo "================================================"
}

print_sep
echo " 임베디드 리눅스 학습 환경 점검"
echo " $(date '+%Y-%m-%d %H:%M:%S')"
print_sep
echo ""

# 1. 운영체제 확인
echo "[1] 운영체제"
OS_NAME=$(grep "^NAME=" /etc/os-release | cut -d= -f2 | tr -d '"')
KERNEL=$(uname -r)
ARCH=$(uname -m)
printf "    OS:       %s\n" "$OS_NAME"
printf "    커널:     %s\n" "$KERNEL"
printf "    아키텍처: %s\n" "$ARCH"
echo ""

# 2. 필수 도구 확인
echo "[2] 필수 도구"
for tool in $REQUIRED_TOOLS; do
    if command -v "$tool" > /dev/null 2>&1; then
        VER=$("$tool" --version 2>&1 | head -1)
        printf "    [OK]    %-10s %s\n" "$tool" "$VER"
    else
        printf "    [없음]  %-10s 설치 필요\n" "$tool"
        ALL_OK=false
    fi
done
echo ""

# 3. 작업 디렉토리 확인
echo "[3] 작업 디렉토리"
if [ -d "$STUDY_DIR" ]; then
    SIZE=$(du -sh "$STUDY_DIR" 2>/dev/null | cut -f1)
    FILE_COUNT=$(find "$STUDY_DIR" -type f | wc -l)
    printf "    [OK]  %s\n" "$STUDY_DIR"
    printf "          크기: %s, 파일 수: %d개\n" "$SIZE" "$FILE_COUNT"
else
    printf "    [없음] %s — 생성합니다.\n" "$STUDY_DIR"
    mkdir -p "$STUDY_DIR"
    ALL_OK=false
fi
echo ""

# 4. 디스크 여유 공간 확인
echo "[4] 디스크 여유 공간"
AVAIL_GB=$(df / | tail -1 | awk '{printf "%.1f", $4/1024/1024}')
if awk "BEGIN{exit !($AVAIL_GB >= $MIN_DISK_GB)}"; then
    printf "    [OK]  여유 공간: %s GB (최소 %d GB)\n" "$AVAIL_GB" "$MIN_DISK_GB"
else
    printf "    [경고] 여유 공간: %s GB — %d GB 이상 권장\n" "$AVAIL_GB" "$MIN_DISK_GB"
    ALL_OK=false
fi
echo ""

# 5. 결과 요약
print_sep
if $ALL_OK; then
    echo " 결과: 준비 완료 — 학습을 시작하세요!"
else
    echo " 결과: 일부 항목을 점검해야 합니다."
fi
print_sep
```

저장 후 실행 권한을 주고 실행합니다.

```bash
chmod +x ~/embedded-linux/setup_check.sh
~/embedded-linux/setup_check.sh
```

출력 예:

```text
================================================
 임베디드 리눅스 학습 환경 점검
 2026-05-04 03:45:00
================================================

[1] 운영체제
    OS:       Ubuntu
    커널:     6.8.0-51-generic
    아키텍처: x86_64

[2] 필수 도구
    [OK]    gcc        gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
    [OK]    make       GNU Make 4.3
    [OK]    git        git version 2.43.0
    ...

[3] 작업 디렉토리
    [OK]  /home/ubuntu/embedded-linux
          크기: 48K, 파일 수: 5개

[4] 디스크 여유 공간
    [OK]  여유 공간: 15.3 GB (최소 5 GB)

================================================
 결과: 준비 완료 — 학습을 시작하세요!
================================================
```

---

## 10. 자주 발생하는 실수와 해결법

### 실수 1: `Permission denied` — 실행 권한 없음

```text
bash: ./system_info.sh: Permission denied
```

```bash
chmod +x system_info.sh
```

### 실수 2: Makefile "missing separator" 오류

```text
Makefile:8: *** missing separator.  Stop.
```

명령 앞에 탭 대신 스페이스를 쓴 경우입니다. 확인 방법:

```bash
cat -A Makefile | grep -n "^I\|^ "
```

탭은 `^I`로, 스페이스는 공백으로 표시됩니다. 에디터에서 해당 줄을 탭으로 교체합니다.

### 실수 3: 변수에 공백이 있으면 따옴표가 필요

```bash
WRONG_DIR=/path/with space    # 오류 발생
CORRECT_DIR="/path/with space"
```

특히 경로에 공백이 있을 때 주의합니다.

### 실수 4: `>` 와 `>>` 혼동

```bash
echo "중요한 데이터" > data.txt
echo "새 데이터" > data.txt    # 기존 내용이 사라짐!
echo "추가 데이터" >> data.txt  # 이어 쓰기
```

중요한 파일에 쓰기 전에 `cp`로 백업하는 습관을 들이세요.

### 실수 5: `$` 를 빠뜨림

```bash
MY_VAR="hello"
echo MY_VAR     # MY_VAR 라는 문자열 그대로 출력 (잘못됨)
echo $MY_VAR    # hello 출력 (올바름)
```

---

## 11. 정리 및 다음 단계

이 매뉴얼에서 한 일:

- 셸의 역할과 프롬프트 구조를 이해했다.
- `ls`, `touch`, `cp`, `mv`, `rm`, `mkdir`로 파일과 디렉토리를 조작했다.
- 절대 경로와 상대 경로, 특수 표기(`.`, `..`, `~`)를 익혔다.
- `cat`, `less`, `head`, `tail`, `grep`, `find`로 파일을 조회·검색했다.
- 파이프(`|`)와 리디렉션(`>`, `>>`)으로 명령을 연결했다.
- 환경 변수(`export`, `$PATH`, `.bashrc`)를 설정하는 방법을 배웠다.
- 변수·조건문(`if`)·반복문(`for`)을 사용한 셸 스크립트를 작성·실행했다.
- `Makefile`의 타겟·의존성·명령 구조를 이해하고 `gcc`로 빌드를 자동화했다.
- 파일 권한(`rwx`) 표기를 읽고 `chmod`로 변경했다.

다음 매뉴얼 **M004 — 첫 C 프로그램 네이티브 컴파일**에서는 이 매뉴얼에서 만든 `hello.c`와 `Makefile`을 확장합니다. 컴파일의 네 단계(전처리 → 컴파일 → 어셈블 → 링킹)를 `gcc` 옵션으로 하나씩 분리해서 확인하고, 오브젝트 파일과 실행 파일의 차이를 직접 살펴봅니다.

---

## 부록 A — 자주 쓰는 셸 단축키

| 단축키 | 동작 |
|--------|------|
| `Ctrl+C` | 현재 실행 중인 명령 강제 종료 |
| `Ctrl+Z` | 명령을 백그라운드로 일시 중지 (`fg`로 복귀) |
| `Ctrl+D` | EOF 전송 (터미널 종료) |
| `Ctrl+L` | 화면 지우기 (`clear`와 동일) |
| `Ctrl+A` | 줄의 맨 앞으로 커서 이동 |
| `Ctrl+E` | 줄의 맨 끝으로 커서 이동 |
| `Ctrl+W` | 커서 앞의 단어 하나 삭제 |
| `Ctrl+U` | 커서 앞 전체 삭제 |
| `↑` | 이전 명령 불러오기 (히스토리) |
| `Tab` | 명령·파일명 자동 완성 |
| `Tab Tab` | 완성 후보 목록 표시 |
| `!!` | 바로 직전 명령 재실행 |
| `history` | 명령 히스토리 전체 출력 |
| `history \| grep git` | 히스토리에서 git 관련 명령 검색 |

## 부록 B — 이 매뉴얼에서 사용한 명령어 한 줄 풀이

| 명령 | 한 줄 풀이 |
|------|-----------|
| `ls -lha` | 숨김 파일 포함, 자세히, 사람이 읽기 좋은 크기로 목록 출력 |
| `ls -lhS` | 크기 큰 순 정렬 |
| `ls -lht` | 최신 순 정렬 |
| `touch <파일>` | 빈 파일 생성 또는 수정 시각 갱신 |
| `echo "text" > file` | 덮어쓰기 |
| `echo "text" >> file` | 이어 쓰기 |
| `cp -r <원본> <사본>` | 디렉토리째 복사 |
| `rm -r <디렉토리>` | 디렉토리와 내용 모두 삭제 |
| `cat -n <파일>` | 줄 번호와 함께 전체 출력 |
| `less <파일>` | 스크롤 뷰어 (q로 종료) |
| `head -n <파일>` | 처음 n줄 출력 |
| `tail -f <파일>` | 파일 끝 실시간 추적 |
| `grep -n "패턴" <파일>` | 줄 번호와 함께 패턴 검색 |
| `grep -v "패턴" <파일>` | 패턴이 없는 줄만 출력 |
| `grep -r "패턴" <경로>` | 디렉토리 재귀 검색 |
| `find <경로> -name "*.c"` | 파일명 패턴으로 검색 |
| `find <경로> -type f -size +1k` | 1KB 초과 파일 검색 |
| `wc -l <파일>` | 줄 수 출력 |
| `cut -d= -f2` | 구분자 `=`의 두 번째 필드 추출 |
| `tr -d '"'` | 특정 문자 제거 |
| `awk '{print $5}'` | 다섯 번째 공백 구분 필드 출력 |
| `which <명령>` | 실행 파일 경로 출력 |
| `export <변수>=<값>` | 환경 변수를 자식 프로세스에 전달 |
| `source <파일>` | 현재 셸에서 파일 실행 |
| `chmod +x <파일>` | 실행 권한 추가 |
| `chmod 755 <파일>` | 권한을 rwxr-xr-x로 설정 |
| `make` | Makefile의 기본 타겟 실행 |
| `make clean` | 빌드 결과물 정리 |
| `df -h /` | 루트 파티션 디스크 사용량 출력 |
| `du -sh <경로>` | 디렉토리 총 사용 용량 출력 |
| `free -h` | 메모리 사용량 출력 |

---

*다음 매뉴얼: `manual004.md` — 첫 C 프로그램 네이티브 컴파일 (작성 예정)*
