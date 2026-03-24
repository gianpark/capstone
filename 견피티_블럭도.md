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
        AUTH["소셜 로그인\n카카오 / 구글 / 스팀 OAuth"]
        SCHEDULER["APScheduler\n주기적 데이터 수집"]
        ALARM["가격 알림\n푸시 / 이메일"]
    end

    subgraph DATA["📡 데이터 수집 모듈"]
        NAVER_SHOP["네이버 쇼핑 API\n실시간 최저가"]
        NAVER_NEWS["네이버 뉴스 API\n반도체 관련 뉴스"]
        EXCHANGE["한국수출입은행 API\n환율 정보"]
        YOUTUBE["YouTube Data API\n리뷰 / 비교 영상"]
        STEAM["Steam Web API\n게임 라이브러리"]
        USED["번개장터 / 당근마켓\n중고 시세 크롤링"]
    end

    subgraph AI["🤖 AI 분석 모듈"]
        NLP["HuggingFace NLP\n뉴스 감성 분석"]
        SUMMARY["텍스트 요약 모델\n유튜브 자막 요약"]
        PRICE_REASON["가격 변동\n원인 분석"]
        COMPAT["호환성 체크\n부품 DB 매칭"]
        FPS_SIM["FPS 시뮬레이션\n게임별 예상 성능"]
    end

    subgraph DB["🗄️ 데이터베이스"]
        PARTS_DB["부품 정보 DB\n스펙 / 브랜드 / 호환성"]
        PRICE_DB["가격 이력 DB\n신품 / 중고 날짜별 시세"]
        GAME_DB["게임 DB\n부품 의존도 / FPS 데이터"]
        USER_DB["유저 DB\n견적 / 관심 / 스팀 연동"]
        COMMUNITY_DB["커뮤니티 DB\n댓글 / 좋아요"]
    end

    WEB & PWA --> ROUTER
    ROUTER --> AUTH
    SCHEDULER --> DATA
    DATA --> DB
    ROUTER --> AI
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
    subgraph COLLECT["📡 데이터 수집 (APScheduler 주기 실행)"]
        PRICE_CRAWL["네이버 쇼핑 API\n부품별 최저가 수집"]
        NEWS_CRAWL["네이버 뉴스 API\n반도체 관련 뉴스 수집"]
        USED_CRAWL["번개장터 / 당근마켓\n중고 시세 크롤링"]
        EXCHANGE_CRAWL["환율 API\n달러 환율 수집"]
    end

    subgraph STORE["🗄️ DB 저장"]
        PRICE_SAVE["가격 이력 저장\n신품 / 중고 날짜별 누적"]
        NEWS_SAVE["뉴스 저장\n키워드 인덱싱"]
    end

    subgraph AI_ANALYZE["🤖 AI 분석"]
        DETECT["가격 변동 감지\n전일 대비 ±5% 이상"]
        SENTIMENT["뉴스 감성 분석\nHuggingFace NLP"]
        MATCH["뉴스 ↔ 가격 변동\n연관 매칭"]
        REASON["변동 원인 문장 생성"]
    end

    subgraph DISPLAY["📊 사용자에게 표시"]
        CARD["왜 올랐나요? 카드"]
        TEMP["시장 온도계"]
        TIMING["구매 타이밍 카드\n지금 / 기다려 / 예산 늘려"]
        ALERT["가격 알림\n목표가 도달 시 푸시"]
    end

    PRICE_CRAWL & USED_CRAWL --> PRICE_SAVE
    NEWS_CRAWL --> NEWS_SAVE
    EXCHANGE_CRAWL --> AI_ANALYZE
    PRICE_SAVE --> DETECT
    NEWS_SAVE --> SENTIMENT
    DETECT --> MATCH
    SENTIMENT --> MATCH
    MATCH --> REASON
    REASON --> CARD
    DETECT --> TEMP & TIMING & ALERT
```

---

## 4. 호환성 자동 체크 플로우

```mermaid
flowchart TD
    ADD["부품 추가\n예: i9-13900K 선택"]

    subgraph CHECK["🔍 호환성 실시간 체크"]
        CPU_MB["CPU ↔ 메인보드\n소켓 / 칩셋 / 세대"]
        RAM_MB["RAM ↔ 메인보드\nDDR4 vs DDR5 / 최대 용량"]
        GPU_PSU["GPU ↔ 파워서플라이\n권장 와트 / 커넥터"]
        COOL_CASE["쿨러 ↔ 케이스\n높이 호환"]
        SSD_MB["SSD ↔ 메인보드\nM.2 슬롯 / PCIe 버전"]
    end

    subgraph RESULT["📊 결과 처리"]
        OK["✅ 호환 이상 없음"]
        WARN["⚠️ 경고 발생\n문제 부품 명시"]
        ALT["🔧 대안 부품 자동 추천\n호환 가능한 제품 리스트"]
    end

    subgraph FINAL["📋 최종 리포트"]
        REPORT["견적 저장 전\n전체 호환성 요약 리포트"]
    end

    ADD --> CPU_MB & RAM_MB & GPU_PSU & COOL_CASE & SSD_MB
    CPU_MB & RAM_MB & GPU_PSU & COOL_CASE & SSD_MB --> OK
    CPU_MB & RAM_MB & GPU_PSU & COOL_CASE & SSD_MB --> WARN
    WARN --> ALT
    OK & ALT --> REPORT
```

---

## 5. 게임별 맞춤 추천 & 스팀 연동

```mermaid
flowchart TD
    subgraph INPUT["🎮 게임 정보 입력"]
        MANUAL["직접 게임 선택\n배그 / 사이버펑크 / LoL 등"]
        STEAM_LOGIN["스팀 계정 연동\nSteam OpenID 로그인"]
    end

    subgraph STEAM_ANALYZE["📊 스팀 자동 분석"]
        LIBRARY["보유 게임 목록 수집\nSteam Web API"]
        PLAYTIME["플레이 시간 TOP 5 추출"]
        GAME_TYPE["게임 유형 분류\nGPU / CPU / RAM / 밸런스"]
    end

    subgraph RECOMMEND["💡 맞춤 추천"]
        WEIGHT_PARTS["부품 우선순위 산정\n예산 배분 비율 계산"]
        BUDGET_ALLOC["예산별 최적 부품 조합\n현재 시세 반영"]
        FPS_PREDICT["예상 FPS 시뮬레이션\n게임별 성능 미리보기"]
    end

    subgraph UPGRADE["🔧 업그레이드 가이드"]
        CURRENT_PC["현재 PC 사양 입력"]
        BOTTLENECK["병목 부품 감지\nGPU / CPU / RAM 분석"]
        UPGRADE_SUGGEST["업그레이드 대상 추천\n최소 비용 최대 효과"]
    end

    MANUAL --> GAME_TYPE
    STEAM_LOGIN --> LIBRARY --> PLAYTIME --> GAME_TYPE
    GAME_TYPE --> WEIGHT_PARTS --> BUDGET_ALLOC --> FPS_PREDICT
    CURRENT_PC --> BOTTLENECK --> UPGRADE_SUGGEST
```

---

## 6. 예산 초과 시 중고 시세 연동

```mermaid
flowchart TD
    BUDGET["예산 입력\n예: 80만원"]
    BUILD["견적 구성 중"]
    TOTAL["총액 계산"]

    subgraph OVER["💸 예산 초과 감지"]
        EXCEED["초과 금액 표시"]
        USED_PRICE["중고 시세 자동 조회\n번개장터 / 당근마켓"]
        COMPARE["신품 vs 중고 비교 카드\n가격 / AS / 주의사항"]
    end

    subgraph OPTIMIZE["✅ 최적화 견적 제안"]
        MIX["신품 + 중고 혼합 견적\nGPU 중고 + CPU 신품 등"]
        RECALC["재계산 후 예산 내 맞춤"]
        GUIDE["중고 구매 주의사항\n채굴 이력 / 직거래 팁"]
    end

    subgraph USED_TREND["📈 중고 시세 추이"]
        USED_HISTORY["중고 가격 이력 그래프\n신품 시세와 함께 표시"]
        USED_RATE["중고 가격 유지율\n구매가 대비 현재 중고가"]
    end

    BUDGET --> BUILD --> TOTAL
    TOTAL -->|초과| EXCEED
    EXCEED --> USED_PRICE --> COMPARE
    COMPARE --> MIX --> RECALC
    RECALC --> GUIDE
    USED_PRICE --> USED_HISTORY & USED_RATE
```

---

## 7. 시리즈 & 모델 비교 흐름

```mermaid
flowchart TD
    SEARCH["부품 검색\n예: RTX 4070"]

    subgraph SERIES["📊 시리즈 라인업 비교"]
        LINEUP["같은 시리즈 자동 수집\n4070 / 4070 Super / 4070 Ti / 4070 Ti Super"]
        SPEC_TABLE["스펙 비교 테이블\n가격 / VRAM / 성능 차이 / 가성비 지수"]
        BEST_VALUE["가성비 최적 모델 표시"]
    end

    subgraph PEER["⚖️ 비슷한 성능 구간 비교"]
        PERF_GROUP["성능 구간 자동 그룹핑\n타 브랜드 경쟁 제품"]
        BENCH_TAB["용도별 벤치마크 탭\n게이밍 / 렌더링 / 그래픽"]
        YT_COMPARE["유튜브 비교 영상\nAI 요약 + 원본 링크"]
        COMMUNITY["커뮤니티 반응\n긍정 / 부정 비율"]
    end

    subgraph TIMING["⏰ 구매 타이밍"]
        PRICE_TREND["가격 추이 분석\n3개월 방향성"]
        NEWS_CHECK["관련 뉴스 체크\n신제품 루머 여부"]
        TIMING_CARD["구매 타이밍 카드\n지금 사도 무난 / 기다려요"]
    end

    SEARCH --> LINEUP --> SPEC_TABLE --> BEST_VALUE
    SEARCH --> PERF_GROUP --> BENCH_TAB --> YT_COMPARE --> COMMUNITY
    SEARCH --> PRICE_TREND & NEWS_CHECK --> TIMING_CARD
```

---

## 8. 커뮤니티 & 견적 공유 흐름

```mermaid
flowchart TD
    subgraph BUILD["🔧 견적 구성"]
        PARTS["부품 검색 & 추가"]
        COMPAT_CHECK["호환성 실시간 체크"]
        YT_CARD["부품별 유튜브 요약 카드"]
        FPS_CHECK["예상 FPS 확인"]
    end

    subgraph SAVE["💾 저장 & 공유"]
        SAVE_QUOTE["견적 저장"]
        SHARE_LINK["공유 링크 생성\n저장 시점 vs 현재 가격 비교"]
        COPY["다른 사람 견적 복사\n내 계정으로 가져오기"]
    end

    subgraph COMMUNITY["🌐 커뮤니티"]
        FEED["견적 피드\n인기 견적 모아보기"]
        COMMENT["댓글 & 좋아요\n마니아 피드백"]
        RANK["인기 부품 TOP 랭킹\n관심 등록 + 조회수 종합"]
    end

    subgraph PERSONAL["🔔 개인화"]
        WATCHLIST["관심 부품 등록"]
        PRICE_ALERT["목표 가격 알림\n푸시 / 이메일"]
        HISTORY["내 견적 히스토리\n가격 변화 자동 추적"]
    end

    PARTS --> COMPAT_CHECK & YT_CARD & FPS_CHECK
    COMPAT_CHECK --> SAVE_QUOTE
    SAVE_QUOTE --> SHARE_LINK
    SHARE_LINK --> FEED
    FEED --> COPY & COMMENT
    COMMENT --> RANK
    PARTS --> WATCHLIST --> PRICE_ALERT
    SAVE_QUOTE --> HISTORY
```

---

## 9. 기술 스택 구성도

```mermaid
flowchart LR
    subgraph FRONTEND["🖥️ 프론트엔드"]
        HTML["HTML / CSS / JS"]
        CHARTJS["Chart.js\n가격 그래프"]
        PWA2["PWA\nmanifest + Service Worker"]
    end

    subgraph BACKEND["⚙️ 백엔드"]
        FLASK["Python Flask\nREST API"]
        APSCHEDULER["APScheduler\n주기적 데이터 수집"]
        OAUTH["OAuth 2.0\n카카오 / 구글 / 스팀"]
    end

    subgraph AI_STACK["🤖 AI / NLP"]
        HF["HuggingFace\n감성 분석"]
        SUMMARIZER["텍스트 요약 모델\n유튜브 자막"]
        COMPAT_AI["호환성 매칭\n부품 DB 룰 기반"]
        FPS_AI["FPS 시뮬레이터\n벤치마크 DB 기반"]
    end

    subgraph EXTERNAL["📡 외부 API"]
        N_SHOP["네이버 쇼핑 API"]
        N_NEWS["네이버 뉴스 API"]
        YT["YouTube Data API"]
        EX["환율 API"]
        STEAM_API["Steam Web API"]
        USED_CRAWL["번개장터/당근 크롤링"]
    end

    subgraph DATABASE["🗄️ 데이터베이스"]
        PARTS_D["부품 / 호환성 DB"]
        PRICE_D["가격 이력 DB\n신품 + 중고"]
        GAME_D["게임 / FPS DB"]
        USER_D["유저 / 커뮤니티 DB"]
    end

    FRONTEND <--> BACKEND
    BACKEND <--> AI_STACK
    BACKEND <--> EXTERNAL
    BACKEND <--> DATABASE
    EXTERNAL --> DATABASE
```
