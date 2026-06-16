# LLM Wiki(Knowledge Context) 데모 실습 가이드

> 영상: [언제까지 AI에게 매번 같은 설명을 다시 해야할까? | 현업이 알려주는 LLM Wiki (Feat. RAG와의 차이)](https://www.youtube.com/watch?v=4XBofMHVyyc) — 전현준의 현업 에이전트 (34:07)
>
> 이 문서는 영상에서 설명한 **Obsidian + Git + LLM Wiki + RAG(OpenViking) + 사람 승인 프로세스** 조합을 직접 손으로 따라 해 볼 수 있도록 정리한 핸즈온 가이드입니다. 1인 로컬 데모부터 시작해 팀 단위 운영 구조까지 단계적으로 구성합니다.

---

## 0. 영상이 말하는 핵심 (5분 요약)

영상의 출발점은 단순합니다. **"왜 AI에게 매번 같은 설명을 다시 해야 하는가?"** 채팅 세션에서 좋은 답변을 받아도 세션이 끝나면 날아가고, 회의 결정과 그 근거는 따로 흩어지며, 개인 메모는 팀에 공유되지 않습니다. 결국 사람도 AI도 매번 맥락을 다시 찾고 다시 설명하는 비용을 치릅니다.

이 문제를 해결하는 두 가지 접근을 비교합니다.

| | RAG | LLM Wiki |
|---|---|---|
| 정리 시점 | **질문할 때(Read time)** 매번 청크를 재검색·재조립 | **문서가 들어올 때(Write time)** 한 번 정리·연결("컴파일") |
| 문서 상태 | "일단 때려넣기" 식 저장소, 무엇이 들어왔는지 모름 | 문서끼리, 인덱스끼리 **연결**되어 무엇이 있고 없는지 앎 |
| 비유 | 매번 다시 읽는 검색 | 한 번 읽고 결론을 정리해 다음 질문의 기본 컨텍스트로 사용 |

**Karpathy가 제시한 LLM Wiki의 3개 레이어** (영상에서 핵심으로 강조한 "계층 분리"):

1. **Source of truth (원본 데이터)** — 불변의 원본을 특정 경로에 보관
2. **Wiki (정보 계층)** — 요약·개념 정리·문서 간 연결을 담은 지식 그래프 (`INDEX.md`로 탐색, `LOG.md`로 이력 기록)
3. **Harness (규칙 계층)** — 어떻게 읽고/쓰고/저장할지 스키마와 룰을 정해 "쓰레기 산"이 되지 않게 유지

> 영상의 핵심 통찰: **LLM Wiki의 진짜 가치는 "문서 구조를 AI에게 만들게 하는 것"이 아니라 "유지보수를 자동화하는 것"에 있다.** 마지막 단계인 *문서 품질 보존*이 가장 어렵고, 이걸 안 하면 즉시 쓰레기장이 된다.

**영상에서 보여준 발전 단계** (이 데모도 이 순서를 따릅니다):

1. Obsidian 볼트 + Git 레포로 마크다운 지식 저장소 구성
2. `context/`(팀 공유) · `members/`(개인) · `agents/`(스킬·플러그인) · `_private/`(비공유) 구조 설계
3. `AGENTS.md`(시스템 프롬프트) 규칙 하에서만 에이전트가 동작 → PR + **사람 승인**으로 팀 지식 승격
4. 마크다운 문서가 폭발적으로 증가 → **OpenViking(RAG)** 연동으로 L0~L2 계층 로딩
5. **거버넌스**: 사람이 할 일(`_private`, 삭제, 권한·보안)과 에이전트에 위임할 일을 명확히 구분

---

## 1. 데모 아키텍처 한눈에 보기

```
knowledge-context/                # = Git 레포지토리 = Obsidian 볼트(Vault)
├── AGENTS.md                     # 에이전트 시스템 프롬프트(규칙 = Harness 레이어)
├── INDEX.md                      # 전체 지식 인덱스(Karpathy 개념)
├── LOG.md                        # 변경 이력(추가/삭제/머지, 시간순 append-only)
├── context/                      # [팀 공유 레이어] 합의된 지식만 들어옴
│   ├── agents/                   #   팀 공용 스킬/플러그인
│   └── ...
├── members/                      # [개인 작업 레이어]
│   └── ted/
│       ├── agents/               #   개인 스킬/플러그인
│       ├── notes/
│       └── _private/             #   비공유(Git 싱크 제외, 사람만 관리)
├── sources/                      # [Source of truth] 불변 원본 보관
└── .obsidian/                    # Obsidian 설정(볼트 메타데이터)

         ┌─ 비개발자: Obsidian GUI로 노트 작성 ─┐
마크다운 저장소 ┤                                  ├─→ AI 에이전트(Claude Code/Codex)
         └─ 개발자: git PR/리뷰/롤백 ───────────┘     는 AGENTS.md 규칙 안에서만 동작

문서 폭증 시 → OpenViking(RAG)이 저장소를 L0/L1/L2로 인덱싱해 토큰 절감 검색 제공
```

두 가지 뷰가 같은 저장소를 바라봅니다. **비개발자는 Obsidian GUI**로 노트를 쓰고, **개발자는 Git(PR/리뷰/롤백)**으로 다룹니다. 둘 다 AI 에이전트를 쓸 때는 `AGENTS.md`라는 공식 규칙을 따릅니다.

---

## 2. 사전 준비물 (Prerequisites)

| 도구 | 용도 | 비고 |
|---|---|---|
| **Git** | 변경 추적·통제 (`git --version`) | 영상의 "동기화/승인" 기반 |
| **Obsidian 1.12.4+** | 사람이 쓰기 쉬운 마크다운 볼트 + **CLI** | CLI는 1.12부터 정식 제공 |
| **Claude Code 또는 Codex CLI** | 에이전트 (저장소 규칙 기반 동작) | 둘 중 하나 |
| **Python 3.10+** | OpenViking 실행(RAG 단계) | 4단계에서만 필요 |
| **LLM API 키** | 임베딩 + VLM 모델 | OpenAI/Claude/Gemini/Ollama 등 |

> 💡 1~3단계(LLM Wiki 기본)는 API 키 없이도 가능합니다. RAG 연동(4단계)부터 모델 키가 필요합니다. 먼저 1~3단계로 개념을 잡고, 문서가 늘어났다는 가정 하에 4단계를 붙이는 흐름을 추천합니다.

---

## 3. STEP 1 — Git 레포 + 폴더 구조 만들기

영상에서 강조한 **계층 분리**(원본 / 위키 / 규칙)를 폴더로 구현합니다.

```bash
mkdir knowledge-context && cd knowledge-context
git init

# 공유 레이어 / 개인 레이어 / 원본 보관소
mkdir -p context/agents
mkdir -p members/ted/agents members/ted/notes members/ted/_private
mkdir -p sources

# 핵심 파일 골격
touch AGENTS.md INDEX.md LOG.md
```

**`_private`를 Git에서 제외** (영상: 민감하거나 공유하기 껄끄러운 자료는 싱크하지 않음):

```bash
cat > .gitignore <<'EOF'
# 개인 비공유 영역 — Git 싱크 제외, 사람만 관리
members/*/_private/
.obsidian/workspace.json
EOF

git add . && git commit -m "chore: bootstrap knowledge-context structure"
```

폴더 의미 정리:

- `context/` — **팀/회사 전체가 합의한 공유 지식.** 여기에 들어오려면 승인 절차를 거칩니다.
- `members/<이름>/` — 개인 작업 공간. 개발 프로젝트·기획·디자인 등.
- `members/<이름>/agents/`, `context/agents/` — 스킬·플러그인. 개인이 만든 걸 팀으로 승격 가능.
- `sources/` — **Source of truth.** 결정의 근거가 되는 불변 원본.
- `_private/` — 절대 공유/싱크되지 않는 개인 영역.

---

## 4. STEP 2 — Obsidian 볼트 + CLI 설정

### 4-1. 볼트 열기
Obsidian 실행 → **Open folder as vault** → 방금 만든 `knowledge-context` 폴더 선택. 이제 같은 폴더가 **Git 레포이자 Obsidian 볼트**가 됩니다(영상: "그냥 깃 레포지토리다").

### 4-2. CLI 활성화 (Obsidian 1.12+)
**Settings → General → Command line interface** 토글 ON. 이제 터미널에서 `obsidian` 바이너리가 명령을 받습니다. 노트 생성·검색·템플릿 적용·태그 일괄 변경 등 GUI에서 하던 작업을 명령 한 줄로 처리할 수 있어, Claude Code의 다단계 tool 호출이 단일 bash 명령으로 줄어듭니다.

```bash
# 예시 (설치 환경에 따라 명령명이 obsidian / obs 등으로 다를 수 있음)
obsidian --help
obsidian search "회의록"
```

### 4-3. 프론트매터 템플릿 (선택)
영상은 Obsidian의 **Templates** 확장 + 프론트매터로 문서 메타데이터를 사람이 직접 관리한다고 설명합니다. 예시 프론트매터:

```yaml
---
title: 결제 모듈 설계 결정
status: draft        # draft | review | approved
owner: ted
scope: members       # members | context(팀 공유)
source: sources/payment-spec-v3.pdf
tags: [payment, decision]
created: 2026-06-16
---
```

---

## 5. STEP 3 — LLM Wiki 핵심 파일 (INDEX / LOG / AGENTS)

Karpathy 개념의 두 특수 파일과, 영상이 팀용으로 확장한 `AGENTS.md`를 작성합니다.

### 5-1. `INDEX.md` — 내용 중심 목차
에이전트가 각 페이지의 TL;DR만 훑고 관련 페이지로 drill-down 하도록 직접 링크를 거는 네비게이션 파일입니다.

```markdown
# Knowledge Context — INDEX

## context/ (팀 공유)
- [[context/agents/README|팀 공용 에이전트 스킬]] — 승인된 스킬 모음

## members/ted/
- [[members/ted/notes/payment-decision|결제 모듈 설계 결정]] — v3 스펙 기반, 카드 우선
- [[members/ted/notes/onboarding|온보딩 메모]] — 신규 합류자용 컨텍스트

## sources/ (원본 / source of truth)
- payment-spec-v3.pdf — 결제 스펙 원본
```

### 5-2. `LOG.md` — 시간순 운영 이력 (append-only)
모든 ingest/query/머지/삭제를 시간순으로 남겨 `grep`으로 추적 가능하게 합니다.

```markdown
# LOG (append-only)

- 2026-06-16T10:00 ingest  sources/payment-spec-v3.pdf → members/ted/notes/payment-decision
- 2026-06-16T10:05 update  INDEX.md (+payment-decision)
- 2026-06-16T14:20 promote members/ted/notes/payment-decision → context/ (PR #1, approved by ted)
```

### 5-3. `AGENTS.md` — 에이전트 규칙(Harness 레이어)
에이전트가 저장소를 **마음대로 다루지 않고 정해진 규칙 안에서만** 동작하도록 하는 시스템 프롬프트입니다. Claude Code / Codex가 작업 전에 읽습니다.

```markdown
# AGENTS.md — Knowledge Context 운영 규칙

너는 이 저장소의 지식 관리 에이전트다. 아래 규칙을 반드시 따른다.

## 레이어
- `sources/`  : 원본(불변). 절대 수정/삭제하지 않는다.
- `members/*` : 개인 작업 공간. 자유롭게 생성/수정 가능.
- `context/`  : 팀 공유 지식. **직접 쓰지 않는다.** 반드시 PR로 제안한다.
- `_private/` : 접근/수정 금지. 사람만 다룬다.

## 문서 작성
1. 새 자료는 핵심을 요약하고, 관련 문서로 위키링크([[...]])를 연결한다.
2. 모든 추가/수정/삭제는 LOG.md에 시간순으로 한 줄 기록한다.
3. INDEX.md에 새 문서의 TL;DR 항목을 추가한다.

## 승격(팀 공유)
- 사용자가 "이거 팀에 공유하고 싶어"라고 하면:
  members/<user>/ 의 문서를 근거(sources/)와 함께 묶어
  context/ 로 옮기는 **PR**을 생성한다. (직접 머지 금지)
- 기존 지식과 충돌하는지 점검하고, 들어갈 적절한 위치를 추천한다.

## 금지 (사람만 수행)
- context/ 문서 삭제, 권한/보안 관련 변경, _private 접근.
```

> 영상의 운영 묘미: 비개발자에게는 "PR이 뭐냐"는 부담을 주지 않습니다. **"어디에 적을지"만 알려주고**, 사용자가 "이거 팀에 공유하고 싶어"라고 한마디만 하면 에이전트가 내부 저장 구조·리뷰 프로세스를 알아서 태웁니다.

---

## 6. STEP 4 — 에이전트 연동 (Obsidian Skills)

Obsidian CEO(Steph Ango / kepano)가 공식 배포한 **Obsidian Skills**(SKILL.md 5종)를 붙이면 Claude Code·Codex가 볼트의 문법·CLI·캔버스 포맷·웹 클리핑을 이해하고 다룹니다. (영상에서 "이틀 전에 옵시디언 CEO가 스킬을 올렸다"며 추가 자료로 소개한 바로 그것)

```bash
# 레포 루트에서 (Claude Code 기준 .claude/skills 위치 예시)
mkdir -p .claude/skills
git clone https://github.com/kepano/obsidian-skills .claude/skills/obsidian
```

그다음 Claude Code를 레포 루트에서 실행하면 `AGENTS.md` 규칙 + Obsidian Skills를 함께 읽습니다.

**시연 시나리오 (1인 데모):**

1. `sources/`에 원본 문서(PDF/MD)를 하나 넣는다.
2. 에이전트에게: *"sources/payment-spec-v3.pdf 핵심을 요약해서 내 멤버 폴더에 노트로 정리하고 INDEX와 LOG도 갱신해줘."*
3. 에이전트가 `members/ted/notes/`에 요약 노트 생성 + 위키링크 연결 + `INDEX.md`/`LOG.md` 갱신.
4. 만족스러운 답변이 나오면: *"이거 팀에 공유하고 싶어."* → 에이전트가 `context/`로 승격하는 **PR**을 생성.

---

## 7. STEP 5 — 승인 프로세스 (PR + 사람 승인)

영상이 가장 강조한 지점: **AI 검증만으로 끝내지 않고 반드시 사람이 머지를 승인한다.** 이걸 빼면 저장소는 즉시 쓰레기장이 됩니다.

```bash
# 에이전트가 만든 승격 브랜치 확인
git checkout -b promote/payment-decision
# ... 에이전트가 context/ 로 파일 이동 + INDEX/LOG 갱신 ...
git add . && git commit -m "promote: payment-decision to team context"

# 자체 검증(에이전트): 규칙 준수 여부 / 기존 지식 충돌 여부 리뷰
# 최종 머지 승인(사람):
git checkout main
git merge --no-ff promote/payment-decision   # ← 사람이 판단해서 승인
```

> GitHub/GitLab을 쓴다면 실제 Pull Request로 올리고, `CODEOWNERS`로 리뷰어를 지정하면 영상의 "코드 오너 → 리뷰 → 롤백" 흐름이 그대로 재현됩니다. Git이 원래 변경 관리 플랫폼이므로 *누가/언제/무엇을* 바꿨는지 항상 검색·롤백 가능합니다.

**승인 게이트 요약:**

- 에이전트: 규칙 준수·충돌 점검·위치 추천(자동)
- 사람: 팀 지식으로 확정할지 **최종 머지 승인**(수동, 생략 금지)

---

## 8. STEP 6 — RAG(OpenViking) 연동: 문서 폭증 해결

마크다운이 수십 개·수천 줄로 폭증하면 Obsidian/인덱스만으로는 감당이 안 됩니다. 영상은 **OpenViking**(파일시스템 패러다임의 컨텍스트 DB)을 연동해 L0~L2 계층 로딩으로 토큰을 절감합니다.

> OpenViking은 메모리·리소스·스킬을 `viking://` 가상 파일시스템으로 통합 관리하고, 문서를 **L0(제목 ~100토큰) / L1(요약 ~2k토큰) / L2(원문)** 으로 자동 분할해 필요한 계층만 로드합니다. 벤치마크상 전통적 RAG 대비 토큰 비용 83~92% 절감.

### 8-1. 설치
[Releases](https://github.com/Open-Viking/OpenViking/releases)에서 OS별 설치 파일을 받습니다.

- **macOS**: `OpenViking_macOS.dmg` → OpenClaw를 Applications로 드래그 → 실행해 초기화 (보안 경고 시 우클릭 → "열기")
- **Windows**: `OpenViking_x64.exe` 실행 → Deer-Flow 앱 실행

### 8-2. 모델 설정
`~/.openviking/ov.conf` 생성 (임베딩 + VLM 모델 필요). OpenAI 예시:

```json
{
  "storage": { "workspace": "/path/to/knowledge-context" },
  "embedding": {
    "dense": {
      "provider": "openai",
      "model": "text-embedding-3-large",
      "api_key": "YOUR_API_KEY",
      "api_base": "https://api.openai.com/v1"
    }
  },
  "vlm": {
    "provider": "openai",
    "model": "gpt-4o",
    "api_key": "YOUR_API_KEY",
    "api_base": "https://api.openai.com/v1"
  }
}
```

```bash
export OPENVIKING_CONFIG_FILE=~/.openviking/ov.conf   # Linux/macOS
```

> Claude/Gemini/Ollama를 쓰려면 LiteLLM provider로 설정하면 됩니다. `storage.workspace`를 우리 `knowledge-context` 레포 경로로 잡는 것이 핵심 — 영상의 "기존 깃 레포와 연동이 잘됐다"는 부분입니다.

### 8-3. 서버 실행 + 인덱싱
```bash
openviking-server                       # 서버 기동
# (백그라운드: nohup openviking-server > /tmp/ov.log 2>&1 &)

ov status                               # 상태 확인
ov add-resource ./sources               # 원본을 L0/L1/L2로 자동 인덱싱
ov tree viking://resources -L 2         # 가상 폴더 구조 확인
ov find "결제 모듈 카드 우선 정책"          # 디렉토리 재귀 검색
```

### 8-4. 대화형 에이전트 (선택)
```bash
pip install "openviking[bot]"
openviking-server --with-bot
ov chat                                 # 새 터미널에서 인터랙티브 채팅
```

**연동 흐름 정리:** 저장(Write time)에 원본을 정제 문서로 한 번 정리해 넣고(L1), 질문할 때는 L0/L1으로 관련 "폴더"를 먼저 찾은 뒤 필요한 경우에만 L2(원문)를 로드합니다. 인덱스를 *지속 관리하는 프로세스*와 *검색하는 프로세스*를 분리하는 것이 운영 포인트입니다.

---

## 9. STEP 7 — 거버넌스: AI에게 어디까지 맡길까

영상 마지막 주제. **"사람이 할 일을 먼저 정의하고, 나머지 귀찮은 일을 에이전트에 위임"**합니다.

| 사람이 직접 (위임 금지) | 에이전트에 위임 |
|---|---|
| `_private` 영역 관리 | 원본 요약·정제 문서 작성 |
| 팀 지식(`context/`) **삭제** 결정 | 위키링크 연결·INDEX/LOG 갱신 |
| 민감 권한·보안 변경 | 승격 PR 초안 생성, 충돌 점검 |
| PR **최종 머지 승인** | 중복 문서 탐지·병합 제안(self-evolving) |

이 경계만 `AGENTS.md`(Harness)에 명시하면 거버넌스는 의외로 간단해지고 시스템이 안정화됩니다.

---

## 10. 데모 실행 체크리스트

- [ ] `knowledge-context` 레포 생성 + 폴더 구조 + `.gitignore`(`_private` 제외)
- [ ] Obsidian으로 볼트 열기 + CLI 활성화(1.12+)
- [ ] `INDEX.md` / `LOG.md` / `AGENTS.md` 작성
- [ ] Obsidian Skills 클론 + Claude Code/Codex 연동
- [ ] 원본 → 멤버 노트 요약 시연 (INDEX/LOG 자동 갱신 확인)
- [ ] "팀에 공유하고 싶어" → 승격 PR 생성 → **사람이 머지 승인**
- [ ] (문서 많아졌다 가정) OpenViking 설치·설정·`ov add-resource`·`ov find`
- [ ] 거버넌스 경계(`_private`/삭제/권한)를 `AGENTS.md`에 명시

---

## 11. 한 장 요약

> 작은 아이디어인 **LLM Wiki**(INDEX/LOG)에서 출발해, **Git**이라는 동기화·승인 시스템 위에서 **Obsidian**으로 사람이 운영 가능하게 만들고, 문서가 폭증하면 **OpenViking(RAG)**으로 계층 검색을 붙이며, 마지막에 **사람-에이전트 역할 분담(거버넌스)**으로 안정화한다. 핵심은 도구가 아니라 — **어떤 지식을, 어디에 두고, 언제 제공할지**를 결정하는 것.

---

## 출처 (Sources)

- 원본 영상: [언제까지 AI에게 매번 같은 설명을 다시 해야할까? | LLM Wiki (Feat. RAG와의 차이)](https://www.youtube.com/watch?v=4XBofMHVyyc)
- LLM Wiki 개념(INDEX.md/LOG.md): [Karpathy's LLM Wiki as Agent Memory — AAIF](https://aaif.io/blog/karpathys-llm-wiki-as-agent-memory/), [LLM Wiki Tutorial — Data Science Dojo](https://datasciencedojo.com/blog/llm-wiki-tutorial/)
- OpenViking: [Open-Viking/OpenViking (GitHub)](https://github.com/Open-Viking/OpenViking), [volcengine/OpenViking (GitHub)](https://github.com/volcengine/OpenViking)
- Obsidian CLI: [The Obsidian CLI: Complete Guide — Frank Anaya](https://frankanaya.com/obsidian-cli/), [Obsidian CLI + Claude Code — Jim Christian](https://jimchristian.net/blog/2026/02/28/obsidian-cli-claude-code/)
- Obsidian Skills(Steph Ango/kepano): [Steph Ango's Official Obsidian Skills — aiwithmo](https://www.aiwithmo.com/blog/steph-ango-obsidian-skills), [Obsidian Skills — DEV Community](https://dev.to/stevengonsalvez/obsidian-skills-let-your-agent-manage-your-second-brain-4fel)
