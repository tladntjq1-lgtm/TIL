# 개발 환경 세팅 (Git, VS Code, venv)

## 1. 기본 프로그램 설치 및 GitHub 준비

- GitHub 계정 생성
- Git 설치
- Python 설치 (**PATH 체크 필수**)
- VS Code 설치 및 GitHub 계정 로그인
- GitHub에서 새 레포지토리 생성 (예: `ai-data-analysis`)
- 로컬에 작업용 폴더(예: `dev`) 미리 생성

## 2. 레포지토리 클론

```bash
cd dev
git clone https://github.com/내계정명/ai-data-analysis
```

VS Code로 프로젝트 열기:

```bash
code .
```

### `code .`가 안 될 때

1. 윈도우 검색창에 `환경 변수` 검색 → `시스템 환경 변수 편집`
2. `환경 변수` 버튼 → `Path` 선택 → `편집`
3. `새로 만들기` → Git 설치 경로 중 `git.exe`가 있는 폴더 경로 추가
4. 그래도 안 되면 새 터미널에서 `dir code.exe /s`로 경로를 찾아 직접 실행

VS Code 확장 프로그램: **Python**, **Jupyter** 설치.

## 3. requirements.txt 생성 및 첫 커밋

강의 저장소의 `requirements.txt` 내용을 복사해서 새 파일로 저장한 뒤, Source Control(`Ctrl+Shift+G`)에서 Stage → Commit → Push.

처음 푸시 시 사용자 정보를 요구하면:

```bash
git config --global user.name "내이름(또는 깃허브아이디)"
git config --global user.email "내이메일@example.com"
```

## 4. Git 기초 명령어

| 명령어 | 역할 |
|---|---|
| `git status` | 현재 상태(브랜치, 변경된 파일) 확인 |
| `git remote -v` | 연결된 원격 저장소 URL 확인 |
| `pwd` | 현재 작업 경로 확인 |
| `dir` | 현재 폴더의 파일/서브폴더 목록 확인 |

## 5. 파이썬 가상환경(.venv)

```bash
python -m venv .venv                # 가상환경 생성
.venv\scripts\activate              # 활성화 (Windows)

# 최초 1회, 실행 정책 문제 해결
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

터미널 앞에 초록색 `(.venv)`가 뜨면 성공.

## 6. 패키지 설치와 .gitignore

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

`.gitignore`는 팀 강의 저장소의 내용을 그대로 복사해 프로젝트 루트에 생성한다 — **가상환경 폴더, `__pycache__`, `.env` 등 민감/불필요 파일이 커밋되지 않도록 막아준다.**

## 7. 환경 변수(.env) 관리

- `.env.example`: 팀에서 공유하는 **설정 키 이름의 샘플** (실제 값은 비워두거나 더미 값). 이 파일은 커밋한다.
- `.env`: 개인 컴퓨터에만 적용되는 실제 값. `.gitignore`에 등록되어 있어 Git에 잡히지 않는 것이 **정상**이다.

## 8. 새 Git 저장소 처음부터 만들기

```bash
mkdir git-basic-lab
cd git-basic-lab
git init                    # .git 폴더 생성 (Get-ChildItem -Force로 확인 가능)
git branch -M main          # 기본 브랜치를 main으로 설정 (master로 되어있을 수 있음)
```
