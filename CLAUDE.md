# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 저장소의 정체

이것은 코드 프로젝트가 아니라 **LLM Wiki(지식 컨텍스트) 저장소**다. 하나의 디렉터리가 동시에 **Git 레포 = Obsidian 볼트**로 동작한다. 빌드/린트/테스트 같은 명령은 없으며, "작업"은 마크다운 노트를 정리·연결하고 INDEX/LOG를 갱신하는 것이다.

목적: 채팅 세션이 끝나면 맥락이 날아가는 문제를, **문서가 들어올 때(Write time) 한 번 요약·연결해 "컴파일"**해 두는 것. (배경/전체 흐름은 `sources/LLM-Wiki-데모-실습-가이드.md` 및 `members/ted/notes/llm-wiki-demo-guide.md` 참고)

## 운영 규칙 (반드시 준수 — `AGENTS.md`가 정본)

`AGENTS.md`가 이 저장소의 시스템 프롬프트(Harness 레이어)다. 작업 전 항상 따른다. 핵심:

- **레이어별 권한**
  - `sources/` — 원본(불변). **절대 수정·삭제 금지** (source of truth).
  - `members/<이름>/` — 개인 작업 공간. 자유롭게 생성·수정.
  - `context/` — 팀 합의 공유 지식. **직접 쓰지 않는다. 반드시 PR로 제안** (직접 머지 금지).
  - `_private/` — 접근·수정 금지. 사람만 다룬다 (`.gitignore`로 싱크 제외).
- **문서 작성 3원칙** — ① 핵심을 요약하고 관련 문서로 위키링크(`[[...]]`) 연결, ② 모든 추가·수정·삭제를 `LOG.md`에 시간순 1줄 기록, ③ `INDEX.md`에 새 문서 TL;DR 항목 추가.
- **금지(사람만 수행)** — `context/` 문서 삭제, 권한·보안 변경, `_private` 접근.

## 표준 워크플로

원본 → 멤버 노트 ingest (이 저장소의 가장 흔한 작업):
1. `sources/`의 원본을 읽고 핵심 요약 → `members/<user>/notes/<slug>.md` 생성
2. 노트 상단에 `_templates/note.md` 형식의 프론트매터(`title/status/owner/scope/source/tags/created`)
3. 원본·관련 문서로 `[[위키링크]]` 연결
4. `INDEX.md`에 항목 추가 + `LOG.md`에 `ingest`/`update` 줄 추가

팀 승격(사용자가 "팀에 공유하고 싶어"라고 할 때만):
- `members/<user>/`의 문서를 근거(`sources/`)와 함께 묶어 `context/`로 옮기는 **PR/브랜치 초안**을 만든다. 기존 지식과 충돌 점검 + 적절한 위치 추천. **최종 머지는 사람이 승인** — 자동 머지 금지.

## 규약

- **LOG.md** — append-only. 형식: `2026-06-16T17:01 ingest  sources/... → members/...` (동사: `ingest`/`update`/`promote`/`delete`). 기존 줄을 고치지 말고 끝에 추가만.
- **INDEX.md** — 내용 중심 목차. 섹션(`context/` · `members/<user>/` · `sources/`)별로 `[[경로|표시명]] — 한 줄 TL;DR` 형식.
- **위키링크** — Obsidian 문법 `[[note-slug]]` 또는 `[[경로|표시명]]`. 파일명(slug) 기준.
- **프론트매터** — `status: draft|review|approved`, `scope: members|context`. 새 노트는 `_templates/note.md`를 기준으로.

## Git / GitHub

- 원격: `withwooyong/knowledge-context` (public). 기본 브랜치 `master`.
- 커밋 메시지는 **한글**로 작성. 이 레포 로컬 git identity는 `withwooyong <withwooyong@gmail.com>`.
- **푸시는 사용자가 명시적으로 요청할 때만.** 커밋과 푸시를 한 명령에 묶지 않는다.
- `context/` 승격은 GitHub면 실제 PR + `CODEOWNERS` 리뷰로, 로컬이면 `git merge --no-ff promote/<slug>`로 사람이 승인.

## Obsidian Skills

`.claude/skills/obsidian/`에 kepano의 Obsidian Skills를 vendoring해 두었다(`obsidian-cli`, `obsidian-markdown`, `obsidian-bases`, `json-canvas`, `defuddle`). 볼트 문법·CLI·캔버스·웹 클리핑을 다룰 때 이 SKILL.md들을 참고. (third-party — nested `.git` 제거해 일반 파일로 추적 중)
