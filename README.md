# capstone
flowchart TD
    subgraph INPUT["🎵 입력부"]
        MIC[USB 마이크 모듈]
        WEB[웹 컨트롤러\n스마트폰 / PC]
    end

    subgraph RPI["🖥️ 라즈베리파이 (메인 컨트롤러)"]
        direction TB
        AUDIO[오디오 캡처\nPyAudio / sounddevice]
        FFT[주파수 분석\nFFT - NumPy]
        LOGIC[LED 제어 로직\n주파수 → 색상 / 밝기 / 패턴 매핑]
        FLASK[웹 서버\nFlask]
        MODE[모드 관리\n음악반응 / 수동 / 무지개]

        AUDIO --> FFT
        FFT --> LOGIC
        FLASK --> MODE
        MODE --> LOGIC
    end

    subgraph FREQ["🎚️ 주파수 대역 분리"]
        LOW[저음 Bass\n20~300Hz]
        MID[중음 Mid\n300~3kHz]
        HIGH[고음 High\n3k~20kHz]
    end

    subgraph OUTPUT["💡 출력부"]
        LED[WS2812B LED 스트립\nRGB 풀컬러]
        RED[빨간색 펄스\n베이스 반응]
        GREEN[초록색 흐름\n중음 반응]
        BLUE[파란색 반짝임\n고음 반응]
    end

    MIC --> AUDIO
    WEB --> FLASK

    FFT --> LOW
    FFT --> MID
    FFT --> HIGH

    LOW --> RED
    MID --> GREEN
    HIGH --> BLUE

    RED --> LED
    GREEN --> LED
    BLUE --> LED

    LOGIC -->|GPIO PWM\nrpi_ws281x| LED
