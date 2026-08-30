# 프로젝트 컨텍스트 — 기획 산출물

이 레포의 `01-planning/` 은 **prototype-agent 스킬**이 관리하는 제품 기획 산출물이다.

## 자동 발동 트리거

아래 요청이 들어오면 `prototype-agent` 스킬을 사용한다 (사용자가 스킬을 명시하지 않아도):

- 요구사항 자료 투입 (회의록·프로토타입·기능목록·엑셀·캡처)
- "PRD 작성/수정", "기획서", "기능정의서", "FS", "사용자 시나리오", "US"
- "스토리 정리", "백로그", "중복 제거", "Epic 매핑", "IA"

## 산출물 위치

```
01-planning/
  _project.md           프로젝트 메타
  _registry/            stories.md(발번 대장 포함) · ia.md · _HISTORY.md 등
  research/_sources/    원본 자료 (SRC-####)
  stories/{slug}/PRD.md Story 본문
  features/{slug}/PRD.md
  epics/{slug}/Epic.md
  fs/{slug}/FS.md
  _scenarios/US-*.md
```

## 불변 규칙 (요약)

1. Story 마스터는 `_registry/stories.md` 하나. 다른 문서는 투영이다
2. 모든 ID 는 `stories.md` 상단 **발번 대장**에서만 발급한다. 번호를 추측하지 않는다
3. Story 행을 삭제하지 않는다. 상태를 `폐기`로 바꾼다
4. 기존 산출물 변경은 **변경 계획 승인** 후 착수
5. 산출물 확정 전 **독립 검증 루프 3렌즈** 통과 필수

상세는 스킬의 `references/_SHARED_CONVENTIONS.md`.
