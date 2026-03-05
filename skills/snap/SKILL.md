---
name: snap
description: 진행 중인 작업의 토큰 사용량 스냅샷을 보여줍니다.
---

# usage:snap

현재(또는 지정한) 작업이 지금까지 소모한 토큰/비용을 보여줍니다.

## 트리거

- "usage snap"
- "usage check"
- "얼마나 썼나"
- "토큰 사용량"

## 파라미터

- **taskId** (선택): 특정 작업 ID. 생략하면 모든 진행 중인 작업을 보여줍니다.

## 절차

### 1. 작업 목록 로드

```bash
cat "$HOME/.claude/usage-tracker/tasks.json"
```

진행 중인 작업(status = "active")만 필터링합니다. taskId가 지정되어 있으면 해당 작업만 필터링합니다.

### 2. 각 작업의 세션별 ccusage 조회

각 작업에 대해 세션 데이터를 조회합니다:

```bash
ccusage session --since {task.startedAt as YYYYMMDD} --json --breakdown
```

JSON 출력을 파싱합니다. 작업의 sessions 배열에서 sessionId로 매칭합니다.

**매칭 로직:**
- ccusage 세션은 `sessionId` 필드를 가집니다 (프로젝트 경로 기반, 예: `-Users-nhn-git-project-aetherion-spec-guide`)
- 작업 세션은 `id` 필드를 가집니다 (UUID, 예: `ebedca79-...`)
- 매칭: 작업 세션의 `id`가 ccusage 세션의 프로젝트 경로 하위 JSONL 파일명으로 존재하는지 확인
- 간편 방식: 작업의 바인딩 프로젝트와 일치하는 모든 ccusage 세션을 집계

### 3. 지표 계산

각 작업별:
- **총 토큰**: 매칭된 세션의 totalTokens 합계
- **총 비용**: totalCost 합계
- **입력/출력/캐시 내역**
- **세션 수**: 세션 개수
- **기간**: startedAt부터 현재까지
- **일일 비용**: totalCost / 활성 일수

### 4. 결과 출력

```
[Usage Tracker] 스냅샷 - {label}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
| 항목              | 값           |
|-------------------|--------------|
| 총 비용           | ${cost}      |
| 총 토큰           | {tokens}     |
| 입력 토큰         | {input}      |
| 출력 토큰         | {output}     |
| 캐시 읽기         | {cacheRead}  |
| 캐시 생성         | {cacheCreate}|
| 세션 수           | {count}      |
| 기간              | {days}일     |
| 일일 비용         | ${perDay}    |
| 작업 방식         | {approach}   |
```

진행 중인 작업이 여러 개이면 비교 표로 보여줍니다.
