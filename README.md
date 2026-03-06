"# middleProject_4team_202603" 

# 🐰 AI Math Tutor — 루미와 함께하는 초등 수학

> 초등학교 5학년 학생을 위한 AI 기반 1:1 수학 튜터링 서비스  
> LangGraph 워크플로우 + FastAPI 백엔드 + HTML/JS 프론트엔드

---

## 📌 프로젝트 개요

**AI Math Tutor**는 LLM(GPT-4o)과 LangGraph를 핵심 엔진으로 사용하는 교육용 AI 튜터입니다.  
토끼 캐릭터 **루미 선생님**이 개념 설명부터 문제 채점, 실시간 질의응답까지 학습 전 과정을 함께합니다.

### 핵심 학습 흐름

```
단원 선택 → 루미의 개념 설명 → 학생 역설명 → 이해도 평가(PASS/FAIL)
    ↓ FAIL                          ↓ PASS
보충 설명 ──────────────────→ 문제 풀기 → AI 채점 → 오답 저장
```

---

## 🏗️ 기술 스택

| 구분 | 기술 |
|------|------|
| AI 워크플로우 | LangGraph + LangChain |
| LLM | OpenAI GPT-4o |
| TTS | OpenAI TTS-1 (nova 보이스) |
| 백엔드 | FastAPI + Uvicorn |
| 인증 | JWT (python-jose) + bcrypt |
| DB | SQLite (via sqlite3) |
| 프론트엔드 | Vanilla HTML / CSS / JavaScript |

---

## 📁 프로젝트 구조

```
ai_math_tutor/
│
├── server.py                        # FastAPI 앱 진입점 (CORS, 라우터 등록, DB 초기화)
├── requirements.txt
├── .env.example                     # 환경변수 템플릿
│
├── app/
│   ├── routers/
│   │   ├── auth.py                  # POST /auth/login · GET /auth/me · POST /auth/logout
│   │   └── tutor.py                 # GET /api/units · /api/problem · POST /api/explain 등
│   │
│   ├── tutor/
│   │   └── integration.py           # ⭐ LangGraph 핵심 워크플로우 (AI 두뇌)
│   │
│   ├── services/
│   │   └── tutor_service.py         # 라우터 ↔ LangGraph 연결 서비스 계층
│   │
│   └── utils/
│       └── db_manager.py            # SQLite CRUD (users, learning_history)
│
├── frontend/
│   ├── login.html                   # 로그인 페이지
│   ├── app.html                     # 메인 앱 (사이드바 + SPA 컨테이너)
│   ├── assets/
│   │   ├── app.js                   # JWT 인증 + API 연동 + SPA 라우팅
│   │   └── style.css
│   └── pages/
│       ├── today.html               # 오늘 학습 (단원선택 → 채점까지 전 단계)
│       ├── report.html              # 학습 리포트
│       └── wrongnote.html           # 오답 노트
│
├── data/
│   └── processed/
│       └── math_tutor_dataset.csv   # 문제 데이터셋
│
├── database/                        # SQLite DB 파일 생성 위치 (자동 생성)
└── assets/
    └── audio/                       # TTS 음성 캐시 (자동 생성)
```

---

## ⭐ LangGraph 워크플로우 (`integration.py`)

이 프로젝트의 핵심입니다. 하나의 `StateGraph`가 세 가지 흐름을 통합합니다.

```
                    ┌─────────────────────┐
                    │    __start__         │
                    └────────┬────────────┘
                             │ entry_router(state)
              ┌──────────────┼──────────────┐
              │              │              │
       task="concept"  task="answer"   (없음)
              ↓              ↓              ↓
       eval_concept    eval_answer    get_units
              │              │              │
              ↓              ↓         get_problem
             END            END             │
                                           END
```

### State 필드

| 필드 | 용도 |
|------|------|
| `selected_unit` | 선택된 단원명 |
| `task_type` | `"concept"` / `"answer"` / `None` |
| `student_explanation` | 학생의 개념 역설명 |
| `student_answer` | 학생의 문제 풀이 |
| `feedback` | AI 평가 결과 (PASS/FAIL, 정답/오답) |
| `messages` | Q&A 대화 기록 (add_messages 누적) |
| `problem` | 현재 문제 dict |
| `units` | 전체 단원 목록 |

---

## 🚀 시작하기

### 1. 환경 설정

```bash
# 저장소 클론
git clone https://github.com/ilangs/middleProject_4team_202603

# 가상환경 생성 및 활성화
python -m venv .venv
.venv\Scripts\activate        # Windows

# 패키지 설치
pip install -r requirements.txt
```

### 2. 환경변수 설정

`.env` 파일을 만들고 아래 두 값을 채워주세요:

```env
OPENAI_API_KEY=sk-...
JWT_SECRET_KEY=<아래 명령어로 생성>
```

```bash
# JWT 비밀키 생성
python -c "import secrets; print(secrets.token_hex(32))"
```

### 3. 서버 실행

```bash
uvicorn server:app --reload --port 8000
```

### 4. 프론트엔드 실행

VS Code **Live Server** 확장을 설치 후 `frontend/login.html`을 오른쪽 클릭 → **Open with Live Server** (기본 포트: 5500)

| 서비스 | 주소 |
|--------|------|
| API 서버 | http://localhost:8000 |
| API 문서 (Swagger) | http://localhost:8000/docs |
| 프론트엔드 | http://localhost:5500/frontend/login.html |

### 5. 테스트 계정

| 아이디 | 비밀번호 |
|--------|---------|
| student01 | 1234 |

---

## 🔌 API 엔드포인트

### 인증

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/auth/login` | 로그인 → JWT 발급 |
| GET | `/auth/me` | 현재 사용자 정보 |
| POST | `/auth/logout` | 로그아웃 |

### 튜터 기능 (JWT 인증 필요)

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/units` | 전체 단원 목록 |
| GET | `/api/problem?unit=단원명` | 단원별 문제 조회 |
| POST | `/api/explain` | 루미의 개념 설명 생성 |
| POST | `/api/explain/evaluate` | 학생 역설명 이해도 평가 |
| POST | `/api/ask` | 실시간 Q&A (챗봇) |
| POST | `/api/evaluate` | 학생 답변 채점 |
| POST | `/api/history` | 학습 결과 저장 |
| GET | `/api/history` | 전체 학습 이력 조회 |
| GET | `/api/history/incorrect` | 오답 목록 조회 |

---

## 👥 팀 구성 및 담당

| 담당 | 파일 | 설명 |
|------|------|------|
| AI / LangGraph | `app/tutor/integration.py` | LangGraph 워크플로우, LangChain 체인, TTS |
| 백엔드 | `server.py`, `app/routers/` | FastAPI 서버, JWT 인증, API 라우터 |
| DB | `app/utils/db_manager.py` | SQLite 사용자/학습이력 관리 |
| 서비스 계층 | `app/services/tutor_service.py` | 라우터 ↔ LangGraph 연결 |
| 프론트엔드 | `frontend/` | HTML/CSS/JS, SPA 라우팅, API 연동 |

---

## 🔒 보안 구현 사항

- **비밀번호**: bcrypt 해싱 저장 (평문 저장 없음)
- **인증**: JWT Bearer 토큰 (만료 60분)
- **SQL Injection 방지**: 모든 쿼리 파라미터 바인딩(`?`) 적용
- **CORS**: 허용 오리진 명시적 지정

---

## 🗺️ 업데이트 예정

- [ ] **Frontend 개선** — UI/UX 고도화, LaTeX 수식 렌더링(MathJax), TTS 재생 연동
- [ ] **멀티모달 교신** — 이미지 문제 인식, 학생 손글씨 풀이 입력 지원
- [ ] **RAG 기반 Q&A** — ChromaDB 연동, 교과서 기반 검색 강화
- [ ] **LangGraph 확장** — 대화 기록 노드, 힌트 단계별 제공 노드 추가
- [ ] **학습 리포트 시각화** — 단원별 정답률 차트, 취약 단원 분석

---

## 📋 개발 환경

- Python 3.12
- Windows 11
- VS Code + Live Server 확장

---

## 📄 라이선스

본 프로젝트는 교육 목적으로 제작되었습니다.