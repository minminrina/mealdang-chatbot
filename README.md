# 밀당 (MealDang) — RAG FAQ Chatbot

**임신성 당뇨(GDM) 관련 질문에 한국어로 응답하는 FAQ 챗봇 모듈**

> **프로젝트 성격:** 3인 팀 졸업 프로젝트의 개인 담당 모듈  
> **담당 영역:** 챗봇 프론트엔드 · FastAPI 백엔드 · RAG 파이프라인 구현  
> **공개 범위:** 전체 밀당 서비스가 아닌 챗봇 기능 중심의 분리 저장소

## 프로젝트 소개

밀당은 임신성 당뇨 관리 지원을 목적으로 기획된 팀 프로젝트입니다. 이 저장소는 그중 **FAQ 챗봇** 기능을 담당하며, 사용자가 입력한 한국어 질문에서 혈당 수치와 측정 시점 등의 정보를 추출하고 관련 FAQ를 검색해 답변을 생성하도록 설계했습니다.

## 주요 기능

- **한국어 채팅 UI:** React Native에서 대화 내역, 입력창, 빠른 질문 선택지 및 로딩 상태 구현
- **질문 분석:** 식전/식후 시점, 혈당 수치 등 주요 표현 추출 및 의도 분류
- **FAQ 검색(RAG):** Sentence Transformers 임베딩과 ChromaDB를 사용한 관련 FAQ 검색
- **응답 생성:** 규칙 기반 답변 및 선택적 Gemini API 기반 자연어 생성, 오류 시 규칙 기반 폴백
- **REST API:** FastAPI의 `POST /chat`, `GET /health` 엔드포인트
- **앱–서버 연동:** Expo Router 기반 화면과 채팅 API 연결

## 기술 스택

| 구분 | 사용 기술 |
| --- | --- |
| Mobile Frontend | React Native, Expo, TypeScript, Expo Router |
| Backend | Python, FastAPI, Pydantic |
| Retrieval | ChromaDB, Sentence Transformers |
| Optional LLM | Gemini API (`google-genai`) |

## 처리 흐름

```text
React Native 채팅 화면
        │
        ▼
FastAPI POST /chat
        │
        ▼
질문 분석 (한국어 의도 분류 · 혈당/시점 추출)
        │
        ▼
Sentence Transformers + ChromaDB
        │
        ▼
관련 FAQ 근거 검색
        │
        ▼
규칙 기반 답변 또는 Gemini 답변
        │
        ▼
채팅 화면에 응답 표시
```

## 디렉터리 구조

```text
mealdang-fe/
├── app/faq-chat.tsx          # 챗봇 화면
├── components/ChatBubble.tsx # 메시지 UI
└── lib/
    ├── api.ts                # 백엔드 API 연결
    └── types.ts

mealdang-be/
├── main.py                   # FastAPI 서버
├── requirements.txt
└── rag/
    ├── gemini_answer.py      # 선택적 Gemini 답변
    ├── ingest_faq.py         # FAQ 임베딩/색인 생성
    ├── prompt_ko.py          # 한국어 분석 및 답변 구성
    └── retriever.py          # ChromaDB 검색
```

상세 설명: [Frontend README](mealdang-fe/README.md) · [Backend README](mealdang-be/README.md)

## 로컬 실행 안내

**1. 백엔드 의존성 설치**

```bash
cd mealdang-be
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**2. FAQ 데이터 준비 및 ChromaDB 색인 생성**

FAQ 파일을 `mealdang-be/knowledge/faq/faq.jsonl`에 배치한 다음:

```bash
python -m rag.ingest_faq
```

> **주의:** 원본 FAQ 데이터와 로컬 ChromaDB 인덱스는 이 공개 저장소에 포함되지 않습니다. 따라서 저장소만 클론해도 RAG 챗봇이 즉시 실행되는 상태는 아닙니다. 적절한 사용 권한이 있는 FAQ 데이터를 별도로 준비해야 합니다.

**3. FastAPI 서버 실행**

```bash
uvicorn main:app --reload --port 8000
```

`http://localhost:8000/health`에서 서버 상태를 확인할 수 있습니다.

**4. Expo 앱 실행** (별도 터미널)

```bash
cd mealdang-fe
npm install
npx expo start
```

실제 모바일 기기에서는 앱이 접근할 수 있는 개발 서버 주소를 설정해야 합니다. 자세한 연결 방법은 [Frontend README](mealdang-fe/README.md)를 참고해 주세요.

Gemini 기능은 선택 사항입니다. 서버 환경 변수 `USE_GEMINI=true`와 `GEMINI_API_KEY`를 설정하면 사용할 수 있으며, **실제 API 키를 Git에 커밋하지 마세요.**

## 개인 기여 범위

- React Native FAQ 챗봇 화면 구현 및 FastAPI 연동
- 한국어 질문의 의도/수치/측정 시점 추출 로직 구성
- 다국어 임베딩 모델과 ChromaDB를 활용한 FAQ 검색 파이프라인 구축
- 검색 근거 기반 응답 구성 및 선택적 Gemini 연동

## 참고

이 프로젝트는 교육 및 포트폴리오 용도의 프로토타입입니다. 챗봇 응답은 의료진의 진단, 치료 또는 개인별 의학적 조언을 대신하지 않습니다.
