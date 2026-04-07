# 🖥️ 견피티 (GeonPiTi)
**AI & YouTube 기반 PC 부품 정보 플랫폼**

> 2025 캡스톤 디자인 | 1인 프로젝트 | Python Flask

---

## 📌 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | 견피티 (GeonPiTi) — 견적 + PC + 티어 |
| 개발 목적 | PC 부품 선택이 어려운 일반 사용자를 위한 AI 추천 + 유튜브 리뷰 통합 플랫폼 |
| 개발 형태 | 1인 캡스톤 디자인 |
| 개발 환경 | Ubuntu 24.04 + VMware Workstation + VSCode SSH |
| 기술 스택 | Python 3.11 · Flask · SQLAlchemy · SQLite → PostgreSQL |
| 버전 | v0.7 (2025.04 기준) |

---

## 🗺️ 시스템 아키텍처 블록도

```
┌─────────────────────────────────────────────────────────────────┐
│                        사용자 (Browser)                          │
│              Chrome / Safari / Mobile Web                        │
└───────────────────────────┬─────────────────────────────────────┘
                            │  HTTP Request
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Flask 웹 서버 (run.py)                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   라우터 (Blueprints)                     │   │
│  │  /main  │  /parts  │  /quote  │  /auth  │  /community    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            │                                     │
│  ┌─────────────┐  ┌────────▼────────┐  ┌────────────────────┐  │
│  │  Jinja2     │  │   비즈니스 로직  │  │   Flask-Login      │  │
│  │  Templates  │  │  (routes/*.py)  │  │   세션 관리         │  │
│  └─────────────┘  └────────┬────────┘  └────────────────────┘  │
│                            │                                     │
│  ┌─────────────────────────▼────────────────────────────────┐   │
│  │               SQLAlchemy ORM (models/)                    │   │
│  │   Part │ PriceHistory │ User │ Quote │ QuoteItem │ Comment │  │
│  └─────────────────────────┬────────────────────────────────┘   │
└────────────────────────────┼────────────────────────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
┌──────────────┐   ┌──────────────────┐  ┌───────────────────┐
│   SQLite DB  │   │   외부 API       │  │  APScheduler       │
│   (개발용)   │   │                  │  │  (자동 가격 수집)  │
│              │   │ 네이버 쇼핑 API  │  │                    │
│  배포 시     │   │ YouTube Data v3  │  │  6시간 주기        │
│ PostgreSQL   │   │ 카카오 OAuth     │  │  가격 업데이트     │
│  전환 예정   │   │ 구글 OAuth       │  │  + PriceHistory    │
└──────────────┘   │ 환율 API (예정)  │  │  기록              │
                   └──────────────────┘  └───────────────────┘
```

### 데이터 흐름

```
[가격 수집]
APScheduler (6h) → naver_api.py → Part.current_price 업데이트 → PriceHistory 기록

[유튜브 리뷰]
사용자 요청 → youtube_api.py → 키워드 검색 → 영상 목록 반환 → 메인 페이지 표시

[호환성 체크]
견적 부품 선택 → /parts/compatibility → CPU소켓·RAM타입·파워용량·쿨러높이 검증 → 결과 표시

[로그인 흐름]
카카오/구글 OAuth 콜백  ┐
자체 이메일+비밀번호    ├→ _get_or_create_user() → Flask-Login 세션
```

---

## 📁 파일 구조

```
geonpiti/
├── run.py                      # 앱 실행 진입점
├── requirements.txt            # 패키지 목록
├── .env                        # API 키 (비공개)
│
└── app/
    ├── __init__.py             # Flask 앱 팩토리 + DB 초기화
    ├── seed_data.py            # 더미 부품 데이터 (52개)
    │
    ├── models/
    │   ├── part.py             # 부품 모델 (CPU/GPU/RAM 등 전체 스펙)
    │   ├── user.py             # 사용자 모델 (소셜 + 자체 로그인)
    │   ├── quote.py            # 견적 + 견적 아이템
    │   └── community.py        # 커뮤니티 게시글 + 댓글
    │
    ├── routes/
    │   ├── main.py             # 메인 페이지, 뉴스, 추천 영상 API
    │   ├── parts.py            # 부품 목록/상세/비교/호환성
    │   ├── quote.py            # 견적 빌더, 저장, 공유
    │   ├── auth.py             # 카카오/구글 OAuth + 자체 로그인
    │   └── community.py        # 커뮤니티 피드, 게시글 CRUD
    │
    ├── services/
    │   ├── naver_api.py        # 네이버 쇼핑 API (가격 조회)
    │   ├── youtube_api.py      # YouTube Data API v3 (리뷰 검색)
    │   ├── scheduler.py        # APScheduler 가격 자동 수집
    │   └── news_api.py         # 네이버 뉴스 API
    │
    ├── templates/
    │   ├── base.html           # 공통 레이아웃 (네비게이션, 푸터)
    │   ├── auth/
    │   │   └── login.html      # 로그인 페이지 (탭: 소셜 / 자체)
    │   ├── main/
    │   │   └── index.html      # 메인 (히어로 + 유튜브 + 기능 소개)
    │   ├── parts/
    │   │   ├── list.html       # 부품 목록 + 카테고리 필터
    │   │   ├── detail.html     # 부품 상세 + 가격 그래프
    │   │   └── compare.html    # 동일 시리즈 비교 + 레이더 차트
    │   ├── quote/
    │   │   ├── builder.html    # 견적 빌더 (프리셋 + 호환성 체크)
    │   │   ├── my_quotes.html  # 내 견적 목록
    │   │   └── share.html      # 견적 공유 페이지
    │   └── community/
    │       └── feed.html       # 커뮤니티 피드
    │
    └── static/
        ├── css/style.css       # 전역 스타일
        └── js/main.js          # 공통 JS
```

---

## ✅ 구현 현황

| # | 기능 | 상태 | 비고 |
|---|------|:----:|------|
| 1 | 부품 DB 구축 (52개 시드 데이터) | ✅ 완료 | CPU·GPU·RAM·MB·SSD·PSU·CASE·COOLER |
| 2 | 부품 목록 페이지 (카테고리 필터) | ✅ 완료 | |
| 3 | 부품 상세 페이지 | ✅ 완료 | 스펙 전체 표시 |
| 4 | 가격 히스토리 그래프 | ✅ 완료 | Chart.js 라인 차트 |
| 5 | 동일 시리즈 비교 + 레이더 차트 | ✅ 완료 | Chart.js Radar |
| 6 | 견적 빌더 (부품 추가/삭제) | ✅ 완료 | JSON 충돌 버그 수정 |
| 7 | 견적 프리셋 (게이밍/3D/사무/개발) | ✅ 완료 | 키워드 매칭 자동 선택 |
| 8 | 호환성 체크 | ✅ 완료 | 소켓·RAM·파워·쿨러 검증 |
| 9 | 견적 저장 & 공유 | ✅ 완료 | URL 공유 방식 |
| 10 | 카카오 로그인 (OAuth) | ⚙️ 키 필요 | `.env` 설정 시 활성화 |
| 11 | 구글 로그인 (OAuth) | ⚙️ 키 필요 | `.env` 설정 시 활성화 |
| 12 | 자체 이메일/비밀번호 로그인 | ✅ 완료 | bcrypt 해시 저장 |
| 13 | 네이버 쇼핑 API 가격 수집 | ✅ 완료 | 키 설정됨 |
| 14 | APScheduler 자동 가격 업데이트 | ✅ 완료 | 6시간 주기 |
| 15 | YouTube 리뷰 영상 메인 표시 | ✅ 완료 | API 미설정 시 Mock 데이터 |
| 16 | 커뮤니티 피드 | ⚙️ 개발 중 | 기본 CRUD 구현됨 |

---

## 🔑 API 키 설정 (`.env`)

```env
# 네이버 쇼핑/뉴스 API (설정됨)
NAVER_CLIENT_ID=NMlkZguRB6c40GSYzKwi
NAVER_CLIENT_SECRET=dF6Mj4_e5a

# YouTube Data API v3 (발급 완료, 키 입력 필요)
YOUTUBE_API_KEY=your_youtube_api_key_here

# 카카오 OAuth (키 발급 필요)
KAKAO_CLIENT_ID=your_kakao_rest_api_key
KAKAO_REDIRECT_URI=http://localhost:5000/auth/kakao/callback

# 구글 OAuth (키 발급 필요)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=http://localhost:5000/auth/google/callback

# Flask
SECRET_KEY=your_secret_key_here
```

---

## 🚀 실행 방법

```bash
# 1. 가상환경 활성화
cd ~/geonpiti
source venv/bin/activate

# 2. 패키지 설치 (최초 1회)
pip install -r requirements.txt

# 3. DB 마이그레이션 (최초 또는 모델 변경 후)
flask db init        # 최초 1회만
flask db migrate -m "init"
flask db upgrade

# 4. 시드 데이터 입력 (최초 1회)
python3 -c "from app import create_app, db; from app.seed_data import seed; app=create_app(); app.app_context().push(); seed()"

# 5. 서버 실행
python3 run.py
# → http://localhost:5000 접속
```

---

## 🛣️ 향후 개발 계획

| 우선순위 | 기능 | 설명 |
|:--------:|------|------|
| 🔴 High | YouTube API 연동 완성 | 실제 리뷰 영상 API 호출 |
| 🔴 High | 카카오/구글 OAuth 키 등록 | 소셜 로그인 활성화 |
| 🟡 Mid | AI 부품 추천 | 예산·용도 입력 → GPT/로컬 모델 추천 |
| 🟡 Mid | 커뮤니티 기능 완성 | 게시글 작성·댓글·좋아요 |
| 🟢 Low | PostgreSQL 전환 | 배포 환경용 DB 마이그레이션 |
| 🟢 Low | 환율 API 연동 | 해외 직구 가격 비교 |

---

> 📄 상세 기획서: `geonpiti_기획서_v2.docx`
