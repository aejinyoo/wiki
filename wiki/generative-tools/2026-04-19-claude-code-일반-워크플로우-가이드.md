---
author: ''
body_ko: '이 문서는 Claude Code를 활용하여 개발자의 일상적인 작업을 효율화하는 다양한 워크플로우를 소개합니다. 코드베이스 탐색,
  버그 수정, 리팩토링, 테스트 작성, Pull Request 생성 등 일반적인 개발 프로세스에 대한 단계별 가이드와 예제 프롬프트를 제공합니다.
  특히, 코드베이스를 읽기 전용으로 분석하고 계획을 수립하는 ''Plan Mode''는 복잡한 변경 계획이나 코드 검토에 유용하며, Shift+Tab
  단축키로 쉽게 전환할 수 있습니다. 또한, `--permission-mode plan` 플래그나 `-p` 옵션을 사용하여 Plan Mode에서
  직접 쿼리를 실행하는 방법도 설명합니다.


  테스트 작성 시에는 Claude가 프로젝트의 기존 패턴을 학습하여 일관성 있는 테스트 코드를 생성하며, 엣지 케이스까지 고려한 포괄적인 테스트
  커버리지를 확보하도록 요청할 수 있습니다. ''Extended Thinking'' 기능은 복잡한 문제에 대한 깊이 있는 추론을 가능하게 하여,
  아키텍처 결정이나 어려운 버그 해결에 도움을 줍니다. Opus 4.6 및 Sonnet 4.6 모델에서는 ''Adaptive Reasoning''을
  통해 추론 깊이를 동적으로 조절할 수 있습니다.


  여러 작업을 동시에 진행할 때 발생할 수 있는 변경 충돌을 방지하기 위해 ''Git Worktrees''를 활용하는 방법도 상세히 설명합니다.
  `--worktree` 플래그를 사용하면 각기 다른 작업 디렉토리와 브랜치를 가진 격리된 환경에서 Claude 세션을 실행할 수 있으며, 이는
  Subagent 작업에도 적용됩니다. 또한, `.worktreeinclude` 파일을 통해 추적되지 않는 환경 변수 파일 등을 자동으로 복사하도록
  설정할 수 있습니다.


  마지막으로, Claude의 작업 완료 또는 입력 필요 시 데스크톱 알림을 받을 수 있도록 ''Notification Hook''을 설정하는 방법을
  안내합니다. `settings.json` 파일에 hook을 추가하고 필요에 따라 `matcher`를 설정하여 특정 이벤트에 대한 알림만 받도록
  구성할 수 있습니다.'
captured_at: '2026-04-19T13:03:26Z'
category: generative-tools
confidence: 0.9
id: 9c8dfbf35131
key_takeaways:
- Plan Mode는 코드베이스 분석 및 계획 수립에 유용하며, Shift+Tab으로 전환 가능합니다.
- 테스트 작성 시 기존 패턴을 따르고 엣지 케이스를 요청하여 포괄적인 커버리지를 확보할 수 있습니다.
- Extended Thinking은 복잡한 문제 해결 및 의사 결정에 깊이 있는 추론을 제공합니다.
- Git Worktrees는 여러 작업을 동시에 충돌 없이 수행하기 위해 격리된 환경을 제공합니다.
- Notification Hook을 설정하여 Claude의 작업 완료 또는 입력 필요 시 데스크톱 알림을 받을 수 있습니다.
original_language: ko
source: Manual
summary_3lines: '- Claude Code의 다양한 개발 워크플로우를 안내하는 문서입니다.

  - 코드 탐색, 디버깅, 테스트 작성, PR 생성 등 실용적인 예제를 제공합니다.

  - Plan Mode, Worktrees, Extended Thinking 등 고급 기능을 활용하는 방법을 설명합니다.'
tags:
- claude-code
- developer-workflow
- plan-mode
- worktrees
- testing
title: Claude Code 일반 워크플로우 가이드
tried: false
tried_at: null
url: https://code.claude.com/docs/ko/common-workflows
what_to_try: Claude Code의 Plan Mode를 활성화하여 현재 작업 중인 코드베이스를 분석하고 개선 방안을 요청해보세요.
why_it_matters: 개발 생산성 향상을 위한 Claude Code의 다양한 활용법을 익힐 수 있습니다. 특히 Plan Mode와 Worktrees는
  복잡한 프로젝트 관리 및 협업에 큰 도움이 될 것입니다.
---

## 한국어 요지

이 문서는 Claude Code를 활용하여 개발자의 일상적인 작업을 효율화하는 다양한 워크플로우를 소개합니다. 코드베이스 탐색, 버그 수정, 리팩토링, 테스트 작성, Pull Request 생성 등 일반적인 개발 프로세스에 대한 단계별 가이드와 예제 프롬프트를 제공합니다. 특히, 코드베이스를 읽기 전용으로 분석하고 계획을 수립하는 'Plan Mode'는 복잡한 변경 계획이나 코드 검토에 유용하며, Shift+Tab 단축키로 쉽게 전환할 수 있습니다. 또한, `--permission-mode plan` 플래그나 `-p` 옵션을 사용하여 Plan Mode에서 직접 쿼리를 실행하는 방법도 설명합니다.

테스트 작성 시에는 Claude가 프로젝트의 기존 패턴을 학습하여 일관성 있는 테스트 코드를 생성하며, 엣지 케이스까지 고려한 포괄적인 테스트 커버리지를 확보하도록 요청할 수 있습니다. 'Extended Thinking' 기능은 복잡한 문제에 대한 깊이 있는 추론을 가능하게 하여, 아키텍처 결정이나 어려운 버그 해결에 도움을 줍니다. Opus 4.6 및 Sonnet 4.6 모델에서는 'Adaptive Reasoning'을 통해 추론 깊이를 동적으로 조절할 수 있습니다.

여러 작업을 동시에 진행할 때 발생할 수 있는 변경 충돌을 방지하기 위해 'Git Worktrees'를 활용하는 방법도 상세히 설명합니다. `--worktree` 플래그를 사용하면 각기 다른 작업 디렉토리와 브랜치를 가진 격리된 환경에서 Claude 세션을 실행할 수 있으며, 이는 Subagent 작업에도 적용됩니다. 또한, `.worktreeinclude` 파일을 통해 추적되지 않는 환경 변수 파일 등을 자동으로 복사하도록 설정할 수 있습니다.

마지막으로, Claude의 작업 완료 또는 입력 필요 시 데스크톱 알림을 받을 수 있도록 'Notification Hook'을 설정하는 방법을 안내합니다. `settings.json` 파일에 hook을 추가하고 필요에 따라 `matcher`를 설정하여 특정 이벤트에 대한 알림만 받도록 구성할 수 있습니다.

## 원문 발췌

Claude Code를 사용하여 코드베이스 탐색, 버그 수정, 리팩토링, 테스트 및 기타 일상적인 작업을 위한 단계별 가이드입니다.
이 페이지는 일상적인 개발을 위한 실용적인 워크플로우를 다룹니다: 낯선 코드 탐색, 디버깅, 리팩토링, 테스트 작성, PR 생성 및 세션 관리. 각 섹션에는 자신의 프로젝트에 맞게 조정할 수 있는 예제 프롬프트가 포함되어 있습니다. 더 높은 수준의 패턴과 팁은 모범 사례를 참조하십시오.
Plan Mode는 Claude에게 읽기 전용 작업으로 코드베이스를 분석하여 계획을 세우도록 지시하며, 코드베이스 탐색, 복잡한 변경 계획 또는 코드 안전한 검토에 완벽합니다. Plan Mode에서 Claude는 AskUserQuestion을 사용하여 계획을 제안하기 전에 요구사항을 수집하고 목표를 명확히 합니다.
세션 중에 Plan Mode 켜기Shift+Tab을 사용하여 세션 중에 Plan Mode로 전환할 수 있습니다.Normal Mode에 있으면 Shift+Tab은 먼저 Auto-Accept Mode로 전환되며, 터미널 하단에 ⏵⏵ accept edits on으로 표시됩니다. 그 다음 Shift+Tab은 Plan Mode로 전환되며, ⏸ plan mode on으로 표시됩니다.Plan Mode에서 새 세션 시작하기Plan Mode에서 새 세션을 시작하려면 --permission-mode plan 플래그를 사용합니다:
claude --permission-mode plan
Plan Mode에서 “헤드리스” 쿼리 실행하기-p를 사용하여 Plan Mode에서 직접 쿼리를 실행할 수도 있습니다 (즉, “헤드리스 모드”에서):
claude --permission-mode plan -p "Analyze the authentication system and suggest improvements"
find functions in NotificationsService.swift that are not covered by tests
2
테스트 스캐폴딩 생성
add tests for the notification service
3
의미 있는 테스트 케이스 추가
add test cases for edge conditions in the notification service
4
테스트 실행 및 검증
run the new tests and fix any failures
Claude는 프로젝트의 기존 패턴과 규칙을 따르는 테스트를 생성할 수 있습니다. 테스트를 요청할 때 검증하려는 동작에 대해 구체적으로 설명하십시오. Claude는 기존 테스트 파일을 검토하여 이미 사용 중인 스타일, 프레임워크 및 어설션 패턴을 일치시킵니다.포괄적인 적용 범위를 위해 Claude에게 놓쳤을 수 있는 엣지 케이스를 식별하도록 요청하십시오. Claude는 코드 경로를 분석하고 오류 조건, 경계값 및 쉽게 간과할 수 있는 예상치 못한 입력에 대한 테스트를 제안할 수 있습니다.
확장된 사고는 기본적으로 활성화되어 있으며, Claude가 복잡한 문제를 단계별로 추론할 수 있는 공간을 제공합니다. 이 추론은 Ctrl+O로 전환할 수 있는 자세한 모드에서 볼 수 있습니다.또한 Opus 4.6 및 Sonnet 4.6은 적응형 추론을 지원합니다: 고정된 사고 토큰 예산 대신 모델은 노력 수준 설정에 따라 동적으로 사고를 할당합니다. 확장된 사고와 적응형 추론은 함께 작동하여 Claude가 응답하기 전에 얼마나 깊이 있게 추론할지 제어할 수 있습니다.확장된 사고는 복잡한 아키텍처 결정, 어려운 버그, 다단계 구현 계획 및 다양한 접근 방식 간의 트레이드오프 평가에 특히 유용합니다.
“think”, “think hard”, “think more”와 같은 구문은 일반 프롬프트 지시로 해석되며 사고 토큰을 할당하지 않습니다.
확장된 사고는 Claude가 응답하기 전에 수행하는 내부 추론의 양을 제어합니다. 더 많은 사고는 솔루션을 탐색하고, 엣지 케이스를 분석하고, 실수를 자체 수정할 수 있는 더 많은 공간을 제공합니다.Opus 4.6 및 Sonnet 4.6의 경우, 사고는 적응형 추론을 사용합니다: 모델은 선택한 노력 수준에 따라 동적으로 사고 토큰을 할당합니다. 이것은 속도와 추론 깊이 간의 트레이드오프를 조정하는 권장 방법입니다.다른 모델의 경우, 사고는 출력 할당에서 최대 31,999개 토큰의 고정 예산을 사용합니다. MAX_THINKING_TOKENS 환경 변수로 이를 제한하거나 /config 또는 Option+T/Alt+T 토글을 통해 사고를 완전히 비활성화할 수 있습니다.Opus 4.6 및 Sonnet 4.6에서 적응형 추론이 사고 깊이를 제어하므로 MAX_THINKING_TOKENS는 0으로 설정하여 사고를 비활성화하거나 CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1이 이러한 모델을 고정 예산으로 되돌릴 때만 적용됩니다. 환경 변수를 참조하십시오.
사고 요약이 편집되더라도 사용된 모든 사고 토큰에 대해 요금이 청구됩니다. 대화형 모드에서 사고는 기본적으로 축소된 스텁으로 나타납니다. settings.json에서 showThinkingSummaries: true를 설정하여 전체 요약을 표시합니다.
활성 세션 내에서 /resume을 사용하여 다른 대화로 전환합니다.세션은 프로젝트 디렉토리별로 저장됩니다. /resume 선택기는 worktree를 포함한 동일한 git 저장소의 대화형 세션을 표시합니다. claude -p 또는 SDK 호출로 생성된 세션은 선택기에 나타나지 않지만 세션 ID를 claude --resume <session-id>에 직접 전달하여 재개할 수 있습니다.
여러 작업을 동시에 수행할 때 각 Claude 세션이 변경사항이 충돌하지 않도록 코드베이스의 자체 복사본을 가져야 합니다. Git worktree는 각각 자체 파일과 분기를 가지면서 동일한 저장소 기록 및 원격 연결을 공유하는 별도의 작업 디렉토리를 만들어 이를 해결합니다. 이는 한 worktree에서 기능을 작업하는 동안 Claude가 다른 worktree에서 버그를 수정할 수 있으며, 어느 세션도 다른 세션을 방해하지 않음을 의미합니다.--worktree (-w) 플래그를 사용하여 격리된 worktree를 만들고 Claude를 시작합니다. 전달하는 값은 worktree 디렉토리 이름과 분기 이름이 됩니다:
# "feature-auth"라는 worktree에서 Claude 시작# 새 분기를 사용하여 .claude/worktrees/feature-auth/ 생성claude --worktree feature-auth# 별도의 worktree에서 다른 세션 시작claude --worktree bugfix-123
이름을 생략하면 Claude가 자동으로 임의의 이름을 생성합니다:
# "bright-running-fox"와 같은 이름 자동 생성claude --worktree
Worktree는 <repo>/.claude/worktrees/<name>에 생성되고 기본 원격 분기에서 분기됩니다. worktree 분기는 worktree-<name>으로 이름이 지정됩니다.기본 분기는 Claude Code 플래그 또는 설정을 통해 구성할 수 없습니다. origin/HEAD는 복제할 때 Git이 설정한 로컬 .git 디렉토리에 저장된 참조입니다. 저장소의 기본 분기가 나중에 GitHub 또는 GitLab에서 변경되면 로컬 origin/HEAD는 이전 분기를 계속 가리키며, worktree는 거기에서 분기됩니다. 로컬 참조를 원격이 현재 기본값으로 간주하는 것과 다시 동기화하려면:
git remote set-head origin -a
이것은 로컬 .git 디렉토리만 업데이트하는 표준 Git 명령입니다. 원격 서버에서는 아무것도 변경되지 않습니다. worktree가 원격의 기본값이 아닌 특정 분기에서 기반하도록 하려면 git remote set-head origin your-branch-name으로 명시적으로 설정합니다.worktree 생성 방식을 완전히 제어하려면 호출당 다른 기반을 선택하는 것을 포함하여 WorktreeCreate hook을 구성합니다. hook은 Claude Code의 기본 git worktree 로직을 완전히 대체하므로 필요한 모든 ref에서 가져오고 분기할 수 있습니다.세션 중에 Claude에게 “work in a worktree” 또는 “start a worktree”를 요청할 수도 있으며, 자동으로 하나를 만듭니다.
Subagent도 worktree 격리를 사용하여 충돌 없이 병렬로 작업할 수 있습니다. Claude에게 “use worktrees for your agents”를 요청하거나 사용자 정의 subagent에서 에이전트의 frontmatter에 isolation: worktree를 추가하여 구성합니다. 각 subagent는 변경사항 없이 완료될 때 자동으로 정리되는 자체 worktree를 가져옵니다.
Git worktree는 새로운 체크아웃이므로 주 저장소의 .env 또는 .env.local과 같은 추적되지 않은 파일을 포함하지 않습니다. Claude가 worktree를 만들 때 이러한 파일을 자동으로 복사하려면 프로젝트 루트에 .worktreeinclude 파일을 추가합니다.파일은 .gitignore 구문을 사용하여 복사할 파일을 나열합니다. 패턴과 일치하고 gitignored인 파일만 복사되므로 추적된 파일은 절대 복제되지 않습니다.
.worktreeinclude
.env.env.localconfig/secrets.json
이것은 --worktree, subagent worktree 및 데스크톱 앱의 병렬 세션으로 생성된 worktree에 적용됩니다.
Worktree 격리는 기본적으로 git과 함께 작동합니다. SVN, Perforce 또는 Mercurial과 같은 다른 버전 제어 시스템의 경우 WorktreeCreate 및 WorktreeRemove hook을 구성하여 사용자 정의 worktree 생성 및 정리 로직을 제공합니다. 구성되면 이러한 hook은 --worktree를 사용할 때 기본 git 동작을 대체하므로 .worktreeinclude는 처리되지 않습니다. hook 스크립트 내에서 대신 로컬 구성 파일을 복사합니다.공유 작업 및 메시징을 사용한 병렬 세션의 자동 조정을 위해 에이전트 팀을 참조하십시오.
장기 실행 작업을 시작하고 다른 창으로 전환할 때 Claude가 완료되거나 입력이 필요할 때 알 수 있도록 데스크톱 알림을 설정할 수 있습니다. 이것은 Claude가 권한을 기다리거나, 유휴 상태이고 새 프롬프트를 기다리거나, 인증을 완료할 때마다 발생하는 Notificationhook 이벤트를 사용합니다.
1
설정에 hook 추가하기
~/.claude/settings.json을 열고 플랫폼의 기본 알림 명령을 호출하는 Notification hook을 추가합니다:
설정 파일에 이미 hooks 키가 있으면 덮어쓰지 말고 Notification 항목을 병합합니다. CLI에서 원하는 것을 설명하여 Claude에게 hook을 작성하도록 요청할 수도 있습니다.
2
선택적으로 matcher 좁히기
기본적으로 hook은 모든 알림 유형에 대해 발생합니다. 특정 이벤트에만 발생하도록 하려면 matcher 필드를 다음 값 중 하나로 설정합니다:
3
hook 검증하기
/hooks를 입력하고 Notification을 선택하여 hook이 나타나는지 확인합니다. 선택하면 실행될 명령이 표시됩니다. 종단 간 테스트하려면 Claude에게 권한이 필요한 명령을 실행하도록 요청하고 터미널에서 전환하거나, Claude에게 직접 알림을 트리거하도록 요청합니다.
Claude Code를 linter 또는 코드 검토자로 사용하고 싶다고 가정해봅시다.빌드 스크립트에 Claude 추가하기:
// package.json{ ... "scripts": { ... "lint:claude": "claude -p 'you are a linter. please look at the changes vs. main and report any issues related to typos. report the filename and line number on one line, and a description