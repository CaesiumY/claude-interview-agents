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
> 다른 JD로 재실행 시 이전 산출물은 `portfolio/<항목-slug>/_archive/[YYYY-MM-DD]/`로 보관합니다 (SKILL.md 재사용 시나리오 참조).

---

## Phase 1: Curation — 후보 선별 (AI 주도)

가이드: `${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/jd-parsing.md`

**재사용 확인 먼저**: `resumes/[이름]_review.md`(`/review-resume` 산출물)가 있으면 로드해
그 **PSR 분해 검증 표**를 4단계 입력으로 재사용하고 PSR 재분해를 생략합니다
("한 번 분해한 것은 다시 분해하지 않는다"). 단 리뷰의 bullet 목록과 현재 이력서의 bullet이
불일치하면(이력서 수정 등) **불일치하는 bullet만** 재분해합니다. 리뷰의 '포트폴리오 전개 추천'은
GATE 1 후보 선별의 참고 자료로 활용합니다. 파일이 없으면 아래 절차를 그대로 수행합니다 (선택적 입력).

1. 이력서를 읽고 경험 한 줄(bullet) 목록을 추출합니다
2. JD 파일이 제공되면 **5슬롯**으로 파싱합니다: 회사·도메인 / 핵심 책임 / 우대 기술 / 시니어리티 / 인재상
3. 각 항목을 **가중치 채점**합니다: 기술 매칭 40 + 책임 매칭 30 + 인재상 매칭 20 + Featured 10
4. 각 항목을 **PSR(문제/해결/결과)로 분해**하고 누락 요소를 `[P?]`/`[S?]`/`[R?]`로 표시합니다
   (위 재사용 확인에서 `[이름]_review.md`의 PSR 표를 로드한 경우, 일치하는 bullet은 그 분해를 그대로 사용합니다)
5. 결과를 `portfolio/_curation/curation-[YYYY-MM-DD].md`에 저장합니다 (AI 산출물은 파일로 남긴다)
   - 같은 날짜에 다른 JD로 재실행하는 경우 기존 파일을 덮어쓰지 않고 `curation-[YYYY-MM-DD]-2.md`처럼 순번 접미사를 붙입니다

### ✋ GATE 1: 항목 선택
AskUserQuestion(multiSelect)으로 채점 상위 항목을 제시하고, **작성할 항목 N개를 사용자가 최종 선택**합니다.

**선택지 형식** (실행마다 동일하게):
- `label`: 항목 축약 (30자 이내)
- `description`: "총점 N점 · PSR 누락 [P?]/[S?]/[R?] · 추천 이유 한 줄"
- 후보가 4개를 초과하면 상위 4개 + "기타 (curation 파일에서 직접 지정)" 옵션으로 제시합니다

**slug 생성 규칙**: 이력서 한 줄의 핵심 명사 2~3개를 영문 소문자 케밥케이스로 변환합니다
(예: "레거시 번들 최적화로 초기 로딩 40% 개선" → `bundle-optimization`).
단, 디렉토리 생성 전에 Glob으로 `portfolio/*/interview-notes.md`를 조회해 각 파일 1행의
이력서 한 줄 원문과 대조하고, **동일 항목이면 새 slug를 만들지 않고 기존 slug를 재사용**합니다
("한 번 답한 것은 다시 답하지 않는다"의 전제 조건).

선택된 항목마다 `portfolio/<항목-slug>/` 디렉토리를 만들고 `jd-context.md`(curation 파일의 5슬롯 파싱 결과를 복사)를 저장합니다.
기존 산출물이 있는 slug를 새 JD로 재실행하는 경우 `interview-notes.md`만 원위치에 유지하고,
나머지 기존 파일은 `portfolio/<slug>/_archive/[YYYY-MM-DD]/`로 이동한 뒤 새로 생성합니다.

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
모든 항목 완료 후:

1. **승인 전 체크리스트**: 메인 루프가 각 `interview-notes.md`를 직접 확인해 아래 표로 표시합니다

   | 항목 | 정량 수치 1개 이상 | P 재료 | S 재료 | R 재료 | 미답변 정량 질문 |
   |------|------------------|-------|-------|-------|----------------|
   | (slug) | ✅/❌ | ✅/❌ | ✅/❌ | ✅/❌ | 있음/없음 |

2. **정량 수치가 없는 항목은 승인 전에 차단**: 보충 정량 질문 1~2개를 추가로 진행합니다
   (또는 사용자가 "수치 없이 진행"을 명시적으로 선택 — 이 경우 Phase 4의 구조 Binary에서
   미달 판정이 반복될 수 있음을 미리 고지합니다).
   노트에 수치가 없으면 strategist는 창작 금지 원칙상 채울 수 없어 재작성 루프로는 절대 해결되지 않습니다.
3. 사용자에게 `interview-notes.md` 파일 경로를 안내하고 **직접 검토·편집한 뒤 진행을 승인**하도록
   요청합니다. 승인 전에는 Phase 3로 넘어가지 않습니다.

---

## Phase 3: Drafting — 마크다운 작성 (AI 주도, 게이트 없음)

> GATE 1에서 N개 항목을 선택한 경우, Phase 3→4(재작성 루프 포함)와 GATE 3·4는 **항목별로** 순차 수행합니다.

**호출 전 준비 (어절 수 계산)**: 메인 루프가 이력서 한 줄의 어절 수(공백 구분)를 Bash 등으로
직접 계산해 프롬프트에 주입합니다 — 에이전트의 자체 카운팅은 부정확할 수 있으므로
오케스트레이터가 검증된 수치를 제공합니다. Bash 사용이 불가한 실행 환경이라면 메인 루프가
이력서 한 줄을 직접 세어서라도 값을 만들어 주입합니다.

**`portfolio-strategist` 서브 에이전트를 호출합니다** (직접 작성하지 않음):

```
Use the portfolio-strategist subagent:

## 작성 대상
- 이력서 원문 한 줄: [원문 그대로]
- 항목 디렉토리: portfolio/<slug>/
- 이력서 한 줄 어절 수: X어절 (오케스트레이터 계산) → 목표 분량 8X~15X어절

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

**호출 전 준비 (어절 수 참고값 계산 — 필수)**: 메인 루프가 초안의 **본문 어절 수**(공백 구분,
마크다운 제목 `#`·인용 `>`·표·`[확인 필요]` 태그 제외)를 Bash 등으로 계산해 프롬프트에 **반드시** 주입합니다.
**참고값 없이 reviewer를 호출하지 않습니다** — reviewer는 Bash가 없어 스스로 셀 수 없고, 참고값이 없을 때의
자체 근사 카운트는 1:8~1:15 경계 근처 판정의 재현성이 낮기 때문입니다. Bash 등 계산 도구를 사용할 수 없는
실행 환경이라면 메인 루프가 초안 본문을 직접 세어서라도(어절 단위 수동 카운트) 참고값을 만들어 주입하고,
이 경우 아래 호출 프롬프트의 어절 수 참고값에 "(메인 루프 근사치)"를 표기해 reviewer가 정확도 차이를 인지하게 합니다.

**`portfolio-reviewer` 서브 에이전트를 호출합니다.**
strategist와 **반드시 별도 호출**이며, strategist의 대화 내용·인터뷰 노트를 전달하지 않습니다:

```
Use the portfolio-reviewer subagent:

## 평가 대상
- 초안: portfolio/<slug>/draft-v{n}.md
- 이력서 원문 한 줄: [원문 그대로]
- 어절 수 참고값: 이력서 한 줄 X어절 / 초안 본문 Y어절 (오케스트레이터 계산, Bash 불가 시 "메인 루프 근사치" 표기)
- JD 컨텍스트: portfolio/<slug>/jd-context.md
- 평가 기준: ${CLAUDE_PLUGIN_ROOT}/skills/portfolio-strategy/review-rubric.md
- 현재 루프 회차: {n}/3

## 요청
review-rubric.md의 다차원 임계 기준으로 평가하고
portfolio/<slug>/review-v{n}.md로 저장한 뒤 JSON 판정을 출력하세요.
인터뷰 노트와 templates.md는 읽지 마세요 (작성자≠평가자 분리).
```

### 자동 재작성 루프 (최대 3회)
- 판정이 `RETRY`이고 `retry_feedback.needs_user_input`이 `true`이면: **루프 회차를 소모하지 않고**
  사용자에게 부족한 사실(예: Before/After 수치)에 대한 보충 질문을 진행합니다
  → 답변을 `interview-notes.md`에 append → Phase 3(strategist) 재호출.
  사실 부재는 재작성으로 해결되지 않으므로, 자동 루프는 표현·구조 개선에만 사용합니다
- 판정이 `RETRY`(needs_user_input이 없거나 `false`)이면: reviewer의 `retry_feedback`을 포함하여
  Phase 3(strategist)를 재호출 → draft-v{n+1} 작성 → Phase 4 재평가
- **최대 3회**. 3회차에도 미달(`FINAL_FAIL`)이면 마지막 초안과 남은 문제를 사용자에게 그대로 제시

### ✋ GATE 3·4: 평가 확인 및 최종 저장
1. 평가 결과를 아래 표준 형식으로 사용자에게 표시합니다:

   | 카테고리 | 점수 | 임계 | 통과 |
   |---------|------|------|------|
   | 구조 | Binary (P/S/R/정량 지표) | 전 항목 있음 | ✅/❌ |
   | 적합도 | N점 | 70 | ✅/❌ |
   | 가독성 | N점 (어절 비율 1:X) | 75 | ✅/❌ |
   | 종합 | N점 | 80 | ✅/❌ |

   > 안내: 전 카테고리가 통과(✅)여도 종합이 80 미만이면 최종 판정은 FINAL_FAIL일 수 있습니다.
   > 종합 산식(적합도·가독성 각 ×0.4 + 트리거)상 경계 통과만으로는 80에 못 미치도록 한 **의도된 품질 게이트**입니다.

   이어서 면접관 질문 트리거 목록(`interviewer_questions_triggered`)을 함께 표시합니다
2. 사용자가 승인하면 통과한 초안을 `portfolio/<slug>/final.md`로 저장
3. 수정 요청이 있으면 반영 후 다시 확인
4. 항목이 여러 개면 항목별로 이 게이트를 반복합니다

---

## 완료 후 안내

- `final.md`는 `/create-questionnaire`와 `/mock-interview`의 질문 소스로 자동 활용됩니다
- `interview-notes.md`는 영구 자산입니다 — 다른 회사 JD로 재실행 시 Phase 2가 대폭 단축됩니다
