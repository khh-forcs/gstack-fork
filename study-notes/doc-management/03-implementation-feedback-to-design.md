# 03. 구현 중 알게 된 사실로 분석·설계 문서를 고쳐야 할 때

## 질문

분석과 설계를 거쳐 구현하다가 새로 알게 된 사실 때문에 앞 단계 문서를 고쳐야 하면, gstack은 어떻게 처리하는가?

## 결론

gstack은 문서를 두 종류로 나눠 다르게 처리한다.

- **현재 상태를 설명하는 문서**(README, ARCHITECTURE, 사용법 문서)는 실제 구현에 맞게 고친다.
- **그때의 의도와 약속을 기록한 문서**(계획, 설계 문서, 결정 기록)는 원문을 덮어쓰지 않는다. 원문은 기준선으로 남긴다. 실제와 달라진 점은 "무엇이 어떻게 달라졌고 왜 그랬는지"로 따로 기록한다.

계획 문서의 완료 기준을 구현 결과에 맞춰 고치는 일은 의심스러운 행동으로 보고 검사한다. 이렇게 하면 "목표를 바꿔서 완료한 것처럼 보이게 만드는 일"을 막을 수 있다.

## 근거

### 1. 계획과 실제 구현을 항목별로 대조한다

근거: `review/sections/plan-completion.md`, `ship/sections/plan-completion.md.tmpl`

`/review`와 `/ship`은 계획 문서의 항목을 실제 코드 변경과 하나씩 대조한다. 각 항목은 다섯 가지로 분류된다.

| 분류 | 뜻 |
|---|---|
| DONE | 계획대로 구현됐다. 근거 파일을 댄다. |
| PARTIAL | 일부만 구현됐다. |
| NOT DONE | 구현되지 않았다. |
| CHANGED | 다른 방법으로 구현했지만 같은 목표를 달성했다. 차이를 적는다. 예: "Redis 큐" → Sidekiq로 구현 |
| UNVERIFIABLE | 코드만으로는 확인할 수 없다. 사용자가 직접 확인할 항목을 적는다. |

결과는 PR 본문의 `## Plan Completion` 섹션에 남는다. 계획 문서 자체를 고치지 않고, 차이를 별도 기록으로 남기는 방식이다.

### 2. 빠진 항목은 사용자가 처리 방법을 정한다

NOT DONE 항목이 있으면 사용자에게 묻는다. 선택지는 지금 구현하기, 후속 작업으로 미루기, 의도적으로 뺀 것으로 기록하기다. 미루면 TODOS에 "Deferred from plan: <계획 파일 경로>"를 붙인 P1 항목을 만든다. 말없이 빠지는 항목이 없게 하려는 장치다.

### 3. 완료 기준을 고치면 별도로 표시한다

근거: `scripts/resolvers/plan-gates.ts`의 `ACCEPTANCE_EDITED`, `docs/designs/HONEST_WORK_GATE_INTEGRITY.md`의 RH-10

계획 문서에서 "완료 기준", "검증", "테스트 계획" 같은 섹션이 바뀌면 `ACCEPTANCE_EDITED`로 표시한다. 기준선은 기본 브랜치에 있는 계획 문서이고, 없으면 브랜치에 처음 들어온 버전이다. 이 경우에도 항목은 **원래 문구**를 기준으로 판정한다. 고친 문구만 만족하면 PARTIAL이나 NOT DONE이 된다. PR 본문에는 원래 문구와 바뀐 문구를 함께 적어 범위 변경으로 드러낸다. gstack은 이 문제를 "spec-editing as progress"(명세를 고치는 것으로 진척을 만드는 일)라고 부른다.

### 4. 설계 문서는 상태와 변경 이력을 남긴다

근거: `docs/designs/` 아래 문서들. 강제 규칙이 아니라 여러 문서에서 공통으로 보이는 관례다.

- 문서 첫머리에 상태를 적는다. 예: `Approved for implementation`, `ACTIVE`, `TABLED`(보류), `LIVING DOCUMENT — update as bugs are found and fixed`.
- 설계가 바뀐 과정을 지우지 않는다. `PLAN_TUNING_V1.md`는 "Revision 1 (rejected)", "Revision 2 (accepted)"처럼 버린 안과 그 이유를 함께 남긴다.
- 구현된 내용을 따로 적는다. `SELF_LEARNING_V0.md`는 릴리스마다 "What shipped"를 적는다.
- 보류한 설계는 다시 시작할 때 할 일을 적어 둔다. `GCOMPACTION.md`의 "If un-tabling" 섹션이 그 예다.

### 5. 결정은 덮어쓰지 않고 대체 표시를 한다

근거: `bin/gstack-decision-log`, `CLAUDE.md`의 "Cross-session decision memory"

결정 기록은 덧붙이기만 하는 파일(`decisions.jsonl`)이다. 예전 결정을 뒤집을 때는 지우지 않고 `--supersede <id>`로 "대체됨"을 표시한다. 그래서 예전 결정과 그 이유를 나중에도 볼 수 있다.

### 6. 계획과 실제의 차이를 다음 작업의 교훈으로 남긴다

근거: `review/sections/plan-completion.md`의 "Learnings Logging", `scripts/resolvers/learnings.ts`

계획 문서에서 나온 차이는 "X를 계획했지만 Z 때문에 Y를 만들었다" 형식으로 학습 기록(`learnings.jsonl`)에 남긴다. 다음 세션이 같은 문제를 미리 알 수 있게 하려는 것이다. 커밋 메시지나 TODOS에서 추정한 차이는 신뢰도가 낮아서 기록하지 않는다.

### 7. 현재 상태 문서는 사실이면 고치고, 판단이면 묻는다

근거: `document-release/sections/release-body.md.tmpl`

README나 ARCHITECTURE가 구현과 다르면 `/document-release`가 고친다. 경로, 개수 같은 사실은 바로 고친다. 설계 근거나 철학처럼 판단이 필요한 내용은 사용자에게 묻는다.

## 사용자 상황에 적용할 때

- 사용자의 분석·설계 문서는 git 밖에 있다. 원문을 덮어쓰면 이전 내용과 바꾼 이유가 사라진다. 그래서 gstack보다 "원문 보존 + 변경 기록"이 더 중요하다. 이 판단은 결정 현황 D1(개발 중 문서 위치)과 이어진다. 문서를 git에 두면 이전 내용은 이력으로 남으므로, 변경 이유만 기록하면 된다.
- 최종 문서(docx, 위키)는 "현재 상태를 설명하는 문서"에 해당한다. 최종 결과에 맞춰 쓰면 된다.
- 결정 현황 D7(구현 중 발견을 앞 단계 문서에 반영하는 방식)이 새로 생겼다.
