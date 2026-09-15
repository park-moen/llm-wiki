# 사내 하네스 3종 실측 — 무게와 greenfield 편향

> Source: Local personal guidance: ~/Documents/Project/blue-print/docs/wiki/02-inhouse-harness-audit.md
> Collected: 2026-08-10
> Published: 2026-08-10
---
title: 사내 하네스 3종 실측 — 무게와 greenfield 편향
category: AI 엔지니어링 / 사내 도구 평가
created: 2026-08-10
status: 실측 완료 / 설계 제안은 미구현
수명: 짧다 — 조사 대상이 한 달에 35회 버전업 중. 아래 만료조건 참조
관련문서:
  - 01-prose-vs-deterministic-gates.md (여기서 쓰는 "산문/결정적 게이트" 개념의 정의)
  - 03-brownfield-harness.md (이 문서가 발견한 공백을 메우는 설계)
조사대상버전:
  - itnew-forge v0.35.0
  - itnew-dev v0.58.0 (itnew-plan + itnew-work)
  - orbit v1.2.7 (미설치, 마켓플레이스 카탈로그 상태)
  - itnew-blueprint v1.5.0
만료조건: |
  아래 중 하나라도 해당하면 §1~§3 전체 재조사.
  - itnew-forge 가 v0.40 대에 진입 (조사 시점 v0.35.0, 월 35회 페이스)
  - 어느 플러그인이든 hooks/hooks.json 에 PreToolUse 또는 PostToolUse 추가
  - itnew-forge 또는 itnew-plan 에 brownfield / 코드 역산 진입점 추가
재조사방법: §5 참조 (명령어 그대로 실행 가능)
---

# 사내 하네스 3종 실측

## 이 문서를 읽기 전에

**이 문서는 만료를 전제로 쓰였다.** 조사 대상인 `itnew-forge` 는 2026-05-28 v0.1.0 에서
2026-06-29 v0.35.0 까지 **한 달에 35회** 버전업했다. 아래 수치는 빠르게 낡는다.

수치가 낡아도 **§4의 관찰**은 유효할 가능성이 높다. 그것이 이 문서를
[01](./01-prose-vs-deterministic-gates.md)·[03](./03-brownfield-harness.md)과 분리한 이유다.

용어(산문 게이트 / 결정적 게이트)는 [01번 문서](./01-prose-vs-deterministic-gates.md)에서 정의한다.

---

## 1. 무게 실측

### 1.1 itnew-forge v0.35.0

| 항목 | 실측값 |
|---|---|
| Stage 1 `--setup` 완주 | 서브에이전트 **6회 스폰** (layered 모드는 9회) + 사용자 협의 8+ 지점 |
| FEAT 하나 구현 | **10~15회 이상** 개별 서브에이전트 호출 |
| 문서량 | `skills/forge/` phases + references **2,988줄** (`SKILL.md` 포함 시 3,141줄) |
| 에이전트 수 | `agents/` **26개 파일** |
| 개발 속도 | v0.1.0(2026-05-28) → v0.35.0(2026-06-29), **월 35회 버전업** |
| 에이전트 재사용 | **금지** — 안티패턴 #1이 phase마다 재스폰 강제 |

**문서 자체의 산술 불일치**: `skills/forge/SKILL.md:67` 은 *"에이전트 16 + cheap-first 로컬 위임 풀 7 = 23명"*
이라고 쓰지만 frontmatter 배열은 26개, `agents/` 실제 파일도 26개다.
`README.md` 는 또 "13 에이전트"라고 쓴다. **문서의 성장 속도가 유지보수를 앞질렀다는 방증이다.**

FEAT 하나에 검증이 5~6겹으로 겹친다 — hard gate(gap-analyzer, e2e-runner, test-presence) +
advisory(visual-verifier, fanout-verifier B4/B5) + cheap-first 조건부 위임 풀(최대 7종) +
review-fix 루프(cap 3).

### 1.2 대조군 — itnew-dev v0.58.0

| 항목 | 실측값 |
|---|---|
| 단일 TASK 실행 (**기본 설정**) | 서브에이전트 **0개** |
| 로컬 에이전트 수 | **3개** (`cto-advisor`, `breakdown-reviewer`, `tdd-guide`) |
| "표준" 프리셋 적용 시 | TASK당 2~3개 (+ code-reviewer 재리뷰 최대 4회) |
| "FE 풀 QA" 프리셋 | FE TASK당 최대 5~6개 |

기본 설정에서 `preTaskRun`·`postTaskRun`·`onSensitive` 가 **전부 빈 배열**이라
서브에이전트가 한 개도 호출되지 않는다 (`itnew-dev.config.json:10`).

> **함의**: 같은 사내 도구인데 단일 TASK 기준 **0개 대 10~15개**다.
> 도구가 나쁜 게 아니라 **용도가 다르다** — forge는 N명이 N세션으로 병렬 개발하는 하네스다.
> 1인 단건 작업에 쓰면 오버헤드가 본체를 압도한다.

### 1.3 두 플러그인의 관계

역할 분담을 규정한 문서는 **양쪽 어디에도 없다** (`grep -rln "forge" itnew-dev/` → 0건).
유일한 언급은 forge 쪽 `README.md:6`:

> "기존 `itnew-dev`(plan + work, v0.57.x) 의 정신적 후속이지만 **호환되지 않는 clean-break**."

이는 역할 분담 선언이 아니라 **버전 계승 선언**이다.
"언제 어느 것을 쓰라"는 가이드가 없어, 사용자가 스스로 판단해야 한다.

산출물 경로는 겹치지 않는다 (forge: `docs/dev/harness/**`, itnew-plan: `docs/dev/**`).

---

## 2. TDD 강제 수단 실측

세부 논의는 [01번 문서 §2](./01-prose-vs-deterministic-gates.md) 참조. 결과만 옮기면:

| 대상 | hooks.json | 실제 강제 수단 | 판정 |
|---|---|---|---|
| itnew-forge | **파일 없음** | team-lead LLM이 `git diff` 직접 실행·판정 (`phases/lane.md:166-170`) | 산문 |
| itnew-dev | SessionStart 단독 | `preTaskRun` 은 훅이 아닌 config 키. `run.md` Step 4.5 산문 절차 | 산문 |
| orbit | SessionStart 단독 | `Precondition: currentPhase === "red"` (마크다운) | 산문 |

**활성 플러그인 6종 전체에 PreToolUse 훅 0개.**

주목할 점 세 가지:

1. **forge의 규칙 자체는 훌륭하고, 진단도 정확했다.**
   `phases/lane.md:166-170` 은 이 게이트를 *"LLM 불필요(결정적).
   **ephemeral 화로 사라진 tdd-guard watchdog 의 대체이자 강화**"* 라고 설명한다.
   저자들은 "일회성 조언자는 실시간 감시를 못 한다"는 문제를 알고 대안을 넣었다.

   **다만 신뢰가 한 단계 옮겨갔을 뿐이다.** 판정 로직은 결정적이지만
   *그 검사를 실행할지 말지는 team-lead LLM이 결정한다.*
   훅이 아니므로 건너뛰면 아무 일도 일어나지 않고, 그 결과가 v0.32.0 *"per-TASK d2 미발화"* 다.
   같은 규칙을 훅에 걸면 재량 자체가 사라진다 —
   **"판정이 결정적"인 것과 "게이트가 결정적"인 것은 다르다.**

2. **명시적 예외 조항이 있다** (`lane.md:169`) —
   *"tdd-guard 가 'trivial → 게이트 skip 가능' 판정하고 team-lead 가 기록한 TASK"* 는 게이트를 건너뛴다.

3. **itnew-dev는 기본값이 off다.** 설치만 하고 `/itnew work init-config` 로 프리셋을 고르지 않으면
   `tdd-guide` 는 **한 번도 호출되지 않는다.**

---

## 3. greenfield 편향

### 3.1 세 도구 모두 기획 문서를 하드 전제로 요구한다

| 대상 | 근거 |
|---|---|
| itnew-forge | `skills/forge/phases/setup.md:37` — PRD·spec·traceability-matrix 중 *"하나라도 0 → 즉시 중단 + '/blueprint 선행 안내'. **추측 진행 절대 금지**"* |
| itnew-plan | `skills/itnew-plan/phases/analyze.md:13-19` — *"누락 시 즉시 중단… 중간에 추측하지 말고 **중단**"*, `:76` 에서 `abort()` 재확인 |
| orbit | 진입점이 PRD 단 하나. Error Handling 표에 *"PRD 파일 없음 → 파일 경로 확인 요청"* 이 전부 |

**우회 경로는 문서화되어 있지 않다.** "추측 생성 금지"가 명문화돼 있어,
코드만 보고 스펙을 만들어주는 fallback이 **의도적으로 배제**돼 있다.

### 3.2 brownfield 지원 여부 — grep 결과

itnew-forge 전체:

```
retroactive  → 0건
brownfield   → 0건
인수인계      → 0건
legacy       → 2건 (모두 SQL 예제 컬럼명 "legacy_status" 등, 무관)
amend        → 51건 (전부 "Stage 1 계획을 다시 협의해 재실행". 코드 역산 아님)
```

#### ⚠️ 함정 — 폐기된 문서가 grep에 잡힌다

itnew-dev 를 같은 조건으로 grep 하면 **5건이 나오고, 그중 하나는 매우 그럴듯하다**:

```
itnew-dev/docs/LANE-MODEL.md:104 → "## 시나리오 2: 기존 프로젝트 (Brownfield)"
```

내용도 진짜처럼 보인다 — *"be-api/src/**/*Controller.kt 존재 → **reverse 모드** →
서브에이전트 디스패치 → **OpenAPI 역생성**"*. 코드 역산 기능이다.

**그러나 이 스킬은 존재하지 않는다.**

| 확인 | 결과 |
|---|---|
| `ls skills/` | `itnew-commit`, `itnew-handoff`, `itnew-plan`, `itnew-work` — **devlane 없음** |
| `grep -rn "reverse" skills/` | **0건** |
| `README.md:337` | *"구 devplan/**devlane**/devrun에서 이전"* — 명시적 구버전 |

`docs/LANE-MODEL.md` 는 **폐기된 스킬의 사용 가이드가 레포에 남아 있는 것**이다.
나머지 4건도 `itnew-handoff`(세션 인수인계 = 다른 의미)라 무관하다.

그리고 이 폐기 문서조차 전제가 *"`docs/planning/`, `docs/dev/task-breakdown.md` **존재**"* 였다.
여기서 말하는 "Brownfield"는 **하네스로 이미 작업을 시작한 프로젝트의 중간 재개**를 뜻하며,
"기획 문서 없는 남의 서비스 인수인계"가 아니다.

> **교훈 1**: 사내에도 코드 역산(OpenAPI reverse) 시도가 있었으나 현행 버전에서 사라졌다.
> **교훈 2**: **문서 grep 결과를 `skills/` 실재 여부와 반드시 교차 확인할 것.**
> 폐기된 기능의 문서가 남아 "있는데 안 쓴다"는 착각을 만든다.

### 3.3 함정 — itnew-plan 의 `retroactive` 는 이름과 다르다

이름 때문에 "코드에서 역으로 문서를 만드는 기능"으로 오해하기 쉽다. 실제로는 아니다
(`phases/retroactive.md:3`):

> "⚠ 이 phase 는 `docs/dev/task-breakdown.md` + `context/*.md` 가 **이미 존재**해야 의미 있음.
> **신규 프로젝트는 `/itnew plan breakdown` 만으로 충분**."

실체는 기존 `context.md` 의 frontmatter(`designSource`/`refs`)를 신버전 매칭 규칙으로
**재계산하는 마이그레이션 도구**다(`:5`, `:20`). 여전히 `docs/planning/**` 을 glob으로 찾는다.

즉 4개 phase(analyze / breakdown / followup / retroactive) **어느 것도**
`docs/planning/` 없이 처음부터 동작하지 않는다.

### 3.4 흥미로운 반증 — 막히는 건 진입부뿐이다

orbit 의 `orbit-tdd-driver` 에이전트는 컨텍스트로 **"기존 코드 파일 경로"** 를 이미 전달받는다.

즉 **실행부는 기존 코드베이스에서 동작할 수 있다.** 막히는 것은 진입부(태스크 생성)뿐이다.
이는 greenfield 편향이 아키텍처의 본질이 아니라 **입력 계약의 선택**임을 시사한다.

### 3.5 orbit 추가 사항 — 현재 실행 불가

orbit은 bkit(PDCA)과 taskmaster(PRD 파싱)에 전적으로 의존한다.

| 의존성 | 상태 (2026-08-10) |
|---|---|
| bkit | `cache/bkit-marketplace/` 에 `plugin.json` 조차 없음 — 사실상 빈 디렉토리 |
| taskmaster | `cache/taskmaster/…/` 에 파일은 있으나 `installed_plugins.json`·`known_marketplaces.json` 양쪽 미등록 |

`scripts/session-start.js` 가 검사하는 4개 경로가 **전부 MISSING** 이라,
설치 시 매 세션 경고가 뜨고 `/orbit init` 단계에서 진행이 막힌다.

---

## 4. 관찰 — 수치가 낡아도 유효할 것들

### 4.1 무게는 기능이 아니라 "사후 봉인 규칙"의 누적에서 온다

`references/orchestrator-dispatch.md` 에 안티패턴 #1~#16이 명문화돼 있고,
체인지로그는 대부분 "게이트 미발화 fix" 계열이다.
**문서의 상당 부분이 정상 경로가 아니라 과거에 실패했던 경로에 대한 봉인이다.**

### 4.2 산문 강제와 문서 비대는 같은 원인의 두 증상이다

강제가 산문이라 신뢰할 수 없다 → 신뢰를 확보하려고 문서에 "결정적 확인"을 계속 못 박는다
→ 문서량과 체크리스트가 늘어난다 → 지켜야 할 게 많아져 누락이 늘어난다 → 다시 봉인 규칙 추가.

`itnew-dev` 가 addons skip 문제를 "strict directive 문구 강화"로 대응한 것이 전형이다
(`README.md:153-161`).

### 4.3 greenfield 편향의 원인은 "정본을 무엇으로 잡았는가"다

세 도구 모두 **정본(source of truth)을 "사람이 미리 쓴 기획 문서"로 고정**했다.
그 선택이 곧 입력 계약이 되고, 입력 계약이 곧 적용 범위를 결정한다.

> 하네스를 평가할 때는 **"정본을 무엇으로 잡았는가"** 를 먼저 본다.
> 정본이 사람이 생산하는 산출물이면 그 하네스는 구조적으로 greenfield 전용이다.

이 관찰이 [03번 문서](./03-brownfield-harness.md)의 출발점이다.

---

## 5. 재조사 방법

이 문서는 만료가 전제이므로 재현 명령을 남긴다.

```bash
PLUG=~/.claude/plugins/marketplaces/itnew/plugins

# 1) 훅 등록 현황 — PreToolUse/PostToolUse 가 생겼는지
find ~/.claude/plugins -name "hooks*.json" -not -path "*/node_modules/*" -exec sh -c \
  'echo "== $1"; grep -oE "\"(SessionStart|PreToolUse|PostToolUse|Stop|UserPromptSubmit)\"" "$1" | sort -u' _ {} \;

# 2) 실제 차단 코드 존재 여부
grep -rn "exit(2)\|exit 2\|permissionDecision" ~/.claude/plugins/ | grep -v node_modules

# 3) 게이트 목록과 hard/advisory 구분
grep -rn "hard gate" $PLUG/itnew-forge/skills/forge/phases/

# 4) brownfield 진입점이 생겼는지
grep -rniE "brownfield|인수인계|역산|reverse" $PLUG/itnew-forge $PLUG/itnew-dev
#    ⚠️ 히트가 나와도 그대로 믿지 말 것 — 폐기된 스킬 문서가 남아 있다 (§3.2 함정 참조).
#    반드시 아래로 교차 확인:
ls $PLUG/itnew-dev/skills/ $PLUG/itnew-forge/skills/     # 실재하는 스킬만이 현행
grep -rn "reverse" $PLUG/itnew-dev/skills/               # docs/ 가 아니라 skills/ 에 있는지

# 5) 필수 입력 전제가 완화됐는지
grep -rn "즉시 중단\|추측 진행 절대 금지\|추측 생성 금지" $PLUG/

# 6) 무게 — 에이전트 수와 문서량
ls $PLUG/itnew-forge/agents/*.md | wc -l
find $PLUG/itnew-forge/skills/forge/phases $PLUG/itnew-forge/skills/forge/references \
     -name "*.md" -exec cat {} + | wc -l     # §1.1 의 2,988줄과 같은 기준
#    (SKILL.md 포함 전체는 3,141줄 — 기준을 섞지 말 것)

# 7) 현재 버전
grep -h '"version"' $PLUG/*/.claude-plugin/plugin.json

# 8) 실제 활성화된 플러그인 (카탈로그에 있다고 설치된 게 아님)
cat ~/.claude/plugins/installed_plugins.json
```

**주의**: `marketplaces/` 에 소스가 있다고 설치된 것이 아니다.
`installed_plugins.json` 에 없으면 세션에서 활성화되지 않는다. (8번이 그 확인용)

---

## 6. 근거 인덱스

경로는 `~/.claude/plugins/marketplaces/` 기준.

| 주장 | 근거 |
|---|---|
| forge 필수 입력 하드 중단 | `itnew/plugins/itnew-forge/skills/forge/phases/setup.md:37` |
| forge test-presence 규칙 | `.../phases/lane.md:166-170` (예외 조항 `:169`) / feature-team 모드는 `phases/feature-team.md:174` |
| forge 게이트 미발화 이력 | `itnew-forge/CLAUDE.md` v0.32.0, v0.8.10 |
| forge tdd-guard 권한 | `itnew-forge/agents/tdd-guard.md:4` (`tools: Read/Glob/Grep`) |
| forge 에이전트 재사용 금지 | `.../references/orchestrator-dispatch.md:179` (안티패턴 #1) |
| forge 에이전트 수 산술 불일치 | `skills/forge/SKILL.md:67` vs `agents/` 실제 파일 수 |
| forge–dev 관계 선언 | `itnew-forge/README.md:6` |
| itnew-plan 하드 중단 | `itnew-dev/skills/itnew-plan/phases/analyze.md:13-19`, `:76` |
| retroactive 실체 | `.../phases/retroactive.md:3`, `:5`, `:20` |
| itnew-dev addons skip 실증 | `itnew-dev/README.md:155` |
| itnew-dev preTaskRun 해석 위치 | `itnew-dev/skills/itnew-work/phases/run.md` Step 4.5 (210~225행) |
| itnew-dev 기본값 off | `itnew-dev/itnew-dev.config.json:10` |
| orbit TDD 탈출구 | `itnew/plugins/orbit/orbit.config.json` (`tdd.enabled`) |
| orbit PRD 단일 진입점 | `orbit/skills/orbit-workflow/SKILL.md` Error Handling 표 |
| orbit 실행부의 기존 코드 수용 | `orbit/skills/orbit-tdd/SKILL.md` (에이전트 컨텍스트 "기존 코드 파일 경로") |
| orbit 의존성 경로 검사 | `orbit/scripts/session-start.js` |

---

## 7. 한계 및 미검증

- **에이전트 스폰 수는 문서상 호출 경로를 세어 추정한 값**이며, 실행 로그 계측이 아니다.
  실제 실행 시 조건 분기로 더 적거나 많을 수 있다.
- 조사는 **문서와 코드 정적 분석** 기준이다. 실제 세션에서 게이트가 어떻게 동작하는지는
  별도 계측이 필요하다.
- **"무겁다"는 것이 결함이라는 뜻은 아니다.** forge는 N명 병렬 개발을 전제로 설계됐고,
  그 목적에서는 검증 다중화가 합리적이다. 이 문서의 지적은 **용도와 도구를 맞추라**는 것이다.
- 이 문서는 도구 평가이지 **사람 평가가 아니다.** 작성자 개인은 언급하지 않는다.
- 여기 담긴 설계 제안([03번](./03-brownfield-harness.md))은 **아직 구현·검증되지 않았다.**

