---
author: ''
captured_at: '2026-04-19T09:50:51Z'
category: generative-tools
confidence: 0.9
id: 4d1728102932
source: Manual
summary_3lines: '- Claude Code는 코드 작성, 버그 수정, 테스트 생성 등 개발 워크플로우를 자동화하는 AI 도구입니다.

  - 터미널, VS Code 등 다양한 환경에서 사용할 수 있으며, Git 연동 및 외부 도구와의 통합 기능을 제공합니다.

  - 개발자는 반복적인 작업을 줄이고 핵심 기능 개발에 집중하여 생산성을 높일 수 있습니다.'
tags:
- ai-coding
- developer-tools
- code-generation
- automation
- cli
title: 'Claude Code: 개발 생산성 향상을 위한 AI 도구'
tried: false
tried_at: null
url: https://code.claude.com/docs/en/overview
---

Get started
Choose your environment to get started. Most surfaces require a Claude subscription or Anthropic Console account. The Terminal CLI and VS Code also support third-party providers.- Terminal
- VS Code
- Desktop app
- Web
- JetBrains
The full-featured CLI for working with Claude Code directly in your terminal. Edit files, run commands, and manage your entire project from the command line.To install Claude Code, use one of the following methods:Then start Claude Code in any project:You’ll be prompted to log in on first use. That’s it! Continue with the Quickstart →
- Native Install (Recommended)
- Homebrew
- WinGet
macOS, Linux, WSL:Windows PowerShell:Windows CMD:If you see
The token '&&' is not a valid statement separator
, you’re in PowerShell, not CMD. If you see 'irm' is not recognized as an internal or external command
, you’re in CMD, not PowerShell. Your prompt shows PS C:\
when you’re in PowerShell and C:\
without the PS
when you’re in CMD.Native Windows setups require Git for Windows. Install it first if you don’t have it. WSL setups do not need it.Native installations automatically update in the background to keep you on the latest version.
What you can do
Here are some of the ways you can use Claude Code:Automate the work you keep putting off
Automate the work you keep putting off
Claude Code handles the tedious tasks that eat up your day: writing tests for untested code, fixing lint errors across a project, resolving merge conflicts, updating dependencies, and writing release notes.
Build features and fix bugs
Build features and fix bugs
Describe what you want in plain language. Claude Code plans the approach, writes the code across multiple files, and verifies it works.For bugs, paste an error message or describe the symptom. Claude Code traces the issue through your codebase, identifies the root cause, and implements a fix. See common workflows for more examples.
Create commits and pull requests
Create commits and pull requests
Claude Code works directly with git. It stages changes, writes commit messages, creates branches, and opens pull requests.In CI, you can automate code review and issue triage with GitHub Actions or GitLab CI/CD.
Connect your tools with MCP
Connect your tools with MCP
The Model Context Protocol (MCP) is an open standard for connecting AI tools to external data sources. With MCP, Claude Code can read your design docs in Google Drive, update tickets in Jira, pull data from Slack, or use your own custom tooling.
Customize with instructions, skills, and hooks
Customize with instructions, skills, and hooks
CLAUDE.md
is a markdown file you add to your project root that Claude Code reads at the start of every session. Use it to set coding standards, architecture decisions, preferred libraries, and review checklists. Claude also builds auto memory as it works, saving learnings like build commands and debugging insights across sessions without you writing anything.Create custom commands to package repeatable workflows your team can share, like /review-pr
or /deploy-staging
.Hooks let you run shell commands before or after Claude Code actions, like auto-formatting after every file edit or running lint before a commit.Run agent teams and build custom agents
Run agent teams and build custom agents
Spawn multiple Claude Code agents that work on different parts of a task simultaneously. A lead agent coordinates the work, assigns subtasks, and merges results.For fully custom workflows, the Agent SDK lets you build your own agents powered by Claude Code’s tools and capabilities, with full control over orchestration, tool access, and permissions.
Pipe, script, and automate with the CLI
Pipe, script, and automate with the CLI
Claude Code is composable and follows the Unix philosophy. Pipe logs into it, run it in CI, or chain it with other tools:See the CLI reference for the full set of commands and flags.
Schedule recurring tasks
Schedule recurring tasks
Run Claude on a schedule to automate work that repeats: morning PR reviews, overnight CI failure analysis, weekly dependency audits, or syncing docs after PRs merge.
- Routines run on Anthropic-managed infrastructure, so they keep running even when your computer is off. They can also trigger on API calls or GitHub events. Create them from the web, the Desktop app, or by running
/schedule
in the CLI. - Desktop scheduled tasks run on your machine, with direct access to your local files and tools
/loop
repeats a prompt within a CLI session for quick polling
Work from anywhere
Work from anywhere
Sessions aren’t tied to a single surface. Move work between environments as your context changes:
- Step away from your desk and keep working from your phone or any browser with Remote Control
- Message Dispatch a task from your phone and open the Desktop session it creates
- Kick off a long-running task on the web or iOS app, then pull it into your terminal with
claude --teleport
- Hand off a terminal session to the Desktop app with
/desktop
for visual diff review - Route tasks from team chat: mention
@Claude
in Slack with a bug report and get a pull request back
Use Claude Code everywhere
Each surface connects to the same underlying Claude Code engine, so your CLAUDE.md files, settings, and MCP servers work across all of them. Beyond the Terminal, VS Code, JetBrains, Desktop, and Web environments above, Claude Code integrates with CI/CD, chat, and browser workflows:Next steps
Once you’ve installed Claude Code, these guides help you go deeper.- Quickstart: walk through your first real task, from exploring a codebase to committing a fix
- Store instructions and memories: give Claude persistent instructions with CLAUDE.md files and auto memory
- Common workflows and best practices: patterns for getting the most out of Claude Code
- Settings: customize Claude Code for your workflow
- Troubleshooting: solutions for common issues
- code.claude.com: demos, pricing, and product details