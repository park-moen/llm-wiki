# IntelliJ macOS 핵심 단축키

> Sources: JetBrains, 2026-08-17; JetBrains, 2024-03-18; JetBrains, 2026-08-13; JetBrains, 2024-06-24
> Raw: [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md); [JetBrains macOS Keymap: Development and Git](../../raw/intellij/2024-03-18-predefined-macos-keymap-development-and-git.md); [JetBrains Auto Import](../../raw/intellij/2026-08-17-auto-import.md); [JetBrains Database Versioning](../../raw/intellij/2026-08-13-database-versioning.md); [JetBrains Optimize Imports](../../raw/intellij/2024-06-24-optimize-imports.md); [JetBrains Auto Import: Optimize Imports](../../raw/intellij/2026-08-17-auto-import-optimize-imports.md)
> Updated: 2026-09-22

## Overview

IntelliJ IDEA의 macOS 기본 Keymap에서 먼저 익힐 만한 shortcut은 탐색·편집·refactoring·Git 작업을 키보드에서 이어 주는 항목이다. `Shift` 두 번으로 여는 Search Everywhere는 file, action, class, symbol, setting, UI element, Git 항목을 한 진입점에서 이름으로 찾는다. 파일의 정확한 위치나 먼저 선택할 탐색 창을 몰라도 되지만, 검색할 이름이나 관련 검색어는 입력해야 한다. [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

## 가장 빠른 파일 열기

1. `Shift`를 두 번 눌러 Search Everywhere를 엽니다.
2. 예를 들어 `Edition.kt`처럼 알고 있는 file 이름이나 검색어를 입력합니다.
3. 결과를 선택하고 `Enter`를 눌러 엽니다.

처음 열면 recent file 목록이 보입니다. 같은 `Shift` 두 번을 다시 누르거나 `All Places`를 선택하면 project 밖 항목까지 결과 범위를 넓힐 수 있습니다. [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md)

## 검색 대상을 바꾸는 방법

Search Everywhere 창에서는 `Tab`으로 class, file, symbol, action 등의 검색 context를 전환합니다. macOS 기본 Keymap에서 file 또는 directory만 이름으로 찾고 싶으면 `⌘⇧O`, symbol은 `⌘⌥O`, action은 `⌘⇧A`를 사용합니다. [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

| 찾는 대상 | macOS 기본 shortcut | 용도 |
| --- | --- | --- |
| 통합 검색 | `Shift` 두 번 | file·action·class·symbol 등을 함께 검색 |
| File 또는 directory | `⌘⇧O` | 이름으로 file 또는 directory 검색 |
| Symbol | `⌘⌥O` | method·field·class·constant 등 code element 검색 |
| Action | `⌘⇧A` | menu에 없거나 shortcut이 없는 action도 이름으로 검색 |

## Setting과 plugin 찾기

Search Everywhere에서 `/`를 입력하면 setting group을 찾을 수 있습니다. `/plugins`를 입력하면 plugin을 검색하고 enable 또는 disable할 수 있습니다. macOS 기본 Keymap에서 Settings는 `⌘,`로 열 수 있습니다. [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

## Keymap이 다를 때

`Shift` 두 번이 동작하지 않거나 다른 shortcut으로 바꾸려면 `⌘,`로 Settings를 열고 Keymap에서 `Search Everywhere`를 검색합니다. 개인 Keymap을 선택했거나 shortcut을 바꿨다면 이 문서의 macOS 기본값보다 현재 Keymap에 표시되는 값을 우선합니다. [JetBrains Search Everywhere](../../raw/intellij/2026-08-17-search-everywhere.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

## 개발과 Git에서 자주 쓰는 shortcut

| 목적 | macOS 기본 shortcut | 동작 |
| --- | --- | --- |
| 최근 file 전환 | `⌘E` | Recent Files를 열어 최근 작업한 file로 이동 |
| 코드 format | `⌘⌥L` | Reformat Code 실행 |
| 선언부 또는 사용처 이동 | `⌘B` | Go to Declaration or Usages 실행 |
| 이름 변경 | `⇧F6` | Rename refactoring 실행 |
| import 제안 | `⌥⏎` | 누락된 class의 Import class 제안 확인 |
| import 전체 정리 | `⌃⌥O` | Optimize Imports 실행 |
| entity DDL 생성 | `⌥⏎` | class 이름에서 `Generate DDL` action 선택 |
| Git 창 열기 | `⌘9` | Version Control window 표시 |
| Commit 창 열기 | `⌘K` | Commit 실행 창 표시 |

`⌘9`은 Git이 연결된 project에서 변경 이력과 Git tool window를 여는 출발점으로 사용합니다. `⌘K`는 local commit을 준비하는 창을 엽니다. shortcut은 Keymap을 바꾸면 달라질 수 있으므로 동작하지 않을 때는 Settings의 Keymap에서 action 이름을 검색합니다. [JetBrains macOS Keymap: Development and Git](../../raw/intellij/2024-03-18-predefined-macos-keymap-development-and-git.md)

## 누락된 import 제안 적용

import하지 않은 class·static method·field를 사용하면 IntelliJ IDEA가 누락된 import를 추가하라는 tooltip을 표시할 수 있습니다. macOS 기본 Keymap에서는 해당 reference에 caret을 두고 `⌥⏎`를 눌러 Quick Fix와 Intention Actions를 엽니다. 후보가 하나면 제안을 수락하고, 여러 import 후보가 있으면 목록에서 원하는 `Import class`를 선택합니다. tooltip이 꺼져 있어도 unresolved reference의 빨간 전구 icon 또는 `⌥⏎`로 같은 목록을 열 수 있습니다. [JetBrains Auto Import](../../raw/intellij/2026-08-17-auto-import.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

## 사용하지 않는 import 정리

현재 file의 사용하지 않는 import를 한 번에 지우고 import 문을 정리하려면 `⌃⌥O`로 **Optimize Imports**를 실행합니다. 같은 기능은 **Code → Optimize Imports**에서도 실행할 수 있습니다. Project tool window에서 directory를 선택하면 해당 directory의 모든 file 또는 local 변경 file만 대상으로 실행할 수도 있습니다. [JetBrains Optimize Imports](../../raw/intellij/2024-06-24-optimize-imports.md); [JetBrains Auto Import: Optimize Imports](../../raw/intellij/2026-08-17-auto-import-optimize-imports.md)

사용하지 않는 import 하나만 제거하려면 흐리게 표시된 import 문에 caret을 두고 `⌥⏎`를 누른 뒤 **`Remove unused imports`**를 선택합니다. [JetBrains Auto Import: Optimize Imports](../../raw/intellij/2026-08-17-auto-import-optimize-imports.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)

## JPA entity의 DDL 생성

`Registration`처럼 DDL을 만들 JPA entity source를 editor에서 연 뒤 class 이름에 caret을 둡니다. `⌥⏎`로 context actions를 열고 **`Generate DDL`**을 선택하면 entity 하나의 DDL statement를 생성할 수 있습니다. 공식 action 이름은 `Generate DDL`이며, `Generate DDL by <Entity name>` 창에서 DB type과 저장 위치·형식(File, Scratch File, Clipboard, Database Console)을 정한 뒤 preview하고 저장합니다. macOS 기본 Keymap의 `⌥⏎`는 `Alt+Enter`에 해당합니다. [JetBrains Database Versioning](../../raw/intellij/2026-08-13-database-versioning.md); [JetBrains macOS Keymap](../../raw/intellij/2024-03-18-predefined-macos-keymap.md)
