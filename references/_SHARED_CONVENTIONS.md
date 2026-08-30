# 공통 규칙 (Shared Conventions)

| 항목 | 내용 |
|---|---|
| 문서 | prototype-agent 스킬 공통 규칙 |
| 위치 | `~/.claude/skills/prototype-agent/references/_SHARED_CONVENTIONS.md` |
| 적용 대상 | Ingest·PRD 가이드 및 오케스트레이터(SKILL.md) 전부 |
| 우선순위 | ID·위치·네이밍·버전·톤 충돌 시 본 문서가 최종 기준 |

---

## 데이터 모델 (단일 기준 정의)

핵심 엔티티 4종 (Feature · Epic · Story · Function) + 보조 2종 (Source · IA Node) + 영속 명세 1종 (FS).

```
        요구(문제) 측                                  구현(해결) 측

Source ──(N:N)──▶ Feature ──(1:1)── PRD
                     │
                     │ (N:N)                            (N:1)
                     ▼                                    ▲
                   Epic ──(1:N)── Story ◀──(N:N)──▶ Function ── FS (1:N)
                                    │   Blocks / Blocked by
                                    │ (N:N)
                                    ▼
                                 IA Node
```

| 엔티티 | 정체 | 역할 | 입도 | SoT |
|---|---|---|---|---|
| Source | 입력 자료 | 회의록·프로토타입·기능목록 적재 단위 | 자료 1건 | `INDEX.md` (`SRC-####`) |
| Feature | 기획 단위 | 계획된 변경의 묶음, Feature PRD 보유 | 기능/제품 | `features/{f-slug}/PRD.md` |
| Feature PRD | Feature 집약 본문 | Story PRD 들을 링크 집약 + Functions·임팩트·KPI | Feature 1건 | `features/{f-slug}/PRD.md` |
| Story PRD | 요구 1건 본문 | Story 1건의 본문(기본설명·화면명세·기타정책) | Story 1건 | `stories/{story-slug}/PRD.md` |
| Epic | 전달 단위 | Story 의 전달 배치 묶음 | 전달 배치 | `epics/{e-slug}/Epic.md` (`EP-###`) |
| Story | 요구 원자 | 사용자 관점 요구 1단위 | 원자 | `stories.md` (`ST-###`) + `stories/{slug}/PRD.md` |
| FS | 기능정의서 | 기능 도메인 단위 명세, Function 묶음 | 도메인 | `fs/{fs-slug}/FS.md` |
| Function | 구현 요건 | 시스템 구현 요건 1단위 — 동작 워크플로우·인터페이스/데이터 계약·완료 정의·Blocks Story | 원자 | `fs/{fs-slug}/FS.md` 본문 항목 (`FN-###`) |
| IA Node | 정보 구조 | 페이지·섹션 단위 | 화면 요소 | `ia.md` (`IA-###`) |

**핵심 불변식**:
- 작업 추적(Epic·Story) = **일시적**, 구현 명세(FS·Function) = **영속**
- Story 는 다중 Feature 등장에도 단일 ID·단일 Story PRD 유지
- 모든 Story 는 정확히 1개 Epic 을 부모로 가짐 (Orphan Zero)
- **PRD 2-tier**: Story PRD(요구 1건 본문) + Feature PRD(Story PRD 를 **링크 참조**·복사 X)
- Story 본문 SoT = Story PRD. `stories.md` 는 요구문장·상태·매핑 레지스트리 SoT (본문 미보유)
- Story ↔ Function 의존성은 Blocks/Blocked by 양방향 동기

---

## 1. 저장소 및 운용

- **에이전트 레포(공용)** — 본 스킬 자체. 프로젝트 데이터 미포함
- **프로젝트 레포(프로젝트별)** — 실제 산출물·레지스트리·소스의 SoT. `01-planning/` 하위 일체
- 모든 결과물은 레포 내 파일로 존재. 채팅 임시 산출물을 최종본으로 간주하지 말 것
- **외부 이슈 트래커 비의존** — ID 발급·상태 관리 전부 레포 내에서 완결

> **공개 레포 주의**: 프로젝트 레포가 public 이면 `01-planning/` 전체가 공개된다. 내부 전용 정보(내부 URL·고객 실명·계약 조건·미공개 로드맵)를 본문에 넣지 말 것. 부트스트랩 시 레포 공개 여부를 확인하고 `_project.md`에 기록.

---

## 2. 디렉터리 구조

```
<project>/
  01-planning/
    _project.md                    # 프로젝트 메타 (제품 정의·모듈 범위·Actor 매핑·기본브랜치·레포 공개여부)
    _registry/
      stories.md                   # 발번 대장 + Story 정규 레지스트리 (SoT)
      stories-index.md             # Story 정규문장 빠른 목록 (자동 생성·읽기 전용)
      ia.md                        # Information Architecture (IA-### 노드 트리 + Story 호스팅 N:N)
      canonicals.md                # 영역 정본 프로토타입 대장
      policies.md                  # 정책 레지스트리 (POL-####)
      _HISTORY.md                  # 교차 문서 변경 원장
    _memory/
      SESSION-YYYY-MM-DD-NNN.md    # 라운드 1건 = 1 파일
    research/
      _sources/
        INDEX.md                   # 소스 대장
        SRC-####-<유형>-<날짜>.<ext>
    stories/
      {story-slug}/PRD.md          # Story PRD. frontmatter에 story_id·epic·features[]
    features/
      {f-slug}/
        PRD.md                     # Feature PRD
        _source/SRC-####-prototype-<YYYY-MM-DD>/
        _captures/SRC-####-prototype-<YYYY-MM-DD>/
    epics/
      {e-slug}/Epic.md             # frontmatter에 epic_id·title + Story 목록 미러
    fs/
      {fs-slug}/
        FS.md                      # Function 묶음 (영속 명세)
        DFD.md
```

**경로 안정성 원칙**:
- **slug 만 경로 키**. ID(`ST-###`·`EP-###`)는 폴더명에 박지 않음 — 문서 frontmatter 가 단독 담당
- 산출물 파일명 고정. 기능명 변경 시 파일명 unchanged
- **slug 변경**: 변경 계획 승인 필요. 변경 시 `git mv` + 모든 cross-reference 갱신

### 2.1 모듈 14종 (기본값 · canonical)

인증·계정 / 프로필·마이페이지 / 콘텐츠 / 검색·탐색 / 신청·폼 / 결제 / 알림 / 관리자 콘솔 / 통계·대시보드 / 정보구조·내비게이션 / 국제화 / 접근성·SEO / 외부 연동 / 인프라·배포

**모듈 정의 (오해 방지)**:
- **인증·계정** = 로그인·회원가입·세션·토큰·비밀번호·소셜 로그인
- **프로필·마이페이지** = 로그인 후 개인 영역. 내 활동·설정·내 신청 내역
- **콘텐츠** = 서비스가 보여주는 정보 자산(목록·상세·공지·가이드). 편집 도구 포함
- **신청·폼** = 사용자 입력 수집·검증·제출·상태 추적 (신청/문의/설문)
- **관리자 콘솔** = 운영자 전용 백오피스 화면 일체
- **정보구조·내비게이션** = 라우팅·메뉴·빵부스러기·딥링크·URL 설계
- **외부 연동** = OAuth 공급자·결제 PG·메일·푸시·분석·webhook 등 시스템 간 연결(SYS-02)
- **인프라·배포** = CI/CD·환경변수·도메인·모니터링

> `_project.md`의 모듈 범위로 활성 모듈을 좁히거나 신규 모듈 추가 가능 (사용자 결정)

### 2.2 결정 주체

| 항목 | 결정 주체 | 변경 방법 |
|---|---|---|
| 모듈 표준 14종 | 조직 (부트스트랩 기본값) | `_project.md` 모듈 범위 갱신 |
| Feature 신설·slug | **사용자 승인** (변경 계획 게이트) | 변경 계획에 항목 추가. 1 Feature = 1 PRD |
| Epic 신설·slug | **사용자 승인** | 변경 계획. 발행 후 slug 변경 비권장 |
| Story ↔ Feature 매핑 (N:N) | 에이전트 정합성 검증 + **사용자 승인** | 변경 계획. 단일 `ST-###` 유지 + Feature 컬럼 누적 |
| Story ↔ Epic 매핑 (N:1) | **사용자 승인** | 변경 계획 |
| 기능명·모듈 메타 | **사용자 승인** | PRD/FS frontmatter 수정. 폴더 이동 없음 |

---

## 3. 파일 네이밍

- **캐논 파일 1개 원칙**: 문서당 파일 1개. `v1`·`v2`·`YYMMDD` 접미사 금지 — 이력은 git 이 보유
- 산출물 파일명:

  | 산출물 | 위치 | 단위 |
  |---|---|---|
  | Story PRD | `stories/{story-slug}/PRD.md` | Story 1건 |
  | Feature PRD | `features/{f-slug}/PRD.md` | Feature 1건 |
  | Epic 메타 | `epics/{e-slug}/Epic.md` | 전달 배치 |
  | 기능정의서 | `fs/{fs-slug}/FS.md` | 기능 도메인 |
  | DFD | `fs/{fs-slug}/DFD.md` | 도메인 |
  | US 시나리오 | `_scenarios/US-{시나리오명}.md` | 횡단 |
  | 화면 스펙 | **Feature PRD §4.5 "화면 스펙"** (별도 파일 없음) | 화면(IA 노드) |
  | 프로토타입 | `features/{f-slug}/_source/SRC-####-prototype-{날짜}/` | Feature |

- 레지스트리·소스: `stories.md` · `stories-index.md` · `ia.md` · `canonicals.md` · `policies.md` · `_HISTORY.md` · `INDEX.md` · `SRC-{####}-{유형}-{날짜}.{ext}`
- **파일명에 ID 미사용** — slug 가 정체성을 잡음

### 3.1 영역 정본 프로토타입 (Canonical Area Prototype)

- **영역 정본** = 한 기능 영역의 **최신 상태를 통합한 단일 프로토타입**
- **선언 3요소**: ① 폴더명 접미 `-<영역명>` ② 폴더 `README`에 "영역 정본" 명시 ③ **`_registry/canonicals.md`에 등록**. 한 영역 = 정본 1개
- **정본 게이트**: 새 작업이 정본 있는 영역에 속하면 **"정본에 반영할지" 1회 질문**. 정본 없는 영역은 비적용

---

## 4. ID 체계

**본 스킬은 외부 트래커를 쓰지 않으므로, ID 는 레포 내 발번 대장이 단독 발급한다.**

| 항목 | SoT | 형식 | 부여 시점 |
|---|---|---|---|
| **Story** | `stories.md` | **`ST-###`** | 발번 대장에서 순차 발급 |
| **Epic** | `epics/{e-slug}/Epic.md` frontmatter | **`EP-###`** | 발번 대장 |
| **Function** | `fs/{fs-slug}/FS.md` 본문 항목 | **`FN-###`** | 발번 대장 |
| **IA Node** | `ia.md` | **`IA-###`** | 발번 대장 |
| **Source** | `INDEX.md` | **`SRC-####`** | 적재 즉시, 발번 대장 |
| **Policy** | `policies.md` | **`POL-####`** | 발번 대장 |
| **Feature** | `features/{f-slug}/PRD.md` | kebab-case slug (번호 없음) | 변경 계획 승인 시 |
| DR / AC | Story PRD 내부 | `#1, #2` 카운터 | 즉시 (Story local) |

### 4.1 발번 대장 (Number Ledger)

`_registry/stories.md` **최상단**에 아래 블록을 두고, 모든 신규 ID 는 여기서만 발급한다.

```
<!-- NUMBER LEDGER — 모든 ID의 단일 발급처. 수동 편집 금지(에이전트가 갱신). -->
| prefix | next |
|---|---|
| ST | 1 |
| EP | 1 |
| FN | 1 |
| IA | 1 |
| SRC | 1 |
| POL | 1 |
```

- **발급 절차**: `next` 값을 읽어 ID 를 만들고, 같은 커밋에서 `next`를 +1 한다
- **단조 증가**: 폐기·삭제된 번호를 회수하지 않는다. 재사용 금지
- **동시 작업 충돌**: 두 작업자가 같은 번호를 발급했으면 **나중에 rebase 하는 쪽이 자기 번호를 재발급**하고 자기 변경분의 참조를 일괄 갱신. 먼저 push 된 쪽은 건드리지 않는다
- **추측 금지**: 대장을 읽지 않고 `ST-999` 같은 번호를 만들어내지 않는다

### 4.2 상태 (Status)

`stories.md` 상태 컬럼의 표준값:

`백로그` → `기획대기` → `기획중` → `기획완료` → `개발중` → `개발완료` → `QA중` → `배포완료`
보조 상태: `보류` · `재오픈` · `폐기`

- **종결 상태는 `배포완료` 하나**. 이 상태 Story 의 내용 변경은 **in-place 금지** — 신규 `ST-###` 등록 후 선행 링크로 승계
- 그 외 활성 상태(재오픈 포함)는 in-place 업데이트
- **`폐기`는 삭제가 아니다** — 행을 남기고 상태만 바꾸며 비고에 사유 기록

### 4.3 절대 금지

- 발번 대장을 거치지 않은 임의 ID 부여
- 존재하지 않는 번호 추측·생성
- Story 행 삭제 (상태 `폐기`로 대체)

---

## 5. 버전·이력 관리

- **git commit / PR 이 버전 이력**. 별도 파일 버전 체계 미사용
- **본문 클린 최종본 원칙**: 문서 본문에 변경 메타코멘트("vX에서 추가", "(정정)") 금지. 항상 완성본 형태로 출력
- 변경 설명은 **PR 본문 및 채팅**에서 수행
- **`_HISTORY.md` = 교차 문서 변경 원장**:

  | 일시 | 대상(문서·ID) | 변경유형 | 트리거 | 요약 | 영향 하위 + 상태 |
  |---|---|---|---|---|---|

  - 변경유형: 신규 / 신규(승계) / 병합 / 갱신 / 폐기
  - 트리거: 소스 `SRC-####` 또는 명령
  - 영향 하위: 영향받는 Story PRD·Feature PRD + `☐ 필요 / ☑ 완료`

---

## 6. Cascade 정책 (상향 전파)

- **정상 운영은 상향 전파**: Story 변경 → 영향 Feature PRD 식별 후 플래그
- **2단 게이트**: ① 자가 검증 → ② **독립 전문가 검증 루프**(3렌즈 전원 합격) → ③ **PR 게이트**(사람 승인·머지)
- 영향·전파 내역과 **검증 결과(L1/L2/L3·라운드 수)** 는 `_HISTORY.md` 및 PR 본문에 명시

---

## 7. 소스·출처 원칙

- **소스 불변**: 업로드된 모든 자료는 원본 그대로 보관. 소스 단위 중복 제거 안 함. `SRC-####` 부여 후 `INDEX.md` 등록
- **출처 누적(append-only)**: Story 의 출처는 개수 제한 없이 누적. 기존 출처를 덮어쓰지 않음
- **출처 빈도 = 신호**: 다수 소스에서 반복 등장한 Story 는 우선순위(P0) 가중 근거
- **코드베이스도 소스**: 구동 중인 제품(PUD)의 코드는 `SRC-####-pud-<날짜>` 로 등재하고, 조사 근거를 파일 경로·줄 번호로 남긴다

---

## 8. 엔티티 관계

**요구 측**: Source → Feature(PRD) → Epic ⊃ Story
**구현 측**: FS ⊃ Function
양측은 **Story ↔ Function (N:N)** 브리지로 연결.

- **Source ↔ Feature/Story**: N:N
- **Feature → PRD**: 1:1
- **Feature ↔ Story**: N:N — 동일 의미는 **단일 ID 로 통합** + Feature 컬럼에 slug 누적
- **Feature ↔ Prototype**: 1:N — 메인 1 + Sub N
- **Epic ⊃ Story**: 1:N — 모든 Story 는 단일 Epic 소속 (Orphan Zero)
- **FS ⊃ Function**: 1:N
- **Story ↔ Function (Blocks/Blocked by)**: N:N — 핵심 브리지. 양방향 동기 필수
- **Story ↔ IA Node**: N:N
- 파생: Epic ↔ FS = N:N, Feature ↔ Epic = N:N, Feature ↔ FS = N:N

- **Story 정합성 통일 의무**: 같은 의미 Story 재등장 시 **신규 발번 금지, 기존 `ST-###`으로 통합**
- **Function 재사용 원칙**: 동일 시스템 기능에 여러 Story 의존이면 단일 Function 이 다수 Story 를 Blocks

---

## 9. 작성 톤·언어

- 한국어 작성. ID·기술 용어·영문 고유명사는 영문 유지
- 본문은 **명사형 종결**
- 클린 최종본(§5)
- Markdown 구조(표·코드블록·체크박스) 유지

---

## 10. 표준 Actor

웹 서비스 기본 8종 + 시스템 2종. 프로젝트 고유 역할명이 있으면 `_project.md §3`에 매핑표를 두고 **그 이름을 표준명으로 사용**한다.

| ID | 표준명 | 정의 |
|---|---|---|
| ACT-01 | 방문자 | 비로그인 사용자. 공개 콘텐츠 열람 |
| ACT-02 | 회원 | 로그인 사용자. 개인화 기능 이용 |
| ACT-03 | 신청자 | 특정 신청·참여 절차를 밟는 회원 |
| ACT-04 | 콘텐츠 편집자 | 콘텐츠 등록·수정 권한 보유자 |
| ACT-05 | 운영자 | 일상 운영·승인·관리 담당 |
| ACT-06 | 관리자 | 권한·설정·시스템 구성 담당 |
| ACT-07 | 개발자 | 배포·연동·운영 도구 사용자 |
| ACT-08 | 비회원 구매자 | 로그인 없이 거래하는 사용자 (해당 시) |
| SYS-01 | 배치/스케줄러 | 시간 기반 자동 처리 |
| SYS-02 | 외부 연동 | OAuth 공급자·결제 PG·메일·푸시·분석·webhook |

- **SYS-01·SYS-02 는 사용자 스토리 주체가 아니다.** 시스템 동작은 Function/FS 에서 정의
- Story·US·FS 전부 동일 Actor 집합 사용
- **프로젝트 치환 예**: `ACT-03 신청자 → 멤버`, `ACT-05 운영자 → 캡틴`, `ACT-04 콘텐츠 편집자 → 네비게이터`. 치환 시 원래 표준명을 병기하지 않고 **치환명만** 본문에 사용

---

## 11. 정량·인용 규칙

- 출처 없는 정량 수치 임의 생성 금지. 추정 시 산출 근거·가정 명시
- 페르소나 가공 인물(이름·나이) 임의 생성 금지. 입력에 명시된 경우만 사용
- 코드 실사 수치(파일 수·호출 지점 수 등)는 **실제 조사 명령과 결과**를 근거로 기재

---

## 12. 모드·트리거 요약

| 입력 / 명령 | 모드 | 주 산출 |
|---|---|---|
| 소스 업로드 | Ingest / Reconcile | `stories.md`·`INDEX.md` 갱신 |
| 프로토타입 + PRD 요청 | PRD Create / Update | `stories/{slug}/PRD.md` + `features/{f-slug}/PRD.md` |
| 기획서 작성 (포괄) | 전체 체인 | 위 전부 |

- 기존 산출물을 변경/추가하는 모드는 실행 전 **변경 계획 승인 게이트**(`change-plan.md`) 통과
