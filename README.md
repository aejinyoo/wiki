# design-wiki

AI 디자인 개인 위키. 모바일에서 공유한 링크가 자동으로 정리·분류되어 쌓이는 아카이브.

- **소유자**: 애진 (개인용, 회사 계정 미사용)
- **기획서**: [`design-wiki-agent-plan.md`](./design-wiki-agent-plan.md)

## 구조

```
design-wiki/
├─ README.md                   ← 이 파일
├─ design-wiki-agent-plan.md   ← 기획서 v4
├─ _index.json                 ← 전체 인덱스 (자동 생성)
├─ _stats.json                 ← 카테고리·태그 분포 (자동 생성)
├─ _meta.yaml                  ← 설정 (protected 카테고리 등)
├─ _changelog/                 ← Curator 변경 로그
├─ raw/                        ← 수집 원본 (에이전트 내부)
├─ wiki/                       ← 정식 아카이브
│  ├─ ai-ux-patterns/
│  ├─ prompt-ui/
│  ├─ agent-interaction/
│  ├─ generative-tools/
│  ├─ design-system-automation/
│  └─ trend-reports/
└─ daily/                      ← 일일 브리프
```

## 파이프라인

1. **Ingester** (Python, 5분마다) — iCloud `inbox.md` → `raw/` 저장
2. **Classifier** (Haiku, 15분마다) — `raw/` → `wiki/{category}/*.md`
3. **Curator** (Sonnet, 주 1회 일요일 23:00) — 태그 정규화·카테고리 재정비
4. **Daily Brief** (Sonnet, 매일 08:00) — `daily/YYYY-MM-DD.md` 생성

## 수동 작업

```bash
# 인덱스 전체 재생성 (LLM 미호출, 복구용)
uv run agents/rebuild_index.py
```

## 동기화

- Git 원본: 이 repo (로컬 소스 오브 트루스 + GitHub private 주1 백업)
- iCloud 미러: `~/Library/Mobile Documents/.../DesignWikiSync/` (rsync, `.git` 제외)
- 모바일 뷰어: Obsidian vault는 iCloud 미러를 가리킴
