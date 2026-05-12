# 실습 환경 (Exec)

이 폴더는 **각 매뉴얼의 실습용 학습 자료**를 담고 있습니다.

## 구조

```
exec/
└── man_01/              Manual 001 — 임베디드 리눅스 첫걸음
    ├── README.md        이 수업 전체 개요
    ├── concepts/        핵심 개념 정리 파일들
    │   ├── 01_embedded_system.md       개념 01: 임베디드 시스템
    │   ├── 02_embedded_linux.md        개념 02: 임베디드 리눅스
    │   └── 03_key_terms.md            개념 03: 핵심 용어 7가지
    ├── references/      외부 학습 자료 및 링크
    │   └── README.md    참고 자료 목록
    └── notes/          학습 중 작성하는 노트
        └── learning_log.md             학습 진행 상황 기록
```

## 학습 방법

### 1단계: 개념 이해
[Manual 001](../docs/manual001.md)을 읽으면서:
- **concepts/** 폴더의 자세한 설명을 참고하세요
- 어려운 부분은 여러 번 읽으세요

### 2단계: 개념 정리
**concepts/** 폴더의 파일들을 읽으면서:
- 핵심 용어의 정의를 이해하세요
- 각 개념이 어떻게 연결되는지 파악하세요

### 3단계: 학습 기록
**notes/learning_log.md**에:
- 자신의 말로 개념을 정리하세요
- 이해가 안 가는 부분을 기록하세요
- 다음 학습 준비 사항을 체크하세요

### 4단계: 추가 학습
더 알고 싶으면:
- **references/README.md**의 자료들을 참고하세요
- 공식 문서와 온라인 커뮤니티를 활용하세요

## 빠른 시작

```bash
# 1. Manual 001 읽기
cat ../docs/manual001.md

# 2. 핵심 개념 3개 학습
cat man_01/concepts/01_embedded_system.md
cat man_01/concepts/02_embedded_linux.md
cat man_01/concepts/03_key_terms.md

# 3. 학습 진행 기록
nano man_01/notes/learning_log.md

# 4. 추가 자료 확인
cat man_01/references/README.md
```

## 다음 단계

Manual 002 학습을 준비하면서:
- `man_02/` 폴더가 추가될 예정입니다
- 유사한 구조로 학습 자료가 제공됩니다

---

**상태**: 2026-05-09 생성  
**최신 버전**: v0.8.0
