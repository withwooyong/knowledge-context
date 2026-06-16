---
title: LLM Wiki(Knowledge Context) 데모 실습 가이드 요약
status: review
owner: ted
scope: context
source: sources/LLM-Wiki-데모-실습-가이드.md
tags: [llm-wiki, rag, obsidian, knowledge-management, governance]
created: 2026-06-16
---

# LLM Wiki 데모 실습 가이드 — 핵심 요약

> 원본: [[LLM-Wiki-데모-실습-가이드]] (sources/) · 영상 "언제까지 AI에게 매번 같은 설명을 다시 해야할까?" (전현준의 현업 에이전트)

## TL;DR
채팅 세션이 끝나면 맥락이 날아가는 문제를, **문서가 들어올 때(Write time) 한 번 정리·연결**하는 LLM Wiki로 해결한다. Obsidian(사람) + Git(승인) + LLM Wiki(INDEX/LOG) + RAG(OpenViking) + **사람 머지 승인**을 조합해 1인 데모부터 팀 운영까지 단계적으로 구성하는 핸즈온 가이드.

## 1. RAG vs LLM Wiki
| | RAG | LLM Wiki |
|---|---|---|
| 정리 시점 | 질문할 때(Read time) 매번 재검색·재조립 | 문서 들어올 때(Write time) 한 번 "컴파일" |
| 문서 상태 | 때려넣기 — 뭐가 있는지 모름 | 문서·인덱스가 연결되어 유무를 앎 |

→ 둘은 배타적이지 않다. 평소엔 LLM Wiki, 문서가 폭증하면 RAG를 **검색 계층**으로 덧붙인다.

## 2. Karpathy의 3개 레이어
1. **Source of truth** — 불변 원본(`sources/`)
2. **Wiki(정보 계층)** — 요약·연결한 지식 그래프 (`INDEX.md` 탐색 / `LOG.md` 이력)
3. **Harness(규칙 계층)** — 읽기/쓰기/저장 스키마·룰(`AGENTS.md`)로 "쓰레기 산" 방지

> 핵심 통찰: LLM Wiki의 진짜 가치는 "구조를 AI가 만들게 하는 것"이 아니라 **"유지보수(품질 보존)를 자동화하는 것"**. 이걸 빼면 즉시 쓰레기장이 된다.

## 3. 폴더 구조 = 계층 분리
- `sources/` — 불변 원본 (절대 수정/삭제 X)
- `context/` — 팀 합의 공유 지식 (직접 쓰기 X, **PR로만** 승격)
- `members/<이름>/` — 개인 작업 공간 (자유)
- `_private/` — 비공유, Git 싱크 제외, **사람만** 관리
- 한 폴더가 동시에 **Git 레포 = Obsidian 볼트**. 비개발자는 GUI, 개발자는 PR/롤백.

## 4. 핵심 파일
- `INDEX.md` — 각 페이지 TL;DR + 위키링크로 drill-down 하는 내용 중심 목차
- `LOG.md` — ingest/update/promote/삭제를 시간순 append-only 기록 (grep 추적)
- `AGENTS.md` — 에이전트가 규칙 안에서만 동작하게 하는 시스템 프롬프트(Harness)

## 5. 운영 흐름 (STEP 1~7)
1. Git 레포 + 폴더 구조 + `.gitignore`(`_private` 제외)
2. Obsidian 볼트 열기 + CLI 활성화(1.12+) → tool 다단계 호출이 bash 한 줄로
3. INDEX/LOG/AGENTS 작성
4. Obsidian Skills(kepano) 연동 → 원본 요약 노트 자동 생성 시연
5. "팀에 공유하고 싶어" 한마디 → 에이전트가 **승격 PR 초안** 생성
6. 문서 폭증 시 OpenViking(RAG) 연동 — L0(제목)/L1(요약)/L2(원문) 계층 로딩, 토큰 83~92% 절감
7. **거버넌스**: 사람 일과 위임 일을 경계 짓는다

## 6. 거버넌스 경계
| 사람이 직접 (위임 금지) | 에이전트에 위임 |
|---|---|
| `_private` 관리, `context/` **삭제**, 권한·보안, PR **최종 머지 승인** | 요약·정제, 위키링크·INDEX/LOG 갱신, 승격 PR 초안, 중복 탐지·병합 제안 |

## 한 장 요약
작은 **LLM Wiki**(INDEX/LOG)에서 출발 → **Git**(동기화·승인) 위에 → **Obsidian**(사람 운영) → 폭증 시 **OpenViking(RAG)** 검색 → **사람-에이전트 역할 분담**으로 안정화. 핵심은 도구가 아니라 **어떤 지식을, 어디에 두고, 언제 제공할지**.

## 관련 문서
- 원본: [[LLM-Wiki-데모-실습-가이드]]
- 운영 규칙: [[AGENTS]]
