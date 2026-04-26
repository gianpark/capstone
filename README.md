# 🖥️ 견피티 (GeonPiTi) — 개발 진행 보고서

> **캡스톤 디자인 2026년 1학기**  
> AI & 유튜브 기반 PC 부품 종합 정보 플랫폼  
> 작성일: 2026년 4월 26일


---

## 동작영상 : https://youtu.be/LR3T-Mydtzo

## 1. 프로젝트 개요

### 기획 배경

PC 견적을 처음 접하는 사용자는 수십 가지 부품의 호환성, 가격, 성능을 스스로 판단하기 어렵습니다. 기존 다나와·퀘이사존 같은 플랫폼은 정보가 방대하지만 초보자 친화적이지 않고, AI 기반 조언 기능이 없습니다.

**견피티**는 이 문제를 해결하기 위해 세 가지 핵심 가치를 제공합니다:

1. 실시간 다나와 크롤링으로 항상 최신 가격 정보 제공
2. Groq LLM 기반 AI가 부품 진단 · 견적 검증 · 자연어 추천을 자동화
3. YouTube 영상 자막을 AI로 요약해 리뷰 탐색 시간을 단축

**총 추가 비용: 0원** (Groq, YouTube, 네이버 쇼핑 API 모두 무료 플랜 활용)

---

## 2. 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                     Client (Browser)                     │
│         Jinja2 Templates + Vanilla JS + Chart.js         │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP
┌──────────────────────▼──────────────────────────────────┐
│                   Flask Application                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐  │
│  │  /parts  │ │  /quote  │ │    /ai   │ │/community │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └─────┬─────┘  │
│       │            │            │              │         │
│  ┌────▼────────────▼────────────▼──────────────▼──────┐ │
│  │              SQLAlchemy ORM (SQLite)                │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  danawa      │  │  Groq API    │  │  YouTube     │   │
│  │  crawler     │  │  (LLaMA 3.3) │  │  Data API v3 │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                 │                  │           │
│  ┌──────▼───────┐  ┌──────▼───────┐          │           │
│  │  APScheduler │  │  ai_diagnose │          │           │
│  │  (가격 자동  │  │  ai_validate │          │           │
│  │   업데이트)  │  │  ai_recommend│          │           │
│  └──────────────┘  └──────────────┘          │           │
│                                   ┌──────────▼───────┐   │
│                                   │  youtube_transcript│  │
│                                   │  _api (자막 수집) │   │
│                                   └──────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## 3. 구현 기능 현황

### ✅ 완료

#### 3.1 부품 정보 시스템

| 기능 | 설명 |
|------|------|
| 부품 DB | 8개 카테고리 (CPU / GPU / RAM / MB / SSD / PSU / CASE / COOLER), 230+ 부품 |
| 다나와 크롤러 | `div.items` 구조 파싱, 가격/이미지/스펙 자동 수집 |
| 스펙 재수집 | 다나와 HTML 구조 변경 대응 → 전체 DB 일괄 재파싱 기능 |
| 가격 이력 | 신품/중고 날짜별 가격 이력 저장, Chart.js 가격 추이 차트 |
| 부품 비교 | 동일 카테고리 유사 부품 최대 3개 사이드바이사이드 비교 |
| 글로벌 검색 | 네비게이션 바 전체 텍스트 검색 + 실시간 드롭다운 |
| 인기 랭킹 | 카테고리별 찜 수 기반 랭킹 |

#### 3.2 AI 기능

| 기능 | API | 모델 | 설명 |
|------|-----|------|------|
| 내 부품 진단 | `POST /api/ai-diagnose` | Groq llama-3.3-70b | 현재 스펙 → 업그레이드 우선순위 3개 + DB 부품 연결 |
| 견적 AI 검증 | `POST /api/ai-validate` | Groq llama-3.3-70b | 호환성·병목 지수(0~100)·가성비 점수(0~100) |
| 자연어 추천 | `POST /api/ai-recommend` | Groq llama-3.3-70b | "50만원으로 롤 하고 싶어요" → 부품 조합 추천 |
| 유튜브 리뷰 요약 | `GET /api/review-summary` | Groq llama-3.3-70b | 자막 수집 → 장점/단점/한줄 요약 AI 생성 |

> **AI 전환 이력**: 초기 Ollama(로컬) → Google Gemini API → Groq API  
> Gemini API 무료 플랜 quota 0 문제 발생 → Groq(무료 30req/min, 14,400/day)으로 최종 교체

#### 3.3 견적 구성

| 기능 | 설명 |
|------|------|
| 드래그&드롭 빌더 | 8개 슬롯 부품 검색·선택 |
| 용도별 프리셋 | 게이밍/영상편집/개발/사무 키워드 기반 자동 필터링 |
| AI 검증 버튼 | 구성된 견적 → 실시간 호환성·병목·가성비 분석 |
| 견적 저장·공유 | 로그인 사용자 견적 저장, URL 공유 |

#### 3.4 커뮤니티

| 기능 | 설명 |
|------|------|
| 피드 | 견적 공유, 좋아요, 댓글 |
| 관심 목록 | 부품 찜하기 (Watchlist) |
| 인기 랭킹 | 카테고리별 조회수/찜수 기반 |

#### 3.5 인증

| 기능 | 설명 |
|------|------|
| Google OAuth 2.0 | Authlib 기반 소셜 로그인 |
| Flask-Login 세션 | 로그인 상태 유지 |

#### 3.6 메인 페이지

| 기능 | 설명 |
|------|------|
| 인기 GPU/CPU | 성능 점수 기준 Top 4 카드 |
| 최근 가격 하락 | 7일 대비 하락률 상위 부품 |
| 시장 온도계 | CPU/GPU/RAM/SSD 가격 변동 시각화 |
| YouTube 추천 영상 | DB 인기 부품 기반 리뷰 영상 자동 수집 (YouTube API) |

---

## 4. 기술적 해결 과제

### 4.1 다나와 HTML 구조 변경 대응

**문제**: 기존 `.spec-list table tr` 셀렉터가 동작하지 않아 전체 부품의 스펙이 `None`으로 저장됨

**원인 분석**: 다나와 상품 상세 페이지의 스펙 HTML 구조가 변경됨
```
(구) <table class="spec-list"><tr><th>코어 수</th><td>8코어</td></tr>...</table>
(신) <div class="items">인텔(소켓1851) / P6+E12코어 / 18스레드 / 기본 클럭 : 4.2GHz / ...</div>
```

**해결 방법**: `fetch_product_spec` 함수를 전면 재작성
```python
# 신 구조: div.items 텍스트 파싱
for div in soup.select("div.items"):
    raw = div.get_text(separator=" / ", strip=True)
    for chunk in re.split(r"\s*/\s*", raw):
        if ":" in chunk:
            k, _, v = chunk.partition(":")
            spec[k.strip()] = v.strip()   # key:value
        else:
            tags.append(chunk)             # 독립 태그 ("P6+E12코어" 등)
```

`map_spec_to_part`도 `__tags` 리스트 기반 정규식 파싱으로 재설계:
- CPU: `P(\d+)\+E(\d+)코어` → P코어+E코어 합산, `소켓([A-Za-z0-9]+)` → 소켓 추출
- GPU: `(\d+)\s*GB` → VRAM, `(GDDR\d+X?)` → VRAM 타입
- RAM: `(DDR\d+)` → 규격, `\b(\d{4,5})\b` → 동작 클럭

**추가 대응**: `/admin/rescrape` API와 관리자 페이지 버튼으로 기존 DB 전체 일괄 재파싱 기능 구현

---

### 4.2 YouTube 자막 수집 안정화

**문제**: 한국어 자막이 없는 영상에서 `TranscriptsDisabled` 오류 발생

**해결 방법**: 다중 폴백 전략 적용
```
1차: ['ko'] 자막
2차: ['ko-KR'] 자막
3차: ['en'] 자막
4차: 자동 생성 자막 (generated)
5차: 자막 없으면 영상 제목+설명으로 요약 대체
```

---

### 4.3 AI API 전환 (Gemini → Groq)

**문제**: Google Gemini API 무료 플랜 quota 0 (`free_tier_requests=0`)으로 사실상 사용 불가

**해결**: Groq API로 전환
- 모델: `llama-3.3-70b-versatile`
- 무료 한도: 30 req/min, 14,400 req/day
- 기존 `gemini_service.py` 파일명 유지, 내부 구현만 교체 → 서비스 레이어 변경 최소화

---

### 4.4 danawa_crawler.py 파일 손상 복구

**문제**: VM 터미널에서 heredoc 교체 명령 실행 중 마커 탐색 실패(`start=-1`)로 문자열이 238,000줄 이상 반복되는 파일 손상 발생

**해결**: base64 인코딩 방식으로 파일 교체
```bash
python3 -c "
import base64
data = '<base64 encoded content>'
with open('app/services/danawa_crawler.py', 'wb') as f:
    f.write(base64.b64decode(data))
"
```

---

## 5. 데이터베이스 스키마

```
parts                    price_histories         used_price_histories
────────────────         ────────────────        ────────────────────
id (PK)                  id (PK)                 id (PK)
name                     part_id (FK)            part_id (FK)
brand                    price                   price
category                 recorded_at             recorded_at
series                   source                  source
current_price
used_price               quotes                  quote_items
image_url                ────────────            ────────────
naver_product_id         id (PK)                 id (PK)
release_year             user_id (FK)            quote_id (FK)
tdp                      title                   part_id (FK)
[CPU 스펙 컬럼 6개]       budget                  category
[GPU 스펙 컬럼 5개]       usage                   quantity
[RAM 스펙 컬럼 4개]       is_public
[MB 스펙 컬럼 6개]        created_at              users
[SSD 스펙 컬럼 4개]                               ────────────
[PSU 스펙 컬럼 2개]       community_posts         id (PK)
[CASE 스펙 컬럼 3개]      ────────────            email
[COOLER 스펙 컬럼 3개]    id (PK)                 name
performance_score         user_id (FK)            profile_image
value_score               quote_id (FK)           google_id
ai_review_cache           content                 created_at
ai_review_updated_at      likes_count
created_at                created_at
updated_at
```

---

## 6. 화면 구성

| 페이지 | URL | 설명 |
|--------|-----|------|
| 메인 | `/` | 인기 부품 카드, 가격 하락 알림, 시장 온도계, YouTube 추천 영상 |
| 부품 목록 | `/parts/?category=GPU` | 카테고리 필터, 정렬, 검색 |
| 부품 상세 | `/parts/<id>` | 스펙 전체, 가격 추이 차트, AI 리뷰 요약, 비교하기 |
| 부품 비교 | `/parts/compare` | 최대 3개 나란히 스펙 비교 |
| 견적 구성 | `/quote/` | 드래그&드롭 빌더, 프리셋, AI 검증 |
| 내 견적 | `/quote/my` | 저장된 견적 목록 |
| AI 진단 | `/ai/diagnose` | 현재 스펙 입력 → 업그레이드 분석 |
| 커뮤니티 | `/community/feed` | 견적 공유 피드 |
| 인기 랭킹 | `/community/ranking` | 카테고리별 인기 순위 |
| 관리자 | `/admin/` | 크롤링 실행, 스펙 재수집 |

---

## 7. 향후 개선 계획

| 항목 | 우선순위 | 설명 |
|------|----------|------|
| Google OAuth 리다이렉트 URI 등록 | 높음 | Google Cloud Console 설정 필요 (현재 401 오류) |
| 스펙 재수집 실행 | 높음 | `/admin/` 에서 전체 230+ 부품 스펙 업데이트 |
| 성능 점수 자동화 | 중간 | 현재 수동 입력 → Passmark/UserBenchmark 점수 연동 |
| 중고 가격 수집 | 중간 | 번개장터/당근마켓 API 또는 크롤링 |
| 모바일 반응형 개선 | 낮음 | 현재 데스크탑 위주 레이아웃 |
| 견적 댓글 기능 | 낮음 | 커뮤니티 피드 상호작용 강화 |

---

## 8. 실행 환경

| 항목 | 내용 |
|------|------|
| OS | Ubuntu 22.04 (VMware Workstation) |
| Python | 3.12 |
| 웹 프레임워크 | Flask 3.0.3 |
| DB | SQLite (개발) |
| AI | Groq API — llama-3.3-70b-versatile |
| 배포 | Flask dev server (포트 5000) |
| 추가 비용 | **0원** |

---

## 9. 팀 구성

| 역할 | 담당 |
|------|------|
| 기획 / 풀스택 개발 | 1인 개발 (캡스톤 디자인) |

---

*© 2026 견피티 (GeonPiTi) | 캡스톤 디자인 프로젝트*
