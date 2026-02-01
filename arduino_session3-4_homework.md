# Arduino Session 3-4 숙제

## 제출 안내

**제출 기한:** 다음 수업 시작 전까지

**제출 방법:**
1. Tinkercad 링크 제출
2. 코드 파일 (.ino) 제출 (선택)
3. 동작 스크린샷 또는 영상 (선택)

**평가 기준:**
- **정확성** (40%): 요구사항대로 동작
- **코드 품질** (30%): 가독성, 주석, 변수명
- **창의성** (20%): 추가 기능 또는 개선
- **문서화** (10%): Serial 출력, 주석

---

## 과제 1: 두 개 LED 독립적으로 깜빡이기 ⭐⭐

**학습 목표:** 비차단 타이밍으로 여러 작업 동시 처리

**문제:**
두 개의 LED를 서로 다른 속도로 독립적으로 깜빡이게 만드시오.

**요구사항:**
- LED 2개 (Pin 11, 12)
- LED1: 500ms 간격
- LED2: 1000ms 간격
- `millis()` 사용
- `delay()` 사용 금지
- Serial Monitor에 각 LED 상태 변경 시 출력

**회로:**
```
Arduino Pin 11 → 220Ω → LED1 → GND
Arduino Pin 12 → 220Ω → LED2 → GND
```

**힌트:**
- 각 LED마다 별도의 previousMillis 변수 필요
- 각 LED마다 별도의 interval 상수 필요

**체크리스트:**
□ 2개 타이머 사용  
□ delay() 사용 안 함  
□ 독립적으로 동작  
□ Serial 출력 포함  

---

## 과제 2: 버튼으로 깜빡임 속도 조절 ⭐⭐

**학습 목표:** 버튼 입력과 비차단 타이밍 결합

**문제:**
버튼을 누를 때마다 LED 깜빡임 속도가 3단계로 변하게 만드시오.

**요구사항:**
- LED 1개 (Pin 13)
- 버튼 1개 (Pin 2)
- 속도 3단계: 200ms, 500ms, 1000ms (순환)
- `millis()` 사용
- Serial Monitor에 현재 속도 출력

**회로:**
```
Arduino Pin 13 → 220Ω → LED → GND
Arduino Pin 2 → 버튼 → GND
```

**힌트:**
- 속도를 배열로 저장
- 버튼으로 인덱스 변경
- interval 변수를 동적으로 변경

**체크리스트:**
□ 비차단 타이밍 사용  
□ 버튼 디바운싱  
□ 배열로 속도 관리  
□ Serial 출력 포함  

---

## 과제 3: 3개 LED 독립적으로 페이드 ⭐⭐⭐

**학습 목표:** 여러 LED를 독립적으로 페이드 제어

**문제:**
3개 LED를 각각 다른 속도로 페이드 인/아웃시키시오.

**요구사항:**
- LED 3개 (Pin 9, 10, 11)
- LED1: 10ms 간격
- LED2: 20ms 간격
- LED3: 30ms 간격
- `millis()` 사용
- 각 LED 독립적으로 페이드

**회로:**
```
Arduino Pin 9 → 220Ω → LED1 → GND
Arduino Pin 10 → 220Ω → LED2 → GND
Arduino Pin 11 → 220Ω → LED3 → GND
```

**힌트:**
- Session 3의 여러 LED 독립 제어 참고
- 각 LED마다: brightness, fadeAmount, previousMillis

**체크리스트:**
□ 3개 타이머 사용  
□ 각 LED 독립적 동작  
□ 비차단 타이밍  
□ analogWrite() 사용  

---

## 과제 4: RGB 무지개 페이드 ⭐⭐⭐

**학습 목표:** RGB LED 색상 부드럽게 전환

**문제:**
RGB LED로 무지개 색상을 부드럽게 전환시키시오.

**요구사항:**
- RGB LED 1개 (Pin 9, 10, 11)
- 색상 순서: 빨강 → 노랑 → 초록 → 청록 → 파랑 → 자홍 → 빨강
- 각 색상 전환: 2초 소요 (부드럽게)
- for 반복문 사용
- Serial Monitor에 현재 RGB 값 출력 (100 단위)

**회로:**
```
Arduino Pin 9 → 220Ω → Red (+)
Arduino Pin 10 → 220Ω → Green (+)
Arduino Pin 11 → 220Ω → Blue (+)
공통 Cathode (-) → GND
```

**힌트:**
```c
// 빨강(255,0,0) → 노랑(255,255,0)
for (int i = 0; i <= 255; i++) {
  setColor(255, i, 0);
  delay(8);  // 약 2초
}
```

**체크리스트:**
□ 6단계 색상 전환  
□ 부드러운 전환  
□ setColor() 함수 사용  
□ Serial 출력 포함  

---

## 과제 5: 신호등 시스템 (PWM 버전) ⭐⭐⭐

**학습 목표:** 비차단 타이밍 + analogWrite 결합

**문제:**
신호등을 구현하되, LED 밝기를 부드럽게 변화시키시오.

**요구사항:**
- LED 3개 (Pin 9, 10, 11 - 빨강, 노랑, 초록)
- 신호등 순서: 빨강(5초) → 노랑(2초) → 초록(5초) → 노랑(2초)
- 각 신호 전환 시 페이드 효과 (0.5초)
- `millis()` 사용
- 버튼(Pin 2)으로 긴급 모드: 모든 LED 깜빡임
- Serial Monitor에 현재 상태 출력

**회로:**
```
Arduino Pin 9 → 220Ω → Red LED → GND
Arduino Pin 10 → 220Ω → Yellow LED → GND
Arduino Pin 11 → 220Ω → Green LED → GND
Arduino Pin 2 → 버튼 → GND
```

**힌트:**
- 상태 변수 사용 (0=빨강, 1=노랑, 2=초록, 3=노랑)
- 각 상태마다 다른 시간
- 전환 시 페이드 인/아웃

**체크리스트:**
□ 비차단 타이밍  
□ 페이드 효과  
□ 긴급 모드 구현  
□ Serial 출력 포함  

---

## 과제 6: 나이트 라이더 + 페이드 ⭐⭐⭐

**학습 목표:** 순차 제어 + 페이드 효과

**문제:**
5개 LED가 순차적으로 페이드 인/아웃하며 왕복 이동하게 만드시오.

**요구사항:**
- LED 5개 (Pin 6, 9, 10, 11, 12)
- 각 LED 페이드 인: 0 → 255 (100ms)
- 각 LED 페이드 아웃: 255 → 0 (100ms)
- 왕복 이동
- 버튼(Pin 2)으로 속도 조절 (50ms, 100ms, 200ms)
- Serial Monitor에 현재 위치와 속도 출력

**회로:**
```
Arduino Pin 6, 9, 10, 11, 12 → 각각 220Ω → LED → GND
Arduino Pin 2 → 버튼 → GND
```

**힌트:**
- 현재 위치(인덱스)와 방향(1 또는 -1) 변수
- for 반복문으로 페이드 인/아웃
- 끝에 도달하면 방향 반전

**체크리스트:**
□ 5개 LED 순차 제어  
□ 페이드 효과  
□ 왕복 이동  
□ 속도 조절 기능  

---

## 과제 7: 촛불 효과 ⭐⭐⭐

**학습 목표:** random() 함수 + analogWrite 활용

**문제:**
LED로 불규칙하게 깜빡이는 촛불 효과를 만드시오.

**요구사항:**
- LED 1개 (Pin 9)
- 밝기: 180~255 사이 랜덤
- 간격: 50~150ms 사이 랜덤
- `millis()` 사용 (비차단)
- Serial Monitor에 현재 밝기 출력

**회로:**
```
Arduino Pin 9 → 220Ω → LED → GND
```

**힌트:**
```c
int brightness = random(180, 256);  // 180~255
int interval = random(50, 151);     // 50~150
```

**출처:** [Arduino random()](https://www.arduino.cc/reference/en/language/functions/random-numbers/random/)

**체크리스트:**
□ random() 사용  
□ 비차단 타이밍  
□ 불규칙한 깜빡임  
□ Serial 출력 포함  

---

## 과제 8: 음악 비트 시뮬레이션 ⭐⭐⭐⭐

**학습 목표:** 여러 LED 독립적 페이드 + 리듬 표현

**문제:**
3개 LED로 음악 비트를 시뮬레이션하시오.

**요구사항:**
- LED 3개 (Pin 9, 10, 11)
- LED1: 저음 (느린 페이드, 500ms 주기)
- LED2: 중음 (중간 페이드, 300ms 주기)
- LED3: 고음 (빠른 페이드, 100ms 주기)
- 각 LED 독립적으로 페이드 인/아웃
- `millis()` 사용
- 버튼(Pin 2)으로 일시정지/재개
- Serial Monitor에 각 LED 상태 출력

**회로:**
```
Arduino Pin 9, 10, 11 → 각각 220Ω → LED → GND
Arduino Pin 2 → 버튼 → GND
```

**힌트:**
- 각 LED마다 독립적인 타이머
- 각 LED마다 독립적인 페이드 변수
- 일시정지 플래그 변수

**체크리스트:**
□ 3개 독립 페이드  
□ 서로 다른 주기  
□ 일시정지 기능  
□ Serial 출력 포함  

---

## 과제 9: 밝기 센서 시뮬레이션 ⭐⭐⭐⭐

**학습 목표:** millis() + analogWrite + 조건문 종합

**문제:**
버튼을 누르고 있는 시간에 따라 LED 밝기가 점진적으로 변하게 만드시오.

**요구사항:**
- LED 1개 (Pin 9)
- 버튼 1개 (Pin 2)
- 버튼을 누르고 있는 동안 밝기 증가 (0 → 255)
- 버튼을 떼면 밝기 감소 (현재 → 0)
- 증가/감소 속도: 1초에 50씩
- `millis()` 사용
- Serial Monitor에 현재 밝기 출력

**회로:**
```
Arduino Pin 9 → 220Ω → LED → GND
Arduino Pin 2 → 버튼 → GND
```

**힌트:**
- 버튼 상태에 따라 증감 방향 결정
- 경계 처리 (0~255)
- 비차단 타이밍으로 부드럽게

**체크리스트:**
□ 버튼 누름: 밝기 증가  
□ 버튼 뗌: 밝기 감소  
□ 비차단 타이밍  
□ Serial 출력 포함  

---

## 과제 10: RGB 색상 믹서 ⭐⭐⭐⭐

**학습 목표:** 버튼 3개 + RGB LED + 배열 종합

**문제:**
버튼 3개로 RGB 각 채널의 밝기를 독립적으로 조절하시오.

**요구사항:**
- RGB LED 1개 (Pin 9, 10, 11)
- 버튼 3개 (Pin 2, 3, 4)
- 버튼1: Red 채널 밝기 조절 (0 → 64 → 128 → 192 → 255 → 0)
- 버튼2: Green 채널 밝기 조절 (동일)
- 버튼3: Blue 채널 밝기 조절 (동일)
- Serial Monitor에 현재 RGB 값 출력
- 형식: "RGB(r, g, b)"

**회로:**
```
Arduino Pin 9 → 220Ω → Red (+)
Arduino Pin 10 → 220Ω → Green (+)
Arduino Pin 11 → 220Ω → Blue (+)
공통 Cathode (-) → GND
Arduino Pin 2, 3, 4 → 각각 버튼 → GND
```

**힌트:**
- 3개 배열: RGB 핀, 버튼 핀, 현재 밝기
- for 반복문으로 각 버튼 체크
- 버튼 디바운싱

**체크리스트:**
□ 3개 버튼 독립 제어  
□ 5단계 밝기  
□ RGB 동시 표시  
□ Serial 출력 형식 정확  

---

## 추가 도전 과제 (선택)

### 보너스 1: 복합 시스템 ⭐⭐⭐⭐⭐

**문제:**
과제 8과 과제 10을 결합하여, 버튼으로 RGB 색상을 설정하고, 해당 색상으로 비트 효과를 만드시오.

### 보너스 2: 게임 ⭐⭐⭐⭐⭐

**문제:**
RGB LED가 랜덤 색상을 보여주면, 사용자가 버튼 3개를 적절히 조합하여 같은 색상을 만드는 게임을 구현하시오. 오차 범위 ±20 이내면 성공.

---

## 제출 전 체크리스트

□ 모든 요구사항 충족  
□ 회로 연결 정확  
□ 코드 주석 작성  
□ Serial 출력 포함  
□ Tinkercad 링크 동작 확인  
□ 변수명 의미 있게 작성  
□ 매직 넘버 상수로 정의  

---

## 평가 예시

**과제 1 만점 예시:**
```c
// 명확한 주석
const int LED1_PIN = 11;
const int LED2_PIN = 12;
const unsigned long INTERVAL1 = 500;
const unsigned long INTERVAL2 = 1000;

unsigned long previousMillis1 = 0;
unsigned long previousMillis2 = 0;
int led1State = LOW;
int led2State = LOW;

void setup() {
  Serial.begin(9600);
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
}

void loop() {
  unsigned long currentMillis = millis();
  
  // LED1 제어
  if (currentMillis - previousMillis1 >= INTERVAL1) {
    previousMillis1 = currentMillis;
    led1State = !led1State;
    digitalWrite(LED1_PIN, led1State);
    
    Serial.print("LED1 ");
    Serial.println(led1State ? "ON" : "OFF");
  }
  
  // LED2 제어
  if (currentMillis - previousMillis2 >= INTERVAL2) {
    previousMillis2 = currentMillis;
    led2State = !led2State;
    digitalWrite(LED2_PIN, led2State);
    
    Serial.print("LED2 ");
    Serial.println(led2State ? "ON" : "OFF");
  }
}
```

**좋은 코드의 특징:**
- 상수 사용 (const)
- 의미 있는 변수명
- 적절한 주석
- Serial 출력 명확
- 들여쓰기 정확

---

**행운을 빕니다!**

문제를 스스로 해결하면서 실력이 크게 향상될 것입니다. 막히는 부분이 있다면 공식 Arduino Reference 문서를 참고하세요.
