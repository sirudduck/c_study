# Arduino Session 2 실습 및 과제 정답

## 기본 버튼-LED 제어 코드

```c
void setup() {
  pinMode(2, INPUT_PULLUP);   // 버튼 핀
  pinMode(13, OUTPUT);        // LED 핀
}

void loop() {
  if (digitalRead(2) == LOW) {  // 버튼 눌렀을 때
    digitalWrite(13, HIGH);     // LED 켜기
  } else {                       // 버튼 안 눌렀을 때
    digitalWrite(13, LOW);      // LED 끄기
  }
}
```

---

## 실습 1: 토글 동작 구현

### 코드 (디바운싱 포함)

```c
int ledState = LOW;              // LED 현재 상태
int buttonState;                 // 버튼 현재 상태
int lastButtonState = HIGH;      // 버튼 이전 상태

void setup() {
  pinMode(2, INPUT_PULLUP);
  pinMode(13, OUTPUT);
  digitalWrite(13, ledState);    // 초기 LED 상태 설정
}

void loop() {
  buttonState = digitalRead(2);
  
  // 버튼 상태가 변했는지 확인
  if (buttonState != lastButtonState) {
    delay(50);  // 디바운싱 (채터링 방지)
    
    // 버튼이 눌렸을 때 (HIGH → LOW 전환)
    if (buttonState == LOW) {
      ledState = !ledState;      // 상태 반전
      digitalWrite(13, ledState);
    }
  }
  
  lastButtonState = buttonState;  // 이전 상태 업데이트
}
```

**설명:**
- `ledState`: LED의 켜짐/꺼짐 상태 저장
- `lastButtonState`: 이전 버튼 상태를 저장하여 상태 변화 감지
- `!ledState`: 논리 NOT 연산자로 상태 반전 (LOW → HIGH, HIGH → LOW)
- `delay(50)`: 버튼의 기계적 진동(채터링) 방지

---

### 간단한 버전 (디바운싱 없음)

```c
int ledState = LOW;
int lastButtonState = HIGH;

void setup() {
  pinMode(2, INPUT_PULLUP);
  pinMode(13, OUTPUT);
}

void loop() {
  int buttonState = digitalRead(2);
  
  if (buttonState == LOW && lastButtonState == HIGH) {
    ledState = !ledState;
    digitalWrite(13, ledState);
  }
  
  lastButtonState = buttonState;
  delay(10);  // 짧은 지연
}
```

---

## 실습 2: 2개 버튼으로 2개 LED 제어

### 회로 구성

```
Arduino Pin 2 → 버튼1 (1a) → 버튼1 (2a) → GND 레일
Arduino Pin 3 → 버튼2 (1a) → 버튼2 (2a) → GND 레일

Arduino Pin 13 → 220Ω → LED1(+) → LED1(-) → GND 레일
Arduino Pin 12 → 220Ω → LED2(+) → LED2(-) → GND 레일

Arduino GND → GND 레일
```

### 코드

```c
void setup() {
  pinMode(2, INPUT_PULLUP);   // 버튼1
  pinMode(3, INPUT_PULLUP);   // 버튼2
  pinMode(13, OUTPUT);        // LED1
  pinMode(12, OUTPUT);        // LED2
}

void loop() {
  // 버튼1 → LED1 제어
  if (digitalRead(2) == LOW) {
    digitalWrite(13, HIGH);
  } else {
    digitalWrite(13, LOW);
  }
  
  // 버튼2 → LED2 제어
  if (digitalRead(3) == LOW) {
    digitalWrite(12, HIGH);
  } else {
    digitalWrite(12, LOW);
  }
}
```

---

## 실습 3: 버튼 길게 누르기 감지

```c
const int BUTTON_PIN = 2;
const int LED_PIN = 13;
const long LONG_PRESS_TIME = 2000;  // 2초 (밀리초)

unsigned long pressedTime = 0;
bool isPressing = false;
bool isLongDetected = false;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW) {  // 버튼 눌림
    if (!isPressing) {
      isPressing = true;
      pressedTime = millis();  // 누른 시간 기록
      isLongDetected = false;
    }
    
    // 2초 이상 눌렀는지 확인
    if (!isLongDetected && (millis() - pressedTime > LONG_PRESS_TIME)) {
      digitalWrite(LED_PIN, HIGH);
      isLongDetected = true;
    }
  } else {  // 버튼 떼짐
    isPressing = false;
    if (!isLongDetected) {
      digitalWrite(LED_PIN, LOW);
    }
  }
}
```

**설명:**
- `millis()`: 프로그램 시작 후 경과 시간(밀리초)
- `pressedTime`: 버튼을 누른 시점의 시간
- `millis() - pressedTime`: 버튼을 누르고 있는 시간
- `isLongDetected`: 긴 누름이 이미 감지되었는지 확인 (중복 방지)

---

## 과제 1: 안전 확인 시스템

### 회로 구성

```
Arduino Pin 2 → 버튼1 → GND 레일
Arduino Pin 3 → 버튼2 → GND 레일
Arduino Pin 13 → 220Ω → LED(+) → LED(-) → GND 레일
Arduino GND → GND 레일
```

### 코드

```c
const int BUTTON1_PIN = 2;
const int BUTTON2_PIN = 3;
const int LED_PIN = 13;

void setup() {
  pinMode(BUTTON1_PIN, INPUT_PULLUP);
  pinMode(BUTTON2_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  // 두 버튼을 모두 눌렀을 때만 LED 켜기
  if (digitalRead(BUTTON1_PIN) == LOW && digitalRead(BUTTON2_PIN) == LOW) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }
}
```

**설명:**
- AND 연산자 (`&&`): 두 조건이 모두 참일 때만 실행
- 버튼1과 버튼2를 모두 누르면 (둘 다 LOW) LED 켜짐
- 하나라도 떼면 (하나라도 HIGH) LED 꺼짐

---

## 과제 2: 카운터 시스템

### 회로 구성

```
Arduino Pin 2 → 버튼 → GND 레일
Arduino Pin 11 → 220Ω → LED1(+) → LED1(-) → GND 레일
Arduino Pin 12 → 220Ω → LED2(+) → LED2(-) → GND 레일
Arduino Pin 13 → 220Ω → LED3(+) → LED3(-) → GND 레일
Arduino GND → GND 레일
```

### 코드 (비트 연산 사용)

```c
const int BUTTON_PIN = 2;
const int LED_PINS[3] = {11, 12, 13};  // LED 핀 배열

int count = 0;
int lastButtonState = HIGH;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  
  for (int i = 0; i < 3; i++) {
    pinMode(LED_PINS[i], OUTPUT);
  }
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  // 버튼이 눌렸는지 확인 (HIGH → LOW 전환)
  if (buttonState == LOW && lastButtonState == HIGH) {
    delay(50);  // 디바운싱
    
    count++;
    if (count > 7) {  // 0~7 범위
      count = 0;
    }
    
    // 카운트를 이진수로 LED에 표시
    displayBinary(count);
  }
  
  lastButtonState = buttonState;
}

void displayBinary(int num) {
  for (int i = 0; i < 3; i++) {
    // 비트 연산: num의 i번째 비트가 1인지 확인
    if (num & (1 << i)) {
      digitalWrite(LED_PINS[i], HIGH);
    } else {
      digitalWrite(LED_PINS[i], LOW);
    }
  }
}
```

**비트 연산 설명:**
- `1 << i`: 1을 왼쪽으로 i비트 이동
  - `1 << 0` = 0001 (1)
  - `1 << 1` = 0010 (2)
  - `1 << 2` = 0100 (4)
- `num & (1 << i)`: num과 비트 AND 연산
  - 예: num=5 (0101) & (1 << 0) = 0001 → true
  - 예: num=5 (0101) & (1 << 1) = 0000 → false

---

### 코드 (조건문 사용 - 더 간단)

```c
const int BUTTON_PIN = 2;
const int LED1_PIN = 11;
const int LED2_PIN = 12;
const int LED3_PIN = 13;

int count = 0;
int lastButtonState = HIGH;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(LED3_PIN, OUTPUT);
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW && lastButtonState == HIGH) {
    delay(50);
    
    count++;
    if (count > 7) {
      count = 0;
    }
    
    displayNumber(count);
  }
  
  lastButtonState = buttonState;
}

void displayNumber(int num) {
  // LED1 (1의 자리)
  digitalWrite(LED1_PIN, (num & 1) ? HIGH : LOW);
  
  // LED2 (2의 자리)
  digitalWrite(LED2_PIN, (num & 2) ? HIGH : LOW);
  
  // LED3 (4의 자리)
  digitalWrite(LED3_PIN, (num & 4) ? HIGH : LOW);
}
```

**이진수 표시 예시:**
- 0 (000): 모든 LED 꺼짐
- 1 (001): LED1만 켜짐
- 2 (010): LED2만 켜짐
- 3 (011): LED1, LED2 켜짐
- 4 (100): LED3만 켜짐
- 5 (101): LED1, LED3 켜짐
- 6 (110): LED2, LED3 켜짐
- 7 (111): 모든 LED 켜짐

---

## 과제 3 (선택): 반응 속도 게임

### 코드

```c
const int BUTTON_PIN = 2;
const int LED_PIN = 13;

void setup() {
  Serial.begin(9600);  // 시리얼 통신 시작
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
  
  randomSeed(analogRead(0));  // 랜덤 시드 초기화
}

void loop() {
  Serial.println("준비...");
  delay(2000);
  
  // 1~5초 사이 랜덤 대기
  long randomDelay = random(1000, 5000);
  delay(randomDelay);
  
  // LED 켜기
  digitalWrite(LED_PIN, HIGH);
  Serial.println("지금!");
  
  unsigned long startTime = millis();
  
  // 버튼 누를 때까지 대기
  while (digitalRead(BUTTON_PIN) == HIGH) {
    // 기다리기
  }
  
  unsigned long reactionTime = millis() - startTime;
  
  // 결과 출력
  digitalWrite(LED_PIN, LOW);
  Serial.print("반응 시간: ");
  Serial.print(reactionTime);
  Serial.println(" ms");
  
  // 평가
  if (reactionTime < 200) {
    Serial.println("결과: 매우 빠름!");
  } else if (reactionTime < 400) {
    Serial.println("결과: 빠름");
  } else if (reactionTime < 600) {
    Serial.println("결과: 보통");
  } else {
    Serial.println("결과: 느림");
  }
  
  delay(3000);  // 다음 라운드 전 대기
}
```

**시리얼 모니터 사용법:**
1. Tinkercad에서 "Serial Monitor" 클릭
2. 또는 Arduino IDE에서 Tools → Serial Monitor
3. Baud rate: 9600

**함수 설명:**
- `Serial.begin(9600)`: 시리얼 통신 초기화
- `Serial.println()`: 시리얼 모니터에 출력
- `random(min, max)`: min 이상 max 미만의 랜덤 정수
- `randomSeed()`: 랜덤 시드 설정 (진정한 랜덤을 위해)

---

## 추가 예제: 디바운싱 라이브러리 방식

복잡한 버튼 제어를 위한 더 견고한 디바운싱 방법:

```c
const int BUTTON_PIN = 2;
const int LED_PIN = 13;
const int DEBOUNCE_DELAY = 50;

int ledState = LOW;
int buttonState;
int lastButtonState = HIGH;
unsigned long lastDebounceTime = 0;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, ledState);
}

void loop() {
  int reading = digitalRead(BUTTON_PIN);
  
  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }
  
  if ((millis() - lastDebounceTime) > DEBOUNCE_DELAY) {
    if (reading != buttonState) {
      buttonState = reading;
      
      if (buttonState == LOW) {
        ledState = !ledState;
        digitalWrite(LED_PIN, ledState);
      }
    }
  }
  
  lastButtonState = reading;
}
```

---

## 핵심 개념 정리

### 1. INPUT_PULLUP 로직

| 버튼 상태 | 전기적 연결 | digitalRead() 결과 |
|-----------|-------------|-------------------|
| 안 누름 | Pin → 5V (저항 통해) | HIGH |
| 누름 | Pin → GND (직접) | LOW |

### 2. 상태 변화 감지 패턴

```c
// 이전 상태 저장
int lastState = HIGH;

// 현재 상태 읽기
int currentState = digitalRead(PIN);

// 변화 감지
if (currentState != lastState) {
  // 상태가 변했을 때 처리
}

// 이전 상태 업데이트
lastState = currentState;
```

### 3. 디바운싱 필요성

버튼의 기계적 접점은 눌릴 때 여러 번 튕기는 현상 발생:
```
물리적: ___/¯¯¯\___/\___/¯¯¯¯¯
실제:   ___/¯¯¯¯¯¯¯¯¯¯¯¯¯¯
```

해결 방법:
- `delay()` 사용 (간단)
- `millis()` 사용 (더 정확)

---

**참고**: 이 정답 파일은 교육용이며, 학생들이 스스로 문제를 해결한 후 확인 용도로만 사용하세요.
