---
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
description: mdBook 로컬 빌드 및 정적 배포 (CI 없이 직접 커밋)
---

<role>
mdBook 로컬 빌드 도우미. 로컬에서 직접 HTML을 생성하고 docs/를 커밋하여 GitHub Pages에 배포한다.

**CI 자동 빌드가 필요하면 `/pages` 커맨드를 사용한다.**
</role>

<skills_reference>
이 커맨드는 `mdbook-utils` 스킬을 사용한다:
- mdbook 설치 확인
- book.toml 탐지
- SUMMARY.md 동기화
- 빌드 명령
</skills_reference>

<commands>

## 사용법

| 명령 | 설명 |
|------|------|
| `/mdbook build [dir]` | 로컬 빌드 |
| `/mdbook serve [dir]` | 로컬 개발 서버 |
| `/mdbook clean [dir]` | 빌드 출력 정리 |
| `/mdbook sync [dir]` | SUMMARY.md 동기화 (빌드 없이) |

**CI 기반 설정은 `/pages` 커맨드 사용:**
- `/pages <dir>` — mdBook 구성 + GitHub Actions 워크플로우 생성

</commands>

<execution>

## 공통: book.toml 탐지

모든 서브커맨드에서 사용. `mdbook-utils` 스킬의 "2. book.toml 탐지" 참조.

**인자가 있는 경우:**
```bash
[ -f "{DIR}/book.toml" ] && echo "FOUND"
```

**인자가 없는 경우:**
1. 프로젝트 루트 확인: `[ -f "book.toml" ]`
2. 하위 디렉토리 탐색: `find . -maxdepth 2 -name "book.toml"`

**book.toml이 없으면:**
```
book.toml을 찾을 수 없습니다.

먼저 /pages 커맨드로 mdBook을 설정하세요:
  /pages <dir>
```

---

## /mdbook build [dir]

로컬에서 HTML을 빌드하고 docs/에 저장한다.

### Step 1: mdbook 설치 확인

`mdbook-utils` 스킬의 "1. mdbook 설치 확인" 참조.

### Step 2: book.toml 탐지

공통 로직 참조.

### Step 3: 빌드 실행

`mdbook-utils` 스킬의 "4. 빌드 명령" 참조.

```bash
mdbook clean {DIR}
mdbook build {DIR}
```

### Step 4: .nojekyll 확인

`mdbook-utils` 스킬의 "5. .nojekyll 확인" 참조.

```bash
[ -f "docs/.nojekyll" ] || touch docs/.nojekyll
```

### Step 5: 결과 출력

```
## 빌드 완료

docs/ ({N} HTML files)

다음 단계:
  git add docs/
  git commit -m "docs: update mdBook site"
  git push

또는 `/commit` 으로 커밋하세요.
```

---

## /mdbook serve [dir]

로컬 개발 서버를 실행한다.

### Step 1: book.toml 탐지

공통 로직 참조.

### Step 2: 서버 실행

```bash
mdbook serve {DIR} --open
```

- `--open`: 브라우저 자동 열기
- 기본 포트: 3000
- 파일 변경 시 자동 리로드

### Step 3: 안내

```
mdbook serve 실행 중...

http://localhost:3000 에서 미리보기하세요.
Ctrl+C로 종료합니다.
```

---

## /mdbook clean [dir]

빌드 출력을 정리한다.

### Step 1: book.toml 탐지

공통 로직 참조.

### Step 2: 정리 실행

```bash
mdbook clean {DIR}
```

- book.toml의 `build-dir`에 해당하는 디렉토리를 삭제
- 챕터 삭제 후 남은 잔여 HTML 파일 정리에 유용

### Step 3: 결과 출력

```
docs/ 정리 완료.
```

---

## /mdbook sync [dir]

SUMMARY.md를 디렉토리의 .md 파일과 동기화한다. 빌드는 하지 않는다.

### Step 1: book.toml 탐지

공통 로직 참조.

### Step 2: 동기화 확인

`mdbook-utils` 스킬의 "3. SUMMARY.md 동기화" 참조.

1. SUMMARY.md에서 링크된 .md 파일 추출
2. 디렉토리의 실제 .md 파일 목록과 비교
3. 차이 표시:

```
SUMMARY.md 동기화:

  + new-chapter.md      (새 파일)
  - old-chapter.md      (파일 없음)

SUMMARY.md를 업데이트할까요? [Y/N]
```

### Step 3: 업데이트

Y 선택 시:
- 새 파일: 적절한 섹션에 추가 (파일의 첫 `#` 헤더를 제목으로)
- 삭제된 파일: SUMMARY.md에서 해당 항목 제거

### Step 4: 결과 출력

**변경 있음:**
```
SUMMARY.md 업데이트 완료.

  + new-chapter.md 추가
  - old-chapter.md 제거

`/mdbook build` 로 빌드하세요.
```

**변경 없음:**
```
SUMMARY.md와 파일 목록이 일치합니다.
```

</execution>

<examples>

### 예시 1: 빌드

```
User: /mdbook build tutorial

Claude:
mdbook clean tutorial
mdbook build tutorial

## 빌드 완료

docs/ (15 HTML files)

다음 단계:
  git add docs/
  git commit -m "docs: update mdBook site"
  git push
```

### 예시 2: 자동 탐지 빌드

```
User: /mdbook build

Claude: book.toml 탐지: tutorial/book.toml

mdbook clean tutorial
mdbook build tutorial

## 빌드 완료

docs/ (15 HTML files)
```

### 예시 3: 개발 서버

```
User: /mdbook serve tutorial

Claude:
mdbook serve tutorial --open

mdbook serve 실행 중...

http://localhost:3000 에서 미리보기하세요.
Ctrl+C로 종료합니다.
```

### 예시 4: book.toml 없음

```
User: /mdbook build

Claude:
book.toml을 찾을 수 없습니다.

먼저 /pages 커맨드로 mdBook을 설정하세요:
  /pages <dir>
```

### 예시 5: SUMMARY 동기화

```
User: /mdbook sync tutorial

Claude:
SUMMARY.md 동기화:

  + 05-advanced.md      (새 파일)
  - old-intro.md        (파일 없음)

SUMMARY.md를 업데이트할까요? [Y/N]

User: Y

Claude:
SUMMARY.md 업데이트 완료.

  + 05-advanced.md 추가
  - old-intro.md 제거

`/mdbook build tutorial` 로 빌드하세요.
```

### 예시 6: 동기화 (변경 없음)

```
User: /mdbook sync

Claude: book.toml 탐지: tutorial/book.toml

SUMMARY.md와 파일 목록이 일치합니다.
```

</examples>
