# Claude Config Command

.claude/ submodule의 변경 사항을 commit, push하고 부모 저장소도 업데이트한다.

## 실행 시 동작

1. **.claude/ 변경 확인**: submodule에서 git status 확인
2. **submodule commit & push**: 변경 사항이 있으면 commit하고 push
3. **부모 저장소 업데이트**: submodule 참조 변경을 commit하고 push

## 실행 방법

### 1. Submodule 상태 확인

```bash
git -C .claude status --short
```

### 2. Submodule 변경 사항 commit & push

변경 사항이 있으면:

```bash
# 모든 변경 사항 stage
git -C .claude add -A

# commit (사용자에게 메시지 요청)
git -C .claude commit -m "커밋 메시지"

# push
git -C .claude push
```

### 3. 부모 저장소 업데이트

submodule 참조가 변경되었으면:

```bash
# submodule 참조 변경 stage
git add .claude

# commit
git commit -m "chore: update .claude submodule"

# push
git push
```

## 인자 처리

| 명령 | 설명 |
|------|------|
| `/claude-config` | 변경 사항 확인 및 commit/push |
| `/claude-config status` | 상태만 확인 (commit 안 함) |
| `/claude-config -m "메시지"` | 지정한 메시지로 commit |

## 워크플로우

```
┌─────────────────────────────────────────────────────────┐
│  /claude-config                                         │
├─────────────────────────────────────────────────────────┤
│  1. git -C .claude status                               │
│     └─ 변경 없음 → "Nothing to commit" 출력 후 종료     │
│     └─ 변경 있음 → 계속                                 │
│                                                         │
│  2. 변경 내용 표시                                      │
│     └─ git -C .claude diff --stat                       │
│                                                         │
│  3. 커밋 메시지 요청 (인자로 안 받았으면)               │
│                                                         │
│  4. Submodule commit & push                             │
│     └─ git -C .claude add -A                            │
│     └─ git -C .claude commit -m "메시지"                │
│     └─ git -C .claude push                              │
│                                                         │
│  5. 부모 저장소 업데이트                                │
│     └─ git add .claude                                  │
│     └─ git commit -m "chore: update .claude submodule"  │
│     └─ git push                                         │
│                                                         │
│  6. 완료 메시지                                         │
└─────────────────────────────────────────────────────────┘
```

## 주의사항

- 부모 저장소의 working directory가 clean해야 함 (다른 변경 사항이 섞이지 않도록)
- push 전에 사용자 확인 받기
- 충돌 발생 시 수동 해결 안내
