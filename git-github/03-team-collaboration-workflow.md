# 팀 협업 워크플로우 (브랜치, PR, 머지)

팀 프로젝트에서 매일 반복하게 되는 Git 실전 사이클을 정리한다.

## 1단계: 내 작업 브랜치 만들고 이동

```bash
git checkout main
git checkout -b feature/본인기능명   # 개인 브랜치 생성
```

## 2단계: 코딩 후 내 브랜치에 Push

미완성이어도 퇴근 시 반드시 실행해서 **코드 유실을 방지**한다.

```bash
git add .
git commit -m "feat: 담당 기능 구현"
git push origin feature/본인기능명
```

> ⚠️ **VS Code 소스 제어 창을 쓸 때 가장 조심할 점**: 커밋/푸시를 누르기 전에, 화면 왼쪽 아래 구석에서 **현재 내 브랜치 이름**을 눈으로 꼭 확인한다. `main`이 아니라 내 전용 브랜치가 맞는지 확인하고 눌러야 안전하게 내 브랜치로만 올라간다.

## 3단계: GitHub에서 PR 생성 후 팀에 공지

**★ 셀프 머지 절대 금지.** 팀 공지 채널(Slack 등)에 알림이 필수다.

1. GitHub에서 `[Compare & pull request]` 클릭
2. 내용 작성 후 PR 제출
3. 공지방에 "`feature/로그인` 작업 완료하여 PR 올렸습니다! 검토 부탁드립니다! [링크]" 게시

## 4단계: 코드 리뷰와 반영

- 리뷰어가 `Files changed` 탭에서 코드를 확인하고 코멘트를 남긴다.
- 작성자는 코멘트를 반영해 수정 후 다시 Push (PR은 자동으로 업데이트됨).
- 문제가 없으면 리뷰어가 **Approve(승인)** 한다.
- 로컬에서 서버(`runserver`)를 실행해 에러 없이 정상 작동하는지 확인한 뒤 승인하는 것이 안전하다.

## 5단계: 최종 Merge

승인이 완료되면 팀장 또는 교차 검토자가 GitHub에서 `[Merge pull request]`를 눌러 `main`에 합친다. **셀프 머지 금지** 규칙은 코드가 최소 한 명 이상에게 검증받도록 보장하는 안전장치다.

## 6~7단계: 로컬 동기화

```bash
# 6단계: main을 최신화
git checkout main
git pull origin main

# 7단계: 내 작업 브랜치로 돌아가 최신 main 내용을 병합
git checkout feature/본인기능명
git merge main
```

이 과정을 거쳐야 내 브랜치도 최신 상태를 유지하며, 나중에 PR을 올릴 때 충돌 위험이 줄어든다.

## 언제 Merge하면 안 되는가

- 코드를 짜다가 완성되지 않은 상태로 퇴근할 때 → **절대 머지 금지.** `git push origin feature/...`로 브랜치에만 올려둔다.
- `python manage.py runserver` 실행 시 에러가 뜰 때 → 내 컴퓨터에서 나는 에러는 머지하면 팀원 전체에서 100% 재현된다. 완전히 해결한 뒤에만 머지한다.

## 충돌(Conflict) 대처

여러 팀원이 동시에 같은 파일(예: `models.py`)을 수정해서 머지하면 충돌이 발생한다.

- VS Code에서 `[Accept Current Change]` / `[Accept Incoming Change]` 중 선택하거나
- AI에게 충돌난 코드를 보여주고 "두 팀원이 동시에 수정해서 Git 충돌이 났어, 어떻게 해결해야 해?"라고 물어보면 해결 방법을 안내받을 수 있다.

## 팀 전체가 하루 두 번 지키면 좋은 루틴

1. **아침에 출근해서**: 밤새 머지된 최신 코드를 내 브랜치에 반영하고 시작.
2. **기능을 다 만들어 Push하기 직전**: 그 사이 main에 새로 올라온 내용이 있는지 내 브랜치에서 먼저 병합해보고 테스트한 뒤 올린다.

```bash
git checkout main
git pull origin main
git checkout feature/내-기능-브랜치
git merge main
```

이 루틴을 팀원 전체가 지키면 Git 충돌로 크게 고생할 일이 줄어든다.

## .gitignore와 브랜치 이름 초기 체크포인트

- Django 프로젝트는 `.pyc` 파일, `venv/` 폴더, `.env`(비밀키/DB 비밀번호) 가 커밋되지 않도록 `.gitignore`를 꼭 설정한다 (AI에게 "Django용 `.gitignore` 만들어줘"라고 요청하면 바로 만들어준다).
- 요즘 GitHub는 저장소 생성 시 기본 브랜치가 이미 `main`인 경우가 많다. `master`로 생성됐다면 `main`으로 바꾼다.
