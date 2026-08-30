# prototype-agent

웹사이트·웹서비스 구축 프로젝트의 **제품 산출물과 Story 백로그**를 관리하는 Claude 스킬입니다.

회의록·프로토타입(JSX/HTML·캡처)·기능목록·엑셀 같은 요구사항 소스를 넣으면 PRD 로 정리하고,
Story 를 발번·중복 제거하고, Epic/IA 에 매핑합니다.
**산출물은 전부 레포 안의 마크다운 파일**이라 외부 이슈 트래커 없이 완결됩니다.

> 의료정보시스템(EMR) 용 스킬은 **이 레포와 무관한 별도 스킬**입니다. 여기 없고, 앞으로도 들어오지 않습니다.

## 설치

```bash
git clone git@github.com:StudyClub-PlusPlus/prototype-agent.git ~/Developer/prototype-agent
ln -s ~/Developer/prototype-agent ~/.claude/skills/prototype-agent
```

클론 위치는 자유입니다. 심링크만 `~/.claude/skills/<스킬이름>` 으로 걸면 Claude Code 가 인식합니다.
(복사해 넣어도 되지만, 심링크로 걸어야 `git pull` 로 갱신을 받습니다.)

설치 확인 — Claude Code 를 새로 열고:

```
/prototype-agent
```

## 쓰는 법

스킬 이름을 부르지 않아도 됩니다. **요구사항 자료를 올리거나 기획 작업을 말하면 알아서 붙습니다.**

```
회의록 정리해서 PRD 로 만들어줘
이 프로토타입 캡처 보고 Story 뽑아줘
스토리 중복 좀 정리해줘
```

명시적으로 부르려면 `/prototype-agent` 를 쓰면 됩니다.

## 구조

```
SKILL.md                    진입점 — 원칙과 작업 흐름
references/                 세부 지침
  _SHARED_CONVENTIONS.md      ID 체계·파일 배치 등 공통 규약
  prd-guide.md                PRD 작성법 (번호별 명세 규칙 포함)
  ingest-reconcile.md         입력 자료 판정과 기존 백로그와의 대조
  change-plan.md              변경 계획 승인 흐름
  review-loop.md              검토 루프
  completion-loop.md          완료 판정
  git-sync.md                 여러 사람이 같이 쓸 때의 병합 규칙
  baseline-requirements.md    기준 요구사항
  session-memory.md           세션 간 맥락 유지
  system-instructions.md      항상 적용되는 지침
templates/                  새 프로젝트 시작용 뼈대
  01-planning/                _project.md · _registry/ · research/
  CLAUDE.md                   프로젝트 레포에 둘 에이전트 컨텍스트
_archive/                   더 이상 쓰지 않는 지침 (참고용)
```

## 고칠 때

스킬은 **여러 사람이 같은 규약을 공유하는 것**이 값어치입니다.
각자 로컬에서 고쳐 쓰면 "누구 스킬이 맞냐"가 생기니, 바꿀 게 있으면 PR 로 올려주세요.

특히 `_SHARED_CONVENTIONS.md` 의 ID 체계와 `prd-guide.md` 의 번호 명세 규칙은
산출물끼리 서로를 참조하는 근거라, 혼자 바꾸면 남의 문서가 깨집니다.
