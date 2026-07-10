---
description: 이력서 한 줄을 포트폴리오 섹션으로 전개하는 4-Phase 하니스를 실행합니다 (Curation→Interview→Drafting→Review)
argument-hint: <이력서파일경로> [JD파일경로]
---

# 포트폴리오 하니스 명령 (4-Phase)

이력서 한 줄을 채용 담당자가 질문하고 싶어지는 포트폴리오 섹션으로 전개합니다.

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Curation   │ →  │  Interview  │ →  │  Drafting   │ →  │   Review    │
│   AI 주도   │    │  인간 주도  │    │   AI 주도   │    │  AI + 인간  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
   ✋ GATE 1         ✋ GATE 2          (자동 진행)        ✋ GATE 3·4
```

## 사용법
```
/portfolio resumes/[이름].md                    # JD 없이 일반 기준으로
/portfolio resumes/[이름].md jd/[회사명].md     # 특정 회사 JD 기준으로
```

## 전략 원칙 참조

시작 전에 `${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/SKILL.md`를 읽어
파이프라인 전체 원칙(작성자≠평가자, 답변 파일 저장, 사용자 게이트)을 확인하세요.

> **경로 규칙**: 산출물은 모두 **커맨드를 실행한 현재 작업 디렉토리의 `portfolio/`** 아래에 생성합니다 (UTF-8).
> 디렉토리 구조 컨벤션은 산출물과 함께 자동 생성되는 파일 경로를 따릅니다:
> `portfolio/_curation/curation-[YYYY-MM-DD].md`, `portfolio/<항목-slug>/{jd-context.md, interview-notes.md, draft-v{n}.md, review-v{n}.md, final.md}`

---

## Phase 1: Curation — 후보 선별 (AI 주도)

가이드: `${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/jd-parsing.md`

1. 이력서를 읽고 경험 한 줄(bullet) 목록을 추출합니다
2. JD 파일이 제공되면 **5슬롯**으로 파싱합니다: 회사·도메인 / 핵심 책임 / 우대 기술 / 시니어리티 / 인재상
3. 각 항목을 **가중치 채점**합니다: 기술 매칭 40 + 책임 매칭 30 + 인재상 매칭 20 + Featured 10
4. 각 항목을 **PSR(문제/해결/결과)로 분해**하고 누락 요소를 `[P?]`/`[S?]`/`[R?]`로 표시합니다
5. 결과를 `portfolio/_curation/curation-[YYYY-MM-DD].md`에 저장합니다 (AI 산출물은 파일로 남긴다)

### ✋ GATE 1: 항목 선택
AskUserQuestion으로 채점 상위 항목을 제시하고, **작성할 항목 N개를 사용자가 최종 선택**합니다.
선택된 항목마다 `portfolio/<항목-slug>/` 디렉토리를 만들고 `jd-context.md`(5슬롯 파싱 결과)를 저장합니다.

---

## Phase 2: Interview — 디테일 채우기 (인간 주도)

가이드: `${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/interview-protocol.md`

1. **재사용 확인 먼저**: `portfolio/<slug>/interview-notes.md`가 이미 있으면 로드하고,
   새 JD 관점에서 부족한 질문만 추가합니다 — "한 번 답한 것은 다시 답하지 않는다"
2. 항목당 3~5개 질문을 4카테고리(앵커링/정량/의사결정·트레이드오프/실패)에서 선별합니다
   - **정량 질문 1개 이상 필수**, Phase 1의 PSR 누락 표시가 질문 선별의 최우선 입력
3. **한 번에 한 질문씩** 제시하고, 답변을 받으면 **즉시** `interview-notes.md`에 append합니다
4. "모르겠다" 답변도 그대로 기록합니다 — 답변을 창작하지 않습니다

### ✋ GATE 2: 인터뷰 노트 승인
모든 항목 완료 후, 사용자에게 `interview-notes.md` 파일 경로를 안내하고
**직접 검토·편집한 뒤 진행을 승인**하도록 요청합니다. 승인 전에는 Phase 3로 넘어가지 않습니다.

---

## Phase 3: Drafting — 마크다운 작성 (AI 주도, 게이트 없음)

**`portfolio-strategist` 서브 에이전트를 호출합니다** (직접 작성하지 않음):

```
Use the portfolio-strategist subagent:

## 작성 대상
- 이력서 원문 한 줄: [원문 그대로]
- 항목 디렉토리: portfolio/<slug>/

## 재료
- 인터뷰 노트: portfolio/<slug>/interview-notes.md
- JD 컨텍스트: portfolio/<slug>/jd-context.md
- 템플릿·톤 가이드: ${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/templates.md

## 요청
templates.md의 구조·전략 원칙·톤 가이드에 따라 초안을 작성하고
portfolio/<slug>/draft-v{n}.md로 저장하세요.
review-rubric.md는 읽지 마세요 (작성자≠평가자 분리).
```

완료되면 게이트 없이 Phase 4로 자동 진행합니다.

---

## Phase 4: Review — 채용 담당자 시점 평가 (AI 평가 + 인간 승인)

**`portfolio-reviewer` 서브 에이전트를 호출합니다.**
strategist와 **반드시 별도 호출**이며, strategist의 대화 내용·인터뷰 노트를 전달하지 않습니다:

```
Use the portfolio-reviewer subagent:

## 평가 대상
- 초안: portfolio/<slug>/draft-v{n}.md
- 이력서 원문 한 줄: [원문 그대로]
- JD 컨텍스트: portfolio/<slug>/jd-context.md
- 평가 기준: ${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/review-rubric.md
- 현재 루프 회차: {n}/3

## 요청
review-rubric.md의 다차원 임계 기준으로 평가하고
portfolio/<slug>/review-v{n}.md로 저장한 뒤 JSON 판정을 출력하세요.
인터뷰 노트와 templates.md는 읽지 마세요 (작성자≠평가자 분리).
```

### 자동 재작성 루프 (최대 3회)
- 판정이 `RETRY`이면: reviewer의 `retry_feedback`을 포함하여 Phase 3(strategist)를 재호출
  → draft-v{n+1} 작성 → Phase 4 재평가
- **최대 3회**. 3회차에도 미달(`FINAL_FAIL`)이면 마지막 초안과 남은 문제를 사용자에게 그대로 제시

### ✋ GATE 3·4: 평가 확인 및 최종 저장
1. 평가 결과(카테고리별 통과 여부, 점수, 면접관 질문 트리거 목록)를 사용자에게 표시
2. 사용자가 승인하면 통과한 초안을 `portfolio/<slug>/final.md`로 저장
3. 수정 요청이 있으면 반영 후 다시 확인

---

## 완료 후 안내

- `final.md`는 `/create-questionnaire`와 `/mock-interview`의 질문 소스로 자동 활용됩니다
- `interview-notes.md`는 영구 자산입니다 — 다른 회사 JD로 재실행 시 Phase 2가 대폭 단축됩니다
