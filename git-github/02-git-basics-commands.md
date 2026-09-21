# Git 기초 명령어와 3영역 모델

## 3영역 모델: Working Directory → Staging Area → Repository

```bash
# README 파일 생성 (PowerShell 예시)
@"
# Git Basic Lab
Git의 기본 동작 원리를 학습하는 프로젝트입니다.
"@ | Set-Content -Encoding UTF8 README.md
```

| 단계 | 명령어 | 의미 |
|---|---|---|
| 1. 상태 확인 | `git status` | `Untracked files: README.md` → 아직 Git이 추적하지 않는 상태 (Working Directory에만 존재) |
| 2. 스테이징 | `git add README.md` | Working Directory → Staging Area로 이동 |
| 3. 커밋 | `git commit -m "docs: add project README"` | Staging Area → Repository (기록으로 저장) |
| 4. 이력 확인 | `git log --oneline` | 커밋 이력 확인 |

## 커밋은 스냅샷이다

Git은 커밋 시점의 전체 프로젝트 파일 상태를 **사진(스냅샷)처럼** 기록한다.

```bash
git ls-tree --name-only <커밋ID>   # 특정 커밋의 파일 목록 확인
git show --stat <커밋ID>            # 커밋 통계 확인
```

## HEAD, 브랜치 포인터, 커밋 ID

- **커밋 ID**: 각 커밋을 구분하는 고유 해시값 (`git log --oneline` 또는 `git rev-parse HEAD`)
- **HEAD**: 현재 작업 중인 위치를 가리키는 포인터 (`HEAD → main → 최신 커밋`)
- **브랜치**: 특정 커밋을 가리키는, 이동 가능한 가벼운 이름표(포인터)

```bash
git log --oneline --decorate --graph --all   # 전체 커밋 그래프 확인
```

## 변경 내용을 골라서 커밋하기 (선택적 스테이징)

여러 파일을 동시에 수정했을 때, 작업 의도별로 쪼개서 각각 커밋할 수 있다.

```bash
git diff   # 변경 내용 확인

# 1. 파이썬 파일만 먼저 스테이징 후 커밋
git add analysis.py
git commit -m "refactor: improve project entry point"

# 2. 나머지 README 수정본을 스테이징 후 커밋
git add README.md
git commit -m "docs: add learning topic"
```

> **Tip**: 하나의 커밋에는 하나의 작업 의도만 담아야 나중에 이력을 이해하고 관리하기 쉽다.

## 기타 명령어

```bash
git rm --cached <파일>   # 로컬 파일은 그대로 두고 Git 추적만 해제
```

## 작업을 잠시 미뤄두고 최신 코드부터 받아오기 (stash)

```bash
git stash                 # 지금 하던 작업을 임시 저장
git switch main
git pull origin main
git switch 내브랜치명
git merge main
git stash pop              # 아까 미뤄둔 작업을 다시 불러오기
```

`merge` 하기 **전에** `stash`로 먼저 작업을 치워두는 이유: 머지 시점에 원래 수정하던 내용과 충돌이 일어나지 않도록, 최신 코드를 먼저 반영한 뒤 미뤄둔 작업을 마지막에 불러오기 위함이다.
