---
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
description: mdBook 로컬 빌드 및 정적 배포 (CI 없이 직접 커밋)
---

<role>
mdBook 로컬 빌드 도우미. 로컬에서 직접 HTML을 생성하고 docs/를 커밋하여 GitHub Pages에 배포한다.

**CI 자동 빌드가 필요하면 `/pages` 커맨드를 사용한다.**
</role>

<commands>

## 사용법

| 명령 | 설명 |
|------|------|
| `/mdbook build [dir]` | 로컬 빌드 |
| `/mdbook serve [dir]` | 로컬 개발 서버 |
| `/mdbook clean [dir]` | 빌드 출력 정리 |

**CI 기반 설정은 `/pages` 커맨드 사용:**
- `/pages <dir>` — mdBook 구성 + GitHub Actions 워크플로우 생성

</commands>

<execution>

## /mdbook build [dir]

로컬에서 HTML을 빌드하고 docs/에 저장한다.

### Step 1: mdbook 설치 확인

```bash
which mdbook || echo "NOT_INSTALLED"
```

설치 안 됨 → 설치 안내:
```
mdbook이 설치되어 있지 않습니다.

설치 방법:
  cargo install mdbook
  # 또는
  brew install mdbook  # macOS
  # 또는
  sudo apt install mdbook  # Ubuntu
```

### Step 2: book.toml 위치 탐지

**인자가 있는 경우 (`/mdbook build tutorial`):**
```bash
[ -f "tutorial/book.toml" ] && echo "FOUND"
```

**인자가 없는 경우 (`/mdbook build`):**
1. 현재 디렉토리에 book.toml 확인:
```bash
[ -f "book.toml" ] && echo "FOUND_ROOT"
```

2. 없으면 하위 디렉토리 탐색:
```bash
find . -maxdepth 2 -name "book.toml" -type f 2>/dev/null
```

**book.toml이 없으면:**
```
book.toml을 찾을 수 없습니다.

먼저 /pages 커맨드로 mdBook을 설정하세요:
  /pages <dir>
```

### Step 3: 빌드 실행

```bash
mdbook clean {DIR}
mdbook build {DIR}
```

- `mdbook clean`은 이전 빌드 출력을 삭제하여 삭제된 챕터의 잔여 HTML이 남지 않도록 한다.

### Step 4: .nojekyll 확인

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

### Step 1: book.toml 위치 탐지

`/mdbook build`와 동일한 로직.

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

### Step 1: book.toml 위치 탐지

`/mdbook build`와 동일한 로직.

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

</examples>
