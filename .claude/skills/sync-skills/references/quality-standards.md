# 스킬 품질 기준

sync-skills의 Step 7에서 생성/업데이트된 스킬의 품질을 검증할 때 참조합니다.

## 필수 요건

생성되거나 업데이트된 모든 verify-* 스킬은 다음을 갖추어야 합니다:

### 1. 실제 파일 경로

- Related Files의 모든 경로가 코드베이스에 실제 존재해야 합니다
- 플레이스홀더(`path/to/file`)는 절대 사용하지 않습니다
- 검증 방법:

```bash
ls <file-path> 2>/dev/null || echo "MISSING: <file-path>"
```

### 2. 작동하는 탐지 명령어

- Workflow의 모든 Grep/Glob/Read 패턴이 현재 파일과 매칭되어야 합니다
- 최소 1개의 명령어를 드라이런하여 문법 유효성을 검증합니다
- 결과가 0건이어도 문법이 올바르면 PASS

### 3. PASS/FAIL 기준

- 각 검사에 대해 통과와 실패의 명확한 조건을 명시합니다
- 실패 시 수정 방법을 코드 예시와 함께 제공합니다

### 4. 현실적인 예외 (Exceptions)

- 프로젝트에 맞게 충분한 "위반이 아닌" 케이스를 포함합니다 (일반적으로 3-5개, 복잡한 도메인은 더 많이)
- False positive를 방지하기 위한 구체적인 패턴 설명
- 스키마나 규칙이 프로젝트 중간에 변경된 경우, 변경 이전 파일에 대한 예외를 포함합니다

### 5. 일관된 형식

필수 섹션:
- `Purpose` — 2-5개의 번호가 매겨진 검증 카테고리
- `When to Run` — 3-5개의 트리거 조건
- `Related Files` — 파일 경로 테이블
- `Workflow` — 검사 단계 (도구, 경로, PASS/FAIL)
- `Output Format` — 결과 마크다운 테이블
- `Exceptions` — 위반이 아닌 케이스

### 6. 등록 테이블 동기화

- `sync-skills/SKILL.md`의 등록 테이블에 포함
- `run-skills/SKILL.md`의 실행 대상 테이블에 포함
- `CLAUDE.md`의 Skills 테이블에 포함
- 세 테이블의 스킬 목록이 동일해야 합니다
- Glob 패턴(`.claude/skills/verify-*/SKILL.md`)으로 실제 존재하는 스킬과 테이블 목록이 일치하는지 확인합니다

### 7. Description 트리거 조건

- YAML frontmatter의 description에 **언제 트리거되는가** 포함
- "무엇을 하는가"만 쓰지 않고, 구체적 상황/조건을 명시
