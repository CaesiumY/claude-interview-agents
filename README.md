# Interview Agents — 프론트엔드 이직 준비 Claude Code 플러그인

프론트엔드 개발자 3년차 이직을 위한 AI 에이전트 시스템입니다.
**이력서 리뷰 → 기술 면접 준비 → 임원/컬처핏 모의면접 → 포트폴리오 제작**까지 이직의 전 여정을 지원합니다.

포트폴리오 파이프라인은 [이력서 한 줄을 포트폴리오로 전개하는 4-Phase 하니스 설계](https://caesiumy.dev/posts/ai/portfolio-strategy-harness-design/)를 구현한 것입니다.

---

## 목차

1. [설치](#설치)
2. [커맨드 한눈에 보기](#커맨드-한눈에-보기)
3. [전체 워크플로우](#전체-워크플로우)
4. [포트폴리오 하니스 (4-Phase)](#포트폴리오-하니스-4-phase)
5. [모의면접 (임원/컬처핏)](#모의면접-임원컬처핏)
6. [에이전트 아키텍처](#에이전트-아키텍처)
7. [모델 교체 가이드 (Fable → Opus)](#모델-교체-가이드-fable--opus)
8. [옵시디언(Claudian)에서 사용하기](#옵시디언claudian에서-사용하기)
9. [평가 기준 요약](#평가-기준-요약)
10. [FAQ](#faq)

---

## 설치

이 레포는 Claude Code **플러그인이자 자체 마켓플레이스**입니다.

### 방법 1: GitHub에서 설치 (권장)

```bash
claude plugin marketplace add caesiumy/claude-interview-agents
claude plugin install interview-agents
```

### 방법 2: 로컬 경로에서 설치

```bash
claude plugin marketplace add C:\path\to\claude-interview-agents
claude plugin install interview-agents
```

### 방법 3: 개발/테스트용 임시 로드

```bash
claude --plugin-dir C:\path\to\claude-interview-agents
```

설치 후 커맨드는 네임스페이스가 붙어 노출됩니다: `/interview-agents:review-resume` 형식.
(이 레포를 프로젝트로 직접 열었을 때가 아니라 **플러그인으로 설치했을 때** 기준)

> **산출물 위치**: 모든 결과 파일(`resumes/`, `portfolio/`)은 플러그인 설치 위치가 아니라
> **커맨드를 실행한 현재 작업 디렉토리**에 생성됩니다. 이직 준비 전용 폴더(또는 옵시디언 vault)에서 실행하세요.

---

## 커맨드 한눈에 보기

| 커맨드 | 하는 일 | 산출물 |
|--------|---------|--------|
| `/review-resume <이력서>` | 이력서 100점 만점 리뷰 + PSR 분해 검증 | `resumes/[이름]_review.md` |
| `/create-questionnaire <이력서>` | 맞춤 면접 질문지 생성 (질문 4카테고리) | `resumes/[이름]_questionnaire.md` |
| `/evaluate <질문지>` | 3-에이전트 병렬 평가 + 최종 판정 | `resumes/[이름]_evaluation.md` |
| `/mock-interview <이력서> [--type executive\|culture]` | 인터랙티브 임원/컬처핏 모의면접 + 평가 | `resumes/[이름]_mock_[type]_[YYYY-MM-DD]_evaluation.md` |
| `/portfolio <이력서> [JD]` | 4-Phase 포트폴리오 하니스 | `portfolio/<항목>/final.md` |

---

## 전체 워크플로우

```
                        ┌──────────────────────────────────────────┐
                        │              이력서 작성                   │
                        └──────────────┬───────────────────────────┘
                                       ▼
                        ┌──────────────────────────────────────────┐
                        │  /review-resume  (PSR 분해로 약점 진단)     │
                        └──────┬───────────────────────┬───────────┘
                               ▼                       ▼
              ┌────────────────────────┐   ┌────────────────────────┐
              │ /portfolio             │   │ /create-questionnaire  │
              │ 4-Phase 하니스로        │   │ 기술 면접 질문지 생성     │
              │ 포트폴리오 제작          │──▶│ (포트폴리오를 질문        │
              └────────────────────────┘   │  소스로 활용)           │
                                           └───────────┬────────────┘
                                                       ▼ 답변 작성
                        ┌──────────────────────────────────────────┐
                        │  /evaluate  (3-에이전트 병렬 평가)          │
                        └──────────────┬───────────────────────────┘
                                       ▼
                        ┌──────────────────────────────────────────┐
                        │  /mock-interview  (임원/컬처핏 모의면접)    │
                        │  → 평가 결과는 /evaluate 재실행 시 참고자료   │
                        └──────────────────────────────────────────┘
```

각 단계는 독립적으로도 사용할 수 있지만, 위 순서대로 진행하면 산출물이 다음 단계의 입력으로 연결됩니다.

---

## 포트폴리오 하니스 (4-Phase)

**"포트폴리오는 면접관에게 질문을 유도하는 트리거다"** — 이 관점으로 이력서 한 줄을 포트폴리오 섹션으로 전개합니다.

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Curation   │ →  │  Interview  │ →  │  Drafting   │ →  │   Review    │
│   AI 주도   │    │  인간 주도  │    │   AI 주도   │    │  AI + 인간  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
   후보 선별        디테일 채우기      마크다운 작성       다차원 평가
   ✋ GATE 1         ✋ GATE 2                            ✋ GATE 3·4
```

| Phase | 내용 |
|-------|------|
| **1. Curation** | JD를 5슬롯(회사·도메인/핵심책임/우대기술/시니어리티/인재상)으로 파싱 → 이력서 항목을 기술40+책임30+인재상20+Featured10으로 채점 → PSR(문제/해결/결과) 분해로 누락 표시 → **GATE 1**: 작성할 항목 선택 |
| **2. Interview** | 항목당 3~5개 질문(앵커링/정량/의사결정/실패 4카테고리, 정량 필수)을 한 번에 하나씩 → 답변은 즉시 `interview-notes.md`에 저장 → **GATE 2**: 노트 검토·편집 승인 |
| **3. Drafting** | `portfolio-strategist` 에이전트가 인터뷰 노트+JD+템플릿+톤 가이드로 초안 작성. 노트에 없는 사실 창작 금지. 게이트 없이 Phase 4로 자동 진행 |
| **4. Review** | `portfolio-reviewer` 에이전트(작성자와 완전 분리)가 다차원 임계 평가: 구조(Binary)/적합도(70+)/가독성(75+, 단어비율 1:8~1:15)/종합(80+). 미달 시 최대 3회 자동 재작성 → **GATE 3·4**: 확인 후 `final.md` 저장 |

### 핵심 설계 원칙

1. **작성자 ≠ 평가자**: 작성(strategist)과 평가(reviewer)는 별도 서브에이전트 + 참조 파일까지 분리.
   AI가 자기 결과물에 관대해지는 경향과 컨텍스트 오염을 차단합니다.
2. **한 번 답한 것은 다시 답하지 않는다**: 인터뷰 답변은 `interview-notes.md`에 영구 저장.
   다른 회사 JD로 재실행하면 Phase 2가 대폭 단축됩니다.
3. **AI 산출물은 파일로 남긴다**: 모든 중간 결과가 파일이므로 언제든 직접 검토·수정 가능합니다.

산출물 구조는 [portfolio/README.md](portfolio/README.md) 참조.

---

## 모의면접 (임원/컬처핏)

```bash
/mock-interview resumes/홍길동.md --type executive   # 임원 면접
/mock-interview resumes/홍길동.md --type culture     # 컬처핏 면접
```

- **한 질문씩** 진행하고, 답변에 따라 **꼬리질문 1~2개**가 즉석에서 나옵니다 (앵커링·정량·일관성·구체화)
- **세션 중에는 피드백이 없습니다** — 실제 면접처럼 긴장감을 유지합니다
- 포트폴리오(`portfolio/*/final.md`)가 있으면 거기 심어둔 질문 트리거에서 질문이 나옵니다 —
  포트폴리오 설계가 실제로 작동하는지 리허설하는 효과
- 종료 후 `culture-fit-evaluator`가 평가: 진정성25/조직적합25/성장서사20/태도15/리스크신호15
- 평가 파일은 이후 `/evaluate` 실행 시 최종 조율자의 **참고 자료**로 자동 활용됩니다

---

## 에이전트 아키텍처

| 에이전트 | 역할 | 성향 | 호출 커맨드 |
|----------|------|------|------------|
| `technical-evaluator` | 기술 역량 평가 (45%) | 엄격·비판적 | `/evaluate` |
| `communication-evaluator` | 소프트 스킬 평가 (30%) | 유연·긍정적 | `/evaluate` |
| `final-arbiter` | 가중치 종합·최종 판정 | 중립 | `/evaluate` |
| `culture-fit-evaluator` | 임원/컬처핏 세션 평가 | 사람을 보는 시선 | `/mock-interview` |
| `portfolio-strategist` | 포트폴리오 초안 작성 | 서사 설계 에디터 | `/portfolio` Phase 3 |
| `portfolio-reviewer` | 채용 담당자 시점 평가 | 바쁜 서류 검토자 | `/portfolio` Phase 4 |

- 기술/커뮤니케이션 평가자는 **병렬 실행**으로 Anchoring Bias를 방지합니다
- strategist/reviewer는 **상호 참조 금지** (templates.md ↔ review-rubric.md 접근 분리)

---

## 모델 교체 가이드 (Fable → Opus)

모든 에이전트는 현재 `model: claude-fable-5`로 고정되어 있습니다.
Fable 지원이 종료되면 아래 한 줄로 전체 교체하세요. **교체 지점은 `agents/*.md`의 frontmatter 6곳뿐입니다**
(각 위치에 `# MODEL SWAP POINT` 주석이 있고, 커맨드/스킬에는 의도적으로 model 필드가 없습니다).

**PowerShell (Windows):**
```powershell
Get-ChildItem agents/*.md | ForEach-Object { (Get-Content $_ -Raw) -replace 'claude-fable-5', 'claude-opus-4-8' | Set-Content $_ -Encoding utf8 }
```

**bash (macOS/Linux):**
```bash
sed -i 's/claude-fable-5/claude-opus-4-8/' agents/*.md
```

**교체 확인:**
```bash
grep -r "claude-fable-5" agents/   # 아무것도 나오지 않아야 함
```

교체 후 플러그인을 재설치하거나 세션에서 `/reload-plugins`를 실행하세요.

---

## 옵시디언(Claudian)에서 사용하기

Claudian은 옵시디언 안에서 Claude Code를 실행하는 확장 프로그램으로, **`~/.claude` 설정을 CLI와 공유**합니다.
따라서 위 [설치](#설치) 방법대로 CLI에서 플러그인을 설치하면 별도 설정 없이 옵시디언에서도 커맨드를 사용할 수 있습니다.

권장 사용법:
1. CLI에서 플러그인 설치 (방법 1 또는 2)
2. 옵시디언 vault 안에 이직 준비 폴더를 만들고 이력서를 `resumes/`에 배치
3. Claudian 채팅에서 `/interview-agents:review-resume resumes/이력서.md` 실행
4. 산출물(리뷰, 질문지, 인터뷰 노트, 포트폴리오)이 vault 안에 마크다운으로 쌓임 —
   옵시디언의 링크·검색과 자연스럽게 결합됩니다

특히 `/portfolio`의 인터뷰 노트는 vault에 영구 자산으로 남아, 회사별 지원 이력과 함께 관리하기 좋습니다.

---

## 평가 기준 요약

### 이력서 리뷰 (100점)
기본 구성 20 / 기술 역량 표현 30 / 성과 중심 기술 25 (PSR 검증 연동) / 프로젝트 경험 15 / 가독성 10

### 3년차 가중치 (/evaluate)
기술 45% / 소프트 스킬 30% / 성장 가능성 25%

### 컬처핏 평가 (100점)
진정성 25 / 조직 적합성 25 / 성장 서사 20 / 태도 15 / 리스크 신호 15

### 포트폴리오 임계 (전부 통과해야 PASS)
구조: PSR+정량지표 (Binary) / 적합도: 70+ / 가독성: 75+ (단어비율 1:8~1:15) / 종합: 80+

### 최종 판정
85-100 강력 합격 / 70-84 합격 / 60-69 보류 / 50-59 조건부 / 0-49 불합격

---

## FAQ

**Q. JD 없이 `/portfolio`를 실행해도 되나요?**
네. JD 슬롯을 비워두고 일반적인 프론트엔드 3년차 채용 기준으로 채점·평가합니다.
목표 회사가 정해지면 JD를 넣고 재실행하세요 — 인터뷰 노트가 재사용되어 빠르게 끝납니다.

**Q. 포트폴리오 재작성 루프가 3회를 넘으면 어떻게 되나요?**
마지막 초안과 남은 문제를 그대로 보여드립니다. 직접 수정하거나, 부족한 디테일을 인터뷰 노트에 추가한 뒤 다시 실행하세요.

**Q. 모의면접 중간에 그만두면?**
답변이 즉시 세션 파일에 저장되므로 유실되지 않습니다. "종료"라고 하면 그 시점까지의 답변으로 평가를 진행합니다.

**Q. 왜 에이전트마다 모델을 고정했나요?**
평가 일관성을 위해서입니다. 교체가 필요하면 [모델 교체 가이드](#모델-교체-가이드-fable--opus)의 한 줄 명령으로 6곳이 한 번에 바뀝니다.

---

## 라이선스

MIT — [LICENSE](LICENSE) 참조
