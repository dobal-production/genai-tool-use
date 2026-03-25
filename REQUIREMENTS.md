# 주식 분석 AI 애플리케이션 요구사항

## 프로젝트 개요

YFinance 라이브러리와 Amazon Bedrock Converse API의 Tool Use 기능을 활용하여 특정 주식 종목을 분석하는 교육용 Streamlit 애플리케이션입니다.

---

## 학습 목표

- **Gen AI Tool Use** 패턴 이해 (AI가 직접 도구를 호출하는 방식)
- **Amazon Bedrock Converse API** 사용법 습득
- **YFinance** API를 통한 주식 데이터 수집 방법 습득
- **Streamlit**으로 빠르게 AI 애플리케이션 UI 구성하기

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| UI | Streamlit |
| AI | Amazon Bedrock Converse API |
| AI 모델 | `anthropic.claude-sonnet-4-5` |
| 주식 데이터 | yfinance |
| 언어 | Python 3.14 |
| 패키지 관리 | uv |

---

## 주요 기능

### 1. 종목 조회 및 채팅 인터페이스
- 사이드바에서 주식 티커(Ticker) 입력 (예: `AAPL`, `005930.KS`)
- 채팅 형태로 AI에게 주식 관련 질문 입력

### 2. AI Tool Use (핵심 기능)
AI는 사용자의 질문에 답하기 위해 아래 도구들을 **자동으로 선택하여 호출**합니다.

| 도구 이름 | 설명 |
|-----------|------|
| `get_stock_price` | 현재 주가 및 기본 정보 조회 |
| `get_stock_history` | 최근 N일간 주가 히스토리 조회 |
| `get_financials` | 재무제표 데이터 조회 (매출, 영업이익 등) |
| `get_stock_news` | 종목 관련 최신 뉴스 조회 (yfinance 내장 기능) |

### 3. 분석 결과 표시
- AI의 텍스트 분석 결과를 채팅 버블로 표시
- 주가 히스토리 요청 시 라인 차트 시각화

---

## 화면 구성 (UI Layout)

```
+----------------------------------+------------------------------------------+
|  사이드바                         |  메인 화면                                |
|                                  |                                          |
|  [티커 입력]  예: AAPL            |  채팅 메시지 영역                         |
|                                  |  - 사용자 메시지                          |
|  분석 예시 질문:                  |  - AI 응답 (Tool Use 결과 포함)           |
|  - 현재 주가 알려줘               |  - 필요시 차트 표시                       |
|  - 최근 한달 주가 흐름 분석해줘   |                                          |
|  - 재무 상태 요약해줘             |  [메시지 입력창]                          |
|  - 최근 뉴스 알려줘              |                                          |
+----------------------------------+------------------------------------------+
```

---

## 파일 구조

```
fsi-demo-genai-tool-use/
├── app.py                  # Streamlit 메인 앱 (UI + 흐름 제어)
├── tools.py                # YFinance Tool 정의 (schema + 실행 함수)
├── utils/
│   └── bedrock.py          # Bedrock Converse API 호출 (Tool Use 루프)
├── pyproject.toml
└── REQUIREMENTS.md
```

---

## 의존성 패키지

```toml
dependencies = [
    "streamlit>=1.55.0",
    "boto3>=1.35.0",
    "yfinance>=0.2.0",
    "pandas>=2.0.0",
]
```

---

## Tool Use 동작 흐름

```
사용자 입력
    │
    ▼
utils/bedrock.py - Converse API 호출 (tools 포함)
    │
    ▼
stopReason == "tool_use"?
    │
   YES ──▶ tools.py - YFinance 데이터 조회
    │           │
    │           ▼
    │      toolResult 포함하여 Converse API 재호출
    │           │
    │           ▼
   NO  ◀── 최종 텍스트 응답 반환
    │
    ▼
app.py - Streamlit 화면에 결과 표시
```

---

## 구현 제약사항 (교육용 단순화)

- Tool Use 루프는 **최대 1회** (단순성 유지)
- 에러 처리는 최소화 (핵심 흐름 집중)
- 스트리밍 없이 **단건 응답** 방식
- 차트는 `st.line_chart`로 단순하게 표시
- AWS 자격증명은 **EC2 IAM Role** 사용 (코드에 키 하드코딩 없음, boto3가 자동으로 인스턴스 메타데이터에서 획득)
