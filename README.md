# 🖥️ 견피티 (GeonPiTi)

> **AI & 유튜브 기반 PC 부품 종합 정보 플랫폼**  
> 캡스톤 디자인 2026년 1학기 | 혼자서도 완벽한 PC 견적을 짤 수 있는 서비스

---

## 프로젝트 개요

견피티는 PC 부품 구매를 처음 접하는 사용자도 쉽게 최적의 견적을 구성할 수 있도록 돕는 플랫폼입니다. 다나와 실시간 크롤링으로 수집한 부품 데이터와 Groq LLM, YouTube Data API를 결합해 AI 기반 진단·추천·리뷰 요약 기능을 제공합니다.

**총 추가 비용: 0원** (모든 API 무료 플랜 활용)

---

## 주요 기능

### 부품 정보
- GPU / CPU / RAM / MB / SSD / PSU / CASE / COOLER 8개 카테고리
- 다나와 실시간 크롤링으로 최저가 자동 수집 (APScheduler)
- 신품/중고 가격 이력 차트 (Chart.js)
- 유사 부품 비교하기 (성능 점수 기반)
- 전체 텍스트 검색 (글로벌 네비게이션 바)

### AI 기능
| 기능 | 설명 |
|------|------|
| 🤖 내 부품 진단 | 현재 PC 스펙 입력 → 업그레이드 우선순위 분석 + DB 부품 연결 |
| ✅ 견적 AI 검증 | 선택한 견적 구성의 호환성·병목·가성비 자동 검증 |
| 📝 유튜브 리뷰 요약 | 부품별 YouTube 영상 자막 수집 → 장단점 AI 요약 |
| 💬 자연어 견적 추천 | "50만원으로 롤 하고 싶어요" → AI 부품 추천 |

### 커뮤니티
- 견적 공유 피드 & 좋아요
- 카테고리별 인기 부품 랭킹
- 관심 부품 찜하기 (Watchlist)

### 견적 구성
- 드래그&드롭 방식 부품 선택
- 용도별 추천 프리셋 (게이밍 / 영상편집 / 개발 / 사무)
- 견적 저장·공유 (로그인 필요)

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| Backend | Python 3.12, Flask 3.0, SQLAlchemy, Flask-Login, Flask-Migrate |
| Database | SQLite (개발), PostgreSQL 호환 |
| Frontend | Jinja2, Vanilla JS, Chart.js, CSS Variables (다크모드 대응) |
| AI | Groq API (llama-3.3-70b-versatile) |
| 크롤링 | BeautifulSoup4, lxml, requests |
| 스케줄링 | APScheduler |
| 인증 | Flask-Login + Authlib (Google OAuth) |
| 배포 환경 | Ubuntu 22.04 (VMware), Python venv |

---

## 프로젝트 구조

```
geonpiti/
├── app/
│   ├── __init__.py              # Flask 앱 팩토리
│   ├── models/
│   │   ├── part.py              # Part, PriceHistory, UsedPriceHistory
│   │   ├── user.py              # User (Google OAuth)
│   │   ├── quote.py             # Quote, QuoteItem
│   │   └── community.py        # Post, Like, Watchlist
│   ├── routes/
│   │   ├── main.py              # 메인 페이지, 통계 API
│   │   ├── parts.py             # 부품 목록/상세/비교
│   │   ├── quote.py             # 견적 구성/저장/공유
│   │   ├── community.py        # 피드/랭킹
│   │   ├── auth.py              # 로그인/로그아웃/Google OAuth
│   │   ├── ai.py                # AI 진단/검증/추천/리뷰 API
│   │   └── admin.py             # 크롤링 관리 페이지
│   ├── services/
│   │   ├── danawa_crawler.py    # 다나와 크롤러 (목록+스펙+가격)
│   │   ├── gemini_service.py    # Groq API 클라이언트
│   │   ├── ai_diagnose.py       # 내 부품 진단 / 견적 검증 로직
│   │   ├── ai_recommend.py      # 자연어 견적 추천 로직
│   │   ├── ai_review.py         # YouTube 리뷰 요약 로직
│   │   ├── youtube_api.py       # YouTube Data API
│   │   ├── naver_api.py         # 네이버 쇼핑 API (가격)
│   │   └── scheduler.py         # 가격 자동 업데이트 스케줄러
│   ├── templates/
│   │   ├── base.html            # 공통 레이아웃 (네비/푸터)
│   │   ├── main/index.html      # 메인 페이지
│   │   ├── parts/               # 부품 목록/상세/비교
│   │   ├── quote/               # 견적 구성/저장
│   │   ├── community/           # 피드/랭킹
│   │   ├── auth/                # 로그인
│   │   └── ai/diagnose.html     # AI 진단 페이지
│   └── static/
│       ├── css/style.css        # 전체 스타일 (CSS 변수 기반)
│       ├── js/main.js           # 공통 JS (검색, 다크모드)
│       ├── manifest.json        # PWA 설정
│       └── sw.js                # Service Worker
├── run.py                       # 앱 실행 진입점
├── requirements.txt
└── .env.example
```

---

## 설치 및 실행

### 1. 환경 변수 설정

```bash
cp .env.example .env
```

`.env` 파일에 아래 값을 채워주세요:

```env
SECRET_KEY=your-secret-key
DATABASE_URL=sqlite:///geonpiti.db

# Groq API (무료: https://console.groq.com)
GROQ_API_KEY=your-groq-api-key

# YouTube Data API v3 (무료: Google Cloud Console)
YOUTUBE_API_KEY=your-youtube-api-key

# 네이버 쇼핑 API (무료: https://developers.naver.com)
NAVER_CLIENT_ID=your-naver-client-id
NAVER_CLIENT_SECRET=your-naver-client-secret

# Google OAuth (선택, 로그인 기능)
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

### 2. 패키지 설치

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. DB 초기화 및 데이터 수집

```bash
flask db init
flask db migrate -m "init"
flask db upgrade
```

### 4. 서버 실행

```bash
flask run --host=0.0.0.0 --port=5000
```

브라우저에서 `http://localhost:5000` 접속

---

## 관리자 기능

`http://localhost:5000/admin/` 에서 다나와 크롤링을 실행할 수 있습니다.

- **스펙 재수집**: 기존 DB 부품의 스펙 정보를 최신 다나와 HTML 구조로 재파싱
- **카테고리별 크롤링**: GPU, CPU 등 특정 카테고리 신규 부품 수집
- **전체 크롤링**: 8개 카테고리 일괄 수집

---

## API 엔드포인트

| Method | URL | 설명 |
|--------|-----|------|
| GET | `/` | 메인 페이지 |
| GET | `/parts/` | 부품 목록 (카테고리 필터) |
| GET | `/parts/<id>` | 부품 상세 |
| GET | `/parts/compare` | 부품 비교 |
| GET | `/quote/` | 견적 구성 빌더 |
| GET | `/ai/diagnose` | AI 부품 진단 페이지 |
| POST | `/api/ai-diagnose` | AI 진단 API |
| POST | `/api/ai-validate` | 견적 AI 검증 API |
| POST | `/api/ai-recommend` | 자연어 견적 추천 API |
| GET | `/api/review-summary` | YouTube 리뷰 요약 API |
| GET | `/api/featured-videos` | 메인 추천 영상 API |
| GET | `/community/feed` | 커뮤니티 피드 |
| GET | `/community/ranking` | 인기 랭킹 |

---

## 데이터 모델

### Part (부품)
- 8개 카테고리 × 카테고리별 상세 스펙 컬럼
- 신품/중고 현재가 + 가격 이력
- AI 리뷰 캐시 (YouTube 요약 결과)
- 성능 점수 / 가성비 점수

### PriceHistory / UsedPriceHistory
- 날짜별 신품/중고 가격 이력 (가격 추이 차트용)

### Quote / QuoteItem
- 사용자 견적 구성 저장 및 공유

### User
- Google OAuth 기반 인증

---

## 라이선스

캡스톤 디자인 프로젝트 — 학술 목적 사용만 허용
