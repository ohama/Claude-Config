---
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
description: mdBook 프로젝트 설정 및 GitHub Pages 배포 준비
---

<role>
mdBook 문서 사이트 설정 도우미. book/ 소스 폴더, docs/ 빌드 출력, GitHub Actions 워크플로우를 생성합니다.
</role>

<commands>

## 사용법

| 명령 | 설명 |
|------|------|
| `/mdbook` | 대화형 설정 시작 |
| `/mdbook init` | 기본값으로 빠른 초기화 |
| `/mdbook build` | 기존 book/ 빌드만 실행 |

</commands>

<execution>

## Step 1: mdbook 설치 확인

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

## Step 2: 프로젝트 정보 수집

AskUserQuestion으로 필수 정보 수집:

**질문 1: 프로젝트 정보**
- 책 제목 (예: "My Project Documentation")
- 저자 이름
- 언어 (ko/en, 기본: ko)
- 설명 (한 줄)

**질문 2: GitHub 정보** (선택)
- Repository URL (예: https://github.com/user/repo)
- 없으면 edit URL 기능 비활성화

**질문 3: 기존 콘텐츠**
- 기존 markdown 파일이 있는 폴더? (예: tutorial/, documentation/)
- 없으면 빈 템플릿 생성

## Step 3: docs/ 폴더 처리

```bash
[ -d "docs" ] && echo "EXISTS"
```

존재하면 사용자에게 질문:
```
docs/ 폴더가 이미 존재합니다.

[B] 백업 후 진행 (docs/ → docs.backup/)
[O] 덮어쓰기
[X] 취소
```

## Step 4: book/ 구조 생성

```
book/
├── book.toml      # 설정
└── src/
    ├── SUMMARY.md     # 목차
    └── introduction.md # 소개
```

### book.toml 템플릿

```toml
[book]
title = "{TITLE}"
authors = ["{AUTHOR}"]
language = "{LANG}"
description = "{DESCRIPTION}"
src = "src"

[build]
build-dir = "../docs"
create-missing = false

[output.html]
default-theme = "light"
preferred-dark-theme = "navy"
{GIT_REPO_CONFIG}

[output.html.search]
enable = true
limit-results = 30
boost-title = 2
boost-hierarchy = 1
```

**GIT_REPO_CONFIG** (repo URL 있을 때만):
```toml
git-repository-url = "{REPO_URL}"
edit-url-template = "{REPO_URL}/edit/master/book/src/{path}"
```

### SUMMARY.md 템플릿

기존 콘텐츠 없을 때:
```markdown
# Summary

[소개](introduction.md)

# 시작하기

- [Chapter 1](chapter-01.md)
- [Chapter 2](chapter-02.md)

# 부록

- [참고 자료](references.md)
```

기존 콘텐츠 있을 때:
- 폴더 스캔하여 .md 파일 목록 생성
- 파일명에서 챕터 구조 추론
- 사용자에게 확인 후 SUMMARY.md 생성

### introduction.md 템플릿

```markdown
# {TITLE}

{DESCRIPTION}

## 시작하기

[Chapter 1](chapter-01.md)부터 시작하세요.
```

## Step 5: 기존 콘텐츠 복사

소스 폴더 지정된 경우:
```bash
cp {SOURCE_DIR}/*.md book/src/
```

## Step 6: mdbook 빌드

```bash
mdbook build book
```

## Step 7: GitHub Pages 설정

### .nojekyll 생성
```bash
touch docs/.nojekyll
```

### GitHub Actions 워크플로우

```yaml
# .github/workflows/mdbook.yml
name: Build mdBook

on:
  push:
    branches:
      - master
      - main
    paths:
      - 'book/**'
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v2
        with:
          mdbook-version: 'latest'

      - name: Build mdBook
        run: mdbook build book

      - name: Check for changes
        id: check
        run: |
          git diff --quiet docs/ || echo "changes=true" >> $GITHUB_OUTPUT

      - name: Commit and push
        if: steps.check.outputs.changes == 'true'
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/
          git commit -m "docs: rebuild mdBook site"
          git push
```

## Step 8: 결과 출력

```markdown
## mdBook 설정 완료

### 생성된 파일
- book/book.toml - 설정 파일
- book/src/SUMMARY.md - 목차
- book/src/introduction.md - 소개 페이지
- docs/ - 빌드 출력 ({N} HTML files)
- .github/workflows/mdbook.yml - 자동 빌드

### 다음 단계
1. book/src/SUMMARY.md 편집하여 목차 구성
2. book/src/에 챕터 파일 추가
3. `mdbook serve book`으로 로컬 미리보기
4. git push 후 GitHub Settings > Pages > Branch: master, Folder: /docs
```

</execution>

<subcommands>

## /mdbook init

기본값으로 빠른 초기화:
- 제목: 현재 폴더명
- 저자: git config user.name
- 언어: ko
- 빈 템플릿 생성

## /mdbook build

기존 book/ 폴더 빌드만 실행:
```bash
mdbook build book
```

## /mdbook serve

로컬 개발 서버:
```bash
mdbook serve book --open
```

</subcommands>

<examples>

### 예시 1: 새 문서 사이트

```
User: /mdbook

Claude: mdBook 설정을 시작합니다.

[프로젝트 정보]
- 책 제목: My API Documentation
- 저자: John Doe
- 언어: ko
- 설명: REST API 사용 가이드

[GitHub 정보]
- Repository URL: https://github.com/johndoe/my-api

기존 markdown 폴더가 있나요?
→ 없음 (빈 템플릿 생성)

## 완료
- book/src/ (3 files)
- docs/ (5 HTML files)

`mdbook serve book`으로 미리보기하세요.
```

### 예시 2: 기존 콘텐츠 변환

```
User: /mdbook

Claude: 기존 markdown 폴더가 있나요?
→ tutorial/

tutorial/ 스캔 결과:
- chapter-01-intro.md
- chapter-02-setup.md
- chapter-03-usage.md

이 순서로 SUMMARY.md를 생성할까요? [Y/n]

User: Y

## 완료
- book/src/ (4 files, 3 copied)
- docs/ (8 HTML files)
```

</examples>
