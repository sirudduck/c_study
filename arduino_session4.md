# Arduino Week 2 Session 4: PWM과 아날로그 출력

## 학습 목표
- `analogWrite()` 함수 사용법 학습
- 0~255 값으로 LED 밝기 조절
- PWM 핀 확인 및 활용
- for 반복문으로 부드러운 페이드 효과 구현
- RGB LED로 색상 제어 (배열 활용)

---

## 1. 동기부여: LED 밝기 조절 문제 (10분)

### 문제: LED를 반만 밝게 켜고 싶다

**현재까지 배운 방법:**
```c
digitalWrite(13, HIGH);  // 완전히 켜짐 (5V)
digitalWrite(13, LOW);   // 완전히 꺼짐 (0V)
```

**한계:**
- ON/OFF 두 가지 상태만 가능
- 중간 밝기를 만들 수 없음

**학생들에게 먼저 질문:**
- 실생활에서 밝기를 조절하는 기기는?
- 5V도 아니고 0V도 아닌 중간 전압(예: 2.5V)을 만들 수 있을까?
- 디지털 핀으로 아날로그 신호를 만들 수 있을까?

**실생활 예시:**
- 조명 디머 스위치
- 노트북 화면 밝기 조절
- 선풍기 속도 조절 (약/중/강)
- 스마트폰 진동 세기

**시연:**
1. `digitalWrite()`로 LED 제어 → 완전히 켜짐/꺼짐만 가능
2. `analogWrite(128)`로 LED 제어 → 반만 밝게!
3. `analogWrite(64)` → 더 어둡게!

---

## 2. `analogWrite()` 함수 소개 (15분)

### 2.1 `digitalWrite()`의 한계

**지금까지 배운 것:**
```c
digitalWrite(13, HIGH);  // LED 완전히 켜짐
digitalWrite(13, LOW);   // LED 완전히 꺼짐
```

**한계**: ON/OFF 두 가지만 가능

**새로운 함수: `analogWrite()`**
```c
analogWrite(9, 128);  // LED 반만 밝게!
analogWrite(9, 64);   // LED 더 어둡게!
analogWrite(9, 255);  // LED 최대 밝기!
```

### 2.2 함수 시그니처

```c
void analogWrite(int pin, int value);
```

**매개변수:**
- `pin`: PWM 핀 번호 (3, 5, 6, 9, 10, 11만 가능)
- `value`: 0~255 사이의 값

**반환값:** 없음 (void)

**출처**: [Arduino Reference - analogWrite()](https://www.arduino.cc/reference/en/language/functions/analog-io/analogwrite/)

### 2.3 값의 의미

| 값 | LED 밝기 |
|----|----------|
| 0 | 꺼짐 |
| 64 | 어두움 (25%) |
| 128 | 중간 (50%) |
| 192 | 밝음 (75%) |
| 255 | 최대 (100%) |

**왜 0~255인가?**
- 8비트 (2^8 = 256가지 값)
- 0부터 세면 0~255

**수학적 대응:**
```
digitalWrite(): {LOW, HIGH} = {0, 1}
analogWrite():  {0, 1, 2, ..., 255} = [0, 255]
```

### 2.4 PWM 핀 확인

**중요**: 모든 디지털 핀이 `analogWrite()`를 지원하는 것은 아님!

**PWM 가능 핀 (Arduino Uno):**
- Pin 3, 5, 6, 9, 10, 11 (핀 옆에 **~** 표시)

**PWM 불가능:**
- Pin 2, 4, 7, 8, 12, 13

```c
// ✓ 가능:
analogWrite(9, 128);

// ✗ 불가능 (컴파일은 되지만 작동 안 함):
analogWrite(13, 128);
```

**Tinkercad에서 확인:**
- Arduino 보드 이미지에서 핀 번호 옆 **~** 기호 확인

**참고**: PWM은 "Pulse Width Modulation"의 약자로, HIGH/LOW를 매우 빠르게 반복하여 평균 밝기를 만드는 기술입니다. 자세한 원리는 몰라도 사용하는 데 문제없습니다.

**출처**: [Arduino Uno Pinout](https://docs.arduino.cc/hardware/uno-rev3/)

### 2.5 기본 사용 예시

```c
const int LED_PIN = 9;  // PWM 핀

void setup() {
  pinMode(LED_PIN, OUTPUT);  // pinMode는 여전히 필요
}

void loop() {
  analogWrite(LED_PIN, 0);    // 꺼짐
  delay(1000);
  
  analogWrite(LED_PIN, 64);   // 어두움
  delay(1000);
  
  analogWrite(LED_PIN, 128);  // 중간
  delay(1000);
  
  analogWrite(LED_PIN, 192);  // 밝음
  delay(1000);
  
  analogWrite(LED_PIN, 255);  // 최대
  delay(1000);
}
```

### 2.6 `digitalWrite()` vs `analogWrite()`

| 함수 | 값 범위 | 용도 |
|------|---------|------|
| `digitalWrite()` | HIGH/LOW | ON/OFF 제어 |
| `analogWrite()` | 0~255 | 밝기/속도 조절 |

**호환성:**
```c
digitalWrite(9, HIGH);  // = analogWrite(9, 255)
digitalWrite(9, LOW);   // = analogWrite(9, 0)
```

---

## 3. LED 밝기 제어 기초 (20분)

## 3. LED 밝기 제어 기초 (20분)

### 3.1 고정 밝기 설정

**회로:**
```
Arduino Pin 9 (~) → 220Ω → LED(+) → LED(-) → GND
```

**코드:**
```c
const int LED_PIN = 9;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  analogWrite(LED_PIN, 100);  // 고정 밝기
}
```

### 3.2 버튼으로 밝기 단계 조절

**회로 추가:**
```
Arduino Pin 2 → 버튼 → GND
```

**코드:**
```c
const int LED_PIN = 9;
const int BUTTON_PIN = 2;

int brightness = 0;  // 현재 밝기 (0~255)
int lastButtonState = HIGH;

void setup() {
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  // 버튼 눌렀을 때 (상태 변화 감지)
  if (buttonState == LOW && lastButtonState == HIGH) {
    brightness = brightness + 51;  // 5단계 (0, 51, 102, 153, 204, 255)
    
    if (brightness > 255) {
      brightness = 0;  // 순환
    }
    
    analogWrite(LED_PIN, brightness);
    
    Serial.print("Brightness: ");
    Serial.println(brightness);
    
    delay(200);  // 디바운싱
  }
  
  lastButtonState = buttonState;
}
```

**5단계 밝기:**
- 0: 꺼짐
- 51: 20%
- 102: 40%
- 153: 60%
- 204: 80%
- 255: 100%

### 3.3 변수로 밝기 관리

**밝기를 배열로 관리:**
```c
const int BRIGHTNESS_LEVELS[5] = {0, 64, 128, 192, 255};
int currentLevel = 0;

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW && lastButtonState == HIGH) {
    currentLevel = (currentLevel + 1) % 5;  // 0~4 순환
    
    int brightness = BRIGHTNESS_LEVELS[currentLevel];
    analogWrite(LED_PIN, brightness);
    
    Serial.print("Level: ");
    Serial.print(currentLevel);
    Serial.print(", Brightness: ");
    Serial.println(brightness);
    
    delay(200);
  }
  
  lastButtonState = buttonState;
}
```

---

## 4. 부드러운 페이드 효과 (25분)

### 4.1 for 반복문으로 페이드 인

**페이드 인**: 0에서 255까지 점진적 증가

```c
const int LED_PIN = 9;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  // 페이드 인 (밝아짐)
  for (int brightness = 0; brightness <= 255; brightness++) {
    analogWrite(LED_PIN, brightness);
    delay(10);  // 10ms마다 1씩 증가
  }
  
  delay(1000);  // 1초 대기
}
```

**수학적 표현:**
```
밝기(t) = t, where t ∈ [0, 255]
시간 = 255 × 10ms = 2.55초
```

### 4.2 페이드 아웃

**페이드 아웃**: 255에서 0까지 점진적 감소

```c
void loop() {
  // 페이드 아웃 (어두워짐)
  for (int brightness = 255; brightness >= 0; brightness--) {
    analogWrite(LED_PIN, brightness);
    delay(10);
  }
  
  delay(1000);
}
```

### 4.3 호흡등 효과 (Breathing Effect)

**페이드 인 + 페이드 아웃 반복:**

```c
void loop() {
  // 페이드 인
  for (int brightness = 0; brightness <= 255; brightness++) {
    analogWrite(LED_PIN, brightness);
    delay(5);
  }
  
  // 페이드 아웃
  for (int brightness = 255; brightness >= 0; brightness--) {
    analogWrite(LED_PIN, brightness);
    delay(5);
  }
}
```

**수학적 표현 (사인 함수):**
```
밝기(t) = 128 + 127 × sin(2πt / T)

where:
  t: 시간
  T: 주기
  밝기 범위: [1, 255]
```

### 4.4 `delay()` vs `millis()` 페이드

**`delay()` 사용 (블로킹):**
```c
// 페이드 중에는 다른 작업 불가능
for (int brightness = 0; brightness <= 255; brightness++) {
  analogWrite(LED_PIN, brightness);
  delay(10);  // 멈춤!
}
```

**`millis()` 사용 (비차단):**
```c
const int LED_PIN = 9;
const int BUTTON_PIN = 2;

int brightness = 0;
int fadeAmount = 1;
unsigned long previousMillis = 0;
const unsigned long FADE_INTERVAL = 10;  // 10ms마다 업데이트

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void loop() {
  unsigned long currentMillis = millis();
  
  // 페이드 처리 (비차단)
  if (currentMillis - previousMillis >= FADE_INTERVAL) {
    previousMillis = currentMillis;
    
    brightness = brightness + fadeAmount;
    
    // 경계 처리
    if (brightness <= 0 || brightness >= 255) {
      fadeAmount = -fadeAmount;  // 방향 반전
    }
    
    analogWrite(LED_PIN, brightness);
  }
  
  // 버튼 체크 (동시에 가능!)
  if (digitalRead(BUTTON_PIN) == LOW) {
    // 버튼 처리
  }
}
```

**장점:**
- 페이드 중에도 버튼 입력 감지 가능
- 여러 LED를 독립적으로 페이드 가능

**출처**: [Arduino Fading Example](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Fading)

### 4.5 속도 조절

**페이드 속도 조절 방법:**

1. **delay 값 조절:**
```c
delay(5);   // 빠름
delay(10);  // 중간
delay(20);  // 느림
```

2. **증감량 조절:**
```c
brightness += 1;   // 부드러움, 느림
brightness += 2;   // 중간
brightness += 5;   // 거침, 빠름
```

3. **간격 조절 (millis):**
```c
const unsigned long INTERVAL = 10;   // 빠름
const unsigned long INTERVAL = 20;   // 느림
```

---

## 5. 실습 (30분)

### 실습 1: 호흡등 효과

**문제**: LED가 부드럽게 밝아졌다 어두워지기를 반복하게 만들어라

**요구사항**:
- LED 1개 (Pin 9)
- 페이드 인: 0 → 255
- 페이드 아웃: 255 → 0
- 한 주기: 약 3초
- Serial Monitor에 현재 밝기 출력 (50 단위)

**힌트**: for 반복문 2개 사용

---

### 실습 2: 여러 LED 독립적으로 페이드

**문제**: 3개 LED를 각각 다른 속도로 페이드시켜라

**요구사항**:
- LED 3개 (Pin 9, 10, 11)
- LED1: 10ms 간격
- LED2: 20ms 간격
- LED3: 30ms 간격
- `millis()` 사용
- 각 LED 독립적으로 페이드 인/아웃

**힌트**: 
- Session 3의 여러 LED 독립 제어 참고
- 각 LED마다 brightness, fadeAmount, previousMillis 필요

---

### 실습 3: 버튼으로 페이드 속도 조절

**문제**: 버튼을 누를 때마다 페이드 속도가 변경되게 만들어라

**요구사항**:
- LED 1개 (Pin 9)
- 버튼 1개 (Pin 2)
- 속도 3단계: 느림(30ms), 중간(15ms), 빠름(5ms)
- 버튼 누를 때마다 순환
- Serial Monitor에 현재 속도 출력

**힌트**: 
- 속도를 배열로 저장
- 버튼으로 인덱스 변경

---

## 6. RGB LED 소개 (20분)

### 6.1 RGB LED란?

**정의**: Red, Green, Blue 3개 LED가 하나의 패키지에 들어있는 LED

**구조:**
```
RGB LED (Common Cathode)
    
    ┌─ Red (+)
    ├─ Green (+)
    ├─ Blue (+)
    └─ GND (-)  (공통)
```

**핀 구성:**
- Red 핀
- Green 핀
- Blue 핀
- 공통 핀 (Common Cathode 또는 Common Anode)

**Tinkercad에서:**
- "RGB LED" 검색
- Common Cathode 타입 사용 권장

### 6.2 색상 혼합 원리

**가산 혼합 (Additive Color Mixing):**

빛의 3원색을 혼합하여 다양한 색상 생성

| Red | Green | Blue | 색상 |
|-----|-------|------|------|
| 255 | 0 | 0 | 빨강 |
| 0 | 255 | 0 | 초록 |
| 0 | 0 | 255 | 파랑 |
| 255 | 255 | 0 | 노랑 |
| 255 | 0 | 255 | 자홍 (Magenta) |
| 0 | 255 | 255 | 청록 (Cyan) |
| 255 | 255 | 255 | 흰색 |
| 0 | 0 | 0 | 꺼짐 |
| 128 | 128 | 128 | 회색 |
| 255 | 128 | 0 | 주황 |

**수학적 표현:**
```
색상 = (R, G, B)
where R, G, B ∈ [0, 255]

색 공간: 256³ = 16,777,216 가지 색상
```

### 6.3 회로 연결

**Common Cathode RGB LED:**
```
Arduino Pin 9 (~) → 220Ω → Red (+)
Arduino Pin 10 (~) → 220Ω → Green (+)
Arduino Pin 11 (~) → 220Ω → Blue (+)
공통 Cathode (-) → GND
```

**주의:**
- 각 핀마다 220Ω 저항 필요
- 모든 핀이 PWM 핀이어야 함
- 공통 핀은 GND에 연결 (Common Cathode)

### 6.4 기본 코드

**색상 설정 함수:**
```c
const int RED_PIN = 9;
const int GREEN_PIN = 10;
const int BLUE_PIN = 11;

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
}

void setColor(int red, int green, int blue) {
  analogWrite(RED_PIN, red);
  analogWrite(GREEN_PIN, green);
  analogWrite(BLUE_PIN, blue);
}

void loop() {
  setColor(255, 0, 0);    // 빨강
  delay(1000);
  
  setColor(0, 255, 0);    // 초록
  delay(1000);
  
  setColor(0, 0, 255);    // 파랑
  delay(1000);
  
  setColor(255, 255, 0);  // 노랑
  delay(1000);
  
  setColor(255, 0, 255);  // 자홍
  delay(1000);
  
  setColor(0, 255, 255);  // 청록
  delay(1000);
  
  setColor(255, 255, 255); // 흰색
  delay(1000);
  
  setColor(0, 0, 0);      // 꺼짐
  delay(1000);
}
```

### 6.5 무지개 효과

**색상 변화 순서:**
빨강 → 주황 → 노랑 → 초록 → 파랑 → 남색 → 보라

```c
void loop() {
  // 빨강
  setColor(255, 0, 0);
  delay(500);
  
  // 주황
  setColor(255, 128, 0);
  delay(500);
  
  // 노랑
  setColor(255, 255, 0);
  delay(500);
  
  // 초록
  setColor(0, 255, 0);
  delay(500);
  
  // 파랑
  setColor(0, 0, 255);
  delay(500);
  
  // 남색
  setColor(75, 0, 130);
  delay(500);
  
  // 보라
  setColor(148, 0, 211);
  delay(500);
}
```

**부드러운 색상 전환:**
```c
void loop() {
  // 빨강 → 초록
  for (int i = 0; i <= 255; i++) {
    setColor(255 - i, i, 0);
    delay(10);
  }
  
  // 초록 → 파랑
  for (int i = 0; i <= 255; i++) {
    setColor(0, 255 - i, i);
    delay(10);
  }
  
  // 파랑 → 빨강
  for (int i = 0; i <= 255; i++) {
    setColor(i, 0, 255 - i);
    delay(10);
  }
}
```

---

## 7. 과제 안내 (5분)

### 과제 1: 촛불 효과 ⭐⭐

**요구사항**:
- LED 1개 (Pin 9)
- 불규칙하게 깜빡이는 촛불 효과
- 밝기 범위: 180~255
- `random()` 함수 사용
- Serial Monitor에 현재 밝기 출력

**힌트**:
```c
int brightness = random(180, 256);  // 180~255 랜덤 값
analogWrite(LED_PIN, brightness);
```

**출처**: [Arduino random()](https://www.arduino.cc/reference/en/language/functions/random-numbers/random/)

---

### 과제 2: RGB 무지개 페이드 ⭐⭐⭐

**요구사항**:
- RGB LED 1개
- 부드러운 무지개 색상 전환
- 빨강 → 노랑 → 초록 → 청록 → 파랑 → 자홍 → 빨강 (순환)
- `millis()` 사용 (비차단)
- 버튼(Pin 2)으로 속도 조절
- Serial Monitor에 현재 RGB 값 출력

**힌트**:
- 6단계 색상 배열 사용
- 각 색상 사이를 for 반복문으로 보간

---

### 과제 3: 음악 비트 시뮬레이션 ⭐⭐⭐⭐

**요구사항**:
- LED 3개 (Pin 9, 10, 11)
- 음악 비트에 맞춰 깜빡이는 효과 시뮬레이션
- LED1: 저음 (느린 깜빡임, 500ms)
- LED2: 중음 (중간 깜빡임, 300ms)
- LED3: 고음 (빠른 깜빡임, 100ms)
- 각 LED는 페이드 인/아웃
- `millis()` 사용
- 버튼으로 일시정지/재개

**힌트**:
- 각 LED마다 독립적인 페이드 타이머
- 비차단 타이밍 활용

---

## 8. 정리 및 다음 시간 예고 (10분)

### 오늘 배운 내용
✅ `analogWrite()` 함수 사용법  
✅ 0~255 값으로 밝기 조절  
✅ PWM 핀 확인 (~표시)  
✅ LED 밝기 단계 제어  
✅ for 반복문으로 페이드 효과  
✅ 비차단 페이드 구현  
✅ RGB LED 색상 제어  

### 핵심 개념 정리

**analogWrite() 사용:**
```c
analogWrite(pin, value);  // pin: PWM 핀 (~), value: 0~255
```

**페이드 효과:**
```c
for (int brightness = 0; brightness <= 255; brightness++) {
  analogWrite(LED_PIN, brightness);
  delay(10);
}
```

**RGB 색상:**
```c
void setColor(int r, int g, int b) {
  analogWrite(RED_PIN, r);
  analogWrite(GREEN_PIN, g);
  analogWrite(BLUE_PIN, b);
}
```

### 다음 시간 주제
- `analogRead()` 함수
- 아날로그 입력 센서 (포텐셔미터)
- 센서 값으로 LED 밝기 제어
- 센서 값 매핑 (`map()` 함수)
- 실시간 센서 데이터 처리

---

## 참고 자료

### 공식 문서
- [Arduino Reference - analogWrite()](https://www.arduino.cc/reference/en/language/functions/analog-io/analogwrite/)
- [Arduino Uno Pinout](https://docs.arduino.cc/hardware/uno-rev3/)
- [Arduino Fading Example](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Fading)
- [Arduino random()](https://www.arduino.cc/reference/en/language/functions/random-numbers/random/)

### 추가 학습 자료
- [Arduino PWM](https://www.arduino.cc/en/Tutorial/Foundations/PWM)
- [Arduino Fade Tutorial](https://docs.arduino.cc/built-in-examples/basics/Fade/)

---

## 문제 해결 (Troubleshooting)

### 일반적인 문제

**문제 1**: analogWrite()를 사용했는데 LED 밝기가 안 바뀜
- **원인**: PWM 핀이 아닌 핀 사용
- **해결**: Pin 3, 5, 6, 9, 10, 11 중 하나 사용 (~ 표시 확인)
```c
// ✗ 잘못된 핀:
analogWrite(2, 128);   // Pin 2는 PWM 불가능

// ✓ 올바른 핀:
analogWrite(9, 128);   // Pin 9는 PWM 가능
```

**문제 2**: LED가 깜빡거림
- **원인**: 값이 너무 낮거나 높음
- **해결**: 중간 값(50~200) 사용 확인
```c
analogWrite(9, 5);    // 너무 어두워서 깜빡이는 것처럼 보임
analogWrite(9, 128);  // 적절한 밝기
```

**문제 3**: RGB LED에서 원하는 색상이 안 나옴
- **원인 1**: Common Anode와 Common Cathode 혼동
- **해결**: Common Cathode 사용 확인
- **원인 2**: 저항 누락
- **해결**: 각 핀마다 220Ω 저항 연결
```c
// Common Cathode (일반적):
setColor(255, 0, 0);  // 빨강

// Common Anode인 경우 (반전 필요):
setColor(0, 255, 255);  // 빨강
```

**문제 4**: 페이드가 부드럽지 않음
- **원인**: delay 값이 너무 크거나 증감량이 너무 큼
- **해결**: delay 줄이거나 증감량 줄이기
```c
// ✗ 거침:
for (int i = 0; i <= 255; i += 10) {  // 증감량 10
  analogWrite(9, i);
  delay(50);  // 너무 긴 delay
}

// ✓ 부드러움:
for (int i = 0; i <= 255; i++) {  // 증감량 1
  analogWrite(9, i);
  delay(5);   // 짧은 delay
}
```

**문제 5**: 여러 LED 페이드 시 동기화됨
- **원인**: 하나의 타이머로 모든 LED 제어
- **해결**: 각 LED마다 독립적인 타이머 변수
```c
// 각 LED마다:
unsigned long previousMillis[3] = {0, 0, 0};
int brightness[3] = {0, 0, 0};
int fadeAmount[3] = {1, 1, 1};
```

**문제 6**: analogWrite() 후 digitalWrite()가 작동 안 함
- **원인**: analogWrite()가 PWM 타이머 설정
- **해결**: 다시 digitalWrite() 사용 가능 (자동으로 PWM 해제)
```c
analogWrite(9, 128);   // PWM 시작
digitalWrite(9, HIGH); // PWM 정지, 디지털 출력으로 전환
```

---

## 심화 개념

### RGB로 다양한 색상 표현

RGB LED로 다양한 색상을 만들 수 있습니다:

**기본 색상:**
```c
setColor(255, 0, 0);    // 빨강
setColor(0, 255, 0);    // 초록
setColor(0, 0, 255);    // 파랑
setColor(255, 255, 0);  // 노랑
setColor(255, 0, 255);  // 자홍
setColor(0, 255, 255);  // 청록
```

**중간 색상:**
```c
setColor(255, 128, 0);  // 주황
setColor(128, 0, 128);  // 보라
setColor(0, 128, 128);  // 청녹
setColor(255, 192, 203);  // 분홍
```

**회색 계열:**
```c
setColor(64, 64, 64);   // 어두운 회색
setColor(128, 128, 128); // 중간 회색
setColor(192, 192, 192); // 밝은 회색
setColor(255, 255, 255); // 흰색
```

**C 프로그래밍 관점:**
- RGB 값을 배열로 저장하여 관리
- for 반복문으로 색상 전환
- 구조체 활용 (추후 학습)

---

**수업 준비물 체크리스트**
□ Session 3 회로 유지  
□ PWM 가능 LED 준비 (Pin 9, 10, 11)  
□ RGB LED 1개 (Common Cathode)  
□ 220Ω 저항 추가 (RGB용 3개)  
□ Tinkercad에서 PWM 핀 확인  
□ Serial Monitor 준비  

**과제 제출 마감**: 다음 수업 시작 전까지
