# 프로토타입 만들기 (playground)

> 기획서를 쓰기 전에 **화면부터 만든다.** 이 스킬의 Hard Gate 가 프로토를 요구하는 이유다 —
> 글로만 쓴 화면은 만들어 보면 거의 다 틀려 있다.
>
> **프로토의 화면명세가 PRD 의 화면명세다.** 손으로 두 벌 쓰지 않는다.

---

## 0. 어디에 만드나

| | 경로 | 주소 |
|---|---|---|
| **사용자 사이트** | `frontend/apps/playground/src/app/(proto)/proto/core/[locale]/…` | `/proto/core/ko/…` |
| **운영 콘솔** | `frontend/apps/playground/src/app/(proto)/proto/console/…` | `/proto/console/…` |
| 공용 컴포넌트 | `frontend/packages/ui` (`@studyclub/ui`) | — |
| 더미 데이터 | `frontend/packages/mock` (`@studyclub/mock`) | — |

```
pnpm --filter playground dev      # 4702 번
```

- **`@studyclub/ui` 를 프로토 사정으로 고치지 않는다** — 팀이 같이 쓰는 코드다. 프로토 전용
  속성을 심으면 그게 실서비스로 흘러간다
- 더미는 **난수를 쓰지 않는다.** 새로고침마다 명단이 바뀌면 화면을 판단할 수 없다 —
  id·이메일 같은 고정값에서 결정적으로 만든다

---

## 1. 화면명세는 `spec.ts` 가 원본이다

화면 옆에 `spec.ts` 를 두고, 거기 적은 것이 **① 화면의 「설명 보기」 패널**과 **② PRD §2 화면명세**
양쪽이 된다. **PRD 에 손으로 쓰지 않는다.**

```ts
export const SPEC: ScreenSpec = {
  screen: '알림 발송 이력 (운영 콘솔)',
  chip: '발송 이력',          // 설명 보기에서 고르는 이름
  scope: 'history',           // PRD 의 어느 블록으로 갈지
  prd: 'notification-history',// 어느 PRD 로 갈지 (stories/<slug>)
  notes: [ /* 화면 전체에 걸리는 말 */ ],
  entries: [ /* 번호별 명세 */ ],
};
```

화면이 둘 이상이면 `export const SPECS = [LIST_SPEC, ROLE_SPEC]` 로 모아 둔다.

### 번호 붙이기

요소에 `data-anno="<scope>:<n>"` 을 단다. **번호는 화면 위 위치 기준** — 큰 영역이 `1`,
그 안이 `1-1`·`1-2`.

```tsx
<div data-anno='history:1'> … </div>
```

공용 컴포넌트처럼 속성을 못 다는 요소만 `selector` 로 가리킨다.

### 한 항목에 적는 것

| 칸 | 무엇 |
|---|---|
| `display` | 그 요소가 **사용자에게 알려주는 것**. 형태(도넛·칩·색·배치)는 쓰지 않는다 — 디자이너가 형태를 바꿔도 참이어야 한다 |
| `behavior` | 누르면·입력하면 무엇이 일어나는가 |
| `policy` | 판정 기준·제약. **「이 문장대로 코드를 짤 수 있는가」**를 물어 못 짜면 고쳐 쓴다 |
| `data` | 어떤 값이 어디서 오는가. 미구현이면 `TODO(api)` |
| `when` | 조건부로만 그려질 때의 조건 — 적어 두면 대조 실패로 세지 않는다 |

**미설명 0 이 기준이다.** 화면에 보이는 요소는 빠짐없이 어느 번호의 `display` 나 `data` 에 있어야
한다. 「설명 보기」 패널이 배지와 실제 텍스트를 대조해 주니, 엉뚱한 데 붙으면 바로 드러난다.

---

## 2. PRD 로 내보내기

PRD 의 화면명세 자리에 **받을 자리**를 만들어 둔다.

```markdown
## 2. 화면명세

![화면 이름](https://raw.githubusercontent.com/StudyClub-PlusPlus/studyclub-engineering/beta/01-planning/stories/<slug>/assets/<name>.png)

<!-- spec:start history -->
<!-- spec:end history -->
```

```
pnpm --filter playground export-specs            # spec.ts → PRD 블록 교체
node scripts/export-specs.mjs --check            # CI 검사 — 둘이 다르면 실패
```

- **그 블록 안을 손으로 고치지 않는다.** 다음 실행에 덮인다 — 고치려면 `spec.ts` 를 고친다
- `prd` 를 안 적으면 그 화면은 건너뛴다

---

## 3. 캡처

PRD 화면명세 맨 앞에 **번호 배지가 달린 캡처 한 장**을 넣는다. 번호만 있고 그림이 없으면
어느 번호가 어디인지 글로만 찾아야 한다.

- **「설명 보기」를 켠 상태**로 찍는다 — 주황색 번호 동그라미가 그 상태에서만 보인다
- 설명 패널과 「설명 보기」 버튼은 숨기고, 제목부터 마지막 `data-anno` 까지 잘라 찍는다
- **Playwright 는 `frontend/apps/back-office-front` 에서 실행한다** — 의존성이 거기 있다
- 파일은 `01-planning/stories/<slug>/assets/` 에, PRD 에서는 **`raw.githubusercontent.com` 절대
  주소**로 건다 (상대 경로는 노션으로 옮기면 깨진다)

> 화면을 고쳤으면 **캡처도 다시 찍는다.** 열이 하나 바뀐 캡처가 제일 오래 남는다.

---

## 4. 내기 전에

```
npx turbo run lint typecheck --filter=playground     # eslint --max-warnings 0
pnpm --filter playground export-specs                # PRD 와 맞추기
```

- `beta` 에 머지되면 **playground.studyclub.life** 로 배포된다
- 브랜치 상태로는 PRD 의 캡처가 안 보인다(절대 주소라 beta 에 올라가야 뜬다)
- 화면 문구는 **POL-0008 · POL-0011** 을 열고 그대로 쓴다 (§5-20)
