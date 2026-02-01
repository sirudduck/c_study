# Arduino Week 2 Session 3: Serial 통신과 비차단 타이밍

## 학습 목표
- Serial 통신으로 디버깅
- `delay()`의 문제점 이해
- `millis()` 함수로 시간 측정
- 비차단(non-blocking) 타이밍 구현
- 반복문(for, while) 복습 및 활용

---

## 1. 동기부여: 문제 제시 (10분)

### 문제: LED가 깜빡이는 동안에도 버튼 입력을 받아라

**현재 코드의 문제:**
```c
void loop() {
  digitalWrite(13, HIGH);
  delay(1000);              // 1초 동안 멈춤!
  digitalWrite(13, LOW);
  delay(1000);              // 1초 동안 또 멈춤!
  
  // 이 사이에 버튼을 눌러도 감지 못함
}
```

**학생들에게 먼저 질문:**
- `delay()`를 사용하면 왜 버튼 입력을 놓칠까?
- 프로그램이 멈추지 않고 시간을 재는 방법은?
- 코드가 제대로 작동하는지 어떻게 확인할까?

**시연:**
1. LED 깜빡이기 + 버튼으로 다른 LED 제어
2. `delay()` 사용 버전: 깜빡이는 동안 버튼 무반응
3. `millis()` 사용 버전: 깜빡이면서 버튼도 반응

---

## 2. Serial 통신 기초 (15분)

### 2.1 Serial 통신이란?

**정의**: Arduino와 컴퓨터 간의 데이터 통신 방법

**비유:**
```
Arduino ←[USB 케이블]→ 컴퓨터

Arduino: "현재 버튼 상태는 HIGH입니다"
컴퓨터: Serial Monitor에 표시

마치 전화 통화처럼 양방향 소통 가능
```

**왜 필요한가?**
- LED로는 숫자를 표시하기 어려움
- 변수 값을 확인하고 싶을 때
- 프로그램이 제대로 작동하는지 디버깅

**출처**: [Arduino Reference - Serial](https://www.arduino.cc/reference/en/language/functions/communication/serial/)

### 2.2 Serial Monitor 사용하기

**Tinkercad에서 Serial Monitor 사용:**
1. 코드에 `Serial.begin(9600)`과 `Serial.println()` 작성
2. **"Start Simulation"** 버튼 클릭 (재생 버튼)
3. 시뮬레이션이 시작되면 화면 하단에 **"Serial Monitor" 창이 자동으로 나타남**
4. 보드레이트는 자동으로 코드의 `Serial.begin()` 값에 맞춰짐

**Arduino IDE에서 (참고용):**
- 도구 → 시리얼 모니터 (또는 Ctrl+Shift+M)
- 보드레이트를 코드의 `Serial.begin()` 값과 동일하게 설정 (예: 9600)

**주의:**
- Tinkercad에서는 시뮬레이션을 시작해야만 Serial Monitor가 보입니다
- Serial Monitor는 검색해서 추가하는 부품이 아닙니다

### 2.3 기본 함수

#### `Serial.begin(baud_rate)`

**정의**: Serial 통신 초기화

**매개변수**:
- `baud_rate`: 통신 속도 (보통 9600 사용)

**사용:**
```c
void setup() {
  Serial.begin(9600);  // setup()에서 한 번만 호출
}
```

**출처**: [Arduino Reference - Serial.begin()](https://www.arduino.cc/reference/en/language/functions/communication/serial/begin/)

#### `Serial.println(data)`

**정의**: 데이터를 Serial Monitor에 출력하고 줄바꿈

**매개변수**:
- `data`: 출력할 값 (문자열, 숫자, 변수 등)

**예시:**
```c
Serial.println("Hello Arduino");  // 문자열 출력
Serial.println(123);               // 숫자 출력
Serial.println(digitalRead(2));    // 변수 값 출력
```

**`Serial.print()` vs `Serial.println()`:**
- `print()`: 줄바꿈 없이 출력
- `println()`: 출력 후 줄바꿈

```c
Serial.print("A");    // 출력: A
Serial.print("B");    // 출력: AB
Serial.println("C");  // 출력: ABC (줄바꿈)
Serial.println("D");  // 출력: D (새 줄)
```

**출처**: [Arduino Reference - Serial.println()](https://www.arduino.cc/reference/en/language/functions/communication/serial/println/)

### 2.4 실습 예제

**버튼 상태 출력:**
```c
void setup() {
  Serial.begin(9600);           // Serial 통신 시작
  pinMode(2, INPUT_PULLUP);     // 버튼 설정
}

void loop() {
  int buttonState = digitalRead(2);
  
  Serial.print("Button: ");     // 텍스트 출력
  Serial.println(buttonState);  // 값 출력 (줄바꿈)
  
  delay(500);  // 0.5초마다 출력
}
```

**Serial Monitor 출력:**
```
Button: 1
Button: 1
Button: 0  (버튼 누름)
Button: 0
Button: 1  (버튼 뗌)
```

**변수 값 확인:**
```c
void loop() {
  int count = 0;
  
  for (int i = 0; i < 5; i++) {
    count = count + i;
    Serial.print("i = ");
    Serial.print(i);
    Serial.print(", count = ");
    Serial.println(count);
  }
}
```

**Serial Monitor 출력:**
```
i = 0, count = 0
i = 1, count = 1
i = 2, count = 3
i = 3, count = 6
i = 4, count = 10
```

### 2.5 디버깅 활용

**문제 상황**: 코드가 예상대로 작동하지 않을 때

**해결 방법**: Serial로 중간 값 확인
```c
void loop() {
  int sensorValue = analogRead(A0);
  
  // 디버깅: 값 확인
  Serial.print("Sensor: ");
  Serial.println(sensorValue);
  
  if (sensorValue > 500) {
    digitalWrite(13, HIGH);
  } else {
    digitalWrite(13, LOW);
  }
}
```

**수학적 비유:**
```
수학 문제 풀이:
  중간 과정을 적어가며 확인 → Serial.println()
  
프로그래밍:
  변수 값을 출력하며 확인 → Serial.println()
```

---

## 3. `delay()`의 문제점 (10분)

### 3.1 `delay()`는 블로킹(Blocking) 함수

**블로킹(Blocking)의 의미:**
- 프로그램 실행을 **완전히 멈춤**
- 다른 작업을 **전혀 수행할 수 없음**
- CPU가 **대기 상태**로 들어감

**비유:**
```
식당에서 주문을 받는 직원

[ delay() 사용 ]
손님1: 주문 → 직원: "잠시만요" → (3분 대기) → 음식 나옴
손님2: (기다림) → 직원이 손님1 처리 끝날 때까지 못 받음
손님3: (기다림)

[ millis() 사용 ]
손님1: 주문 → 주방에 전달 → 타이머 확인
손님2: 주문 → 주방에 전달 → 타이머 확인
손님3: 주문 → 주방에 전달 → 타이머 확인
→ 모든 손님 동시에 서빙 가능
```

**출처**: [Arduino Reference - delay()](https://www.arduino.cc/reference/en/language/functions/time/delay/)

### 3.2 다중 작업 불가능

**버튼 체크 못함:**
```c
void loop() {
  digitalWrite(13, HIGH);
  delay(1000);              // 이 동안 버튼 못 읽음
  digitalWrite(13, LOW);
  delay(1000);              // 이 동안 버튼 못 읽음
  
  if (digitalRead(2) == LOW) {  // 2초마다 한 번만 체크
    // 버튼 처리
  }
}
```

**Serial로 확인:**
```c
void loop() {
  Serial.println("Before delay");
  delay(2000);
  Serial.println("After delay");  // 2초 후에야 출력
}
```

---

## 4. `millis()` 함수 (15분)

### 4.1 `millis()` 함수의 원리

**정의**: Arduino가 시작된 후 경과된 시간을 밀리초(ms) 단위로 반환

**함수 시그니처:**
```c
unsigned long millis(void);
```

**반환값:**
- `unsigned long` 타입 (0 ~ 4,294,967,295)
- Arduino 시작 후 경과 시간 (밀리초)
- 약 49.7일 후 0으로 되돌아감 (overflow)

**예시:**
```c
void setup() {
  Serial.begin(9600);
}

void loop() {
  unsigned long currentTime = millis();
  Serial.print("Time: ");
  Serial.println(currentTime);
  delay(1000);
}
```

**Serial Monitor 출력:**
```
Time: 0
Time: 1000
Time: 2000
Time: 3000
```

**출처**: [Arduino Reference - millis()](https://www.arduino.cc/reference/en/language/functions/time/millis/)

### 4.2 스톱워치 비유

**`millis()`는 계속 돌아가는 스톱워치:**

```
Arduino 시작:     millis() = 0
1초 후:          millis() = 1000
2초 후:          millis() = 2000
5.5초 후:        millis() = 5500
1분 후:          millis() = 60000
```

**핵심:**
- `delay()`는 멈춤: "1초 기다려"
- `millis()`는 확인: "지금 몇 초야?"

### 4.3 시간 간격 측정

**경과 시간 계산:**
```c
unsigned long previousTime = 0;  // 이전 시각 저장

void loop() {
  unsigned long currentTime = millis();   // 현재 시각
  unsigned long elapsed = currentTime - previousTime;  // 경과 시간
  
  Serial.print("Elapsed: ");
  Serial.println(elapsed);
  
  if (elapsed >= 1000) {  // 1초 이상 경과?
    Serial.println("1 second passed!");
    previousTime = currentTime;  // 시각 업데이트
  }
}
```

**수학적 표현:**
```
Δt = t_현재 - t_이전

조건: Δt ≥ 1000ms 이면 작업 실행
```

---

## 5. 비차단 LED 깜빡이기 (20분)

### 5.1 `delay()` 버전 vs `millis()` 버전

**`delay()` 사용 (블로킹):**
```c
void loop() {
  digitalWrite(13, HIGH);
  delay(1000);              // 멈춤
  digitalWrite(13, LOW);
  delay(1000);              // 멈춤
}
```

**`millis()` 사용 (비차단):**
```c
const unsigned long INTERVAL = 1000;  // 깜빡임 간격 (1초)
unsigned long previousMillis = 0;     // 이전 시각
int ledState = LOW;                   // LED 현재 상태

void setup() {
  Serial.begin(9600);       // 디버깅용
  pinMode(13, OUTPUT);
}

void loop() {
  unsigned long currentMillis = millis();  // 현재 시각
  
  // 간격이 지났는지 확인
  if (currentMillis - previousMillis >= INTERVAL) {
    previousMillis = currentMillis;  // 시각 업데이트
    
    // LED 상태 토글
    ledState = !ledState;
    digitalWrite(13, ledState);
    
    // 디버깅: 상태 변경 확인
    Serial.print("LED changed at ");
    Serial.println(currentMillis);
  }
  
  // loop()가 계속 실행되므로 다른 작업 가능!
}
```

**출처**: [Blink Without Delay Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/BlinkWithoutDelay)

### 5.2 비차단 타이밍의 핵심

**3가지 핵심 요소:**

1. **간격 상수 (INTERVAL):**
   ```c
   const unsigned long INTERVAL = 1000;  // 작업 간격
   ```

2. **이전 시각 저장 (previousMillis):**
   ```c
   unsigned long previousMillis = 0;  // 마지막 작업 시각
   ```

3. **시간 비교 및 업데이트:**
   ```c
   if (currentMillis - previousMillis >= INTERVAL) {
     previousMillis = currentMillis;  // 시각 업데이트
     // 작업 수행
   }
   ```

**타임라인:**
```
t=0ms:    previousMillis = 0
t=500ms:  500 - 0 = 500 < 1000  → 작업 안 함
t=1000ms: 1000 - 0 = 1000 ≥ 1000 → 작업 수행, previousMillis = 1000
t=1500ms: 1500 - 1000 = 500 < 1000 → 작업 안 함
t=2000ms: 2000 - 1000 = 1000 ≥ 1000 → 작업 수행, previousMillis = 2000
```

### 5.3 LED + 버튼 동시 제어

**비차단 타이밍으로 다중 작업:**
```c
const unsigned long BLINK_INTERVAL = 1000;
unsigned long previousBlinkMillis = 0;
int ledState = LOW;

void setup() {
  Serial.begin(9600);
  pinMode(13, OUTPUT);      // LED
  pinMode(2, INPUT_PULLUP); // 버튼
  pinMode(12, OUTPUT);      // 버튼용 LED
}

void loop() {
  // 작업 1: LED 깜빡이기 (비차단)
  unsigned long currentMillis = millis();
  if (currentMillis - previousBlinkMillis >= BLINK_INTERVAL) {
    previousBlinkMillis = currentMillis;
    ledState = !ledState;
    digitalWrite(13, ledState);
    
    Serial.print("Blink at ");
    Serial.println(currentMillis);
  }
  
  // 작업 2: 버튼 체크 (항상 실행)
  if (digitalRead(2) == LOW) {
    digitalWrite(12, HIGH);
    Serial.println("Button pressed!");
  } else {
    digitalWrite(12, LOW);
  }
}
```

**핵심:**
- `loop()`가 매우 빠르게 반복 실행 (수천~수만 번/초)
- 각 반복마다 시간 체크 + 버튼 체크
- 시간 조건 만족 시에만 LED 상태 변경

---

## 6. 반복문 복습 (15분)

### 6.1 for 반복문

**기본 구조:**
```c
for (초기식; 조건식; 증감식) {
  // 반복할 코드
}
```

**수학적 대응:**
```
합 기호 Σ:
  n
  Σ  f(i) = f(0) + f(1) + f(2) + ... + f(n)
 i=0

C 언어:
for (int i = 0; i <= n; i++) {
  // f(i) 수행
}
```

**예시: 0부터 4까지 출력**
```c
void setup() {
  Serial.begin(9600);
  
  for (int i = 0; i < 5; i++) {
    Serial.println(i);
  }
  // Serial Monitor 출력: 0, 1, 2, 3, 4
}
```

**실행 순서:**
```
1. int i = 0        (초기화, 한 번만)
2. i < 5?           (조건 확인)
3. Serial.println(i) (본문 실행)
4. i++              (증가)
5. 2번으로 돌아감
```

**출처**: [Arduino Reference - for](https://www.arduino.cc/reference/en/language/structure/control-structure/for/)

### 6.2 while 반복문

**기본 구조:**
```c
while (조건식) {
  // 조건이 참인 동안 반복
}
```

**for vs while 선택:**
- **for**: 반복 횟수를 알 때 (배열 순회 등)
- **while**: 조건이 참인 동안 계속 (버튼 대기 등)

**출처**: [Arduino Reference - while](https://www.arduino.cc/reference/en/language/structure/control-structure/while/)

### 6.3 배열과 반복문 조합 (복습)

**이전 실습에서 배운 내용:**

여러 LED를 제어할 때 배열과 for 반복문을 함께 사용하면 코드가 간결해집니다.

```c
const int LED_PINS[3] = {11, 12, 13};
const int NUM_LEDS = 3;

void setup() {
  // 모든 LED 핀을 OUTPUT으로 설정
  for (int i = 0; i < NUM_LEDS; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  // 모든 LED 순차 점등
  for (int i = 0; i < NUM_LEDS; i++) {
    digitalWrite(LED_PINS[i], HIGH);
    delay(200);
    digitalWrite(LED_PINS[i], LOW);
  }
}
```

**여러 LED 독립적으로 제어:**

각 LED마다 독립적인 타이머를 사용하면 서로 다른 속도로 깜빡일 수 있습니다.

```c
const int NUM_LEDS = 3;
const int LED_PINS[NUM_LEDS] = {11, 12, 13};
const unsigned long INTERVALS[NUM_LEDS] = {500, 1000, 1500};

unsigned long previousMillis[NUM_LEDS] = {0, 0, 0};
int ledStates[NUM_LEDS] = {LOW, LOW, LOW};

void setup() {
  Serial.begin(9600);
  for (int i = 0; i < NUM_LEDS; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  unsigned long currentMillis = millis();
  
  for (int i = 0; i < NUM_LEDS; i++) {
    if (currentMillis - previousMillis[i] >= INTERVALS[i]) {
      previousMillis[i] = currentMillis;
      ledStates[i] = !ledStates[i];
      digitalWrite(LED_PINS[i], ledStates[i]);
      
      Serial.print("LED ");
      Serial.print(i);
      Serial.println(" changed");
    }
  }
}
```

---

## 7. 실습 (25분)

### 실습 1: Serial로 millis() 값 확인

**문제**: Serial Monitor로 millis() 값을 1초마다 출력하라

**요구사항**:
- 1초마다 현재 millis() 값 출력
- `delay()` 사용 금지
- "Time: [값]" 형식으로 출력

**힌트**: 비차단 타이밍 패턴 사용

---

### 실습 2: 3개 LED 순차 점등 (비차단)

**문제**: `delay()` 없이 3개 LED를 순차적으로 점등하라

**요구사항**:
- LED 3개 (Pin 11, 12, 13)
- 각 LED 500ms 간격
- `millis()` 사용
- 버튼(Pin 2)을 누르면 다른 LED(Pin 9) 켜짐 (동시에)
- Serial Monitor에 현재 켜진 LED 번호 출력

**힌트**:
- 배열에 LED 핀 저장
- 현재 켜져야 할 LED 인덱스를 변수로 저장
- 시간 간격마다 인덱스 증가

---

### 실습 3: 여러 LED 독립적으로 깜빡이기

**문제**: 3개 LED를 각각 다른 속도로 깜빡이게 만들어라

**요구사항**:
- LED 3개 (Pin 11, 12, 13)
- LED1: 300ms 간격
- LED2: 700ms 간격
- LED3: 1100ms 간격
- 모든 LED가 독립적으로 동작
- 각 LED마다 독립적인 타이머 사용

**힌트**: 
- 배열 3개 필요: LED 핀, 간격, 이전 시각
- for 반복문으로 각 LED 체크

---

## 8. C 문법 심화 (5분)

### 8.1 변수 범위 (Scope)

**전역 변수:**
```c
unsigned long previousMillis = 0;  // 모든 함수에서 접근 가능

void setup() {
  previousMillis = millis();
}

void loop() {
  previousMillis = millis();  // 접근 가능
}
```

**지역 변수:**
```c
void loop() {
  unsigned long currentMillis = millis();  // loop() 내에서만 접근 가능
  // 매번 새로 생성됨
}
```

**비차단 타이밍에서 전역 변수 필요:**
- `previousMillis`는 loop() 호출 사이에 값 유지 필요 → 전역
- `currentMillis`는 매번 새로 계산 → 지역

### 8.2 상수 (`const`)

**상수 선언:**
```c
const int LED_PIN = 13;        // 변경 불가
const unsigned long INTERVAL = 1000;

void setup() {
  // LED_PIN = 12;  // 컴파일 오류!
}
```

**장점:**
- 실수로 값 변경 방지
- 코드 가독성 향상

**출처**: [Arduino Reference - const](https://www.arduino.cc/reference/en/language/variables/variable-scope-qualifiers/const/)

---

## 9. 과제 안내 (5분)

### 과제 1: 신호등 시스템 (비차단) ⭐⭐

**요구사항**:
- LED 3개 (빨강-Pin 11, 노랑-Pin 12, 초록-Pin 13)
- 신호등 패턴:
  - 빨강: 5초
  - 노랑: 2초
  - 초록: 5초
  - 노랑: 2초
  - 반복
- `millis()` 사용 (delay 금지)
- 버튼(Pin 2)으로 긴급 모드: 누르는 동안 모든 LED 깜빡임
- Serial Monitor에 현재 신호등 상태 출력

**힌트**: 
- 현재 신호등 상태를 변수로 저장 (0=빨강, 1=노랑, 2=초록, 3=노랑)
- 각 상태마다 다른 간격 사용

---

### 과제 2: 나이트 라이더 효과 (비차단) ⭐⭐⭐

**요구사항**:
- LED 5개 (Pin 9, 10, 11, 12, 13)
- 한 개씩 순차 이동 (왕복)
- 100ms 간격
- `millis()` 사용
- 버튼(Pin 2)으로 속도 조절: 누를 때마다 50ms, 100ms, 200ms 순환
- Serial Monitor에 현재 위치와 속도 출력

**힌트**:
- 현재 위치(인덱스)와 방향(1 또는 -1) 변수 사용
- 끝에 도달하면 방향 반전

---

### 과제 3: 다중 타이머 시스템 ⭐⭐⭐⭐

**요구사항**:
- LED 4개 (Pin 9, 10, 11, 12)
- 각 LED마다 다른 깜빡임 패턴:
  - LED1: 300ms 켜짐, 300ms 꺼짐
  - LED2: 500ms 켜짐, 200ms 꺼짐
  - LED3: 1000ms 켜짐, 500ms 꺼짐
  - LED4: 800ms 켜짐, 800ms 꺼짐
- 모든 LED 독립적으로 동작
- 버튼(Pin 2)으로 일시정지/재개
- Serial Monitor에 각 LED 상태 변경 시 출력

**힌트**:
- 각 LED마다 2개의 간격 변수 필요 (켜짐 간격, 꺼짐 간격)
- 배열 사용

---

## 10. 정리 및 다음 시간 예고 (5분)

### 오늘 배운 내용
✅ Serial 통신으로 디버깅  
✅ `delay()`의 한계 이해  
✅ `millis()`로 시간 측정  
✅ 비차단 타이밍 구현  
✅ for/while 반복문 복습  
✅ 배열과 반복문 조합  
✅ 여러 작업 동시 처리  

### 핵심 개념 정리

**Serial 통신:**
```c
void setup() {
  Serial.begin(9600);
}

void loop() {
  Serial.println("Debug message");
}
```

**비차단 타이밍 패턴:**
```c
unsigned long previousMillis = 0;
const unsigned long INTERVAL = 1000;

void loop() {
  unsigned long currentMillis = millis();
  
  if (currentMillis - previousMillis >= INTERVAL) {
    previousMillis = currentMillis;
    // 작업 수행
  }
  
  // 다른 작업도 수행 가능
}
```

### 다음 시간 주제
- PWM (Pulse Width Modulation)
- `analogWrite()` 함수
- LED 밝기 조절
- 모터 속도 제어 기초
- 부드러운 페이드 효과

---

## 참고 자료

### 공식 문서
- [Arduino Reference - Serial](https://www.arduino.cc/reference/en/language/functions/communication/serial/)
- [Arduino Reference - Serial.begin()](https://www.arduino.cc/reference/en/language/functions/communication/serial/begin/)
- [Arduino Reference - Serial.println()](https://www.arduino.cc/reference/en/language/functions/communication/serial/println/)
- [Arduino Reference - millis()](https://www.arduino.cc/reference/en/language/functions/time/millis/)
- [Arduino Tutorial - Blink Without Delay](https://www.arduino.cc/en/Tutorial/BuiltInExamples/BlinkWithoutDelay)
- [Arduino Reference - for](https://www.arduino.cc/reference/en/language/structure/control-structure/for/)
- [Arduino Reference - while](https://www.arduino.cc/reference/en/language/structure/control-structure/while/)
- [Arduino Reference - const](https://www.arduino.cc/reference/en/language/variables/variable-scope-qualifiers/const/)

### 추가 학습 자료
- [Arduino Arrays Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Arrays)
- [Understanding millis() - Adafruit](https://learn.adafruit.com/multi-tasking-the-arduino-part-1/overview)
- [Arduino Serial Communication](https://docs.arduino.cc/learn/communication/serial)

---

## 문제 해결 (Troubleshooting)

### 일반적인 문제

**문제 0**: Serial Monitor에 아무것도 안 나옴
- **원인**: Serial 통신 초기화 누락 또는 보드레이트 불일치
- **해결**:
  1. `setup()`에 `Serial.begin(9600)` 확인
  2. Serial Monitor 보드레이트가 9600인지 확인
  3. USB 케이블 연결 확인
  4. 올바른 COM 포트 선택 확인
  5. Tinkercad에서는 시뮬레이션 시작 후 Serial Monitor 확인

**문제 1**: LED가 불규칙하게 깜빡임
- **원인**: `previousMillis` 업데이트 누락
- **해결**: 조건문 안에서 `previousMillis = currentMillis` 확인
```c
if (currentMillis - previousMillis >= INTERVAL) {
  previousMillis = currentMillis;  // 반드시 필요!
  // 작업 수행
}
```

**문제 2**: 시간이 정확하지 않음 (점점 느려짐)
- **원인**: `previousMillis += INTERVAL` 사용
- **해결**: `previousMillis = currentMillis` 사용
```c
// ✗ 잘못된 방법:
previousMillis += INTERVAL;  // 오차 누적

// ✓ 올바른 방법:
previousMillis = currentMillis;  // 현재 시각으로 리셋
```

**문제 3**: Overflow (49.7일 후 millis() 초기화)
- **원인**: `unsigned long` 최대값 초과 시 0으로 돌아감
- **해결**: 뺄셈 사용 시 자동으로 처리됨
```c
// 안전한 방법 (overflow 자동 처리)
if (currentMillis - previousMillis >= INTERVAL) {
  // 정상 작동
}
```
- **설명**: Unsigned 뺄셈은 overflow 시에도 올바른 결과 보장

**문제 4**: 배열 인덱스 범위 초과
- **원인**: 배열 크기보다 큰 인덱스 사용
```c
int ledPins[5] = {9, 10, 11, 12, 13};
digitalWrite(ledPins[5], HIGH);  // 오류! (0~4만 유효)
```
- **해결**: 항상 `0 ~ (크기-1)` 범위 사용
```c
for (int i = 0; i < 5; i++) {  // i는 0~4
  digitalWrite(ledPins[i], HIGH);
}
```

**문제 5**: 여러 LED가 동시에 깜빡임 (독립적으로 안 움직임)
- **원인**: 하나의 타이머로 모든 LED 제어
- **해결**: 각 LED마다 별도의 타이머 변수 사용
```c
// LED마다 독립적인 previousMillis 필요
unsigned long previousMillis[3] = {0, 0, 0};
```

**문제 6**: `delay()`와 `millis()` 혼용 시 문제
- **원인**: `delay()`가 전체 프로그램 멈춤
- **해결**: `delay()` 완전히 제거하고 `millis()`만 사용
```c
// ✗ 혼용하지 말 것:
if (currentMillis - previousMillis >= 1000) {
  digitalWrite(13, HIGH);
  delay(500);  // 다른 타이머에 영향!
}

// ✓ millis()만 사용:
if (currentMillis - previousMillis >= 1000) {
  digitalWrite(13, HIGH);
  previousMillis = currentMillis;
}
```

---

## 심화 개념

### millis() Overflow 처리 원리

**Unsigned 연산의 특성:**
```c
unsigned long a = 4294967295;  // 최대값
unsigned long b = a + 1;       // b = 0 (overflow)

// 하지만 뺄셈은 정상 작동:
unsigned long diff = (a + 2) - a;  // diff = 2 (올바름!)
```

**왜 안전한가?**
- C 표준에서 unsigned 연산은 모듈로 연산
- `(현재 - 이전)`은 항상 올바른 차이를 반환
- Overflow 시에도 수학적으로 정확

**참고**: [Arduino millis() rollover handling](https://arduino.stackexchange.com/questions/12587/how-can-i-handle-the-millis-rollover)

---

**수업 준비물 체크리스트**
□ Session 2 회로 유지  
□ 추가 LED 준비 (총 5개)  
□ 추가 220Ω 저항 준비  
□ Tinkercad 접속 확인  
□ Serial Monitor 사용법 숙지  
□ USB 케이블 및 COM 포트 확인  

**과제 제출 마감**: 다음 수업 시작 전까지
