# 모듈과 가상환경

## 모듈이란

모듈은 파이썬 코드를 담은 `.py` 파일이다. `import`로 표준 라이브러리와 직접 만든 모듈을 불러올 수 있고, 관련 모듈을 패키지 폴더로 묶어 관리할 수 있다.

```python
import math                    # math 모듈 전체를 math라는 이름으로 사용
from math import sqrt          # math에서 sqrt만 가져와 바로 사용
import statistics as stats     # statistics를 stats라는 별칭으로 사용
```

- `import module`: 모듈 전체를 가져와 `모듈이름.함수이름()` 형태로 호출.
- `from module import name`: 모듈 내부의 특정 함수/클래스만 선택해서 가져오며, 모듈 이름 없이 함수 이름만으로 호출 가능.
- `import module as alias`: 별칭 부여.

표준 라이브러리(별도 설치 없이 기본 제공)와 외부 패키지(`pip install`로 설치)는 구분해야 한다.

## 가상환경(venv)

프로젝트마다 독립된 패키지 환경을 만들기 위해 가상환경을 사용한다.

```powershell
python -m venv .venv                              # .venv 가상환경 생성
.venv\scripts\activate                            # 가상환경 활성화 (Windows)
# venv/bin/activate                                # 가상환경 활성화 (Mac)

Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned  # 최초 1회, 실행 권한 정책
```

터미널 앞에 초록색 `(.venv)`가 뜨면 가상환경이 정상 활성화된 상태다.

## 패키지 설치와 requirements.txt

```powershell
python -m pip install --upgrade pip           # pip 최신화
python -m pip install -r requirements.txt     # requirements.txt에 적힌 패키지 설치
pip freeze > requirements.txt                  # 현재 설치된 패키지를 기록
```

`requirements.txt`로 필요한 패키지와 버전을 기록해 두면, 팀원 간 환경을 동일하게 맞출 수 있다. 버전이 다르면 오류가 날 수 있으므로 아래처럼 확인하는 습관이 필요하다.

```powershell
python -c "import sys; print(sys.executable)"       # 실행 중인 파이썬 경로 확인
python -c "import pandas; print(pandas.__version__)" # 라이브러리 버전 확인
```

## VS Code에서 인터프리터 선택

`Ctrl+Shift+P` → `Python: Select Interpreter` → 프로젝트의 `.venv` 선택. `.venv` 폴더가 여러 개 보일 수 있으므로 `Scripts` 폴더 안의 정확한 경로를 선택해야 한다.

## ModuleNotFoundError 점검 순서

1. 파일 위치(경로)가 맞는지 확인
2. 현재 활성화된 파이썬 환경이 맞는지 확인 (베이스 환경으로 실행되고 있지는 않은지)
3. 해당 가상환경에 패키지가 실제로 설치되어 있는지 확인

> 가상환경이 없는데도 `python 00_01.py`가 실행되는 경우, 사실은 시스템의 베이스(전역) 환경으로 실행된 것이다. 프로젝트별로 격리되지 않으므로 좋은 습관이 아니다.
