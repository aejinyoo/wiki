# AI 디자인 개인 위키·브리핑 에이전트 — 기획서 v4

*작성일: 2026-04-19 · 대상: 애진 개인용*

---

## 0. 한 문장 요약

> **모바일에서 공유한 링크를 내가 자고 있는 동안 개인 위키(Git repo)에 정리해두고, 아침에 위젯으로 "오늘 읽어볼 3개 + 해볼 실험 3개"를 받는다. 회사 계정은 한 번도 쓰지 않는다.**

---

## 1. 배경과 동기

- AI 디자인 트렌드 변화 속도 > 수동 팔로업 가능 속도
- 현재 북마크들은 X / YouTube / Threads / Instagram에 파편화
- 원하는 경험: 모바일에서 "공유만" → 나머지는 자동
- 매일 아침 **개인화된 브리핑 + 실행 아이템 추천**을 받고 싶음
- 장기적으로는 개인 아카이브가 쌓이면서 나의 관심사를 학습하는 위키가 되기를

## 2. 핵심 제약

| 제약            | 의미                                                |
| ------------- | ------------------------------------------------- |
| **회사 계정 미사용** | Notion·Slack·Gmail 회사 워크스페이스 0회 호출                |
| 기기            | iPhone + MacBook + iPad (개인 Apple ID로 iCloud 연동됨) |
| 개인 사용         | 협업·공유·승인 플로우 불필요, 내가 알 수 있으면 됨                    |
| 토큰 예산         | 최대한 적게쓰면 좋음                                       |
| 시간            | 초기 구축은 짧게, 매일의 사용 마찰은 0에 가깝게                      |

---

## 3. 전체 아키텍처

```
[수집 레이어 — iPhone]
  Safari/앱 공유시트
      │  (iOS Shortcut = "내 플러그인")
      ▼
  iCloud Drive/DesignWikiSync/inbox.md  ← 단 하나의 수신 파일

[처리 레이어 — MacBook, launchd + 개인 API 키]
  Ingester (코드)
      │  (inbox.md 파싱 → raw/ 저장, inbox 비움)
      ▼
  Classifier (Haiku + 개인화 컨텍스트)
      │  (요약·카테고리·태그)
      ▼
  wiki/{category}/*.md  (Git 커밋)
      │
      ├─► Curator (주 1회, Sonnet)
      │     · 태그 정규화, 중복 병합, 카테고리 분할/병합
      │     · Git 커밋으로 이력, 필요시 git revert
      │
      └─► Daily Brief (매일 아침 08:00, Sonnet)
            · 어제 수집분 + 웹검색 트렌드 → 브리프 생성
            · daily/YYYY-MM-DD.md 저장

[저장 레이어]
  ~/dev/design-wiki/        (Git repo, 소스 오브 트루스)
      │  (주 1회 push)
      ▼
  GitHub private repo (개인 계정)

  ~/iCloud/DesignWikiSync/  (읽기 전용 미러)
      · wiki/, daily/, _index.json만
      · rsync로 갱신 (.git 제외)
      │  (iCloud가 자동 sync)
      ▼
  iPhone/iPad

[뷰어·위젯 레이어]
  Obsidian (맥·아이폰·아이패드 동일 vault 열기)
      · 데일리 브리프 열람
      · 체크리스트 토글
      · 카테고리 탐색

  Scriptable 위젯 (홈스크린)
      · daily/today.md 읽어 카드 렌더
      · 실험 3개 체크박스 + 원본 링크
```

---

## 4. 폴더·파일 스키마

### 4.1 소스 저장소 (`~/dev/design-wiki/`)

```
design-wiki/
├─ .git/
├─ README.md
├─ _index.json                # 전체 인덱스 (자동 생성)
├─ _stats.json                # 카테고리·태그 분포
├─ _meta.yaml                 # 설정 (protected 카테고리 등)
├─ _changelog/                # Curator 변경 로그
│  └─ 2026-04-21.md
├─ raw/                       # 수집 원본 (에이전트 내부)
│  └─ 2026-04-19/
│     └─ x_1791823.json
├─ wiki/                      # 정식 아카이브
│  ├─ ai-ux-patterns/
│  ├─ prompt-ui/
│  ├─ agent-interaction/
│  ├─ generative-tools/
│  ├─ design-system-automation/
│  └─ trend-reports/
└─ daily/                     # 일일 브리프
   └─ 2026-04-19.md
```

### 4.2 iCloud 미러 (`~/Library/Mobile Documents/.../DesignWikiSync/`)

```
DesignWikiSync/
├─ inbox.md          ← iPhone이 쓰는 단 하나의 파일 (append-only)
├─ daily/            ← 에이전트가 복사 (읽기 전용)
├─ wiki/             ← 에이전트가 복사 (읽기 전용)
└─ _index.json       ← Obsidian/Scriptable이 읽음
```

### 4.3 위키 아이템 frontmatter

```yaml
---
id: x_1791823
source: X | YouTube | Threads | Instagram | Manual
url: https://...
author: @elonmusk
captured_at: 2026-04-18T07:20Z
title: "..."
summary_3lines: |
  - ...
  - ...
  - ...
tags: [prompt-ui, streaming, chat-ux]
category: prompt-ui
confidence: 0.82
tried: false          # 실행 아이템으로 뽑혔고 내가 체크했는지
tried_at: null
---

본문/자막/노트
```

### 4.4 inbox.md 포맷 (iOS Shortcut이 쓰는 형식)

```
---
captured_at: 2026-04-19T21:15+09:00
source_hint: twitter
url: https://x.com/...
note: (선택) 내가 남긴 메모
---
```

append-only. Ingester가 읽고 비우거나 `inbox-archive/`로 이관.

---

## 5. 구성 요소별 상세

### 5.1 iOS Shortcut — "Add to Wiki Inbox"

- **트리거**: Safari/앱 공유시트
- **입력**: URL, (선택) 제목, (선택) 메모
- **동작**:
  1. 현재 시각 ISO 포맷
  2. URL 도메인 파싱으로 source_hint 추론 (twitter.com → "twitter" 등)
  3. 위 frontmatter 블록 생성
  4. `iCloud Drive/DesignWikiSync/inbox.md`에 append
  5. 햅틱 피드백 + "Added" 토스트

- **설정 난이도**: 중간 (Shortcut 앱 가이드 따라가면 ~30분)
- **교체 가능성**: 나중에 Telegram bot이나 Cloudflare webhook으로 바꿔도 inbox.md 포맷만 맞추면 됨

### 5.2 Ingester — 수집기 (LLM 0)

- **언어**: Python (uv로 격리된 스크립트)
- **주기**: launchd로 5분마다 실행 (아주 가벼움)
- **하는 일**:
  1. `inbox.md` 읽고 frontmatter 블록별로 파싱
  2. URL 정규화 + 중복 체크 (`_index.json`의 기존 URL 해시 대조)
  3. URL에 따라 본문/메타 추출:
     - 트위터: snscrape 또는 간단 HTML 스크래핑 (공식 API 유료라 우회)
     - YouTube: yt-dlp로 제목·자막 추출
     - 일반: readability + trafilatura로 본문 추출
  4. `raw/YYYY-MM-DD/<id>.json` 저장
  5. 처리한 블록만 `inbox.md`에서 제거
- **실패 시**: 원본은 `inbox-failed.md`로 이관, 에러 로그
- **토큰**: 0 (순수 코드)

### 5.3 Classifier — 분류·요약기 (Haiku)

- **주기**: launchd로 매 15분 (새 raw 있을 때만 실제 호출)
- **LLM 모델**: `claude-haiku-4-5-20251001`
- **입력(아이템당 ~2k 토큰)**:
  - 개인화 컨텍스트 카드 (아래 5.4에서 재생성, ~300토큰)
  - 원본 텍스트 (잘라서 최대 1,500토큰)
- **출력**: 3줄 요약, 카테고리, 태그 3–5개, confidence 0–1
- **저장**: `wiki/{category}/{slug}.md` + frontmatter
- **한도**: 하루 최대 30건, 초과분은 다음날로 큐잉
- **토큰 예산**: 하루 평균 Haiku 5k 이하

### 5.4 개인화 컨텍스트 카드 (비밀 무기)

파인튜닝 없이 Classifier의 출력을 내 스타일에 맞추는 장치.

- **재생성 주기**: 주 1회, Curator가 생성
- **위치**: `_personal_context.md` (Git 제외, 로컬만)
- **내용** (최대 300토큰):
  - 카테고리 분포 요약 ("prompt-ui 많음, agent-interaction 급증")
  - 카테고리별 대표 태그 3개
  - 최근 `tried: true` 아이템 5개 (취향 시그널)
  - 자주 쓰는 태그 상위 10개
  - 거부·재분류한 이력 요약

Classifier 호출 때마다 system prompt 앞단에 주입. 토큰 거의 공짜지만 일관성은 크게 상승.

### 5.5 Curator — 큐레이터 (주 1회 Sonnet)

- **주기**: 매주 일요일 23:00
- **조건**: 위키 아이템 ≥ 50건일 때만
- **LLM 모델**: `claude-sonnet-4-6`
- **하는 일**:
  1. 태그 오타/변형 정규화 (자동반영, 상한 없음)
  2. 명백한 중복 병합 (자동반영)
  3. 카테고리 내 재분류 (자동반영)
  4. 새 카테고리 신설 제안 (주 1건 상한, 자동반영)
  5. 카테고리 병합/분할/삭제 (주 2건 상한, 자동반영)
  6. `_personal_context.md` 재생성
- **가드레일**:
  - `_meta.yaml`의 `protected: [...]` 카테고리는 건드리지 않음
  - 한 변경의 영향 > 100건이면 자동반영 skip, 다음 브리프에 승인 요청
  - 같은 카테고리 2주 내 재변경 금지 (cooldown)
- **기록**: Git 커밋 메시지 + `_changelog/YYYY-MM-DD.md`
- **알림**: 다음 일일 브리프 상단 배너로 표시
- **롤백**: 필요 시 `git revert <sha>` (별도 스냅샷 인프라 없음)
- **토큰**: 1회 5–8k Sonnet

### 5.6 Daily Brief — 일일 브리프 (매일 아침 Sonnet)

- **주기**: 매일 08:00 (launchd)
- **LLM 모델**: `claude-sonnet-4-6`
- **입력**:
  - 어제 수집된 아이템 요약 최대 15개 (초과분은 제외)
  - 최근 24h 웹검색 결과 "AI design trends" 상위 5개
  - `_personal_context.md`
- **출력 템플릿** (`daily/YYYY-MM-DD.md`):

```markdown
# Daily Design Brief — 2026-04-19 (월)

## 🔥 오늘의 3줄
- ...
- ...
- ...

## 📌 어제의 아카이브 하이라이트
1. **[제목]** · 카테고리 · 출처
   - 요약 2줄
   - 왜 봐야 하나: ...

## 🌐 업계 트렌드 (웹)
- (3개, 각 2줄)

## 🧪 오늘 해볼 만한 실험 (Top 3)
| 아이템 | 난이도 | 예상시간 | 원본 링크 |
|---|---|---|---|
| ... | ⭐ | 30m | [link] |

## 🧭 이번 주 위키 변화 (Curator 있을 때만)
- "prompt-ui"가 33건으로 확장, "prompt-ui/streaming"으로 분할
```

- **토큰**: 1회 3–5k Sonnet
- **실패 시**: 전날 브리프 재표시 + 에러 로그

### 5.7 Search (on-demand, 선택)

- **언제**: 내가 Obsidian에서 키워드 검색으로 안 풀릴 때만
- **방법**: Mac 터미널에서 `wiki-search "agent memory ux"` 한 줄
- **하는 일**: frontmatter 태그 grep → 상위 N개 로드 → Sonnet에 요약·답변 요청
- **토큰**: 1회 1–3k Sonnet (거의 안 씀)

### 5.8 인덱스·검증 정책 (컴파일/린트는 별도 스텝 없음)

**원칙**: 별도 "빌드/린트" 커맨드를 두지 않는다. 각 에이전트가 자기 쓰기 시점에 인덱스를 점진 갱신하고, 최소한의 검증만 내장한다.

#### 인덱스 갱신 — incremental · 분산

| 주체 | 타이밍 | 인덱스 쓰기 범위 |
|---|---|---|
| Ingester | 새 링크 수집 시 | `_index.json`에 raw 항목 stub 추가 |
| Classifier | 아이템 분류 완료 시 | 해당 항목을 "분류 완료"로 업데이트 + 카테고리 카운트 +1 |
| Curator | 주 1회 | `_stats.json` 재계산 + `_personal_context.md` 재생성 |
| Daily Brief | 매일 08:00 | 읽기만, 쓰지 않음 |

→ 데일리 브리프는 **컴파일이 아니라 소비자**. 빌드 덩치가 없어 구조가 가볍게 유지됨.

#### 검증(validate) — 에이전트 내부에 내장

별도 lint 커맨드 없음. Classifier/Curator가 `.md` 쓰기 전에 내부적으로:

1. frontmatter YAML 파싱 가능한가 → 실패 시 형식 자동 수정
2. `id` 중복 없나 → 있으면 dedup (URL 해시 기준)
3. `category`가 실제 존재하는 폴더인가 → 없으면 폴더 생성
4. `tags`는 list 타입인가 → 아니면 list로 강제 캐스팅
5. 필수 필드(`id`, `url`, `source`, `captured_at`) 누락 체크

오류는 로그로만 남기고 진행을 막지 않음(개인 사용자의 마찰 0).

#### 수동 full rebuild (가끔만)

다음 상황에만 수동 실행:
- 처음 위키 세팅 직후
- `.md`를 대량 수동 편집한 뒤
- 인덱스가 틀어진 게 의심될 때

```bash
uv run agents/rebuild_index.py
```

하는 일: `wiki/**/*.md` 전수 스캔 → `_index.json`·`_stats.json` 통째 재생성. **LLM 미호출 → 공짜.**

#### 폴더 구조화 자동화의 두 계층

| 계층 | 주체 | 빈도 | 역할 |
|---|---|---|---|
| 즉시 배치 | Classifier | 아이템마다 | 새 아이템을 맞는 카테고리 폴더에 바로 넣음 |
| 구조 재정비 | Curator | 주 1회 | 카테고리 분할·병합·이름변경·중복정리 |

매일의 소소한 정리는 Classifier, 주간의 큰 정리는 Curator. 사용자 개입 0.

---

## 6. 실행 환경 — Option B 확정

**선택한 환경**: MacBook의 launchd + 개인 Anthropic API 키

### 6.1 왜 B인가
- Cowork(Claude 데스크톱 세션)도 회사 계정이라 종속 발생
- 완전한 개인 계정 분리 원칙 유지
- launchd로 진짜 "잠자는 동안 돌아가는" 자동화 구현

### 6.2 구성요소

```
~/dev/design-wiki-agent/
├─ .env                      # ANTHROPIC_API_KEY (chmod 600)
├─ agents/                   # 각 에이전트 Python 스크립트
│  ├─ ingester.py
│  ├─ classifier.py
│  ├─ curator.py
│  └─ daily_brief.py
├─ prompts/                  # 프롬프트 템플릿(.md)
│  ├─ classifier.md
│  ├─ curator.md
│  └─ daily_brief.md
├─ lib/                      # 공통 (API 클라이언트, 파일 IO 등)
├─ launchd/                  # plist 파일들
│  ├─ com.aejin.designwiki.ingester.plist      (5분마다)
│  ├─ com.aejin.designwiki.classifier.plist    (15분마다)
│  ├─ com.aejin.designwiki.curator.plist       (일요일 23:00)
│  └─ com.aejin.designwiki.daily-brief.plist   (매일 08:00)
└─ pyproject.toml
```

### 6.3 시크릿 관리

- `.env`에 `ANTHROPIC_API_KEY` (개인 Anthropic 콘솔에서 발급, 회사 키 아님)
- 권한: `chmod 600 .env`
- `.gitignore`에 `.env`, `_personal_context.md`, `.anthropic_cache/` 포함
- 맥 로그인 키체인 사용도 대안 (나중에 업그레이드 가능)

### 6.4 비용 가드레일

- `.env`에 `DAILY_TOKEN_CAP_SONNET=15000`, `DAILY_TOKEN_CAP_HAIKU=10000` 설정
- 각 에이전트가 호출 전 `_usage.json` 확인 → 초과 시 중단, 다음날 재개
- 월말 요약 스크립트로 Anthropic 청구 vs 로컬 카운터 대조

### 6.5 launchd 샘플 (Daily Brief)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plist version="1.0">
<dict>
  <key>Label</key><string>com.aejin.designwiki.daily-brief</string>
  <key>ProgramArguments</key>
  <array>
    <string>/Users/aejin/.local/bin/uv</string>
    <string>run</string>
    <string>/Users/aejin/dev/design-wiki-agent/agents/daily_brief.py</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict>
    <key>Hour</key><integer>8</integer>
    <key>Minute</key><integer>0</integer>
  </dict>
  <key>StandardOutPath</key>
  <string>/Users/aejin/dev/design-wiki-agent/logs/daily-brief.log</string>
  <key>StandardErrorPath</key>
  <string>/Users/aejin/dev/design-wiki-agent/logs/daily-brief.err</string>
</dict>
</plist>
```

### 6.6 맥이 꺼져 있을 때

- launchd는 맥 꺼지면 실행 못함 → 다음 기상 시 누락된 주기 자동 보충(`RunAtLoad` + `StartCalendarInterval`)
- Daily Brief는 "어제자"가 안 나왔으면 전날 것도 소급 생성하도록 스크립트 내부에 catch-up 로직 추가

---

## 7. 모바일 뷰어·위젯 설정

### 7.1 Obsidian (iPhone/iPad/Mac 공용)

- **Vault 위치**: `~/iCloud/DesignWikiSync/`
- **설정**: 모바일 Obsidian 앱 → "Open vault from your device" → iCloud Drive에서 선택
- **플러그인(선택)**:
  - Daily Notes (데일리 브리프 바로가기)
  - Dataview (카테고리·태그 기반 목록)
  - Tasks (체크리스트 상태 관리)

### 7.2 Scriptable 위젯 (홈스크린)

- 스크립트: `iCloud/DesignWikiSync/daily/` 중 최신 `.md` 읽기
- 첫 헤더 + "실험 Top 3" 섹션 파싱 → 카드 렌더
- 탭하면 Obsidian 해당 페이지로 딥링크
- 위젯 2개 운영:
  - **Medium**: 오늘의 3줄 + 실험 1개
  - **Large**: 3줄 + 하이라이트 3개 + 실험 3개

### 7.3 체크리스트 상호작용

- Obsidian 모바일에서 체크박스 토글 → `.md` 파일 내 `- [ ]` → `- [x]` 변경
- 다음 파이프라인 실행 시 `tried: true` frontmatter 반영 (Classifier/Curator가 감지)
- 이 시그널이 `_personal_context.md`에 반영 → 다음 브리프의 추천 품질 ↑

---

## 8. 개인화 전략 (프롬프트 레벨)

- **바탕**: 지도/비지도 학습 없이 프롬프트 컨텍스트만으로 달성
- **레이어**:
  1. **분포 레이어** — `_stats.json`에서 뽑은 카테고리/태그 빈도
  2. **취향 레이어** — `tried=true` 아이템의 공통 특성
  3. **거부 레이어** — 내가 재분류하거나 삭제한 이력 (있을 때)
- **재생성 타이밍**: Curator 주간 실행 시 + 위키 +20건 때마다 force refresh
- **검증**: 2주 뒤 Classifier 정확도(내가 재분류한 비율)로 품질 체크. 10% 미만 재분류면 합격.

---

## 9. 토큰·비용 예산

| 작업 | 빈도 | 1회 토큰 | 일/주/월 합계 |
|---|---|---|---|
| Ingester | 5분마다 | 0 | 0 |
| Classifier | 새 아이템당 | ~2k Haiku 입력 + ~500 출력 | 10개/일 기준 Haiku 25k/일 |
| Curator | 주 1회 | ~5k Sonnet + 3k 출력 | 주 8k Sonnet |
| Daily Brief | 매일 08:00 | ~3k Sonnet 입력 + 2k 출력 | 월 150k Sonnet |
| Search | 요청 시 | ~2k Sonnet | 가변 |

**예상 월 비용** (2026년 4월 기준 토큰 단가 적용 권장 — 실제 청구는 Anthropic 콘솔에서 확인):
- Haiku 월 ~750k tokens
- Sonnet 월 ~200k tokens
- 대략 몇 달러~십몇 달러 수준

### 하드 가드레일
- 하루 Sonnet 15k / Haiku 10k 초과 시 자동 일시정지
- 월말 마지막 날에 자동 리포트 Obsidian에 작성 (`daily/2026-04-30.md` 상단)

---

## 10. 현실적인 소스 연동 계획

| 소스                    | v1 전략                          |
| --------------------- | ------------------------------ |
| **수동 드롭 (가장 중요)**     | iOS Shortcut → inbox.md        |
| **X 북마크**             | Share 버튼 누르면 IOS Shortcut으로 표시 |
| **YouTube 좋아요/저장**    | Share 버튼 누르면 IOS Shortcut으로 표시 |
| **Threads/Instagram** | Share 버튼 누르면 IOS Shortcut으로 표시 |
| **Safari 탭/읽기목록**     | Shortcut으로 선택 탭만 shared        |

v1에선 **수동 드롭 하나로 시작 → 충분히 돌아가는지 2주 확인 → 나머지 소스 붙이기**가 안전.

---

## 11. 로드맵 (개인 일정 기준, 총 4주)

### W1 — 기반
- [ ] `https://github.com/aejinyoo/wiki` Git repo 생성 + GitHub private repo 연결
- [ ] `~/iCloud/DesignWikiSync/` 생성 + Obsidian vault 연결
- [ ] Anthropic 개인 API 키 발급, `.env` 세팅
- [ ] Python 프로젝트 스캐폴딩 (`uv init`, `anthropic` SDK)
- [ ] iOS Shortcut "Add to Wiki Inbox" 만들기 + 아이콘 홈화면 고정

### W2 — 파이프라인 v1
- [ ] Ingester 스크립트 + launchd 5분 주기
- [ ] Classifier 스크립트 + 개인화 컨텍스트 카드 초안
- [ ] 카테고리 하드코딩 6개로 시작
- [ ] 2–3일 dogfooding, 로그 검토

### W3 — 브리프 & 위젯
- [ ] Daily Brief 스크립트 + 08:00 launchd
- [ ] rsync 미러 스크립트 (iCloud로 복사)
- [ ] Scriptable 위젯 스크립트 작성
- [ ] 위젯을 홈스크린에 배치

### W4 — 큐레이션 & 튜닝
- [ ] Curator 스크립트 + 일요일 23:00 launchd
- [ ] 개인화 컨텍스트 카드 자동 재생성
- [ ] YouTube 좋아요 연동 (YouTube Data API)
- [ ] 개인화 정확도 확인, 프롬프트 튜닝

---

## 12. 결정 이력

| #   | 결정                                                   | 이유                         | 일자         |
| --- | ---------------------------------------------------- | -------------------------- | ---------- |
| 1   | 에이전트 4개로 분할 (Ingester/Classifier/Curator/DailyBrief) | 토큰·복잡도 분리                  | 2026-04-19 |
| 2   | 저장소 = 로컬 Git + GitHub private 주1 백업                  | 버전 이력 + 클라우드 백업            | 2026-04-19 |
| 3   | Curator 자동반영 + Git 이력 + 브리프 배너 알림                    | 승인 플로우는 개인용 오버킬            | 2026-04-19 |
| 4   | 뷰어·위젯 = Obsidian + Scriptable                        | 회사 계정 미사용, 개인 Apple ID로 완결 | 2026-04-19 |
| 5   | 수집 채널 = iOS Shortcut → iCloud Drive inbox.md         | 전 기기 Apple 생태계, 시크릿 0      | 2026-04-19 |
| 6   | 실행 환경 = Option B (launchd + 개인 Anthropic API 키)      | Cowork도 회사 계정이라 종속 발생      | 2026-04-19 |
| 7   | v1 소스 = 수동 드롭만                                       | 먼저 파이프라인 돌아가는지 확인          | 2026-04-19 |
| 8   | Git repo와 iCloud는 물리적으로 분리 + rsync                   | Git 메타파일과 iCloud 동기화 충돌 회피 | 2026-04-19 |
| 9   | 별도 빌드/린트 스텝 없음, 각 에이전트가 incremental 인덱스 갱신           | 개인용이라 마찰 최소화, 구조 경량 유지     | 2026-04-19 |
| 10  | 검증은 에이전트 내부에 "validate & auto-fix"로 내장               | 별도 lint 실패로 파이프라인 멈추는 일 방지 | 2026-04-19 |
| 11  | 수동 `rebuild_index.py`만 복구용으로 보유 (LLM 미호출)            | 전수 재생성이 필요한 희귀 케이스 대응      | 2026-04-19 |

---

## 13. 미해결·추후 결정

- [ ] **브리프 추가 전달 채널**: 이메일 푸시도 받고 싶은지? (현재는 Obsidian + 위젯만)
- [ ] **체크리스트 완료 자동 분석**: `tried=true`가 쌓인 뒤 "너 이런 실험은 안 하더라" 패턴 감지 시 알림?
- [ ] **읽지 않은 누적분 처리**: 바빠서 3일 못 본 브리프의 자동 요약 제공?
- [ ] **이미지·스크린샷 수집**: 공유시트로 이미지도 함께 올리면 OCR로 태그 추출?
- [ ] **v3 자체 PWA**: Obsidian이 2–3달 써보고 부족하면 그때 결정

---

## 14. 작업 착수 전 체크리스트

이 기획서로 실제 제작을 시작하기 전에 확인할 것:

1. Anthropic **개인 계정** 콘솔에서 API 키 발급 (회사 계정 아닌지 더블체크)
2. 개인 GitHub 계정에서 private repo 빈 상태로 하나 생성 (`wiki`)
3. 맥에 Homebrew + `uv` + `yt-dlp` 설치 확인
4. Obsidian 데스크톱·iOS 앱 설치
5. Scriptable 앱 (무료) 설치
6. iCloud Drive가 "Desktop/Documents"가 아니더라도 정상 동기화 중인지 확인

---

## 부록 A — 왜 이 구조인가 (한 줄 정리)

- **Ingester가 코드**라서 → 토큰 0
- **Classifier가 Haiku**라서 → 저렴
- **Curator가 주 1회**라서 → 월 4–5회만 과금
- **Daily Brief가 템플릿 고정**이라서 → 출력 크기 예측 가능
- **Git이 버전관리**라서 → 스냅샷·롤백 인프라 불필요
- **Obsidian이 뷰어**라서 → 자체 앱 개발 0
- **Scriptable이 위젯**이라서 → iOS 개발 0
- **iCloud가 동기화**라서 → 서버 0
- **launchd가 스케줄러**라서 → cron 관리 0
- **개인 API 키**라서 → 회사 계정 종속 0

모든 계층에서 **"최소 추가 인프라"**를 선택해, 남는 에너지를 위키 품질과 개인화에 투자하는 설계입니다.
