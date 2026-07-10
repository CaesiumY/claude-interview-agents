---
description: 3-에이전트 토론 시스템을 통해 모의 면접 결과를 평가합니다
argument-hint: <질문지파일경로>
---

# 모의 면접 평가 명령

3-에이전트 토론 시스템을 통해 모의 면접 결과를 평가합니다.
**서브 에이전트를 활용한 독립적 병렬 평가**를 수행합니다.

## 사용법
```
/evaluate [질문지파일경로]
```

질문지 파일에는 각 질문에 대한 답변이 기록되어 있어야 합니다.

---

## 실행 절차

> **경로 규칙**: 입력 파일과 산출물 경로는 모두 **커맨드를 실행한 현재 작업 디렉토리 기준**입니다.

### 1단계: 평가 자료 수집
1. 이력서 파일 읽기 (`resumes/[이름].md`)
2. 질문지 및 답변 기록 읽기 (`resumes/[이름]_questionnaire.md`)
3. 지원자 이름 추출
4. 모의면접 평가 파일(`resumes/[이름]_mock_*_evaluation.md`) 존재 여부 확인 (있으면 4단계에서 참고 자료로 전달)

### 2단계: 독립 평가 실행 (병렬)

**중요**: 아래 두 평가는 서브 에이전트를 사용하여 **병렬로** 실행합니다.
각 서브 에이전트는 독립된 컨텍스트에서 작동하여 Anchoring Bias를 방지합니다.

#### Task 1: 기술 평가자 서브 에이전트 호출

`technical-evaluator` 서브 에이전트를 사용합니다.

```
Use the technical-evaluator subagent to evaluate the following candidate:

## 평가 대상
- 이력서: [이력서 전문]
- 질문지 및 답변: [질문지 전문]

## 요청
위 자료를 기반으로 기술 역량을 평가하고, 지정된 JSON 형식으로 결과를 출력하세요.
```

#### Task 2: 커뮤니케이션 평가자 서브 에이전트 호출

`communication-evaluator` 서브 에이전트를 사용합니다.

```
Use the communication-evaluator subagent to evaluate the following candidate:

## 평가 대상
- 이력서: [이력서 전문]
- 질문지 및 답변: [질문지 전문]

## 요청
위 자료를 기반으로 커뮤니케이션 및 소프트 스킬을 평가하고, 지정된 JSON 형식으로 결과를 출력하세요.
```

### 3단계: 평가 결과 수집
두 서브 에이전트의 JSON 출력을 수집합니다.

### 4단계: 최종 조율자 서브 에이전트 호출 (순차)

`final-arbiter` 서브 에이전트를 사용합니다.

```
Use the final-arbiter subagent to synthesize the evaluation results:

## 기술 평가 결과
[기술 평가자 JSON 출력]

## 커뮤니케이션 평가 결과
[커뮤니케이션 평가자 JSON 출력]

## 원본 자료
- 지원자 이름: [이름]
- 이력서 경로: resumes/[이름].md
- 질문지 경로: resumes/[이름]_questionnaire.md

## 참고 자료 (존재하는 경우에만)
- 모의면접 평가 경로: resumes/[이름]_mock_[type]_[YYYY-MM-DD]_evaluation.md
  (가중치 계산에는 포함하지 말고 종합 판정 서술에만 반영할 것)

## 요청
두 평가 결과를 종합하여 최종 평가 보고서를 작성하고,
`resumes/[이름]_evaluation.md` 파일로 저장하세요.
```

### 5단계: 결과 확인
- 생성된 `resumes/[이름]_evaluation.md` 파일 내용을 사용자에게 표시

---

## 서브 에이전트 구성

이 커맨드는 플러그인의 서브 에이전트를 활용합니다:

| 에이전트 | 역할 | 가중치 |
|----------|------|--------|
| `technical-evaluator` | 기술 역량 평가 전문가 | 45% |
| `communication-evaluator` | 소프트 스킬 평가 전문가 | 30% |
| `final-arbiter` | 최종 조율 및 판정 | 25% (성장가능성) |

### 병렬 실행 지시

기술 평가자와 커뮤니케이션 평가자는 **반드시 병렬로** 실행되어야 합니다.
다음과 같이 두 Task를 동시에 호출하세요:

> "Run these two subagent tasks in parallel:
> 1. technical-evaluator with [evaluation data]
> 2. communication-evaluator with [evaluation data]"

---

## 출력 형식

최종 보고서 형식은 `final-arbiter` 에이전트에 정의되어 있으며,
`resumes/[이름]_evaluation.md`에 저장됩니다:
지원자 정보 → 에이전트별 평가 → 토론 기록 → 최종 결과(가중 점수) → 개선 액션 플랜 → 예상 면접 결과
