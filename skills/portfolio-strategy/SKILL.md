---
name: portfolio-strategy
description: 이력서 한 줄을 포트폴리오 섹션으로 전개하는 4-Phase 하니스(Curation→Interview→Drafting→Review)의 전략 원칙과 단계별 기준을 제공합니다. /portfolio 커맨드에서 참조됩니다.
---

# 포트폴리오 전략 스킬 (4-Phase 하니스)

## 핵심 관점

**포트폴리오는 면접관에게 질문을 유도하는 트리거다.**

- 블로그 글은 정보 전달이 목적이지만, 포트폴리오는 압축된 형식으로 면접관의 질문을 미리 유도한다
- "면접까지 간다면 이거 한 번 물어봐 주세요"라는 신호를 설계하는 작업이다
- 이력서 한 줄은 진짜 경험을 압축한 결과물이지, 그 자체가 경험은 아니다
- **AI의 역할**: 말투와 표현의 일관성 / **인간의 역할**: 진정한 경험 디테일과 의사결정권

## 전체 파이프라인

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Curation   │ →  │  Interview  │ →  │  Drafting   │ →  │   Review    │
│   AI 주도   │    │  인간 주도  │    │   AI 주도   │    │  AI + 인간  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
   후보 선별        디테일 채우기      마크다운 작성       다차원 평가
   ✋ GATE 1         ✋ GATE 2                            ✋ GATE 3·4
```

| Phase | 주도 | 참조 파일 | 산출물 |
|-------|------|----------|--------|
| 1. Curation | AI | `jd-parsing.md` | `portfolio/_curation/curation-[날짜].md` |
| 2. Interview | 인간 | `interview-protocol.md` | `portfolio/<slug>/interview-notes.md` |
| 3. Drafting | AI (strategist) | `templates.md` | `portfolio/<slug>/draft-v{n}.md` |
| 4. Review | AI (reviewer) + 인간 | `review-rubric.md` | `portfolio/<slug>/review-v{n}.md`, `final.md` |

## 3대 설계 원칙

### 1. 작성자 ≠ 평가자
- Phase 3(portfolio-strategist)과 Phase 4(portfolio-reviewer)는 **반드시 다른 서브에이전트**로 실행
- 이유: 작성 단계의 컨텍스트가 평가를 좁히는 오염 방지 + AI도 자기 결과물에 관대해지는 경향 제거
- 참조 파일도 분리: strategist는 `templates.md`만, reviewer는 `review-rubric.md`만 읽는다

### 2. 인터뷰 답변은 파일로 저장
- 답변은 메모리에만 두지 않고 **즉시** `interview-notes.md`에 기록
- 이유: 사용자가 검토·편집할 승인 지점 확보 + 같은 프로젝트로 다른 JD에 재사용 가능한 영구 자산
- **"한 번 답한 것은 다시 답할 필요가 없어야 한다"** — 기존 노트가 있으면 부족분만 질문

### 3. AI 산출물은 사용자가 볼 수 있는 자리에 남긴다
- 인터뷰 노트 → 파일로 노출
- 평가 결과 → 파일 저장 + 사용자 게이트 화면에 노출
- 초안 → 검토 전 파일로 노출

## 사용자 게이트 (자동 진행 금지 지점)

| 게이트 | 시점 | 사용자 결정 |
|--------|------|------------|
| GATE 1 | Phase 1 완료 후 | 작성할 항목 N개 최종 선택 |
| GATE 2 | Phase 2 완료 후 | 인터뷰 노트 검토·편집 후 진행 승인 |
| GATE 3·4 | Phase 4 평가 후 | 평가 결과 확인 및 최종 저장(final.md) 승인 |

**Phase 3 → Phase 4는 게이트 없이 자동 진행**하며, 임계 미통과 시 최대 3회 자동 재작성 루프를 돕니다.

## 재사용 시나리오 (다른 회사 JD 적용)

같은 이력서 항목으로 다른 JD에 지원할 때:
1. Phase 1을 새 JD로 다시 실행 (채점·선별은 JD 의존적)
2. Phase 2는 기존 `interview-notes.md`를 로드하고 새 JD 관점에서 부족한 질문만 추가
3. Phase 3·4는 새 `jd-context.md`로 정상 실행
