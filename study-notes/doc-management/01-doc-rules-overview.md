# 01. gstack에 있는 문서 관리 규칙 전체 개요

## 질문

gstack 저장소에 문서를 체계적으로 관리하는 규칙이 있는가? 어떤 상황에서 쓰는가?

## 결론

있다. 중심은 Diataxis라는 문서 분류 방식이고, 이를 실행하는 두 스킬이 있다. `/document-release`는 머지 전에 문서를 점검하고, `/document-generate`는 빠진 문서를 새로 쓴다. `/ship`을 할 때마다 문서 점검이 자동으로 돈다. 여기에 CHANGELOG, TODOS, 생성 문서에 대한 형식 규칙이 붙어 있다.

## 근거

### Diataxis: 읽는 사람의 목적으로 문서를 나눈다

근거: `docs/explanation-diataxis-in-gstack.md`

| 분류 | 읽는 사람이 원하는 것 |
|---|---|
| Reference | 정확한 사양 |
| How-to | 특정 작업을 하는 방법 |
| Tutorial | 처음부터 따라 하며 배우기 |
| Explanation | 왜 이렇게 만들었는지 |

한 문서에 여러 분류를 섞지 않는다. gstack이 이 방식을 고른 이유는, 공통 분류가 있어야 "문서가 충분한가"를 기계적으로 점검할 수 있기 때문이다. 팀마다 자기 형식으로 쓰면 점검 도구를 만들 수 없다.

### 상황별 규칙

| 상황 | 규칙 | 근거 파일 |
|---|---|---|
| 기능 구현 후 머지 전 | `/document-release`가 코드 변경분을 읽고 문서에 빠진 분류를 찾는다. 빠진 문서는 표시만 하고 자동으로 쓰지 않는다. | `document-release/SKILL.md.tmpl` |
| 문서를 점검할 때 | 경로, 개수 같은 사실 수정은 바로 하고, 서술 변경이나 섹션 삭제는 사용자에게 묻는다. | `document-release/sections/release-body.md.tmpl` |
| 문서를 찾을 수 있는지 | 모든 문서는 README나 CLAUDE.md에서 링크로 찾아갈 수 있어야 한다. | 같은 파일 Step 6 |
| `/ship`할 때마다 | 문서 점검을 반드시 실행한다. 고칠 것이 없어도 "점검 완료"로 기록한다. | `ship/SKILL.md.tmpl` Step 14.5 |
| 문서를 새로 쓸 때 | `/document-generate`가 코드를 먼저 끝까지 읽고, Reference → Explanation → How-to → Tutorial 순서로 쓴다. | `document-generate/SKILL.md.tmpl` |
| CHANGELOG를 쓸 때 | 브랜치마다 `changelog.d/<브랜치>.md` 파일 하나를 쓴다. 버전은 머지할 때 붙는다. 여러 사람이 같은 파일을 동시에 고치는 충돌을 피하려는 것이다. | `docs/CHANGELOG_STYLE.md` |
| TODOS를 정리할 때 | 정해진 형식을 따르고, 끝난 항목은 Completed로 옮긴다. | `review/TODOS-format.md` |
| 생성되는 문서를 고칠 때 | 원본(템플릿)만 고치고 다시 생성한다. 생성된 파일은 직접 고치지 않는다. | `CLAUDE.md` |
| CLAUDE.md가 커질 때 | 세부 내용은 `docs/`로 옮기고 CLAUDE.md에는 링크만 남긴다. | `docs/CHANGELOG_STYLE.md` 첫 문단 |
| 설계 문서를 남길 때 | 공유할 설계 문서는 `docs/designs/`, 개인 장기 계획은 `~/.gstack-dev/plans/`(커밋 안 함)에 둔다. | `CLAUDE.md` |

## 사용자 상황에 적용할 때

- gstack의 자동 점검은 문서가 코드와 같은 저장소에 있다는 전제 위에 있다. 사용자의 문서는 코드 저장소 밖에 있으므로 그대로 적용되지 않는다. (결정 현황 D1과 관련)
- "개인 계획 → 공유 문서"로 위치를 나누는 방식은 사용자의 "개인 관리 → 완료 후 공유" 흐름과 닮았다. (D3과 관련)
