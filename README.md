# Claude Config - Shared Configuration

이 저장소는 여러 프로젝트에서 공유하는 Claude Code 설정입니다.

## 새 프로젝트에 설치

```bash
# 1. 프로젝트 생성 및 git 초기화
mkdir my-project && cd my-project
git init

# 2. .claude submodule 추가
git submodule add git@github-ohama:ohama/Claude-Config.git .claude

# 3. 초기 커밋
git add .gitmodules .claude
git commit -m "chore: add .claude as submodule"
```

## 기존 프로젝트 클론

```bash
# submodule 포함하여 클론
git clone --recurse-submodules <repo-url>

# 또는 클론 후 submodule 초기화
git clone <repo-url>
cd <repo>
git submodule update --init
```

## 설정 변경 후 Push (Project A)

`.claude/` 내용을 수정한 후:

```bash
/claude-config push
```

또는 커밋 메시지 지정:

```bash
/claude-config push -m "feat: add new skill"
```

### 수동으로 하려면

```bash
# 1. Submodule 커밋
cd .claude
git add -A
git commit -m "변경 내용"
git push

# 2. 부모 저장소 업데이트
cd ..
git add .claude
git commit -m "chore: update .claude submodule"
git push
```

## 다른 프로젝트에서 Pull (Project B)

Project A가 변경한 내용을 가져오려면:

```bash
/claude-config pull
```

### 상태 확인

```bash
/claude-config
```

### 수동으로 하려면

```bash
# 1. Submodule 최신으로 업데이트
cd .claude
git fetch origin
git pull origin master

# 2. 변경 확인
cd ..
git status .claude  # "(new commits)" 표시되면 업데이트됨

# 3. 부모 저장소에 반영
git add .claude
git commit -m "chore: update .claude submodule"
git push
```

## 명령어 요약

| 명령 | 설명 |
|------|------|
| `/claude-config` | 상태 확인 |
| `/claude-config push` | 변경 사항 commit/push |
| `/claude-config push -m "메시지"` | 지정 메시지로 commit |
| `/claude-config pull` | 원격 변경 가져오기 |

## 구조

```
.claude/
├── agents/          # 에이전트 설정
├── commands/        # 슬래시 명령어
├── docs/            # 문서
├── hooks/           # 훅 스크립트
├── skills/          # 스킬 정의
├── settings.json    # 공유 설정
└── settings.local.json  # 로컬 설정 (gitignore)
```

## 주의사항

- `settings.local.json`은 `.gitignore`에 포함되어 있어 공유되지 않음
- Submodule 변경 시 부모 저장소도 함께 커밋해야 다른 프로젝트에서 참조 가능
