# knowledge-context

**LLM Wiki(지식 컨텍스트) 저장소** — 하나의 디렉터리가 동시에 **Git 레포 = Obsidian 볼트**로 동작합니다.

> 채팅 세션이 끝나면 맥락이 날아가는 문제를, **문서가 들어올 때(Write time) 한 번 요약·연결해 "컴파일"**해 두는 방식으로 해결합니다. (RAG가 질문할 때마다 재검색하는 것과 대비)

배경/전체 흐름: [`sources/LLM-Wiki-데모-실습-가이드.md`](sources/LLM-Wiki-데모-실습-가이드.md) · 요약 노트 [`context/notes/llm-wiki-demo-guide.md`](context/notes/llm-wiki-demo-guide.md)

## 레이어 구조

| 경로 | 의미 | 권한 |
|---|---|---|
| `sources/` | 결정의 근거가 되는 불변 원본 (source of truth) | 수정·삭제 금지 |
| `context/` | 팀이 합의한 공유 지식 | **PR로만** 승격 (직접 쓰기 금지) |
| `members/<이름>/` | 개인 작업 공간 | 자유 |
| `members/<이름>/_private/` | 비공유 개인 영역 | 사람만, Git 싱크 제외 |

특수 파일:
- **`INDEX.md`** — 각 문서 TL;DR + 위키링크로 탐색하는 내용 중심 목차
- **`LOG.md`** — 모든 ingest/update/promote/삭제를 시간순으로 남기는 append-only 이력
- **`AGENTS.md`** — 에이전트(Claude Code/Codex)가 규칙 안에서만 동작하도록 하는 시스템 프롬프트(Harness)
- **`CLAUDE.md`** — Claude Code용 운영 가이드 (AGENTS.md 규칙 요약 + 워크플로)

## 두 가지 사용 방식

- **비개발자** — Obsidian GUI로 노트를 쓰고, AI에게 "어디에 적을지"만 부탁. "이거 팀에 공유하고 싶어" 한마디면 승격 PR이 만들어집니다.
- **개발자** — Git(PR/리뷰/롤백)으로 변경을 통제. 누가/언제/무엇을 바꿨는지 항상 추적·되돌리기 가능.

둘 다 AI를 쓸 때는 `AGENTS.md` 규칙을 따릅니다.

## 표준 워크플로

1. **ingest** — `sources/`의 원본을 요약해 `members/<user>/notes/`에 노트 생성 → 위키링크 연결 → `INDEX.md`/`LOG.md` 갱신
2. **promote** — "팀에 공유하고 싶어" → `context/`로 옮기는 PR 생성 (충돌 점검·위치 추천)
3. **approve** — **사람이 최종 머지 승인** (에이전트는 머지하지 않음 — 이 게이트를 빼면 저장소가 쓰레기장이 됩니다)

## 거버넌스 — 사람 vs 에이전트

| 사람이 직접 (위임 금지) | 에이전트에 위임 |
|---|---|
| `_private` 관리, `context/` **삭제**, 권한·보안, PR **최종 머지 승인** | 원본 요약·정제, 위키링크·INDEX/LOG 갱신, 승격 PR 초안, 중복 탐지·병합 제안 |

## 셋업

- **Obsidian 1.12+** — `Open folder as vault`로 이 폴더 열기. `Settings → General → Command line interface` 토글로 CLI 활성화.
- **Obsidian Skills** — `.claude/skills/obsidian/`에 [kepano의 스킬](https://github.com/kepano/obsidian-skills)을 vendoring해 두었습니다.
- 문서가 폭증하면 **OpenViking(RAG)**을 검색 계층으로 연동 (가이드 STEP 6 참고).
