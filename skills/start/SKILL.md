---
name: start
description: 새 작업의 토큰 사용량 추적을 시작합니다. 현재 세션을 기록하고 프로젝트+브랜치를 자동 바인딩합니다.
---

# usage:start

새 작업의 토큰 사용량 추적을 시작합니다.

## 트리거

- "usage start {label}"
- "usage tracking {label}"
- "작업 추적 시작 {label}"

## 파라미터

- **label** (필수): 사람이 읽을 수 있는 작업 이름 (예: "GPU Live 프로젝트 CRUD 정책서")
- **approach** (선택): 작업 방식 태그 (예: "manual", "workflow-runner", "autopilot"). 기본값: "manual"
- **tags** (선택): 분류 태그, 쉼표 구분 (예: "spec,planning")

## 절차

### 1. 현재 컨텍스트 감지

```bash
# 이 프로젝트의 가장 최근 JSONL에서 현재 세션 ID를 가져옵니다
PROJECT_PATH=$(pwd | sed "s|$HOME/||" | sed 's|/|-|g' | sed 's|^|-|')
SESSION_ID=$(ls -t ~/.claude/projects/${PROJECT_PATH}/*.jsonl 2>/dev/null | head -1 | xargs basename | sed 's/.jsonl//')

# 현재 git 브랜치를 가져옵니다 (git 프로젝트인 경우)
GIT_BRANCH=$(git branch --show-current 2>/dev/null || echo "none")
```

### 2. tasks.json 읽기 또는 생성

```bash
TRACKER_DIR="$HOME/.claude/usage-tracker"
mkdir -p "$TRACKER_DIR/reports"
cat "$TRACKER_DIR/tasks.json" 2>/dev/null || echo '{"tasks":{}}'
```

### 3. 작업 ID 생성

label로부터 짧고 URL 안전한 ID를 생성합니다:
- 소문자로 변환, 공백은 하이픈으로 치환
- 최대 30자
- 중복 시 `-2`, `-3` 등을 추가

### 4. 작업 항목 기록

tasks.json에 추가합니다:

```json
{
  "tasks": {
    "{taskId}": {
      "label": "{label}",
      "approach": "{approach}",
      "tags": ["{tag1}", "{tag2}"],
      "bindings": [
        {
          "project": "{PROJECT_PATH}",
          "branch": "{GIT_BRANCH}"
        }
      ],
      "sessions": [
        {
          "id": "{SESSION_ID}",
          "addedAt": "{ISO timestamp}",
          "auto": false
        }
      ],
      "startedAt": "{ISO timestamp}",
      "status": "active"
    }
  }
}
```

### 5. 결과 출력

출력 형식:
```
[Usage Tracker] 작업 추적을 시작합니다
- ID: {taskId}
- 작업명: {label}
- 방식: {approach}
- 세션: {SESSION_ID}
- 바인딩: {project} @ {branch}
```
