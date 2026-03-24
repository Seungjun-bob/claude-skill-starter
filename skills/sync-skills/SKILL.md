---
name: sync-skills
description: 프로젝트를 처음 설정할 때, 또는 새로운 패턴이나 규칙을 도입한 후 프로젝트를 분석하여 맞춤 verify-* 스킬을 자동으로 생성/업데이트합니다.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Edit, Write
argument-hint: "[선택사항: 특정 스킬 이름 또는 집중할 영역]"
---

# 프로젝트 맞춤 스킬 생성 & 유지보수

## 목적

프로젝트를 분석하여 맞춤 verify-* 스킬을 자동으로 생성하고, 프로젝트 변화에 맞춰 유지보수합니다.

**두 가지 모드:**
- **초기 분석 모드** — 등록된 verify-* 스킬이 없을 때. 프로젝트 전체를 스캔하여 구조, 도메인, 규칙을 파악하고 맞춤 verify-* 스킬을 생성합니다.
- **유지보수 모드** — 등록된 verify-* 스킬이 있을 때. 세션 변경사항을 분석하여 기존 verify-* 스킬의 드리프트를 탐지하고 수정합니다.

## 등록된 verify-* 스킬

| 스킬 | 설명 | 커버 파일 패턴 |
|------|------|---------------|
<!-- 아직 등록된 verify-* 스킬이 없습니다. /sync-skills를 실행하면 프로젝트를 분석하여 자동으로 생성합니다. -->

## 참조 파일

| 파일 | 언제 사용 |
|------|----------|
| [references/project-analysis.md](references/project-analysis.md) | 초기 분석 모드: 기술 스택 판별, 도메인 탐지 상세 |
| [references/update-guide.md](references/update-guide.md) | 유지보수 모드: 기존 스킬 업데이트 규칙 및 예시 |
| [references/quality-standards.md](references/quality-standards.md) | 양쪽 모두: 생성/업데이트된 스킬의 품질 검증 기준 |
| [assets/verify-skill-template.md](assets/verify-skill-template.md) | 양쪽 모두: 새 verify-* 스킬 생성 시 SKILL.md 템플릿 |

## 워크플로우

### 모드 결정

등록된 verify-* 스킬을 확인합니다. **두 가지 방법을 병행**합니다:

1. 이 파일의 등록 테이블 확인
2. Glob 패턴으로 `.claude/skills/verify-*/SKILL.md` 검색

두 결과를 합산(중복 제거)하여 판단합니다:

- **0개** → [초기 분석 모드](#초기-분석-모드-step-0)로 진입
- **1개 이상** → [유지보수 모드](#유지보수-모드-step-1-8)로 진입

---

### 초기 분석 모드 (Step 0)

프로젝트 전체를 스캔하여 기술 스택, 도메인, 규칙을 파악하고 맞춤 verify-* 스킬을 생성합니다.

상세 분석 방법은 [references/project-analysis.md](references/project-analysis.md)를 참조합니다.

#### Step 0-1: 프로젝트 구조 파악

[references/project-analysis.md](references/project-analysis.md)의 Step 0-1에 따라 프로젝트 규모를 확인하고, 디렉토리 구조, 파일 확장자 분포, 설정 파일을 스캔합니다.

- **500개 이하:** 전체 파일 목록을 탐색합니다.
- **500개 이상:** 디렉토리 구조만 스캔하고, 각 디렉토리에서 파일을 5개씩 샘플링합니다.

#### Step 0-2: 프로젝트 유형 판별

[references/project-analysis.md](references/project-analysis.md)의 Step 0-2에 따라 감지된 설정 파일을 읽고 프로젝트 유형을 분류합니다.

CLAUDE.md가 있으면 읽어서 추가 컨텍스트를 추출합니다.

**멀티 스택 감지:** 여러 기술 스택이 감지된 경우 (예: backend/ + frontend/, 또는 두 개의 패키지 매니저 파일) 각 스택별로 별도 verify-* 스킬을 생성하고, 스택 간 연결 규칙(API 계약, 환경변수 동기화 등)이 있으면 추가 스킬을 제안합니다.

#### Step 0-3: 규칙 발견

[references/project-analysis.md](references/project-analysis.md)의 Step 0-3에 따라 프로젝트 어디서든 **검증 가능한 규칙**을 찾습니다. 규칙 소스 탐색 순서, 구체적 명령어, 규칙 탐지 질문은 모두 해당 참조 파일에 정의되어 있습니다.

**스킬 생성 기준:** 검증 가능한 규칙이 발견되면 verify-* 스킬 생성 후보로 등록합니다. (파일 수가 아니라 규칙의 존재가 기준)

#### Step 0-4: 스킬 생성 제안

분석 결과를 종합하여 사용자에게 제안합니다:

```markdown
## 프로젝트 분석 결과

**프로젝트 유형:** <감지된 유형>

### 생성 제안 스킬

| # | 우선순위 | 스킬 | 이유 | 커버 영역 |
|---|---------|------|------|----------|
| 1 | 필수 | verify-<name> | <규칙 문서에서 명시된 규칙> | <파일 패턴> |
| 2 | 권장 | verify-<name> | <반복 패턴에서 발견된 규칙> | <파일 패턴> |
| 3 | 선택 | verify-<name> | <약한 패턴 또는 소수 파일> | <파일 패턴> |
```

**우선순위 기준:**
- **필수** — CLAUDE.md, 가이드 문서, 템플릿에서 명시적으로 정의된 규칙
- **권장** — 반복 패턴에서 발견된 암묵적 규칙
- **선택** — 약한 패턴이거나 해당 파일이 소수

`AskUserQuestion`으로 사용자가 선택/수정하도록 합니다.

→ 사용자 확인 후 **Step 6(새 스킬 생성)**으로 진행합니다.

---

### 유지보수 모드 (Step 1~8)

등록된 verify-* 스킬이 있을 때, 세션 변경사항을 기반으로 기존 verify-* 스킬을 유지보수합니다.

#### Step 1: 세션 변경사항 분석

**Case A — git 저장소이고 커밋 히스토리가 있는 경우:**

```bash
git diff HEAD --name-only
git diff main...HEAD --name-only 2>/dev/null
```

변경 파일을 디렉토리별로 그룹화하여 표시합니다.

**Case B — git은 있지만 커밋이 없는 경우:**

`AskUserQuestion`으로 "어떤 파일을 작업했나요?"를 질문하고, 답변에 해당하는 파일/디렉토리를 스캔합니다.

**Case C — git이 없는 경우:**

`AskUserQuestion`으로 "어떤 파일이나 폴더를 수정했나요? (예: campaigns/, reports/2024-Q1.md, data/customers.csv)"를 질문하고, 해당 디렉토리만 스캔하여 변경 파일 목록으로 사용합니다. 답변이 모호하면 초기 분석 모드로 전환합니다.

#### Step 2: 등록된 스킬과 변경 파일 매핑

등록 테이블의 각 스킬에 대해 SKILL.md를 읽고, 변경 파일과 매칭합니다.

#### Step 3: 영향받은 스킬의 커버리지 갭 분석

영향받은 스킬의 SKILL.md를 읽고 점검합니다: 누락 파일, 오래된 명령어, 새 패턴, 삭제된 참조, 변경된 값.

#### Step 4: CREATE vs UPDATE 결정

- 기존 스킬 도메인과 관련 → UPDATE
- 3개+ 관련 파일이 공통 패턴 공유 → CREATE
- 그 외 → 면제

`AskUserQuestion`으로 확인합니다.

#### Step 5: 기존 스킬 업데이트

[references/update-guide.md](references/update-guide.md)의 규칙에 따라 업데이트합니다.

---

### 공통 단계 (양쪽 모드)

#### Step 6: 새 스킬 생성

1. 관련 파일을 읽어 패턴을 깊이 이해합니다
2. `AskUserQuestion`으로 스킬 이름을 확인합니다 (`verify-` 접두사 + kebab-case)
3. [assets/verify-skill-template.md](assets/verify-skill-template.md)를 참조하여 **폴더 구조 전체를 생성**합니다:
   - `.claude/skills/verify-<name>/SKILL.md` — 검증 워크플로우
   - `.claude/skills/verify-<name>/references/` — 참조 문서 디렉토리 (API 문서, 코드 스니펫 등을 넣을 수 있음)
   - `.claude/skills/verify-<name>/assets/` — 템플릿 디렉토리 (코드 템플릿, 수정 가이드 등을 넣을 수 있음)
4. 분석 중 발견한 API 문서, 코드 패턴 등이 있으면 references/ 또는 assets/에 추가하고 SKILL.md에서 링크합니다
5. 연관 파일을 업데이트합니다:
   - 이 파일의 등록 테이블에 추가
   - `run-skills/SKILL.md`의 실행 대상 테이블에 추가
   - `CLAUDE.md`에 Skills 테이블이 있으면 추가 (없으면 건너뜀)
6. 사용자에게 references/, assets/ 활용법을 안내합니다

#### Step 7: 검증

[references/quality-standards.md](references/quality-standards.md)의 기준으로 검증합니다.

#### Step 8: 요약 보고서

최종 결과를 요약합니다.

---

## 예외사항

1. **Lock 파일/빌드 출력물** — 스킬 커버리지 불필요
2. **일회성 설정 변경** — 버전 범프, 린터 설정의 사소한 변경
3. **문서 파일** — README, CHANGELOG, LICENSE
4. **테스트 픽스처** — fixtures/, test-data/ 내 파일
5. **벤더/서드파티** — vendor/, node_modules/ 내 파일
6. **CI/CD 설정** — .github/, Dockerfile
7. **CLAUDE.md 자체** — 문서이며 코드 패턴이 아님
8. **에셋 파일** — 폰트, 이미지, 아카이브 (assets/fonts/, *.zip 등)
9. **가상환경/런타임** — venv/, .venv/, .expo/
10. **플랫폼 빌드** — ios/, android/ (네이티브 빌드 출력물)

## Related Files

| File | Purpose |
|------|---------|
| `.claude/skills/run-skills/SKILL.md` | 통합 검증 스킬 (실행 대상 목록을 관리) |
| `CLAUDE.md` | 프로젝트 지침 (있는 경우 Skills 섹션을 관리) |
