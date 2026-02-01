# Arduino Week 2 Session 3: 비차단 타이밍과 반복문

## 학습 목표
- `delay()`의 문제점 이해
- `millis()` 함수로 시간 측정
- 비차단(non-blocking) 타이밍 구현
- 반복문(for, while) 복습 및 활용
- 배열과 반복문 조합

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
- 여러 LED를 각각 다른 속도로 깜빡이려면?

**시연:**
1. LED 깜빡이기 + 버튼으로 다른 LED 제어
2. `delay()` 사용 버전: 깜빡이는 동안 버튼 무반응
3. `millis()` 사용 버전: 깜빡이면서 버튼도 반응

---

## 2. `delay()`의 문제점 (15분)

### 2.1 `delay()`는 블로킹(Blocking) 함수

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

### 2.2 `delay()`의 한계

**단일 작업만 가능:**
```c
// LED만 깜빡일 수 있음
void loop() {
  digitalWrite(13, HIGH);
  delay(1000);
  digitalWrite(13, LOW);
  delay(1000);
}
```

**다중 작업 불가능:**
```c
// LED 깜빡이는 동안 버튼 체크 못함
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

---

## 3. `millis()` 함수 (20분)

### 3.1 `millis()` 함수의 원리

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
unsigned long currentTime = millis();
// Arduino 시작 후 5초 경과 → currentTime = 5000
```

**출처**: [Arduino Reference - millis()](https://www.arduino.cc/reference/en/language/functions/time/millis/)

### 3.2 스톱워치 비유

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

**수학적 대응:**
```
delay(1000)     → 1초 동안 정지 (명령형)
millis()        → 현재 시각 조회 (함수 호출)

시계 비유:
delay()  = 시계 보지 않고 "1, 2, 3, ..., 60" 세기
millis() = 시계를 계속 보면서 "지금 몇 시?" 확인
```

### 3.3 시간 간격 측정

**경과 시간 계산:**
```c
unsigned long previousTime = millis();  // 이전 시각 저장

// ... 어떤 작업 수행 ...

unsigned long currentTime = millis();   // 현재 시각
unsigned long elapsed = currentTime - previousTime;  // 경과 시간

if (elapsed >= 1000) {  // 1초 이상 경과?
  // 작업 수행
  previousTime = currentTime;  // 시각 업데이트
}
```

**수학적 표현:**
```
Δt = t_현재 - t_이전

조건: Δt ≥ 1000ms 이면 작업 실행
```

---

## 4. 비차단 LED 깜빡이기 (25분)

### 4.1 `delay()` 버전 vs `millis()` 버전

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
  pinMode(13, OUTPUT);
}

void loop() {
  unsigned long currentMillis = millis();  // 현재 시각
  
  // 간격이 지났는지 확인
  if (currentMillis - previousMillis >= INTERVAL) {
    previousMillis = currentMillis;  // 시각 업데이트
    
    // LED 상태 토글
    if (ledState == LOW) {
      ledState = HIGH;
    } else {
      ledState = LOW;
    }
    
    digitalWrite(13, ledState);
  }
  
  // loop()가 계속 실행되므로 다른 작업 가능!
}
```

**출처**: [Blink Without Delay Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/BlinkWithoutDelay)

### 4.2 비차단 타이밍의 핵심

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

### 4.3 LED + 버튼 동시 제어

**비차단 타이밍으로 다중 작업:**
```c
const unsigned long BLINK_INTERVAL = 1000;
unsigned long previousBlinkMillis = 0;
int ledState = LOW;

void setup() {
  pinMode(13, OUTPUT);      // LED
  pinMode(2, INPUT_PULLUP); // 버튼
  pinMode(12, OUTPUT);      // 버튼용 LED
}

void loop() {
  // 작업 1: LED 깜빡이기 (비차단)
  unsigned long currentMillis = millis();
  if (currentMillis - previousBlinkMillis >= BLINK_INTERVAL) {
    previousBlinkMillis = currentMillis;
    ledState = !ledState;  // 상태 반전
    digitalWrite(13, ledState);
  }
  
  // 작업 2: 버튼 체크 (항상 실행)
  if (digitalRead(2) == LOW) {
    digitalWrite(12, HIGH);
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

## 5. 반복문 복습 (20분)

### 5.1 for 반복문

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
  // 출력: 0, 1, 2, 3, 4
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

### 5.2 while 반복문

**기본 구조:**
```c
while (조건식) {
  // 조건이 참인 동안 반복
}
```

**예시:**
```c
int count = 0;

void loop() {
  while (count < 5) {
    Serial.println(count);
    count++;
  }
  // 출력: 0, 1, 2, 3, 4 (한 번만)
}
```

**for vs while 선택:**
- **for**: 반복 횟수를 알 때 (배열 순회 등)
- **while**: 조건이 참인 동안 계속 (버튼 대기 등)

**출처**: [Arduino Reference - while](https://www.arduino.cc/reference/en/language/structure/control-structure/while/)

### 5.3 수학적 활용 예시

**팩토리얼 계산:**
```c
// n! = n × (n-1) × (n-2) × ... × 1

int factorial(int n) {
  int result = 1;
  for (int i = 1; i <= n; i++) {
    result = result * i;
  }
  return result;
}
```

**등차수열 합:**
```c
// S = a + (a+d) + (a+2d) + ... + (a+(n-1)d)

int arithmeticSum(int a, int d, int n) {
  int sum = 0;
  for (int i = 0; i < n; i++) {
    sum = sum + (a + i * d);
  }
  return sum;
}
```

---

## 6. 배열과 반복문 (25분)

### 6.1 배열 복습

**배열의 수학적 개념:**
```
벡터 표현: v = [v₀, v₁, v₂, v₃, v₄]
C 배열:   int v[5] = {9, 10, 11, 12, 13};

인덱스: 0부터 시작
v[0] = 9
v[1] = 10
v[4] = 13
```

**배열 선언:**
```c
int ledPins[5] = {9, 10, 11, 12, 13};  // LED 핀 번호들
```

**배열 크기:**
```c
int size = sizeof(ledPins) / sizeof(ledPins[0]);
// sizeof(ledPins) = 20 바이트 (int 5개 × 4바이트)
// sizeof(ledPins[0]) = 4 바이트
// size = 20 / 4 = 5
```

### 6.2 for 반복문으로 배열 순회

**모든 LED 켜기:**
```c
const int LED_PINS[5] = {9, 10, 11, 12, 13};
const int NUM_LEDS = 5;

void setup() {
  // 모든 LED 핀을 OUTPUT으로 설정
  for (int i = 0; i < NUM_LEDS; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  // 모든 LED 켜기
  for (int i = 0; i < NUM_LEDS; i++) {
    digitalWrite(LED_PINS[i], HIGH);
  }
}
```

**왜 유용한가?**

❌ **배열 없이 (반복 코드):**
```c
pinMode(9, OUTPUT);
pinMode(10, OUTPUT);
pinMode(11, OUTPUT);
pinMode(12, OUTPUT);
pinMode(13, OUTPUT);
```

✅ **배열 + for 반복문 (간결):**
```c
for (int i = 0; i < NUM_LEDS; i++) {
  pinMode(LED_PINS[i], OUTPUT);
}
```

### 6.3 순차 점등 패턴

**왼쪽에서 오른쪽으로:**
```c
const int LED_PINS[5] = {9, 10, 11, 12, 13};
const int NUM_LEDS = 5;

void setup() {
  for (int i = 0; i < NUM_LEDS; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  // 순차 점등
  for (int i = 0; i < NUM_LEDS; i++) {
    digitalWrite(LED_PINS[i], HIGH);
    delay(200);
    digitalWrite(LED_PINS[i], LOW);
  }
}
```

**오른쪽에서 왼쪽으로:**
```c
void loop() {
  // 역순 점등
  for (int i = NUM_LEDS - 1; i >= 0; i--) {
    digitalWrite(LED_PINS[i], HIGH);
    delay(200);
    digitalWrite(LED_PINS[i], LOW);
  }
}
```

### 6.4 여러 LED 독립적으로 깜빡이기

**문제**: 각 LED를 다른 속도로 깜빡이게 하려면?

**해결책: 각 LED마다 타이머 사용**
```c
const int NUM_LEDS = 3;
const int LED_PINS[NUM_LEDS] = {11, 12, 13};
const unsigned long INTERVALS[NUM_LEDS] = {500, 1000, 1500};  // 각각 다른 간격

unsigned long previousMillis[NUM_LEDS] = {0, 0, 0};  // 각 LED의 이전 시각
int ledStates[NUM_LEDS] = {LOW, LOW, LOW};           // 각 LED의 상태

void setup() {
  for (int i = 0; i < NUM_LEDS; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  unsigned long currentMillis = millis();
  
  // 각 LED를 독립적으로 제어
  for (int i = 0; i < NUM_LEDS; i++) {
    if (currentMillis - previousMillis[i] >= INTERVALS[i]) {
      previousMillis[i] = currentMillis;
      ledStates[i] = !ledStates[i];
      digitalWrite(LED_PINS[i], ledStates[i]);
    }
  }
}
```

**핵심 개념:**
- 각 LED마다 독립적인 타이머 (`previousMillis[i]`)
- 각 LED마다 독립적인 간격 (`INTERVALS[i]`)
- for 반복문으로 모든 LED 체크

---

## 7. 실습 (25분)

### 실습 1: 3개 LED 순차 점등 (비차단)

**문제**: `delay()` 없이 3개 LED를 순차적으로 점등하라

**요구사항**:
- LED 3개 (Pin 11, 12, 13)
- 각 LED 500ms 간격
- `millis()` 사용
- 버튼(Pin 2)을 누르면 다른 LED(Pin 9) 켜짐 (동시에)

**힌트**:
- 현재 켜져야 할 LED 인덱스를 변수로 저장
- 시간 간격마다 인덱스 증가

---

### 실습 2: 여러 속도로 깜빡이기

**문제**: 3개 LED를 각각 다른 속도로 깜빡이게 만들어라

**요구사항**:
- LED 3개 (Pin 11, 12, 13)
- LED1: 300ms 간격
- LED2: 700ms 간격
- LED3: 1100ms 간격
- 모든 LED가 독립적으로 동작

**힌트**: 앞의 6.4 예제 참고

---

### 실습 3: 패턴 저장 및 재생

**문제**: 배열에 패턴을 저장하고 순차적으로 재생하라

**요구사항**:
- LED 5개 (Pin 9, 10, 11, 12, 13)
- 패턴 배열: `{1, 0, 1, 1, 0}` (1=켜짐, 0=꺼짐)
- 500ms마다 다음 패턴으로 이동
- 패턴이 끝나면 처음부터 반복

**힌트**:
- 2차원 배열 또는 여러 1차원 배열 사용
- 현재 패턴 인덱스 저장

---

## 8. C 문법 심화 (10분)

### 8.1 변수 범위 (Scope)

**전역 변수:**
```c
int globalVar = 0;  // 모든 함수에서 접근 가능

void setup() {
  globalVar = 10;
}

void loop() {
  globalVar++;  // 접근 가능
}
```

**지역 변수:**
```c
void loop() {
  int localVar = 0;  // loop() 내에서만 접근 가능
  localVar++;
}  // localVar 소멸

void setup() {
  // localVar 접근 불가!
}
```

**비차단 타이밍에서 전역 변수 사용:**
```c
// 전역: loop() 호출 사이에 값 유지
unsigned long previousMillis = 0;

void loop() {
  unsigned long currentMillis = millis();  // 지역: 매번 새로 생성
  
  if (currentMillis - previousMillis >= 1000) {
    previousMillis = currentMillis;  // 전역 변수 업데이트
  }
}
```

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
- 컴파일러 최적화 가능

**출처**: [Arduino Reference - const](https://www.arduino.cc/reference/en/language/variables/variable-scope-qualifiers/const/)

### 8.3 비트 연산 (선택)

**NOT 연산자로 토글:**
```c
int ledState = LOW;

ledState = !ledState;  // LOW → HIGH, HIGH → LOW

// 같은 의미:
if (ledState == LOW) {
  ledState = HIGH;
} else {
  ledState = LOW;
}
```

**XOR 연산자로 토글 (고급):**
```c
ledState ^= 1;  // 0 → 1, 1 → 0
```

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

**힌트**:
- 각 LED마다 2개의 타이머 변수 (켜짐 타이머, 꺼짐 타이머)
- 배열 사용

---

## 10. 정리 및 다음 시간 예고 (10분)

### 오늘 배운 내용
✅ `delay()`의 한계 이해  
✅ `millis()`로 시간 측정  
✅ 비차단 타이밍 구현  
✅ for/while 반복문 복습  
✅ 배열과 반복문 조합  
✅ 여러 작업 동시 처리  

### 핵심 개념 정리

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
- [Arduino Reference - millis()](https://www.arduino.cc/reference/en/language/functions/time/millis/)
- [Arduino Tutorial - Blink Without Delay](https://www.arduino.cc/en/Tutorial/BuiltInExamples/BlinkWithoutDelay)
- [Arduino Reference - for](https://www.arduino.cc/reference/en/language/structure/control-structure/for/)
- [Arduino Reference - while](https://www.arduino.cc/reference/en/language/structure/control-structure/while/)
- [Arduino Reference - const](https://www.arduino.cc/reference/en/language/variables/variable-scope-qualifiers/const/)

### 추가 학습 자료
- [Arduino Arrays Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Arrays)
- [Understanding millis() - Adafruit](https://learn.adafruit.com/multi-tasking-the-arduino-part-1/overview)

---

## 문제 해결 (Troubleshooting)

### 일반적인 문제

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
□ Serial Monitor 사용법 복습  

**과제 제출 마감**: 다음 수업 시작 전까지
