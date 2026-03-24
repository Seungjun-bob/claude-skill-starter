---
name: run-skills
description: 작업을 완료한 후, PR이나 작업물을 제출하기 전, 또는 리뷰 시 모든 verify-* 스킬을 순차 실행하여 통합 검증 보고서를 생성합니다.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
argument-hint: "[선택사항: 특정 verify 스킬 이름]"
---

# 구현 검증

## 목적

프로젝트에 등록된 모든 `verify-*` 스킬을 순차적으로 실행하여 통합 검증을 수행합니다:

- 각 스킬의 Workflow에 정의된 검사를 실행
- 각 스킬의 Exceptions를 참조하여 false positive 방지
- 발견된 이슈에 대해 수정 방법을 제시
- 사용자 승인 후 수정 적용 및 재검증

## 실행 대상 스킬

이 스킬이 순차 실행하는 verify-* 스킬 목록입니다. `/sync-skills`가 스킬을 생성/삭제할 때 이 목록을 자동 업데이트합니다.

| # | 스킬 | 설명 |
|---|------|------|
<!-- 아직 등록된 verify-* 스킬이 없습니다. /sync-skills를 실행하여 프로젝트에 맞는 verify-* 스킬을 생성하세요. -->

## 참조 파일

| 파일 | 언제 사용 |
|------|----------|
| [assets/report-template.md](assets/report-template.md) | Step 3: 통합 보고서 형식 참조 |

## 워크플로우

### Step 1: 실행 대상 스킬 수집

**두 가지 방법을 병행**하여 실행 대상 스킬을 수집합니다:

1. **테이블 확인** — 위 "실행 대상 스킬" 테이블의 등록된 스킬 목록을 확인합니다
2. **동적 발견** — Glob 패턴으로 `.claude/skills/verify-*/SKILL.md`를 검색하여 실제 존재하는 verify-* 스킬을 발견합니다

두 결과를 합산(중복 제거)하여 최종 실행 대상을 결정합니다. 인수가 제공된 경우 해당 스킬만 필터링합니다.

**실행 대상 스킬이 0개인 경우:**

```
verify-* 스킬이 없습니다. /sync-skills를 실행하여 프로젝트에 맞는 verify-* 스킬을 생성하세요.
```

워크플로우를 종료합니다.

**실행 대상 스킬이 1개 이상인 경우:** 실행 대상 목록을 표시하고 검증을 시작합니다.

### Step 2: 순차 실행

각 verify-* 스킬에 대해:

1. 해당 스킬의 SKILL.md를 읽고 Workflow, Exceptions, Related Files를 파싱합니다
2. Workflow의 각 검사를 순서대로 실행합니다
3. Exceptions에 해당하는 패턴은 면제 처리합니다
4. FAIL인 경우 파일 경로, 문제, 수정 방법을 기록합니다
5. 스킬별 결과(검사 항목 수, 통과, 이슈, 면제)를 표시합니다

### Step 3: 통합 보고서

[assets/report-template.md](assets/report-template.md)의 형식으로 결과를 통합합니다.

### Step 4: 사용자 액션 확인

이슈 발견 시 `AskUserQuestion`으로 확인합니다:

1. **전체 수정** — 모든 권장 수정사항을 자동 적용
2. **개별 수정** — 각 수정사항을 하나씩 검토 후 적용
3. **건너뛰기** — 변경 없이 종료

### Step 5: 수정 적용

사용자 선택에 따라 수정을 적용하며 진행 상황을 표시합니다.

**참고:** 생성된 콘텐츠(이미지, 영상 등)의 이슈는 텍스트 수정만으로 해결되지 않을 수 있습니다. 이 경우 수정 방법 대신 재생성 안내를 제공합니다.

### Step 6: 수정 후 재검증

이슈가 있었던 스킬만 다시 실행하여 Before/After를 비교합니다. 잔여 이슈가 있으면 수동 해결을 안내합니다.

---

## 예외사항

1. **등록된 스킬이 없는 프로젝트** — 안내 메시지를 표시하고 종료
2. **스킬의 자체적 예외** — 각 verify 스킬의 Exceptions에 정의된 패턴은 이슈로 보고하지 않음
3. **run-skills 자체** — 실행 대상에 자기 자신을 포함하지 않음
4. **sync-skills** — `verify-`로 시작하지 않으므로 실행 대상에 포함되지 않음

## Related Files

| File | Purpose |
|------|---------|
| `.claude/skills/sync-skills/SKILL.md` | 스킬 유지보수 (실행 대상 스킬 목록을 관리) |
