# TIL | Chapter 14-1 로컬 데이터 분석 자동화

- 학습일: 2026-10-15
- 주제: NAVER 검색 트렌드와 한국관광공사 지역별 관광 강도 데이터를 수집·검증하고, 보고서 생성 과정을 로컬 Python 파이프라인으로 실행하기

## 오늘 배운 것

### 1. 자동화 전에 단계별 검증이 필요하다

자동화는 아래 작업을 정해진 순서로 실행한다.

```text
API 수집 → 데이터 검증·분석 → LLM 요약 → PDF 생성
```

HTTP 요청이 성공해도 데이터가 올바르다는 뜻은 아니다. 응답 코드, 실제 데이터 존재 여부, 필드, 기준 기간, 숫자 변환, 중복 등을 확인해야 한다. 검증된 작업만 파이프라인으로 묶는다.

### 2. 사용한 데이터와 해석 범위

- **NAVER Search Trend:** 여행·축제·캠핑·맛집의 월별 상대 검색 관심도. `ratio`는 실제 검색 건수가 아니라 조회 조건에서 정규화된 상대 지수다.
- **관광 강도 API:** 서울(`areaCd=11`), 2025년 9월(`baseYm=202509`) 기준 체류 강도와 소비 강도. 전체 지역 코드(체류 `21`, 소비 `22`)를 요청해 서울 합계와 구별 자료를 받았다.
- 관광 자료는 체류 26행, 소비 26행으로 총 52행이 처리되었다. 이 지표를 실제 방문객 수나 원화 소비액으로 해석하면 안 된다.
- NAVER와 관광 자료는 분석 단위·범위가 다르므로 기본 실습에서는 행 단위로 병합하지 않고, 각각 분석해 한 보고서에서 함께 설명한다.

### 3. 실패를 상태 그대로 기록하고 복구한다

- 관광 소비 API에서 `ConnectTimeout`이 발생했을 때 파이프라인은 해당 단계에서 멈췄다. 실패한 소비 자료를 성공한 것처럼 처리하지 않았다.
- 이후 노트북에서 관광 수집 스크립트를 다시 실행했고, 체류·소비 각각 26행을 받아 처리 CSV를 만들었다.
- Gemini의 일시적인 `503 UNAVAILABLE`은 서비스 혼잡 오류였다. 후속 파이프라인 성공 로그를 확인했다.
- 파이프라인의 **Fail Fast** 동작: 한 단계가 실패하면 그 뒤 분석·요약·PDF 단계는 실행하지 않는다.

## 실행 방법과 파일

### 개별 수집 재시도: VS Code 노트북

```python
import runpy

runpy.run_path(
    str(PROJECT_DIR / "src" / "collect_tourism.py"),
    run_name="__main__",
)
```

`runpy.run_path()`는 실행한 모듈의 네임스페이스도 반환하므로, 노트북 셀 마지막 줄에 그대로 쓰면 함수·변수 정보가 길게 표시될 수 있다. 필요하면 `result = runpy.run_path(...)`처럼 변수에 담는다.

### 전체 로컬 파이프라인

프로젝트 폴더에서:

```powershell
python run_pipeline.py
```

Windows 배치 파일 `run_pipeline.bat`은 같은 파이프라인을 실행하는 간편 실행기다. 가상환경은 `chapter14-1` 상위 폴더에 있으므로 Python 경로는 `..\.venv\Scripts\python.exe`를 사용한다.

파이프라인 단계:

1. `src/collect_naver.py`
2. `src/collect_tourism.py`
3. `src/analyze.py`
4. `src/generate_llm_report.py`
5. `src/generate_pdf.py`

주요 결과물:

- 원본 응답: `data/raw/`
- 처리 데이터: `data/processed/naver_trend.csv`, `data/processed/tourism_strength.csv`
- 분석 결과: `reports/analysis_summary.csv`
- 그래프: `reports/naver_trend.png`, `reports/tourism_strength_top10.png`
- LLM 요약: `reports/llm_summary.md`
- PDF: `reports/tourism_trend_report_YYYY-MM-DD.pdf`
- 실행 로그: `logs/pipeline_YYYYMMDD.log`

API 키는 코드나 GitHub에 직접 적지 않고 `.env`에 보관한다. `.env`와 원본 데이터·로그는 Git에 올리지 않도록 `.gitignore`를 확인한다. 키의 실제 값은 터미널·노트북 출력이나 보고서에 노출하지 않는다.

## 자동화 방식은 두 가지이며 서로 독립이다

### PART 1: 로컬 자동화

내 PC에서 Python 파이프라인을 실행해 PDF를 만든다. `run_pipeline.bat`은 Windows 작업 스케줄러가 호출할 수 있는 실행 파일이다. 작업 스케줄러 등록 마법사를 열어 작업 이름을 입력하는 단계까지 진행했으나, 등록 완료와 예약 실행 검증은 아직 확인하지 않았다.

작업 스케줄러에 등록할 때의 핵심 값:

- 프로그램: `C:\Windows\System32\cmd.exe`
- 인수: `/c "C:\dev\-llm-data-analysis-course\chapter14-1\run_pipeline.bat"`
- 시작 위치: `C:\dev\-llm-data-analysis-course\chapter14-1`

예약 실행 전 수동 실행이 성공하는지, 로그의 마지막이 `PIPELINE SUCCESS`인지, PDF가 실제로 열리는지 확인한다. 매 실행마다 외부 API와 LLM을 다시 호출하므로 불필요하게 자주 예약하지 않는다.

### PART 2: Make 온라인 자동화 — 아직 별도 실습

Make는 로컬 Python·PDF·`reports` 폴더를 사용하지 않는다. Make 안에서 Scheduler → HTTP API → 간단한 데이터 처리 → LLM → Gmail 순으로 별도 Scenario를 만든다. 우선 `Run once`로 각 모듈의 입력·출력을 확인하고 Gmail 테스트 메일을 받은 다음에만 정기 예약을 활성화한다. Gmail 연결 및 Make Scenario는 오늘 아직 완료하지 않았다.

## 다음에 할 일

- [ ] 작업 스케줄러 마법사에서 트리거와 `cmd.exe` 작업 경로를 입력해 등록
- [ ] 등록한 작업을 수동 실행해 로그와 PDF 확인
- [ ] `llm_summary.md` 내용을 검토하고 PDF에서 한글·그래프·본문을 확인
- [ ] 로컬 PART 1 체크리스트를 마친 뒤 Make PART 2 Scenario 시작
- [ ] Make에서 API 입력·출력을 단계별 확인한 뒤 Gmail 테스트 발송
- [ ] 테스트 수신과 데이터 검증을 모두 확인한 후 적절한 예약 주기 설정

## 오늘의 핵심 정리

> API 호출 성공은 분석 성공과 다르다. 자동화는 검증된 절차를 반복할 뿐이므로, 먼저 사람이 데이터를 확인하고 실패 원인을 구분해야 한다.
