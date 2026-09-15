# setup-matt-pocock-skills

> Source: https://www.skills.sh/mattpocock/skills/setup-matt-pocock-skills; https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/SKILL.md
> Collected: 2026-09-15
> Published: Unknown

## Source record

`setup-matt-pocock-skills`는 Matt Pocock의 engineering Skill이 repository별 규칙을 찾을 수 있도록 초기 설정을 만드는 사용자 호출형 Skill이다. skills.sh 페이지와 연결된 GitHub `SKILL.md`를 함께 확인했다. 저작권이 있는 페이지 전문은 복제하지 않고, Wiki의 근거가 되는 설정 구조와 동작을 기록한다.

설치 명령:

```text
npx skills add https://github.com/mattpocock/skills --skill setup-matt-pocock-skills
```

Frontmatter의 주요 값:

```yaml
name: setup-matt-pocock-skills
disable-model-invocation: true
```

설명에는 다른 engineering Skill을 처음 사용하기 전에 한 번 실행해 issue tracker, triage label vocabulary와 domain document layout을 설정하라고 적혀 있다.

## Process

이 Skill은 deterministic script가 아니라 prompt-driven Skill이다. 현재 repository를 탐색하고, 발견한 내용을 보여주고, 사용자에게 확인받은 뒤 file을 작성한다.

### Explore

다음을 확인한다.

- `git remote -v`와 `.git/config`
- root의 `AGENTS.md`와 `CLAUDE.md`
- `CONTEXT.md`와 `CONTEXT-MAP.md`
- `docs/adr/`, `src/*/docs/adr/`, `docs/agents/`
- local Markdown issue tracker의 신호인 `.scratch/`
- `triage` Skill 설치 여부
- `pnpm-workspace.yaml`, `package.json`의 `workspaces`, `packages/*/src/` 같은 monorepo 신호

### Guided decisions

1. Issue tracker
   - GitHub remote면 GitHub Issues와 `gh` CLI를 제안한다.
   - GitLab remote면 GitLab Issues와 `glab` CLI를 제안한다.
   - Remote가 없거나 사용자가 원하면 `.scratch/<feature>/`의 local Markdown을 쓸 수 있다.
   - Jira, Linear 등은 사용자가 workflow를 설명하면 freeform prose로 기록한다.
2. Triage labels
   - `triage` Skill이 설치된 경우에만 묻는다.
   - 기본 role은 `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`다.
3. Domain docs
   - 기본은 root `CONTEXT.md`와 `docs/adr/`를 사용하는 single-context다.
   - 실제 monorepo 신호가 있을 때만 root `CONTEXT-MAP.md`와 context별 `CONTEXT.md`를 쓰는 multi-context를 제안한다.

질문은 한 section씩 진행하고, 이미 탐색으로 결정된 항목은 다시 묻지 않는다.

### Confirm and write

쓰기 전에 다음 초안을 사용자에게 보여주고 수정 기회를 제공한다.

- root instruction file에 넣을 `## Agent skills` block
- `docs/agents/issue-tracker.md`
- `docs/agents/domain.md`
- `triage` Skill이 있을 때만 `docs/agents/triage-labels.md`

Instruction file 선택 규칙:

- `CLAUDE.md`가 있으면 그 file을 수정한다.
- 없고 `AGENTS.md`가 있으면 그 file을 수정한다.
- 둘 다 없으면 어느 file을 만들지 사용자에게 묻는다.
- 기존 file의 주변 사용자 내용을 덮어쓰지 않는다.
- `## Agent skills` block이 이미 있으면 중복 추가하지 않고 해당 block을 갱신한다.

생성한 `docs/agents/*.md`는 `to-tickets`, `triage`, `to-spec`, `diagnose`, `tdd`, `improve-codebase-architecture` 같은 downstream Skill이 읽는다. 이후에는 이 문서를 직접 편집할 수 있으며, tracker를 바꾸거나 처음부터 다시 설정할 때만 setup Skill을 재실행한다.
