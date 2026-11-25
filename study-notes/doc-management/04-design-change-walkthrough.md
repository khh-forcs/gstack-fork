# 04. 구현 중 설계가 바뀔 때 gstack이 무엇을 어디에 어떻게 고치는가 (예시로 따라가기)

## 질문

03번에서 설명한 gstack의 방법 1~6을, 사용자가 한 행동에 따라 어디를 어떻게 고치는지 구체적으로 보고 싶다.

## 결론

gstack은 계획 문서를 고치는 대신 **다른 곳에 기록을 남긴다.** 사용자가 구현 중에 계획과 다르게 행동하면, 기록은 네 곳에 나뉘어 남는다.

| 사용자 행동 | 적용되는 방법 | 기록되는 곳 | 기록하는 주체 |
|---|---|---|---|
| 구현 방식을 바꿈 | 1. 항목별 대조 (CHANGED) | PR 설명의 `## Plan Completion` | `/ship`, `/review` |
| | 5. 결정 대체 표시 | `~/.gstack/projects/<프로젝트>/decisions.jsonl` | Claude가 `gstack-decision-log` 실행 |
| | 4. 설계 문서 변경 이력 | `docs/designs/<설계>.md`에 변경 섹션 추가 | 사람 또는 Claude가 직접 편집 (관례) |
| 일부를 구현하지 못함 | 1. 항목별 대조 (NOT DONE) | PR 설명 | `/ship` |
| | 2. 사용자 선택 후 미룸 | `TODOS.md`에 P1 항목 추가 | `/ship` |
| 완료 기준을 고침 | 3. ACCEPTANCE_EDITED | PR 설명에 원래 문구와 고친 문구 | `/ship` |
| (위 차이 전체) | 6. 교훈 기록 | `~/.gstack/projects/<프로젝트>/learnings.jsonl` | `/review` |

`docs/designs/`와 `TODOS.md`는 저장소 안에 있다. `decisions.jsonl`과 `learnings.jsonl`은 저장소 밖 개인 공간에 있다. PR 설명은 GitHub에 있다.

## 예시 상황

회원가입 기능을 만든다. 브랜치는 `feature/signup`이다. 설계 문서 `docs/designs/SIGNUP.md`에는 이렇게 적혀 있다.

```markdown
# 회원가입
Status: Approved for implementation (2026-11-01)

## 구현 항목
1. 인증 메일을 Redis 큐로 발송한다.
2. 가입이 끝나면 환영 메일을 보낸다.

## Acceptance criteria
- 가입 요청은 200ms 안에 응답한다.
```

`/ship`은 이 브랜치에서 바뀐 `docs/designs/*.md` 파일이나 브랜치 이름이 적힌 계획 파일을 찾는다. 찾은 파일을 이 브랜치의 계획 문서로 쓴다.

## 행동 1. 구현 중에 Redis를 쓸 수 없다는 것을 알고 DB 테이블 큐로 바꿨다

**설계 문서 본문은 고치지 않는다.** 1번 항목 "Redis 큐로 발송한다"는 그대로 둔다.

**결정 기록에 두 줄이 추가된다.** Claude는 결정이 뒤집힐 때 `gstack-decision-log`를 실행한다. 파일은 덧붙이기만 한다.

```jsonl
{"id":"a1","kind":"decide","decision":"인증 메일은 Redis 큐로 보낸다","rationale":"재시도와 지연 발송이 쉽다","scope":"repo","date":"2026-11-01T...","source":"user"}
{"id":"b2","kind":"supersede","supersedes":"a1","scope":"repo","date":"2026-11-10T...","source":"agent"}
{"id":"c3","kind":"decide","decision":"인증 메일은 DB 테이블 큐로 보낸다","rationale":"운영 환경에 Redis가 없다","alternatives_considered":"Redis 도입","scope":"repo","date":"2026-11-10T...","source":"user"}
```

첫 줄은 처음 결정이다. 둘째 줄은 "a1은 대체됐다"는 표시다. 셋째 줄이 새 결정이다. 첫 줄은 지워지지 않는다. 나중에 `gstack-decision-search`로 찾으면 현재 유효한 결정만 보이고, 정리(`--compact`)하면 대체된 결정은 `decisions.archive.jsonl`로 옮겨져 계속 남는다.

**설계 문서에 변경 이력을 덧붙일 수 있다.** 이 부분은 도구가 하지 않는다. gstack의 설계 문서들에서 보이는 관례다.

```markdown
## Revisions
**Revision 2 (2026-11-10).** 인증 메일 큐를 Redis에서 DB 테이블로 바꿨다. 운영 환경에 Redis가 없다.
```

**`/ship` 때 PR 설명에 CHANGED로 남는다.** 같은 목표(인증 메일 발송)를 다른 방법으로 달성했으므로 통과로 본다.

```
## Plan Completion
[CHANGED] "인증 메일을 Redis 큐로 발송" → DB 테이블 큐로 구현 (src/mail/queue.ts)
```

## 행동 2. 시간이 부족해서 환영 메일을 구현하지 못했다

**`/ship`이 NOT DONE으로 분류하고 사용자에게 묻는다.** 선택지는 지금 구현하기, 미루고 출시하기 등이다.

**미루기를 고르면 `TODOS.md`에 P1 항목이 생긴다.**

```markdown
## Signup
### 환영 메일 발송
**What:** 가입 완료 시 환영 메일을 보낸다.
**Why:** 설계에 있었지만 이번 출시에서 빠졌다.
**Context:** Deferred from plan: docs/designs/SIGNUP.md
**Effort:** S
**Priority:** P1
```

PR 설명의 `## Plan Completion`에도 미룬 항목으로 적힌다. 설계 문서의 2번 항목은 여전히 그대로다.

## 행동 3. 200ms를 맞추지 못해서 설계 문서의 완료 기준을 500ms로 고쳤다

**`/ship`이 기준선과 비교한다.** 기준선은 기본 브랜치(main)에 있는 `SIGNUP.md`다. `## Acceptance criteria` 섹션 내용이 다르면 `ACCEPTANCE_EDITED`로 표시한다.

**판정은 원래 문구(200ms)로 한다.** 500ms만 만족하면 PARTIAL이나 NOT DONE이 되고, 행동 2처럼 사용자에게 묻는다.

**PR 설명에 두 문구가 함께 남는다.**

```
[NOT DONE][ACCEPTANCE_EDITED] 가입 응답 시간
  기준선: 가입 요청은 200ms 안에 응답한다.
  작업 중: 가입 요청은 500ms 안에 응답한다.
```

주의할 조건이 두 가지 있다.
- 섹션 제목이 영어 키워드(`Acceptance`, `Success criteria`, `Verification`, `Test plan`, `Testing`, `Done when`, `Definition of done`)로 시작해야 검사한다. `## 완료 기준`이라고 쓰면 이 검사가 찾지 못한다.
- 계획 문서가 git 저장소 밖에 있으면 기준선이 없다. 이때는 `baseline unavailable`로만 기록하고 비교하지 못한다.

## 행동 4. 위 차이들이 다음 작업에 남는 방식

**`/review`가 계획 파일에서 나온 차이를 교훈으로 기록한다.**

```jsonl
{"skill":"review","type":"pitfall","key":"plan-delivery-gap-redis-queue","insight":"Redis 큐를 계획했지만 운영 환경에 Redis가 없어서 DB 테이블 큐를 만들었다","confidence":8,"source":"observed","files":["docs/designs/SIGNUP.md"]}
```

다음에 다른 기능을 계획할 때 Claude는 이 파일을 읽는다. 관련 있으면 "Prior learning applied: plan-delivery-gap-redis-queue"라고 밝히고 계획에 반영한다.

## 사용자 상황에 적용할 때

- gstack에서 "어디에 기록할지"를 정하는 기준은 **기록의 성격**이다. 처음 의도는 설계 문서에, 이번 작업의 차이는 PR에, 뒤집힌 결정은 결정 기록에, 남은 일은 할 일 목록에, 다음을 위한 교훈은 교훈 기록에 둔다.
- 행동 3의 검사처럼 원문과 비교하는 장치는 문서가 git에 있어야 작동한다. 사용자의 분석·설계 문서는 git 밖에 있으므로, 원문을 보존하는 방법을 따로 정해야 한다. (결정 현황 D1, D7)
- 사용자에게는 PR 대신 "개발 완료 후 작성하는 docx, 위키"가 있다. 행동 1~3에서 PR 설명에 남는 내용(계획 대비 결과)이 최종 문서의 재료가 될 수 있다. (결정 현황 D2)
