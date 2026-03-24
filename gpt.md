# 견피티 (GeonPiTi) 시스템 블럭도

---

## 1. 시스템 전체 아키텍처

```mermaid
flowchart TD
    subgraph CLIENT["🖥️ 클라이언트"]
        WEB["웹 브라우저"]
        PWA["PWA (모바일)"]
    end

    subgraph FLASK["⚙️ Python Flask 서버"]
        ROUTER["라우터 / API 엔드포인트"]
        AUTH["소셜 로그인\n카카오 / 구글 OAuth"]
        ALARM["가격 알림\n푸시 / 이메일"]
    end

    subgraph DATA["📡 데이터 수집 모듈"]
        NAVER_SHOP["네이버 쇼핑 API\n실시간 최저가"]
        NAVER_NEWS["네이버 뉴스 API\n반도체 관련 뉴스"]
        EXCHANGE["한국수출입은행 API\n환율 정보"]
        YOUTUBE["YouTube Data API\n리뷰 / 비교 영상"]
    end

    subgraph AI["🤖 AI 분석 모듈"]
        NLP["HuggingFace NLP\n뉴스 감성 분석"]
        SUMMARY["텍스트 요약 모델\n유튜브 자막 요약"]
        PRICE_REASON["가격 변동\n원인 분석"]
        WEIGHT["유튜버 신뢰도\n가중치 산정"]
    end

    subgraph DB["🗄️ 데이터베이스"]
        PARTS_DB["부품 정보 DB\n스펙 / 브랜드"]
        PRICE_DB["가격 이력 DB\n날짜별 시세"]
        USER_DB["유저 DB\n견적 / 관심 등록"]
        COMMUNITY_DB["커뮤니티 DB\n댓글 / 좋아요"]
    end

    WEB & PWA --> ROUTER
    ROUTER --> AUTH
    ROUTER --> DATA
    ROUTER --> AI
    ROUTER --> DB
    DATA --> DB
    AI --> DB
    DB --> ROUTER
    ROUTER --> ALARM
```

---

## 2. 핵심 기능 - 유튜브 AI 분석 파이프라인

```mermaid
flowchart LR
    INPUT["사용자가 부품 검색\n예: RTX 4070"]

    subgraph COLLECT["📥 영상 수집"]
        YT_API["YouTube Data API\n키워드 검색"]
        FILTER["조회수 / 관련도 필터링\n상위 5~10개 선별"]
        CLASSIFY["영상 유형 자동 분류\n리뷰 / 비교 / 벤치마크"]
    end

    subgraph ANALYZE["🤖 AI 분석"]
        CAPTION["자막 텍스트 추출"]
        WEIGHT_CALC["유튜버 신뢰도\n가중치 적용"]
        EXTRACT["핵심 정보 추출\n장점 / 단점 / 수치"]
        SUMMARIZE["AI 요약 생성\n한줄 / 요약 / 상세"]
    end

    subgraph OUTPUT["📊 결과 표시"]
        CARD_1["한줄 요약 카드"]
        CARD_2["장점 / 단점 카드"]
        CARD_3["벤치마크 수치"]
        LINK["원본 영상 링크"]
    end

    INPUT --> YT_API
    YT_API --> FILTER
    FILTER --> CLASSIFY
    CLASSIFY --> CAPTION
    CAPTION --> WEIGHT_CALC
    WEIGHT_CALC --> EXTRACT
    EXTRACT --> SUMMARIZE
    SUMMARIZE --> CARD_1 & CARD_2 & CARD_3 & LINK
```

---

## 3. 가격 변동 원인 분석 파이프라인

```mermaid
flowchart TD
    subgraph COLLECT["📡 데이터 수집 (주기적 실행)"]
        PRICE_CRAWL["네이버 쇼핑 API\n부품별 최저가 수집"]
        NEWS_CRAWL["네이버 뉴스 API\n반도체 관련 뉴스 수집"]
        EXCHANGE_CRAWL["환율 API\n달러 환율 수집"]
    end

    subgraph STORE["🗄️ DB 저장"]
        PRICE_SAVE["가격 이력 저장\n날짜별 누적"]
        NEWS_SAVE["뉴스 저장\n키워드 인덱싱"]
    end

    subgraph AI_ANALYZE["🤖 AI 분석"]
        DETECT["가격 변동 감지\n전일 대비 ±5% 이상"]
        SENTIMENT["뉴스 감성 분석\nHuggingFace NLP"]
        MATCH["뉴스 ↔ 가격 변동\n연관 매칭"]
        REASON["변동 원인 문장 생성"]
    end

    subgraph DISPLAY["📊 사용자에게 표시"]
        CARD["왜 올랐나요? 카드\n뉴스 목록 + AI 코멘트"]
        TEMP["시장 온도계\n카테고리별 상태"]
        ALERT["가격 알림\n목표가 도달 시 푸시"]
    end

    PRICE_CRAWL --> PRICE_SAVE
    NEWS_CRAWL --> NEWS_SAVE
    EXCHANGE_CRAWL --> AI_ANALYZE
    PRICE_SAVE --> DETECT
    NEWS_SAVE --> SENTIMENT
    DETECT --> MATCH
    SENTIMENT --> MATCH
    MATCH --> REASON
    REASON --> CARD
    DETECT --> TEMP
    DETECT --> ALERT
```

---

## 4. 견적 구성 & 커뮤니티 흐름

```mermaid
flowchart TD
    subgraph USER_ACTION["👤 사용자 행동"]
        SEARCH["부품 검색"]
        VIEW["부품 상세 페이지 조회\n시세 + 유튜브 요약 + 비교"]
        BUILD["견적 구성\n부품 추가 / 제거"]
    end

    subgraph QUOTE["📋 견적 기능"]
        SAVE["견적 저장"]
        SHARE["공유 링크 생성\n현재 가격 vs 저장 시점 가격"]
        COPY["다른 사람 견적 복사\n내 계정으로 가져오기"]
    end

    subgraph COMMUNITY["🌐 커뮤니티"]
        FEED["견적 피드\n인기 견적 모아보기"]
        COMMENT["댓글 & 좋아요"]
        RANK["인기 부품 TOP 랭킹\n관심 등록 수 + 조회수 종합"]
    end

    subgraph PERSONAL["🔔 개인화"]
        WATCHLIST["관심 부품 등록"]
        ALERT["목표 가격 알림\n푸시 / 이메일"]
        HISTORY["내 견적 히스토리"]
    end

    SEARCH --> VIEW
    VIEW --> BUILD
    BUILD --> SAVE
    SAVE --> SHARE
    SAVE --> HISTORY
    FEED --> COPY
    COPY --> BUILD
    SHARE --> FEED
    FEED --> COMMENT
    COMMENT --> RANK
    VIEW --> WATCHLIST
    WATCHLIST --> ALERT
```

---

## 5. 기술 스택 구성도

```mermaid
flowchart LR
    subgraph FRONTEND["🖥️ 프론트엔드"]
        HTML["HTML / CSS / JS"]
        CHARTJS["Chart.js\n가격 그래프"]
        PWA2["PWA\nmanifest + Service Worker"]
    end

    subgraph BACKEND["⚙️ 백엔드"]
        FLASK["Python Flask\nREST API"]
        SCHEDULER["APScheduler\n주기적 데이터 수집"]
        OAUTH["OAuth 2.0\n소셜 로그인"]
    end

    subgraph AI_STACK["🤖 AI / NLP"]
        HF["HuggingFace\n감성 분석"]
        SUMMARIZER["텍스트 요약 모델"]
        YOUTUBE_AI["YouTube 자막 분석"]
    end

    subgraph EXTERNAL["📡 외부 API"]
        N_SHOP["네이버 쇼핑 API"]
        N_NEWS["네이버 뉴스 API"]
        YT["YouTube Data API"]
        EX["환율 API"]
        KAKAO["카카오 OAuth"]
        GOOGLE["구글 OAuth"]
    end

    subgraph DATABASE["🗄️ 데이터베이스"]
        SQLITE["SQLite\n개발 환경"]
        POSTGRES["PostgreSQL\n배포 환경"]
    end

    FRONTEND <--> BACKEND
    BACKEND <--> AI_STACK
    BACKEND <--> EXTERNAL
    BACKEND <--> DATABASE
```

---

## 6. 용도별 인사이트 & 모델 비교 흐름

```mermaid
flowchart TD
    SELECT["용도 선택\n게이밍 / 사무 / 그래픽 / 3D / 영상편집"]

    subgraph INSIGHT["📊 용도별 인사이트"]
        POPULAR["인기 부품 랭킹\n이번 달 판매 통계"]
        IMPORTANCE["부품 중요도 설명\nAI 코멘트"]
        BUDGET["예산 배분 가이드\n현재 시세 기반"]
    end

    subgraph COMPARE["⚖️ 모델 비교"]
        SPEC["스펙 비교 테이블"]
        BENCH["용도별 벤치마크 탭\n게이밍 / 렌더링 / 그래픽"]
        VALUE["가성비 지수\n점수 ÷ 가격"]
        YT_COMPARE["유튜브 비교 영상\nAI 요약 + 원본 링크"]
        COMMUNITY_REACT["커뮤니티 반응\n긍정 / 부정 비율"]
    end

    subgraph RESULT["✅ 최종 정보 제공"]
        INFO["지금 이 부품은 이렇습니다\n추천이 아닌 정보 제공"]
    end

    SELECT --> POPULAR & IMPORTANCE & BUDGET
    POPULAR --> SPEC
    SPEC --> BENCH
    BENCH --> VALUE
    VALUE --> YT_COMPARE
    YT_COMPARE --> COMMUNITY_REACT
    IMPORTANCE & BUDGET & COMMUNITY_REACT --> INFO
```
