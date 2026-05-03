# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a **documentation-only** repository — there is no source code, no build system, and no tests. It is a personal knowledge-management (KM) log for studying Embedded Linux. Content is written in **Korean**.

## Layout (the two `doc` folders are intentional)

- [docs/km.md](docs/km.md) — the **main KM document**. This is the canonical, time-series, append-only log.
- [doc/research.md](doc/research.md) — research notes used as source material before promoting findings into `km.md`.
- [doc/plan.md](doc/plan.md) — the implementation plan for new KM entries (what to write next, why, and when).

`doc/` (singular) holds working/planning material. `docs/` (plural) holds the published KM log. Do not consolidate them.

## Writing conventions for `docs/km.md`

These are load-bearing — the value of the file is its history, so format and append-discipline matter:

- **Append-only.** Never edit or rewrite past entries. New knowledge goes at the bottom.
- **Each entry starts with a timestamped, versioned heading** in this exact form:
  `## [YYYY-MM-DD HH:MM] <title> (vX.Y.Z)`
  Example already in the file: `## [2026-05-04 02:10] 임베디드 리눅스 개발 기초 자료 (v0.1.0)`
- **Bump the version** on every new entry. Patch (`v0.1.1`) for small additions, minor (`v0.2.0`) for a new topic, major for a structural shift.
- **Keep the trailing marker line** (`*(이후 기록은 시계열에 따라 아래에 추가됩니다.)*`) at the very end of the file. New entries are inserted **above** it.
- **Write in Korean**, matching the existing tone (concise headings, bulleted definitions, bold key terms).

## Workflow before adding a KM entry

1. Capture raw material in [doc/research.md](doc/research.md) if it isn't there yet.
2. Update [doc/plan.md](doc/plan.md) with the next entry's intent (topic, version bump, date).
3. Append the polished entry to [docs/km.md](docs/km.md) following the heading format above.

## Commit message convention

Existing history uses: `vX.Y.Z: <short description>` (e.g., `v0.1.0: Recorded initial Embedded Linux basics in docs/km.md`). The version in the commit message must match the version of the `km.md` entry being added.

## Date handling

The repository records absolute dates in entry headings. When the user gives a relative date ("오늘", "어제"), resolve it to an absolute `YYYY-MM-DD HH:MM` before writing the heading.
