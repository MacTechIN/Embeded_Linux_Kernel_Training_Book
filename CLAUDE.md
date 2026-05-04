# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

이 저장소는 **임베디드 리눅스 실습 교과서**를 제작하는 프로젝트이다. 대상 독자는 **초보 엔지니어**이며, 각 학습 단계마다 **소스 코드와 실습 방법**을 자세히 문서화한다. 모든 콘텐츠는 한국어로 작성한다.

핵심 산출물은 두 종류이다:

1. **튜토리얼 매뉴얼 시리즈** — `docs/manual001.md`, `docs/manual002.md`, … (학습 순서대로 번호 부여)
2. **KM 시계열 로그** — [docs/km.md](docs/km.md) (학습 중 정리한 개념·메모를 append-only로 누적)

## Layout

- [docs/](docs/) — **published artifacts**. 매뉴얼(`manual00x.md`)과 KM 로그(`km.md`)가 함께 들어간다.
- [doc/](doc/) — **working material** (singular, not plural).
  - [doc/research.md](doc/research.md) — 매뉴얼/KM 항목으로 정제하기 전의 리서치 메모.
  - [doc/plan.md](doc/plan.md) — 다음에 무엇을 어떤 버전으로 작성할지의 계획.

`doc/`(작업물)와 `docs/`(공식 산출물)의 분리는 의도적이다. 통합하지 말 것.

## 매뉴얼(`docs/manual00x.md`) 작성 규칙

새 실습 단계를 추가할 때:

- **파일명**: `docs/manualNNN.md` — `NNN`은 0-padded 3자리, 다음 미사용 번호. (예: 현재 `manual001.md`이 마지막이면 다음은 `manual002.md`.)
- **대상이 초보임을 잊지 말 것.** 용어는 처음 등장할 때 풀어쓰고, 명령어·코드는 **복붙 가능한 완전한 형태**로 제공한다 (생략·축약 금지).
- **권장 섹션 순서**:
  1. 학습 목표 (이 단계를 마치면 무엇을 할 수 있게 되는가)
  2. 사전 준비 (필요한 도구·환경·이전 단계)
  3. 단계별 절차 (번호 매긴 hands-on 스텝)
  4. 소스 코드 (인라인 코드 블록, 언어 태그 명시)
  5. 검증 방법 (예상 결과물·확인 명령)
  6. 트러블슈팅 / 다음 단계
- 코드 예제용 별도 디렉토리(예: `examples/manualNNN/`)가 필요해지면 **사용자와 먼저 합의**한다 — 현재는 존재하지 않는다.

## `docs/km.md` 작성 규칙 (KM 로그)

매뉴얼과 별개로 운영되는 시계열 학습 노트. 형식이 load-bearing 하므로 엄격히 따른다:

- **Append-only.** 과거 항목 수정·재작성 금지. 새 내용은 맨 아래에 추가.
- **각 항목 헤더는 정확히 이 형식**:
  `## [YYYY-MM-DD HH:MM] <제목> (vX.Y.Z)`
  예시: `## [2026-05-04 02:10] 임베디드 리눅스 개발 기초 자료 (v0.1.0)`
- **버전 bump**: 항목마다 올림. 작은 추가는 patch (`v0.1.1`), 새 토픽은 minor (`v0.2.0`).
- **마지막 marker 줄** (`*(이후 기록은 시계열에 따라 아래에 추가됩니다.)*`) 은 항상 파일 맨 끝에 유지. 새 항목은 그 **위에** 삽입.

## 작업 흐름

새 매뉴얼 / KM 항목을 추가하기 전에:

1. 필요시 [doc/research.md](doc/research.md) 에 원자료 정리.
2. [doc/plan.md](doc/plan.md) 를 업데이트 (다음 산출물의 의도·버전·날짜).
3. `docs/` 에 정제된 결과 추가 (매뉴얼이면 `manualNNN.md`, KM 메모면 `km.md` append).

## 커밋 메시지 규칙

기존 히스토리 형식: `vX.Y.Z: <짧은 설명>` (예: `v0.1.0: Recorded initial Embedded Linux basics in docs/km.md`). 커밋의 버전은 추가된 산출물의 버전과 일치해야 한다.

## 날짜 처리

항목 헤더는 절대 시각을 기록한다. 사용자가 상대 시각("오늘", "어제")으로 말하면, 헤딩에 적기 전에 절대 `YYYY-MM-DD HH:MM` 으로 변환할 것.
