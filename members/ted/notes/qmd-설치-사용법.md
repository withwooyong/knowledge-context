---
title: QMD 설치 및 사용법
status: draft
owner: ted
scope: members
source: qmd --help (v2.1.0), qmd status
tags: [qmd, search, markdown, rag, mcp, tooling]
created: 2026-06-16
---

# QMD 설치 및 사용법

> QMD = **Quick Markdown Search**. 로컬 마크다운 문서에 대한 검색 엔진으로, BM25 키워드 + 벡터 시맨틱 + 리랭킹을 합친 하이브리드 검색을 제공한다. MCP 서버로 Claude Code 등 AI 에이전트에 연결된다.
>
> 패키지: `@tobilu/qmd` · 확인 시점 기준 버전 v2.1.0 · 인덱스: `~/.cache/qmd/index.sqlite`

---

## 1. 설치

```bash
# npm 전역 설치
npm install -g @tobilu/qmd

qmd --version          # 버전 확인 (예: qmd 2.1.0)
qmd status             # 인덱스/컬렉션 상태
```

- 인덱스는 `~/.cache/qmd/index.sqlite`에 저장된다.

---

## 2. 검색 (3종류)

```bash
qmd query "결제 모듈 카드 우선 정책"        # 추천 — 자동확장 + 리랭킹 하이브리드
qmd search "error handling"               # BM25 키워드 (LLM 없이 빠름)
qmd vsearch "어떻게 에러를 우아하게 처리하나"   # 벡터 시맨틱만
```

### 구조화 쿼리 (정밀 제어)

```bash
qmd query $'intent: 인증 흐름\nlex: CAP theorem\nvec: consistency'
```

- 줄마다 타입 지정: `lex:` 키워드 / `vec:` 시맨틱 / `hyde:` 가상문서
- `intent:` 줄로 검색 의도를 명시하면 스니펫 품질이 좋아진다.

---

## 3. 문서 보기

```bash
qmd get path/to/doc.md            # 단일 문서
qmd get doc.md:120 -l 30          # 120번 줄부터 30줄
qmd multi-get "**/*.md"           # glob / 쉼표목록 일괄 조회
qmd ls accounting-handbook        # 특정 컬렉션의 인덱스된 파일 목록
```

---

## 4. 컬렉션 관리 (폴더 인덱싱)

```bash
qmd collection add <이름> <경로> --pattern "**/*.md"
qmd collection list
qmd collection remove/rename/show ...

qmd context add <이름> "이 컬렉션 한 줄 설명"   # 사람이 쓴 요약을 첨부
qmd context list / rm

qmd update [--pull]      # 재인덱싱 (옵션: git pull 먼저)
qmd embed [-f]           # 벡터 임베딩 생성/갱신
```

> 벡터 검색(`qmd vsearch`, `qmd query`의 시맨틱 부분)을 쓰려면 `qmd embed`로 임베딩을 만들어야 하며, 임베딩 모델(API 키) 설정이 필요하다. 키워드 검색(`qmd search`)은 키 없이 바로 동작한다.

---

## 5. 유지보수 / MCP 연동

```bash
qmd status              # 인덱스 건강 상태 (문서 수, 임베딩, 갱신 시점)
qmd cleanup             # 캐시 정리 + DB vacuum
qmd mcp                 # MCP 서버 기동 (AI 에이전트용 stdio transport)
qmd skill show/install  # 패키지된 QMD 스킬 표시/설치
qmd bench <fixture.json> # 검색 품질 벤치마크
```

---

## 6. 이 볼트(knowledge-context)에 적용하기

문서가 늘어나면 LLM Wiki 가이드의 STEP 6(RAG 계층 검색)을 OpenViking 대신 QMD로 바로 체험할 수 있다.

```bash
qmd collection add knowledge-context /Users/heowooyong/cursor/learning/knowledge-context --pattern "**/*.md"
qmd context add knowledge-context "LLM Wiki 데모 볼트 — sources/members/context 레이어"
qmd update
```

---

## 출처

- `qmd --help` (v2.1.0) 및 `qmd status` 출력 기준으로 정리.
- 관련: LLM Wiki 데모 가이드의 STEP 6(RAG 연동) — 문서 폭증 시 계층 검색.
