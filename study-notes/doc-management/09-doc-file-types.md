# 09. gstack이 문서 작성에 쓰는 파일 종류

## 질문

gstack에서 문서 작성에 사용하는 파일의 종류는 무엇인가?

## 결론

gstack의 문서는 모두 Markdown(`.md`)으로 쓴다. 다른 파일 종류는 Markdown 옆에서 역할을 나눠 맡는다. 역할은 다섯 가지다.

1. 사람이 읽는 문서: `.md`
2. 문서를 만들어 내는 원본: `.md.tmpl`, 그리고 일부 `.ts`
3. 프로그램이 읽는 기록: `.jsonl`, `.json`
4. 그림의 원본과 결과물: `.mmd`, `.excalidraw`, `.svg`, `.png`
5. 밖으로 내보내는 결과물: `.pdf`, `.html`, `.docx`

원칙은 하나다. 고치는 파일은 텍스트 원본 하나로 두고, 나머지는 그 원본에서 만들어 낸다. PDF, Word, 그림 이미지는 직접 고치지 않는다.

## 근거

### 1. 사람이 읽는 문서: `.md`

저장소에 커밋된 `.md` 파일은 603개다(`git ls-files` 기준, 테스트 픽스처 포함). 위치에 따라 용도가 다르다.

| 위치 | 무엇이 있나 |
|---|---|
| 저장소 최상위 | `README.md`, `CLAUDE.md`, `AGENTS.md`, `ARCHITECTURE.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `TODOS.md`, `DESIGN.md`, `ETHOS.md` 등 14개 |
| `docs/` | 사용 안내와 내부 설명. 파일 이름 앞에 `howto-`, `tutorial-`, `explanation-`, `reference-`를 붙인 것이 있다. 01번에서 본 Diataxis 분류를 이름에 드러낸 것이다. |
| `docs/designs/` | 팀이 공유하는 설계 문서(38개). 예: `DESIGN_TOOLS_V1.md`, `PLAN_TUNING_V1.md` |
| `changelog.d/` | 브랜치마다 하나씩 쓰는 CHANGELOG 조각. 머지할 때 `CHANGELOG.md`로 합쳐진다. |
| `review/` 등 스킬 폴더 | 스킬이 읽는 점검표와 형식 규칙. 예: `review/TODOS-format.md` |

스킬이 작업 중에 만드는 문서도 `.md`다. 이 문서는 저장소가 아니라 개인 상태 폴더 `~/.gstack/projects/<프로젝트>/`에 쌓이고 커밋되지 않는다.

| 파일 | 만드는 스킬 |
|---|---|
| `*-design-*.md` | `/office-hours`의 설계 문서. 팀과 공유할 때는 `docs/designs/{주제}.md`로 저장소에도 쓴다(`office-hours/sections/design-and-handoff.md.tmpl`). |
| `ceo-plans/*.md` | `/plan-ceo-review`의 계획. 공유하기로 하면 `docs/designs/`로 옮긴다(`plan-ceo-review/sections/review-sections.md.tmpl`). |
| `checkpoints/<시각>-<제목>.md` | `/context-save`의 작업 상태 저장(`context-save/SKILL.md.tmpl`) |
| `*-test-plan-*.md`, `test-audit-*.md` | 테스트 계획과 점검 결과 |

### 2. 문서를 만들어 내는 원본: `.md.tmpl`, `.ts`

`SKILL.md`는 직접 쓰지 않는다. `SKILL.md.tmpl`과 `sections/*.md.tmpl`(저장소에 149개)을 원본으로 두고 `bun run gen:skill-docs`가 만들어 낸다. 이유는 02번 기록에 있다.

TypeScript 파일이 문서의 원본인 경우도 있다. `review/design-checklist.md`는 `lib/design-catalog.ts`의 데이터에서 만들어진다(`CLAUDE.md`의 SKILL.md workflow 절). 같은 내용을 여러 스킬이 쓰기 때문에, 데이터를 한곳에 두고 문서는 생성한다.

### 3. 프로그램이 읽는 기록: `.jsonl`, `.json`

사람이 읽는 설명이 아니라 프로그램이 다시 읽어서 계산하는 기록은 JSON 계열로 쓴다.

- `.jsonl`(한 줄에 JSON 하나): 계속 쌓이는 기록이다. `decisions.jsonl`, `learnings.jsonl`, `*-reviews.jsonl`, `timeline.jsonl`, `question-log.jsonl`, `health-history.jsonl`이 있다. 자세한 내용은 05번과 07번 기록에 있다.
- `.json`: 한 시점의 상태나 단계 사이에 넘기는 결과다. 예: `designs/*/approved.json`(승인한 디자인), `taste-profile.json`, `.gstack/tmp/docs-audit.json`(문서 점검 결과), `.gstack/tmp/ship-gate.json`

### 4. 그림: `.mmd`가 원본이다

근거: `docs/howto-diagrams-and-formats.md`

- 문서 안의 그림은 Markdown 안에 ` ```mermaid ` 코드 블록으로 쓴다. `/make-pdf`가 이 블록을 벡터 그림으로 바꾼다. ` ```excalidraw ` 블록도 같은 방식으로 처리한다.
- `/diagram` 스킬은 그림 하나를 세 종류 파일로 만든다.

| 파일 | 용도 |
|---|---|
| `<이름>.mmd` | 원본. 고칠 때는 이 파일을 고치고 다시 그린다. |
| `<이름>.excalidraw` | excalidraw.com에서 열어 손으로 위치를 옮길 때 |
| `<이름>.svg` / `.png` | 문서, 이슈, README, 채팅에 붙일 때 |

이 안내서는 문서에 PNG 대신 `.mmd` 원본을 넣으라고 권한다. 그래야 그림을 계속 고칠 수 있다.

### 5. 밖으로 내보내는 결과물: `.pdf`, `.html`, `.docx`

근거: `docs/howto-diagrams-and-formats.md`의 "Export single-file HTML or Word" 절

`/make-pdf`는 Markdown 하나에서 세 형식을 만든다.

```bash
make-pdf generate doc.md out.pdf
make-pdf generate doc.md out.html --to html
make-pdf generate doc.md out.docx --to docx
```

- `.html`은 그림과 이미지를 모두 파일 안에 넣은 단일 파일이다. 메일에 붙여도 그대로 열린다.
- `.docx`는 내용을 옮기는 데 목적이 있다. 제목, 표, 코드 블록, 목록, 그림(300dpi PNG)은 옮겨지지만, 페이지 배치까지 똑같이 맞추지는 않는다.

디자인 스킬은 결과물로 `.png`(시안 이미지 `variant-*.png`)와 `.html`(확정 화면 `finalized.html`)을 만든다(`design-html/SKILL.md.tmpl`).

## 사용자 상황에 적용할 때

- 개발 중 문서를 Markdown으로 쓰면 Claude가 읽고 고치기 쉽고, git으로 변경 이력을 남길 수 있다. gstack이 모든 문서를 `.md`로 쓰는 이유도 같다.
- 사람이 읽는 문서(`.md`)와 프로그램이 읽는 기록(`.jsonl`)을 나누는 방식은 그대로 가져올 만하다. 결정 기록처럼 나중에 찾아보고 집계할 것은 JSONL, 설명은 Markdown으로 둔다.
- 최종 문서가 docx인 상황(결정 현황 D2)에서는 `make-pdf --to docx`처럼 Markdown에서 변환하는 길이 있다. 다만 gstack도 페이지 배치는 맞추지 않는다고 밝힌다. 회사 양식이 서식을 엄격하게 요구하면 변환 결과를 양식에 옮겨 붙이는 작업이 남는다. 이 점을 확인한 뒤 D2를 정한다.
- 그림은 `.mmd` 같은 텍스트 원본으로 두면, 설계가 바뀔 때 그림도 같이 고칠 수 있다. 이미지 파일만 남기면 고칠 때 처음부터 다시 그려야 한다.
