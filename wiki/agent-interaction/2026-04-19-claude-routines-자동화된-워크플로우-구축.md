---
author: ''
captured_at: '2026-04-19T09:25:58Z'
category: agent-interaction
confidence: 0.9
id: a4cc7467e695
source: Manual
summary_3lines: '- Claude Routines는 예약, API 호출, GitHub 이벤트에 반응하여 작업을 자동화하는 기능입니다.

  - 반복적이고 예측 가능한 작업을 AI 에이전트에게 위임하여 개발 생산성을 높일 수 있습니다.

  - 백로그 관리, 알림 분류, 코드 검토 등 다양한 예시를 참고하여 자신만의 자동화 워크플로우를 설계해 보세요.'
tags:
- automation
- workflows
- scheduled-tasks
- api-triggers
- github-events
title: 'Claude Routines: 자동화된 워크플로우 구축'
tried: false
tried_at: null
url: https://code.claude.com/docs/en/routines
---

Routines are in research preview. Behavior, limits, and the API surface may change.
- Scheduled: run on a recurring cadence like hourly, nightly, or weekly
- API: trigger on demand by sending an HTTP POST to a per-routine endpoint with a bearer token
- GitHub: run automatically in response to repository events such as pull requests or releases
/schedule
.
This page covers creating a routine, configuring each trigger type, managing runs, and how usage limits apply.
Example use cases
Each example pairs a trigger type with the kind of work routines are suited to: unattended, repeatable, and tied to a clear outcome. Backlog maintenance. A schedule trigger runs every weeknight against your issue tracker via a connector. The routine reads issues opened since the last run, applies labels, assigns owners based on the area of code referenced, and posts a summary to Slack so the team starts the day with a groomed queue. Alert triage. Your monitoring tool calls the routine’s API endpoint when an error threshold is crossed, passing the alert body astext
. The routine pulls the stack trace, correlates it with recent commits in the repository, and opens a draft pull request with a proposed fix and a link back to the alert. On-call reviews the PR instead of starting from a blank terminal.
Bespoke code review. A GitHub trigger runs on pull_request.opened
. The routine applies your team’s own review checklist, leaves inline comments for security, performance, and style issues, and adds a summary comment so human reviewers can focus on design instead of mechanical checks.
Deploy verification. Your CD pipeline calls the routine’s API endpoint after each production deploy. The routine runs smoke checks against the new build, scans error logs for regressions, and posts a go or no-go to the release channel before the deploy window closes.
Docs drift. A schedule trigger runs weekly. The routine scans merged PRs since the last run, flags documentation that references changed APIs, and opens update PRs against the docs repository for an editor to review.
Library port. A GitHub trigger runs on pull_request.closed
filtered to merged PRs in one SDK repository. The routine ports the change to a parallel SDK in another language and opens a matching PR, keeping the two libraries in step without a human re-implementing each change.
The sections below walk through creating a routine and configuring each of these trigger types.
Create a routine
Create a routine from the web, the Desktop app, or the CLI. All three surfaces write to the same cloud account, so a routine you create in the CLI shows up at claude.ai/code/routines immediately. In the Desktop app, click New task and choose New remote task; choosing New local task instead creates a local Desktop scheduled task, which runs on your machine and is not a routine. The creation form sets up the routine’s prompt, repositories, environment, connectors, and triggers. Routines run autonomously as full Claude Code cloud sessions: there is no permission-mode picker and no approval prompts during a run. The session can run shell commands, use skills committed to the cloned repository, and call any connectors you include. What a routine can reach is determined by the repositories you select and their branch-push setting, the environment’s network access and variables, and the connectors you include. Scope each of those to what the routine actually needs. Routines belong to your individual claude.ai account. They are not shared with teammates, and they count against your account’s daily run allowance. Anything a routine does through your connected GitHub identity or connectors appears as you: commits and pull requests carry your GitHub user, and Slack messages, Linear tickets, or other connector actions use your linked accounts for those services.Create from the web
Open the creation form
Visit claude.ai/code/routines and click New routine.
Name the routine and write the prompt
Give the routine a descriptive name and write the prompt Claude runs each time. The prompt is the most important part: the routine runs autonomously, so the prompt must be self-contained and explicit about what to do and what success looks like.The prompt input includes a model selector. Claude uses the selected model on every run.
Select repositories
Add one or more GitHub repositories for Claude to work in. Each repository is cloned at the start of a run, starting from the default branch. Claude creates
claude/
-prefixed branches for its changes. To allow pushes to any branch, enable Allow unrestricted branch pushes for that repository.Select an environment
Pick a cloud environment for the routine. Environments control what the cloud session has access to:
- Network access: set the level of internet access available during each run
- Environment variables: provide API keys, tokens, or other secrets Claude can use
- Setup script: install dependencies and tools the routine needs. The result is cached, so the script doesn’t re-run on every session
Select a trigger
Under Select a trigger, choose how the routine starts. You can pick one trigger type or combine several.
- Schedule
- GitHub event
- API
Pick a preset frequency: hourly, daily, weekdays, or weekly. See Add a schedule trigger for timezone handling, stagger, and custom cron intervals.
Review connectors
All of your connected MCP connectors are included by default. Remove any that the routine doesn’t need. Connectors give Claude access to external services like Slack, Linear, or Google Drive during each run.
Create the routine
Click Create. The routine appears in the list and runs the next time one of its triggers matches. To start a run immediately, click Run now on the routine’s detail page.Each run creates a new session alongside your other sessions, where you can see what Claude did, review changes, and create a pull request.
Create from the CLI
Run/schedule
in any session to create a scheduled routine conversationally. You can also pass a descr