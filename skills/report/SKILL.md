---
name: report
description: 모든 추적 작업의 비용 비교 대시보드를 보여줍니다.
---

# usage:report

모든 추적된 작업의 비용을 비교하는 대시보드를 출력합니다.

## 트리거

- "usage report"
- "usage dashboard"
- "작업 비교"

## 파라미터

- **filter** (선택): "active", "completed", "all" (기본값: "all")
- **sort** (선택): "cost", "date", "efficiency" (기본값: "date")

## 절차

### 1. 전체 작업 로드

```bash
cat "$HOME/.claude/usage-tracker/tasks.json"
```

### 2. ccusage 조회

```bash
ccusage session --since {가장 이른 작업의 startedAt, YYYYMMDD 형식} --json --breakdown
```

각 작업의 sessions 배열에서 세션 ID를 매칭하여 해당 작업에 속하는 비용만 집계합니다.

**매칭 로직:**
- 작업의 bindings[].project와 ccusage의 sessionId를 매칭합니다
- 같은 프로젝트에 여러 작업이 있을 경우, 세션 시작 시간과 작업의 sessions[].id로 구분합니다

### 3. 비교 표 출력

```
[Usage Tracker] 대시보드

| 작업명 | 방식 | 비용 | 토큰 | 세션 | 일수 | 일일비용 | 상태 |
|--------|------|------|------|------|------|----------|------|
| {label} | {approach} | ${cost} | {tokens} | {count} | {days} | ${perDay} | {status} |

총 추적 비용: ${totalCost}
```

### 4. 방식별 비교 (여러 approach가 있을 때)

```
[방식별 비교]
| 방식 | 평균 비용/작업 | 평균 토큰 | 평균 세션 | 작업 수 |
|------|---------------|-----------|-----------|---------|
| manual   | ${avg} | {avg} | {avg} | {n} |
| autopilot| ${avg} | {avg} | {avg} | {n} |
```

이 비교가 플러그인의 핵심 가치입니다 — 워크플로우 변경이 실제로 효율적인지 데이터로 증명합니다.
