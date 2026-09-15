# 산문 게이트 vs 결정적 게이트 — AI 하네스의 강제력은 어디서 오는가

> Source: Local personal guidance: ~/Documents/Project/blue-print/docs/wiki/01-prose-vs-deterministic-gates.md
> Collected: 2026-08-10
> Published: 2026-08-10
---
title: 산문 게이트 vs 결정적 게이트 — AI 하네스의 강제력은 어디서 오는가
category: AI 엔지니어링 / 하네스 설계 원리
created: 2026-08-10
status: 실측 기반 (플러그인 4종 조사 완료)
수명: 길다 — 특정 도구와 무관한 원리
관련문서:
  - 02-inhouse-harness-audit.md (사내 3종 실측 상세)
  - 03-brownfield-harness.md (이 원리를 적용한 설계)
조사대상버전:
  - itnew-forge v0.35.0 / itnew-dev v0.58.0 / orbit v1.2.7 / superpowers v6.2.0
  - Claude Code 훅 문서 (code.claude.com/docs/en/hooks, 2026-08-10 열람)
만료조건: |
  Claude Code 훅 스키마에 새 이벤트가 추가되거나 exit code 의미가 바뀌면 §2 재확인.
  Stop 훅에 공식 무한 루프 방지 장치가 생기면 §3 폐기.
---

# 산문 게이트 vs 결정적 게이트

## 요약

AI 개발 하네스가 "TDD를 강제한다", "이 게이트를 반드시 통과해야 한다"고 말할 때,
그 강제력은 두 가지 중 하나다:

- **산문 게이트(prose gate)** — 마크다운에 "반드시", "MUST", "절대 금지"라고 쓰여 있다.
  LLM이 그 지침을 읽고 **스스로 지키기로 하는 것**이다.
- **결정적 게이트(deterministic gate)** — 훅이 도구 호출이나 턴 종료를 실제로 거부한다.
  LLM의 의사와 무관하다.

**판별 기준은 하나다:**

> **모델이 그 지침을 무시했을 때, 시스템이 막는가?**

실측 결과 조사한 하네스 4종의 강제 수단은 **전부 산문**이었다.

---

## 1. 왜 이 구분이 중요한가

산문 게이트는 쓸모없지 않다. 잘 쓴 지침은 준수율을 상당히 올린다.
문제는 **보장이 없다는 것**이고, 더 큰 문제는 **보장이 있다고 착각하게 만든다는 것**이다.

산문 게이트의 실패는 조용하다. 게이트가 발화하지 않아도 로그에 아무것도 남지 않는다.
"통과했다"와 "검사하지 않았다"가 겉보기에 동일하다.

실제 사례 — itnew-forge 체인지로그:

| 버전 | 항목 |
|---|---|
| v0.8.10 | "BE lane FEAT 완료 게이트 **누락** fix" |
| v0.32.0 | "tdd-guard **0회 발화**·per-TASK d2 **미발화**" |

게이트가 있다고 문서에 쓰여 있었고, 실제로는 한 번도 발화하지 않았다.
이런 버그는 사후에 우연히 발견된다.

또 다른 사례 — itnew-dev 는 이 문제를 실측으로 확인해 README에 기록했다
(`itnew-dev/README.md:155`):

> "v0.36.3 의 단순 자연어 prompt("run.md Step 따름")만으로는 child Claude 가 자유 해석으로
> addons(preTaskRun/postTaskRun/lint/autoCommit) **통째로 skip** … (사용자 TASK-32 실증)"

대응은 "strict directive 문구 강화"였다 — **산문의 결함을 더 강한 산문으로 고치려는 시도**다.

---

## 2. 실측 — 하네스 4종의 강제 수단

### 2.1 조사 방법

각 플러그인에 대해 세 가지를 확인했다:

1. `hooks/hooks.json` 에 등록된 이벤트
2. 차단 판정 코드(`exit 2` / `permissionDecision: "deny"`)의 존재
3. 강제 문구가 코드인지 산문인지

```bash
# 재현 명령
find ~/.claude/plugins -name "hooks*.json" -not -path "*/node_modules/*"
grep -rn "PreToolUse\|PostToolUse" ~/.claude/plugins/*/…/hooks.json
grep -rn "exit(2)\|exit 2\|permissionDecision" <플러그인 경로>
```

### 2.2 결과

| 대상 | 표방 | hooks.json 등록 이벤트 | 실제 강제 수단 | 판정 |
|---|---|---|---|---|
| itnew-forge | "TDD 강제" | **hooks.json 파일 자체 없음** | 메인 LLM이 `git diff` 를 직접 실행·판정 | 산문 |
| itnew-dev | "preTaskRun 훅" | SessionStart 단독 | `preTaskRun` 은 훅이 아니라 **config 키**. `run.md` Step 4.5 의 산문 절차 | 산문 |
| orbit | "TDD 통합" | SessionStart 단독 | `Precondition: currentPhase === "red"` (마크다운) | 산문 |
| superpowers | "Iron Law" | SessionStart 단독 | SKILL.md 본문을 컨텍스트에 주입 | 산문 |

**활성 플러그인 6종 전체에 PreToolUse 훅이 0개.**
유일한 PostToolUse(itnew-blueprint 와이어프레임 드리프트 감지)는 항상 `sys.exit(0)` 으로 끝나는
안내성 훅이다.

### 2.3 특징적인 사례

**규칙은 좋은데 집행자가 잘못된 경우 — itnew-forge**

강제의 실체는 `phases/lane.md:166-170` 의 "결정적 test-presence 게이트"다:

> team-lead 가 이 TASK 의 변경 집합을 `git diff --name-only` 로 검사.
> **src 파일이 변경됐는데 대응 테스트 파일의 추가/수정이 하나도 없으면 → block, engineer 반송**
> ("실패 테스트 먼저" — TDD 우회 차단). cap 3 회 후 사용자 escalate.

**이 사례는 특히 교훈적이다. forge 저자들은 문제를 정확히 진단했다.**
같은 문단이 이렇게 잇는다:

> "LLM 불필요(결정적). **ephemeral 화로 사라진 tdd-guard watchdog 의 대체이자 강화.**"

즉 "조언자 에이전트는 일회성이라 실시간 감시를 못 한다"는 것을 알았고,
그 대안으로 **판정 로직이 결정적인 검사**를 도입했다. 실제로 `git diff` 기반 판정은 결정적이다.

**그런데 신뢰가 한 단계 옮겨갔을 뿐이다.** 판정 *로직*은 결정적이지만,
그 검사를 **실행할지 말지는 여전히 team-lead LLM이 결정**한다.
훅이 아니므로 team-lead가 이 단계를 건너뛰면 아무 일도 일어나지 않는다.
그 결과가 체인지로그에 남아 있다 — v0.32.0 *"per-TASK d2 **미발화**"*.

여기에 명시적 예외 조항도 있다(`lane.md:169`):

> "예외: step a 에서 tdd-guard 가 '**trivial → 게이트 skip 가능**' 판정하고 team-lead 가 기록한 TASK."

전담 에이전트 `tdd-guard` 는 `tools: ["Read","Glob","Grep"]` 으로 차단 권한이 없고, 스스로 인정한다:

> "engineer 가 테스트 없이 구현 시도 | **본 에이전트 책임 아님 (ephemeral 이라 live 감시 불가).**"

> **정리**: 규칙도 좋고 진단도 정확했다. 부족했던 것은 **집행 위치**다.
> 같은 규칙을 훅에 걸면 "실행할지 말지"라는 재량 자체가 사라진다.
> **"판정이 결정적"인 것과 "게이트가 결정적"인 것은 다르다.**

**공식 탈출구가 있는 경우 — orbit**

`orbit.config.json` 에 `tdd.enabled: false` 가 있고, 문서에 *"false면 Do 단계에서 TDD 없이 구현"*
이라고 명시돼 있다. 강제라고 부르기 어렵다.

**선수와 채점자가 같은 경우 — orbit**

`Precondition: currentPhase === "red"` 를 검사하는데, 그 `currentPhase` 를
`.orbit-state.json` 에 기록하는 주체도 모델 자신이다. **자기 신고다.**

**문구가 가장 센 경우 — superpowers**

> "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
> "Write code before the test? Delete it."

session-start 훅이 `using-superpowers/SKILL.md` **본문 전체를 매 세션 주입**하므로
준수율은 실제로 높다. 그러나 차단 코드는 없다.

---

## 3. 결정적 게이트를 만드는 법

### 3.1 차단 능력이 있는 위치

훅만이 도구 호출과 턴 종료를 실제로 막는다. `exit 2` 의 효과는 이벤트별로 다르다:

| 이벤트 | 차단 | `exit 2` 효과 |
|---|---|---|
| **PreToolUse** | O | 도구 호출 자체를 거부 |
| **Stop** | O | *"Prevents Claude from stopping, continues the conversation"* |
| PostToolUse | O (사후) | 이미 실행된 행위에 대해 모델을 다시 깨움 |
| SessionStart | X | 컨텍스트 주입만 |

`exit 2` 시 **stdout은 무시되고 stderr가 모델에게 에러 메시지로 전달**된다.
exit 0 과 함께 `{"decision":"block","reason":"..."}` JSON을 내보내는 방식도 있다.

### 3.2 이벤트 선택이 곧 UX다

같은 규칙이라도 어느 이벤트에 거느냐에 따라 개발 경험이 완전히 달라진다:

| 이벤트 | 체감 | 적합한 규칙 |
|---|---|---|
| PreToolUse | 작업 중 상시 차단. 강력하지만 성가심 | 위험한 명령, 비밀정보 유출 |
| **Stop** | **턴 경계에서만 발화. TASK 단위 체크포인트** | TDD 준수, 완료 조건 검사 |
| PostToolUse | 사후 피드백 | 린트, 드리프트 감지 |

"TDD를 강제하되 작업 중에는 방해받고 싶지 않다"면 답은 **Stop 훅**이다.
Stop은 matcher를 지원하지 않고 **항상 발화**하므로, 훅 스크립트 내부에서 조건을 걸러야 한다.

### 3.3 함정 — 무한 루프

**Stop 훅으로 차단할 때 반드시 설계해야 한다.** 공식 문서에 안전장치가 **없다**:

> "there is no documented safeguard mentioned. The responsibility appears to fall on
> the hook author to avoid creating conditions that would cause infinite blocking."

조건 없이 `exit 2` 를 반복하면 대화가 영원히 끝나지 않는다.
모델이 지침을 따를 수 없는 상황(예: 테스트를 쓸 수 없는 정당한 이유)에서도 계속 막히면
사용자가 강제 종료하는 수밖에 없다.

**해법 — 상태 해시 기반 1회 차단:**

```
1. 차단 조건에 해당하는 대상(예: 변경 파일 집합)의 해시를 계산
2. 이전에 같은 해시로 차단한 적이 있으면 → exit 0 + 경고만 출력
3. 처음 보는 해시면 → 해시 기록 후 exit 2
```

모델이 테스트를 추가하면 해시가 바뀌어 리셋되고,
사람에게 escalate 하기로 했으면 두 번째 Stop은 통과한다.
**차단은 1회, 종료는 보장.**

### 3.4 대조군 — 실제 차단 코드는 어떻게 생겼나

조사한 플러그인 중 실제 차단 코드를 가진 것은 `bkit`(외부 OSS, 미설치) 하나였다:

```js
// bkit/hooks/pre-write.js
// Philosophy: Automation First — Guide, don't block (exception: permission=deny / explicit danger).
if (perm.deny) {
    outputBlock(perm.denyReason);
    process.exit(2);
}
```

주석의 철학이 시사적이다 — *"Guide, don't block"*. 차단 능력을 갖추고도 기본은 비차단이고,
명시적 deny 규칙에서만 막는다. **차단은 비싸므로 아껴 쓴다**는 판단이다.

반대 사례로 `hookify`(미설치)는 PreToolUse 프레임워크를 제공하지만
`finally: sys.exit(0)` 로 **절대 차단하지 않는 철학**이라 강제 용도로는 부적합하다.
훅이 등록돼 있다고 해서 차단하는 건 아니다.

---

## 4. 도출되는 원리

> **원리 1 — soft + soft = soft**
> 산문 게이트를 여러 개 겹쳐도 결정적 게이트가 되지 않는다.
> 준수율은 오르지만 보장은 생기지 않는다.
> *(예: "TDD가 약한 하네스에 TDD 지침이 강한 플러그인을 얹으면 되지 않을까?" → 안 된다)*

> **원리 2 — 일하는 자가 채점하면 게이트가 아니다**
> 상태를 기록하는 주체가 그 상태를 검사하면 자기 신고다.
> 검사 주체는 검사 대상과 분리돼야 하고, 가장 확실한 분리는 **기계**다.

> **원리 3 — 차단하는 훅에는 종료 보장을 설계해 넣는다**
> 무한 루프 방지는 훅 작성자의 책임이다. 프레임워크가 해주지 않는다.

> **원리 4 — 산문으로 강제하려 하면 문서가 비대해진다**
> 강제가 신뢰할 수 없으니 문서에 "결정적 확인"을 계속 못 박게 되고,
> 그것이 다시 문서량과 체크리스트를 늘린다.
> 하네스가 급격히 무거워지고 있다면 강제 수단을 잘못 골랐다는 신호일 수 있다.
> *(실측: [02-inhouse-harness-audit.md](./02-inhouse-harness-audit.md) §1 참조)*

> **원리 5 — 강제를 도구 밖에 두면 재사용된다**
> TDD 게이트를 하네스 A에 넣으면 A에서만 작동한다.
> 훅으로 빼면 A를 쓰든 B를 쓰든, 심지어 맨손 작업에도 걸린다.
> **강제 층과 워크플로우 층을 분리하라.**

> **원리 6 — 차단 이벤트 선택이 개발 경험을 결정한다**
> 같은 규칙도 PreToolUse에 걸면 성가시고 Stop에 걸면 체크포인트가 된다.
> "무엇을 막을까"보다 "언제 막을까"를 먼저 정한다.

---

## 5. 실무 체크리스트

새 하네스·스킬·플러그인을 도입하거나 만들 때:

- [ ] 이 도구가 "강제"한다는 것이 **산문인가 훅인가** — `hooks.json` 을 직접 열어본다
- [ ] 게이트를 검사하는 주체와 게이트 상태를 기록하는 주체가 **분리돼 있는가**
- [ ] 설정에 **게이트를 끄는 스위치**가 있는가 (있으면 그것은 권고다)
- [ ] 게이트 기본값이 **on인가 off인가** (off면 아무도 안 켠다)
- [ ] 차단 훅이라면 **무한 루프 종료 보장**이 있는가
- [ ] 게이트가 발화했다는 **증거가 로그에 남는가** (안 남으면 미발화를 못 잡는다)

---

## 6. 한계

- 본 문서의 실측은 **2026-08-10 시점, 특정 머신의 설치 상태** 기준이다.
  같은 플러그인이라도 버전과 설정에 따라 다를 수 있다.
- "산문 게이트는 보장이 없다"는 명제는 참이지만, **산문 게이트가 무용하다는 뜻은 아니다.**
  준수율 향상 효과는 실재하며, 특히 superpowers처럼 세션마다 강하게 주입하는 방식은 효과가 크다.
  이 문서의 주장은 **산문을 보장으로 착각하지 말라**는 것이지 산문을 버리라는 것이 아니다.
- 결정적 게이트에도 한계가 있다. 훅은 **기계적으로 판정 가능한 것만** 막을 수 있다.
  "이 테스트가 의미 있는 테스트인가"는 훅이 판정할 수 없다.
  실제 설계에서는 기계 판정과 사람 판정을 어디서 나눌지가 핵심 문제가 된다
  ([03-brownfield-harness.md](./03-brownfield-harness.md) Stage 3 참조).

