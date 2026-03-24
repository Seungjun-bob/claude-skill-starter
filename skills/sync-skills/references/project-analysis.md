# 프로젝트 초기 분석 가이드

sync-skills의 초기 분석 모드(Step 0)에서 참조합니다.

## 분석 원칙

**규칙을 발견하라.** 파일 수를 세는 것이 아니라, 프로젝트 어디서든 **검증 가능한 규칙**을 찾는다. 규칙이 발견되면 verify-* 스킬을 만든다.

어떤 프로젝트든 규칙이 있다:
- 코드 프로젝트: import 규칙, 타입 규칙, 패턴 강제
- 문서 프로젝트: 구조 규칙, 용어 규칙, 필수 섹션
- 마케팅 프로젝트: 브랜드 가이드, 템플릿 준수, 금지 표현
- 데이터 프로젝트: 스키마 규칙, 네이밍 규칙, 검증 조건
- 인프라 프로젝트: 네이밍 규칙, 모듈 구조, 환경 분리
- 연구/학술 프로젝트: 인용 규칙, 필수 섹션, 참고문헌 일관성
- 영업/CRM 프로젝트: 데이터 파일 컬럼 규칙, 필수 필드, 형식 규칙

## Step 0-1: 프로젝트 구조 파악

먼저 프로젝트 규모를 확인합니다:

```bash
# 파일 수 확인
find . -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/vendor/*' -not -path '*/__pycache__/*' -not -path '*/dist/*' -not -path '*/build/*' -not -path '*/venv/*' -not -path '*/.venv/*' -not -path '*/.expo/*' -not -path '*/ios/*' -not -path '*/android/*' | wc -l
```

- **500개 이하:** 전체 파일 목록을 탐색합니다.
- **500개 이상:** 디렉토리 구조만 스캔하고, 각 디렉토리에서 파일을 5개씩 샘플링합니다.

```bash
# 디렉토리 구조
find . -maxdepth 3 -type d -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/vendor/*' -not -path '*/__pycache__/*' -not -path '*/dist/*' -not -path '*/build/*' -not -path '*/venv/*' -not -path '*/.venv/*' -not -path '*/.expo/*' -not -path '*/ios/*' -not -path '*/android/*'

# 파일 확장자 분포
find . -type f -not -path '*/node_modules/*' -not -path '*/.git/*' | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20

# 설정 파일
ls -la *.json *.toml *.yaml *.yml *.cfg *.ini Makefile Dockerfile 2>/dev/null
```

## Step 0-2: 프로젝트 유형 판별

설정 파일의 **내용을 읽고** 프로젝트 유형을 판별합니다. 여러 유형이 겹칠 수 있습니다.

패키지 매니저 파일이 있으면 의존성에서 주요 프레임워크를 식별합니다. 없으면 파일 확장자 분포와 디렉토리 구조로 판별합니다.

CLAUDE.md가 있으면 읽어서 추가 컨텍스트를 추출합니다.

## Step 0-3: 규칙 발견

### 규칙 소스 탐색 순서

다음 소스들을 **순서대로** 탐색하여 검증 가능한 규칙을 발견합니다:

#### 1순위: 명시적 규칙 문서

```bash
# CLAUDE.md (가장 풍부한 소스)
cat CLAUDE.md 2>/dev/null

# 가이드/규칙 문서
find . -maxdepth 2 -iname "*guide*" -o -iname "*rule*" -o -iname "*convention*" -o -iname "*standard*" -o -iname "CONTRIBUTING*" -o -iname "STYLE*" 2>/dev/null
```

이 문서들을 읽고 **검증 가능한 규칙**을 추출합니다:
- "~해야 한다" / "~하지 마라" 형태의 명시적 규칙
- 금지 표현, 필수 표현, 필수 구조
- 네이밍 규칙, 형식 규칙

#### 1.5순위: 기존 워크플로우 스킬

`.claude/skills/` 디렉토리에 `verify-*`가 아닌 워크플로우 스킬이 있으면 SKILL.md를 읽어 규칙을 추출합니다.

```bash
# verify-* 아닌 기존 스킬 탐색
find .claude/skills -name "SKILL.md" -not -path "*/verify-*/*" -not -path "*/sync-skills/*" -not -path "*/run-skills/*" 2>/dev/null
```

워크플로우 스킬에는 실행 절차뿐 아니라 **금지 패턴, 형식 규칙, 필수 구조** 등 검증 가능한 규칙이 포함되어 있을 수 있습니다. 특히 콘텐츠 생성 프로젝트에서는 워크플로우 스킬이 가장 풍부한 규칙 소스일 수 있습니다.

#### 2순위: 템플릿 파일

```bash
find . -maxdepth 3 -path "*/template*" -o -path "*/양식*" -o -path "*/boilerplate*" 2>/dev/null
```

**템플릿 파일의 구조 자체가 규칙입니다.** 템플릿을 읽고:
- 필수 섹션/헤딩이 무엇인지 파악
- 파일 구조가 어떻게 되어야 하는지 파악
- 이 구조를 따르는지 다른 파일들에서 검증 가능

#### 3순위: 설정 파일

```bash
# 린터/포매터
ls .eslintrc* .prettierrc* .pylintrc .flake8 .rubocop.yml rustfmt.toml .golangci.yml .markdownlint* .editorconfig 2>/dev/null

# 타입/빌드
ls tsconfig.json mypy.ini pyproject.toml 2>/dev/null
```

설정 파일에서 도구 기반 규칙을 추출합니다.

#### 4순위: 반복 패턴 발견

명시적 규칙 문서가 없는 경우, **파일을 샘플로 읽어 반복 패턴**을 발견합니다:

1. 각 주요 디렉토리에서 파일 3~5개를 샘플로 읽습니다
2. 파일 간 공통 구조(헤딩, import 순서, 함수 시그니처 등)를 찾습니다
3. 공통 구조가 있으면 그것이 암묵적 규칙입니다

#### 5순위: 사용자 질문

위 소스에서 규칙을 충분히 발견하지 못한 경우, `AskUserQuestion`으로 사용자에게 질문합니다. 프로젝트 유형에 맞는 질문을 선택합니다:

**코드 프로젝트:** "이 프로젝트에서 지켜야 할 규칙이나 컨벤션이 있나요? (예: 네이밍 규칙, 필수 구조, 금지 패턴 등)"

**문서/기획/마케팅 프로젝트:** "문서 작성 시 따라야 할 형식이나 규칙이 있나요? (예: 필수 섹션, 용어 규칙, 금지 표현, 템플릿 등)"

**데이터 프로젝트:** "데이터 파일에 지켜야 할 규칙이 있나요? (예: 컬럼명 규칙, 필수 필드, 데이터 형식 등)"

**판별 불가:** "이 프로젝트에서 일관되게 지켜야 할 규칙이나 형식이 있나요?"

### 규칙을 발견하는 질문

각 소스를 읽을 때 이 질문들로 규칙을 탐지합니다:

- **구조 규칙이 있나?** — 파일/폴더 네이밍, 디렉토리 구성, 필수 파일 존재
- **필수 패턴이 있나?** — 특정 도구, 형식, 구조를 반드시 사용해야 하는 규칙
- **금지 패턴이 있나?** — 특정 도구, 방식, 표현 사용 금지
- **일관성 규칙이 있나?** — 형식, 스타일, 용어 사용의 통일
- **완성도 규칙이 있나?** — 필수 섹션, 필수 필드, 빈 값 금지
- **참조 규칙이 있나?** — 링크, import, 인용이 유효한 대상을 가리켜야 함

## 범용 검증 카테고리

발견된 규칙을 다음 카테고리로 분류합니다. 어떤 프로젝트에서든 적용 가능합니다:

| 카테고리 | 설명 | 검증 방법 | 예시 |
|----------|------|----------|------|
| **구조 검증** | 파일/폴더 네이밍, 필수 파일 존재, 디렉토리 구성 | Glob, ls | 캠페인 폴더가 YYYY-QN/ 형태인지 |
| **일관성 검증** | 형식, 스타일, 용어, 패턴의 통일 | Grep, Read | 같은 개념을 다른 단어로 쓰지 않는지 |
| **완성도 검증** | 필수 섹션/필드 존재, 빈 값 없음 | Grep, Read | 각 문서에 "## 목적" 섹션이 있는지 |
| **참조 검증** | 링크, import, 인용이 유효한 대상을 가리킴 | Grep + 파일 존재 확인 | 마크다운 링크가 실제 파일을 가리키는지 |
| **규칙 준수** | 가이드/템플릿에 정의된 규칙 따르기 | Grep (금지/필수 패턴) | brand-guide의 금지 표현이 사용되지 않는지 |
| **출력물 검증** | 생성된 출력물의 구조, 필수 파일, 스키마 | Glob, Read, JSON 파싱 | 각 output 폴더에 필수 파일이 모두 있는지 |

## 스킬 생성 조건

**검증 가능한 규칙이 발견되면** verify-* 스킬 생성을 제안합니다.

"검증 가능"의 기준:
- Grep/Glob/Read/Bash 도구로 자동 검사할 수 있는 규칙
- PASS/FAIL 기준을 명확히 정의할 수 있는 규칙

**제안하지 않는 경우:**
- 의미적 판단이 필요한 규칙 ("글이 잘 읽히는가?", "디자인이 적절한가?")
- 도구로 자동 검사할 수 없는 규칙
- 이미 외부 도구가 완전히 커버하는 규칙 (ESLint, Prettier 등과 100% 겹침)

### 생성 지침과 검증 규칙의 구분

하나의 문서에 "어떻게 만들 것인가"(생성 지침)와 "결과물이 맞는지 어떻게 확인할 것인가"(검증 규칙)가 섞여 있을 수 있습니다. 특히 콘텐츠 생성 프로젝트의 워크플로우 스킬에서 자주 발생합니다.

**검증 규칙으로 추출하는 것:**
- 결과물에서 Grep/Glob/Read로 확인 가능한 규칙 (금지 표현, 필수 필드, 형식 패턴)
- PASS/FAIL 기준이 명확한 규칙 (예: "JSON에 verse_ref 필드가 있어야 한다")

**검증 규칙으로 추출하지 않는 것:**
- 생성 과정에서만 의미 있는 지침 (예: "담담한 독백체로 쓴다", "5개 안을 만들어라")
- 최종 결과물에서 확인할 수 없는 규칙 (예: "인과관계의 방향이 올바른지")

## 프로젝트 유형별 발견 예시

이 예시는 **발견을 돕는 힌트**입니다.

**코드 프로젝트:**
- CLAUDE.md의 Key Conventions → verify-conventions
- DB 모듈의 반복 패턴 (파라미터화 쿼리, 트랜잭션) → verify-db
- 테스트 파일의 공통 구조 → verify-tests

**문서/기획 프로젝트:**
- templates/ 파일의 구조 → verify-doc-structure (각 문서가 템플릿을 따르는지)
- 문서 간 링크 → verify-references (링크가 유효한지)
- 용어집이 있으면 → verify-terminology (용어 일관성)

**마케팅 프로젝트:**
- brand-guide.md의 규칙 → verify-brand (금지 표현, 필수 문구)
- templates/ 의 캠페인 구조 → verify-campaign-structure (필수 파일, 섹션)

**데이터/분석 프로젝트:**
- 스키마 정의 파일 → verify-schema (데이터 구조 준수)
- 노트북/스크립트의 반복 패턴 → verify-pipeline (입출력 구조)
- CSV/TSV 파일 → verify-data-format (컬럼명 규칙, 구분자, 필수 필드 존재)
- 노트북(.ipynb)은 JSON 구조이므로 Read 도구로 셀 내용을 읽어 검증합니다 (Grep은 셀 경계를 인식하지 못함)

**인프라 프로젝트:**
- 모듈 구조의 반복 패턴 → verify-modules (네이밍, 변수 구조)
- 환경별 설정의 일관성 → verify-environments (dev/prod 구조 동일)

**연구/학술 프로젝트 (LaTeX/BibTeX):**
- `\cite{}` 참조가 .bib 파일의 실제 키를 가리키는지 → verify-references
- 논문 구조의 필수 섹션 (\section{Introduction}, \section{Conclusion} 등) → verify-paper-structure
- BibTeX 키 네이밍 일관성 → verify-bibliography

**영업/CRM 프로젝트:**
- 데이터 파일(.csv, .tsv)의 컬럼명 규칙, 필수 필드 존재 → verify-data-schema
- 보고서/제안서 템플릿 준수 → verify-doc-structure
- 고객 데이터의 형식 규칙 (날짜, 전화번호, 이메일 등) → verify-data-format

**콘텐츠 생성/자동화 프로젝트:**
- 출력물 폴더 구조 → verify-output-structure (필수 파일 존재, 폴더 네이밍 규칙)
- 출력물 스키마 → verify-output-schema (JSON 필수 필드, 값 형식 규칙)
- 금지 패턴 → verify-content-rules (AI 생성 흔적, 금지 표현, 금지 문장 구조)
- 워크플로우 스킬(.claude/skills/)에서 규칙을 추출하여 결과물 검증
- 생성 지침(어떻게 쓸 것인가)과 검증 규칙(결과물이 맞는가)을 구분하여 검증 가능한 것만 추출
