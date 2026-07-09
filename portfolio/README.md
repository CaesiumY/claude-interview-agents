# portfolio/ — 포트폴리오 하니스 산출물 디렉토리

`/portfolio` 커맨드(4-Phase 하니스)의 모든 산출물이 이 구조로 저장됩니다.
**산출물은 커맨드를 실행한 현재 작업 디렉토리 기준**으로 생성됩니다.

## 디렉토리 컨벤션

```
portfolio/
├── _curation/
│   └── curation-YYYY-MM-DD.md    # Phase 1 결과: JD 파싱 + 항목별 채점 + PSR 분해
└── <항목-slug>/                   # 선택된 이력서 항목 하나당 디렉토리 하나
    ├── jd-context.md              # 이 항목에 적용된 JD 파싱 결과 (5슬롯)
    ├── interview-notes.md         # Phase 2 인터뷰 답변 (영구 자산 — 다른 JD에 재사용)
    ├── draft-v1.md                # Phase 3 초안 (재작성 루프마다 v2, v3 추가)
    ├── review-v1.md               # Phase 4 평가 결과 (draft 버전과 짝을 이룸)
    └── final.md                   # GATE 통과 후 최종 확정본
```

## 원칙

1. **한 번 답한 것은 다시 답하지 않는다** — `interview-notes.md`는 삭제하지 마세요.
   같은 프로젝트로 다른 회사 JD에 지원할 때 Phase 2를 거의 건너뛸 수 있습니다.
2. **AI 산출물은 파일로 남긴다** — 모든 중간 결과가 파일로 저장되므로
   언제든 직접 열어 검토·수정할 수 있습니다.
3. **draft/review는 버전을 유지한다** — 최대 3회 자동 재작성 루프의 이력이 보존되어
   평가 기준이 어떻게 반영되었는지 추적할 수 있습니다.
