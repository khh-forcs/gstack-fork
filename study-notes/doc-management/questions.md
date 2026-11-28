# 질문 목록

사용자가 한 질문을 순서대로 모은다. 답변 항목은 그 질문의 결론을 기록한 주제 파일로 연결한다. 관리 규칙은 `.claude/rules/question-log.md`에 있다.

| 번호 | 질문 내용 | 답변 |
|---|---|---|
| 1 | gstack 저장소에서 문서를 체계적으로 관리하는 규칙에 관한 내용이 있는지 확인하고, 어떤 상황에서 사용하는지를 목록으로 정리해줘. | [01. gstack에 있는 문서 관리 규칙 전체 개요](01-doc-rules-overview.md) |
| 2 | SKILL.md.tmpl 파일 형식을 사용하는 이유가 뭐지? | [02. SKILL.md.tmpl 템플릿 형식을 쓰는 이유](02-skill-template-reason.md) |
| 3 | 분석, 설계를 거쳐 구현을 하다가 알게된 사실이 분석 또는 설계 단계에서 작성한 문서의 수정이 필요한 경우 어떻게 처리하고 있어? | [03. 구현 중 알게 된 사실로 분석·설계 문서를 고쳐야 할 때](03-implementation-feedback-to-design.md) |
| 4 | 직전에 답한 gstack 쓰는 방법 1~6 번을 사용자가 한 행동에 따라 어디를? 어떻게? 고치는지 보면서 이해하고 싶어. 지금 설명은 구체적이지 않아서 이해할수 없어. | [04. 구현 중 설계가 바뀔 때 무엇을 어디에 어떻게 고치는가](04-design-change-walkthrough.md) |
| 5 | 결정기록을 json 형식을 사용하는 이유가 뭐야? | [05. 결정 기록에 JSON 형식을 쓰는 이유](05-decision-log-format.md) |
| 6 | 사용자 행동에 따라 변경 되는 파일들만 처리 순서대로 보여줘.흐름을 보고 싶어. | [06. 설계 변경이 처리되는 흐름 — 바뀌는 파일 순서](06-change-flow-files-and-skills.md#바뀌는-파일-순서) |
| 7 | 사용자 행동에 따라 호출 하는 스킬을 순서대로 보여줘. | [06. 설계 변경이 처리되는 흐름 — 호출하는 스킬 순서](06-change-flow-files-and-skills.md#호출하는-스킬-순서) |
| 8 | 섹션 제목이 한국어 이면 찾지 못한다는게 사실이야? | [04. 확인: 한국어 제목이면 정말 찾지 못하는가](04-design-change-walkthrough.md#확인-한국어-제목이면-정말-찾지-못하는가) |
| 9 | gstack 은 jsonl 파일을 주로 어떤 목적으로 사용하는지 확인하고, 하나의 jsonl 파일을 동시에 수정하는 경우 각 라인의 순서가 문제가 되는지? 아니면 각 라인의 추가 순서는 중요하지 않은건지? 여러 에이전트가 동시에 읽기,쓰기 하는데 왜 문제가 발생하지 않는다고 했는데 사실인지? | [07. JSONL 기록 — 용도와 동시 쓰기](07-jsonl-stores.md#1-jsonl을-쓰는-곳) |
| 10 | jsonl 을 프로그램이 필드로 계산하는 동작이 어떻게 진행되는지 궁금해. | [07. JSONL 기록 — 필드로 계산하는 과정](07-jsonl-stores.md#3-필드로-계산하는-과정) |
| 11 | decisions.jsonl,learnings.jsonl 파일을 사용하는 목적과 json 을 추가하는 순서와 읽는 순서는 어떻게 되어 있어? | [07. JSONL 기록 — decisions.jsonl과 learnings.jsonl](07-jsonl-stores.md#4-decisionsjsonl과-learningsjsonl) |
| 12 | /ship 단계에서 한꺼번에 정리하는 것이 무엇인지 궁금해. | [06. /ship에서 한꺼번에 정리하는 것](06-change-flow-files-and-skills.md#ship에서-한꺼번에-정리하는-것) |
| 13 | gstack 에서 정규식으로 섹션 제목을 찾는 이유는 무엇인지 궁금해. | [08. 정규식으로 섹션 제목을 찾는 이유](08-regex-vs-judgment.md) |
| 14 | gstack 에서 문서 작성에 사용하는 파일의 종류를 보여줘. | [09. gstack이 문서 작성에 쓰는 파일 종류](09-doc-file-types.md) |
| 15 | md.tmpl 는 왜 사용하는걸까? 어떤 이득이 있지? | [02. 후속: sections/*.md.tmpl까지 포함한 .md.tmpl의 이득](02-skill-template-reason.md#후속-sectionsmdtmpl까지-포함한-mdtmpl의-이득) |
