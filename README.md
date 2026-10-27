# News_Content_Analysis_with_GPT-4.1: News LLM Coding Toolkit

OpenAI API(ChatGPT)를 활용해 **뉴스 기사를 자동으로 내용분석(content analysis)** 하는 Jupyter Notebook 모음입니다.
기사 제목·본문이 담긴 CSV를 입력하면, 분석 유형에 맞는 결과(분류값 · 근거 · 신뢰도)를 CSV로 저장합니다.

## 노트북 구성

| 노트북 | 목적 | 결과 컬럼 |
|---|---|---|
| `GPT_Framing_analysis_vscode_.ipynb` | 뉴스 기사를 5가지 프레임(갈등 / 책임귀인 / 인간적 흥미 / 경제적 결과 / 도덕성) 중 하나로 분류 | `프레임`, `근거`, `신뢰도`, `비용` |
| `GPT_perspective_analysis_vscode_.ipynb` | 특정 행위자에 대한 기사의 관점(긍정 / 중립 / 부정) 분류 | `관점`, `근거`, `신뢰도`, `비용` |
| `GPT_binary_coding_analysis_vscode_.ipynb` | 연구자가 정의한 항목에 해당하는지를 예(1) / 아니오(0)로 판단하는 이진 코딩 | `코딩결과`, `근거`, `신뢰도`, `비용` |
| `GPT_News_Summary_vscode_.ipynb` | 기사를 정확히 3문장으로 요약 (5W1H 기반) | `요약`, `비용` |

## 공통 동작 방식

- 기사 **제목 + 본문**을 합쳐 프롬프트에 삽입합니다 (토큰 제한을 위해 최대 4,000자로 자름).
- 기사마다 **독립된 API 호출**을 사용해 이전 기사의 영향을 받지 않습니다.
- `temperature=0`, JSON 형식 응답(`response_format=json_object`)으로 일관성을 높였습니다.
- JSON 파싱이 실패할 경우를 대비한 다단계 파서(`EnhancedJSONParser`)를 포함합니다.
- 비동기(`asyncio`) 배치 처리 + 진행바(`tqdm`) + **5배치마다 중간 저장**을 지원합니다.
- 분석에 실패한 기사는 각각 `분류 실패`(프레임·관점), `-1`(이진 코딩), `요약 실패`(요약)로 표시됩니다.

기본 설정값 (`Analyzer` 생성자 인자):

| 인자 | 기본값 | 설명 |
|---|---|---|
| `concurrent_requests` | 3 | 동시 API 요청 수 |
| `rate_limit_delay` | 1.0 | 요청 후 대기 시간(초) |
| `batch_size` | 5 | 한 번에 처리하는 기사 수 |

> ⚠️ 결과의 `비용` 컬럼은 기사당 **$0.002로 고정한 추정치**이며, 실제 과금액이 아닙니다. 실제 사용량은 OpenAI 대시보드에서 확인하세요.

## 입력 데이터 형식

CSV 파일(UTF-8)에 아래 두 컬럼이 **반드시** 있어야 합니다.

| 컬럼 | 설명 |
|---|---|
| `title` | 기사 제목 |
| `text` | 기사 본문 |

그 외 컬럼은 결과 CSV에 그대로 유지됩니다. 데이터 파일은 이 저장소에 포함되어 있지 않습니다.

## 사용 방법

### 1. 패키지 설치

```bash
pip install -r requirements.txt
```

(Colab/Jupyter에서는 각 노트북의 설치 셀로도 가능합니다.)

### 2. 설정 입력

각 노트북 **마지막 실행 셀**의 설정 블록을 수정합니다.

```python
MODEL = "gpt-4.1-mini"
API_KEY = "Your-OpenAI-API-Key-Here"          # 본인의 OpenAI API 키
INPUT_CSV_PATH = "Your-Input-CSV-Path-Here"   # 입력 CSV 경로
OUTPUT_CSV_PATH = "Your-Output-CSV-Path-Here" # 결과 CSV 저장 경로
```

### 3. 실행

노트북을 위에서부터 순서대로 실행하면 결과 CSV가 `OUTPUT_CSV_PATH`에 저장됩니다.

> `GPT_Framing_analysis_vscode_.ipynb`, `GPT_News_Summary_vscode_.ipynb`의 첫 셀(`drive.mount`)은 Google Colab 전용입니다. 로컬 환경(VS Code 등)에서는 해당 셀을 건너뛰세요.

## 연구에 맞게 수정하기

- **프레임 / 관점 / 판단 기준**: 각 노트북의 `ALLOWED_FRAMES`, `ALLOWED_TONES`, `ANALYSIS_CRITERIA`와 `PROMPT_TEMPLATE`을 연구 주제에 맞게 수정하세요.
- **분석 대상 행위자·코딩 항목**: 관점 분석과 이진 코딩 노트북의 프롬프트에는 `[분석 대상 행위자]`, `[코딩 대상 항목]` 같은 **자리표시자(placeholder)** 가 들어 있습니다. 실행 전에 본인의 연구 대상과 판단 기준으로 반드시 교체하세요.
- **모델**: `MODEL` 값을 `gpt-4o-mini`, `gpt-4o` 등으로 바꿀 수 있습니다.

## 보안 및 데이터 관련 주의

- **API 키를 코드에 직접 입력한 상태로 커밋/푸시하지 마세요.** 노트북을 저장하기 전에 키를 다시 placeholder로 되돌리거나, 키를 입력한 노트북은 저장소에 올리지 않는 것을 권장합니다.
- 뉴스 기사 원문 CSV와 결과 CSV에는 저작권이 있는 텍스트가 포함될 수 있으므로 저장소에 올리지 않는 것을 권장합니다 (`.gitignore`에 `*.csv` 포함).
- LLM의 분류 결과는 연구 목적으로 사용할 때 **연구자의 직접 코딩과의 신뢰도 검증**(예: Cohen's κ)을 거치는 것이 좋습니다.
