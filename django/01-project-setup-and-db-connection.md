# Django 프로젝트 세팅과 DB 연동

## 프로젝트 초기 세팅

```bash
mkdir C:\dev\django_members
cd C:\dev\django_members
code .

python -m venv .venv                 # 오류가 나면 한 번 더 실행
.\.venv\Scripts\Activate.ps1          # 가상환경 활성화

pip install django "psycopg[binary]" python-dotenv
python -m pip install --upgrade pip
pip list
pip freeze > requirements.txt
```

`.venv` 생성 후 `Scripts` 폴더 안에 `pip.exe`, `python.exe`, `Activate`/`Deactivate` 스크립트가 있는지 확인하면 정상 생성된 것이다.

## 프로젝트/앱 생성

```bash
django-admin startproject config      # settings.py와 urls.py가 들어있는 핵심 폴더
python manage.py startapp members     # members 앱 생성
python manage.py runserver
```

새로 만든 앱은 `settings.py`의 `INSTALLED_APPS` 리스트 **맨 끝에** 앱 이름(`"members"`)을 추가해줘야 Django가 인식한다.

## PostgreSQL 연동 (특정 스키마 지정)

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "...",
        "USER": "...",
        "PASSWORD": "...",   # .env로 관리, 절대 커밋하지 않는다
        "HOST": "...",
        "PORT": "...",
        "OPTIONS": {
            "options": "-c search_path=practice",  # practice 스키마를 우선 검색
        },
    }
}
```

`OPTIONS`의 `search_path`를 지정하면, 여러 스키마 중 원하는 스키마(예: `practice`)를 우선적으로 검색하도록 만들 수 있다.

## 환경변수(.env)로 비밀정보 관리

민감한 정보(DB 비밀번호, API 키)는 `settings.py`에 직접 쓰지 않고 `.env` 파일로 분리해 `python-dotenv`로 불러온다. `.env`는 `.gitignore`에 등록해서 절대 커밋되지 않도록 한다. 팀원들과는 값이 비어있는 `.env.example`만 공유한다.

## 실습에서 겪은 점

- `startapp`으로 앱을 만든 뒤 `settings.py`에 앱을 등록하는 걸 깜빡하면, 모델/뷰가 인식되지 않아서 헤맬 수 있다 → 새 앱을 만들 때마다 `INSTALLED_APPS` 등록을 세트로 기억한다.
- DB 연동 오류가 나면 가장 먼저 `.env` 값, `OPTIONS`의 `search_path`, 그리고 PostgreSQL 서버가 실제로 떠 있는지부터 확인한다.
