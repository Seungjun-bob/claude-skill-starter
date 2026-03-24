# Claude Skill Starter

Claude Code로 작업하는 모든 프로젝트에서 사용할 수 있는 스킬 스타터 킷.

코드, 문서, 데이터 분석, 기획, 자동화 등 어떤 프로젝트든 — 프로젝트에 복사하면 `/sync-skills`가 프로젝트를 분석하여 맞춤 검증 스킬을 자동으로 생성합니다.

## 포함된 스킬

| 스킬 | 역할 |
|------|------|
| `/sync-skills` | 프로젝트의 변경사항을 분석하고, verify-* 검증 스킬을 자동으로 생성/업데이트 |
| `/run-skills` | 등록된 모든 verify-* 스킬을 순차 실행하여 통합 검증 보고서 생성 |

## 구조

```
skills/
├── sync-skills/
│   ├── SKILL.md                        # 핵심 워크플로우
│   ├── references/
│   │   ├── quality-standards.md        # 스킬 품질 기준
│   │   └── update-guide.md            # 기존 스킬 업데이트 규칙
│   └── assets/
│       └── verify-skill-template.md    # 새 스킬 생성 템플릿
└── run-skills/
    ├── SKILL.md                        # 검증 실행 워크플로우
    └── assets/
        └── report-template.md          # 보고서 출력 형식
```

## 설치

```bash
# 클론 후 복사
git clone https://github.com/your-username/claude-skill-starter.git
cp -r claude-skill-starter/skills/* your-project/.claude/skills/
```

## 사용법

### 1. 스킬 생성

프로젝트에서 작업한 후:

```
/sync-skills
```

sync-skills가 자동으로:
1. 세션에서 변경된 파일을 감지
2. 변경 패턴을 분석하여 필요한 검증 스킬을 제안
3. 사용자 확인 후 verify-* 스킬을 생성

### 2. 검증 실행

```
/run-skills
```

run-skills가 자동으로:
1. 등록된 모든 verify-* 스킬을 순차 실행
2. 통합 보고서를 생성
3. 이슈 발견 시 수정을 제안

## 동작 원리

```
/sync-skills 실행
    ↓
[처음] 프로젝트 전체 스캔 → 규칙 발견 → verify-* 스킬 생성
[이후] git diff로 변경 파일 감지 → 기존 스킬과 매핑 → 업데이트


/run-skills 실행
    ↓
등록 테이블 + Glob 동적 발견으로 verify-* 스킬 수집
    ↓
각 스킬의 Workflow 실행
    ↓
통합 보고서 생성
    ↓
이슈 발견 시 수정 제안
```

### 기존 스킬이 있는 프로젝트

이미 `.claude/skills/`에 verify-* 스킬이 있는 프로젝트에 설치하면, sync-skills가 기존 스킬을 인식하고 유지보수 모드로 진입합니다. 기존 스킬을 덮어쓰지 않습니다.

## 범위와 한계

**자동 검증 가능:**
- 구조 규칙 — 파일/폴더 네이밍, 필수 파일 존재, 디렉토리 구성
- 패턴 규칙 — 금지 표현, 필수 표현, import/참조 규칙
- 일관성 규칙 — 용어, 형식, 스타일의 통일
- 참조 규칙 — 링크/import/인용이 유효한 대상을 가리키는지

**자동 검증 불가:**
- 의미적 품질 — "글이 잘 읽히는가?", "설계가 적절한가?", "톤이 적절한가?"
- 생성 과정의 지침 — 콘텐츠 생성 프로젝트에서 "어떻게 쓸 것인가"는 검증 불가, "결과물이 규칙에 맞는가"만 검증 가능
- 비-텍스트 파일 — 이미지, PDF, 영상 내부 내용

## 요구사항

- [Claude Code](https://claude.ai/code) CLI
- 로컬 파일시스템 기반 프로젝트
- Git 저장소 (권장, 없어도 동작)

## 라이선스

MIT
