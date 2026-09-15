# Orca 리뷰·편집·브라우저 작업 흐름

> Sources: Stably, Inc., Unknown
> Raw: [Diff viewer](../../raw/orca/review-diff-viewer.md); [Annotate AI Diff](../../raw/orca/review-annotate-ai-diff.md); [Attribution](../../raw/orca/review-attribution.md); [Commit & push from Orca](../../raw/orca/review-commit-push.md); [Hosted reviews, issues & Actions](../../raw/orca/review-github.md); [Linear items drawer](../../raw/orca/review-linear.md); [Jira items drawer](../../raw/orca/review-jira.md); [Monaco editor & autosave](../../raw/orca/editing-monaco.md); [Rich markdown editor](../../raw/orca/editing-markdown.md); [Mermaid, PDF & image viewers](../../raw/orca/editing-viewers.md); [File explorer & external drag-drop](../../raw/orca/editing-file-explorer.md); [Per-worktree browser](../../raw/orca/browser-overview.md); [Design Mode](../../raw/orca/browser-design-mode.md); [Browser-use profiles](../../raw/orca/browser-profiles.md); [Review an AI diff line-by-line](../../raw/orca/recipes-review-ai-diff.md); [Fix a UI bug with Design Mode](../../raw/orca/recipes-design-mode-fix.md)
> Updated: 2026-08-12

## Overview

Orca의 편집 기능은 범용 IDE를 흉내 내는 데 목적이 있지 않다. Agent가 worktree에서 만든 변경을 빠르게 확인하고, line-anchored feedback을 되돌려 보내고, browser에서 결과를 검증한 뒤 Git review로 승격시키는 폐쇄 루프를 만드는 데 초점이 있다. Monaco·rich Markdown·format viewer·Chromium browser는 모두 이 review loop의 작업면이다.

## Diff를 중심에 둔 검토

각 worktree의 diff는 기본적으로 start-from ref와 비교한다. Staged, unstaged, untracked change를 합쳐 보거나 다른 commit·branch·base ref로 비교 대상을 바꿀 수 있다. 주요 기능은 다음과 같다.

- File·hunk·line 단위 stage와 line number 표시
- Image의 side-by-side, swipe, onion-skin 비교
- Merge conflict의 three-way view와 inline resolution
- Combined diff의 file tree와 HTML side preview
- Long line word wrap와 changed file·hunk keyboard navigation

AI가 작업한 branch를 고를 때는 먼저 compare base가 의도한 start-from ref인지 확인하고, 전체 file tree로 변경 범위를 본 다음 hunk 단위로 내려간다. Untracked file도 combined diff에 들어가므로 tracked change만 확인했다는 착각을 줄인다.

## Annotate AI Diff: feedback를 revision으로 연결하기

Diff line의 gutter에서 comment를 만들면 note가 source line에 고정된다. Review를 마친 뒤 **Send to agent**를 누르면 unresolved note들을 하나의 line-anchored prompt로 묶어 현재 worktree의 agent에게 보낸다. Agent가 수정한 뒤에도 comment가 남으므로 해결 여부를 같은 자리에서 재검토하고 resolve할 수 있다.

Batch feedback가 중요한 이유는 comment마다 agent가 방향을 바꾸는 현상을 피하기 위해서다. 실전에서는 다음 순서가 안정적이다.

1. Scope와 file list를 먼저 훑는다.
2. Correctness, security, test, maintainability 문제를 line note로 남긴다.
3. 한 번에 agent에게 보내 하나의 revision pass를 요청한다.
4. Diff를 새로 읽고 note마다 해결 여부를 확인한다.
5. 모두 해결되기 전에 commit panel로 넘어가지 않는다.

Attribution은 Orca가 관찰한 agent edit range를 gutter에 표시해 human change와 AI-originated line을 구분한다. Human이 AI code를 다시 편집하면 attribution이 human으로 바뀐다. 이 정보는 local metadata이며 Git commit에 자동으로 들어가지 않는다. Persistent audit가 필요하면 diff toolbar에서 metadata를 export해야 한다.

## Commit, push, hosted review

Diff에서 file이나 hunk를 stage하고 Source Control panel에서 commit message를 작성한다. Repository의 pre-commit hook은 그대로 실행되며 실패 output은 inline으로 나타난다. **Fix with AI**는 실패 정보와 staged file을 agent에게 전달해 수정하도록 하지만 hook bypass, commit, push까지 자동 승인하는 기능은 아니다.

Plain **Push**는 upstream을 설정해 normal push하며 remote보다 뒤처진 branch를 몰래 force-push하지 않는다. History rewrite 뒤에는 별도의 **Force push with lease** action이 나타나고 `--force-with-lease` 의미로 동작한다. 즉 destructive remote rewrite는 normal Push와 분리된 명시적 단계다.

Branch를 push한 뒤 create-review dialog에서 base, title, description, draft를 검토한다. AI-generated commit message와 review detail은 action recipe로 구성되며 global default와 repository override를 나눌 수 있다. Template variable에 linked issue나 patch context를 넣을 수 있지만 값이 없는 경우를 고려해 prompt를 작성해야 한다.

## Hosted provider와 task source

GitHub가 review·Actions·issue integration이 가장 깊고, GitLab·Bitbucket·Azure DevOps·Gitea도 지원 범위 안에서 review state와 check를 보여 준다. GitHub pull request에서는 comment thread, failing check handoff, auto-merge 또는 merge queue, registered stack 정보를 Orca 안에서 다룰 수 있다.

Linear와 Jira는 task source로 worktree에 연결된다. Issue에서 workspace를 만들면 이름과 issue link가 채워지고, review context가 작업에 붙는다. Linear가 suggested branch name을 제공하면 worktree branch에 반영할 수 있다. Jira는 cloud와 self-hosted 연결을 지원하며 issue transition, field, comment를 task drawer에서 다룬다.

핵심은 issue tracker와 Git review를 같은 것으로 보지 않는 것이다. Linear·Jira issue는 worktree의 작업 목적을 제공하고, hosted PR/MR은 변경을 검토하고 합치는 절차를 제공한다. Orca workspace가 두 context를 이어 준다.

## Monaco editor의 역할

Code file은 Monaco에서 열리며 blur 또는 짧은 idle 뒤 자동 저장된다. Editor tab에서 **Changes view mode**로 전환하면 cursor 위치를 유지한 채 HEAD와 working tree의 in-tab diff를 보고 hunk 이동·stage를 할 수 있다.

Orca는 editor-first이지 language-server 중심의 완전한 IDE를 지향하지 않는다. Syntax highlighting과 기본 navigation은 제공하지만 type checker와 linter는 terminal pane에서 실행하는 흐름을 전제로 한다. 따라서 agent가 수정한 직후 editor에서 빠르게 확인하고, 실제 correctness gate는 repository command로 검증하는 편이 맞다.

## Markdown과 문서 artifact

Markdown은 rich editor가 기본이며 slash menu, wiki-style internal link autocomplete, rendered-text search, review annotation, front matter 표시, table editing, heading outline을 지원한다. 필요하면 raw Monaco view로 전환한다. Toggle block은 portable `<details>`와 `<summary>` 형태로 저장되므로 외부 renderer에서도 유지된다.

Public artifact sharing은 명시적 opt-in 뒤 local Markdown이나 self-contained HTML을 view link로 게시한다. Relative HTML asset은 함께 upload되지 않으므로 공유할 HTML은 self-contained 또는 absolute asset URL이어야 한다. Public publishing은 편리하지만 source code나 secret이 포함되지 않았는지 별도로 검토해야 한다.

Built-in viewer는 Mermaid, PDF, image, CSV·TSV, Jupyter notebook을 repository context 안에서 보여 준다. Notebook editing은 beta이며 execution과 rich output은 안정화 중이므로 notebook correctness는 외부 runtime 검증과 병행한다.

## File explorer와 agent change

File explorer는 disk change를 실시간으로 반영하므로 agent가 만든 file도 즉시 나타난다. Git status color, stage·discard·rename·path copy를 제공한다. External drag-drop은 local file copy, Markdown image 삽입, agent terminal에 path 전달로 맥락에 따라 동작한다.

SSH worktree에서는 drag한 file을 remote host에 upload한 뒤 agent에게 실제 remote path를 전달한다. Remote file·folder download는 connection capability와 desktop client 여부에 따라 제한된다. 이 경계를 모르고 local path를 remote agent에 붙여 넣으면 존재하지 않는 path를 주게 되므로, remote file operation은 explorer action을 우선한다.

## Per-worktree browser

각 worktree는 독립 browser tab과 session을 가진다. 구현 중인 application, scroll position, local dev URL이 다른 task와 섞이지 않는다. Chromium 기반 browser에는 address·history·reload·devtools, download shelf, viewport emulation이 있으며 CLI automation도 같은 tab을 제어한다.

Link routing은 Orca browser 또는 system browser 중 기본을 정하고 modifier click으로 일회성 반전을 할 수 있다. 다만 remote·SSH source의 link는 built-in browser로 열리지 않는다는 제한이 있다.

### Browser profile

Profile은 cookie, local storage, cache를 별도 storage partition에 격리한다. 서로 다른 login role, session-specific bug, custom user-agent, viewport를 재현할 때 쓴다. Agent-driven browser command도 active profile을 상속하므로, automation 전에 어떤 identity가 선택되어 있는지 확인해야 한다.

### Design Mode

Design Mode에서 rendered element를 클릭하면 해당 element의 HTML, 주변 DOM, computed CSS, cropped screenshot, 가능한 경우 source map의 file·line을 active agent에 attachment로 보낸다. Agent가 source를 바꾸고 hot reload된 결과를 다시 클릭해 검증한다.

UI bug를 “버튼이 이상하다”는 자연어만으로 전달하는 대신 실제 DOM·style·pixel context를 함께 주므로 ambiguity가 줄어든다. Source map이 없는 production build에서는 file·line 연결이 약해질 수 있으므로 local dev mode에서 쓰는 편이 좋다.

## 권장 end-to-end loop

1. Task source에서 worktree를 만들고 agent에게 변경을 맡긴다.
2. Browser나 test terminal에서 현상을 재현한다.
3. UI 문제라면 Design Mode로 정확한 element context를 보낸다.
4. Combined diff에서 전체 scope를 확인하고 line note를 모은다.
5. Note batch를 agent에게 보내 revision을 받는다.
6. Test·lint를 terminal에서 실행하고 diff를 다시 읽는다.
7. Hunk 단위로 stage하고 hook이 통과한 commit만 만든다.
8. Push 뒤 base·description·draft state를 확인해 hosted review를 생성한다.

## See Also

- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
- [Orca 에이전트와 세션 운영](orca-agent-runtime-and-session-management.md)
- [Orca 원격 실행과 모바일 운영](orca-remote-and-mobile-runtime.md)
