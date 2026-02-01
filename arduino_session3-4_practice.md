# Arduino Session 3-4 수업 실습 문제

## 안내사항

**제출 방법:**
- 실습 시간 내에 완성
- Tinkercad 링크 제출 (선택)
- 막히는 부분은 질문

**평가:**
- 실습 참여도
- 문제 해결 과정

---

## 실습 1: Serial Monitor로 카운터 출력 ⭐

**학습 목표:**
- Serial 통신 기본 사용
- for 반복문 복습

**문제:**
0부터 10까지 숫자를 Serial Monitor에 1초 간격으로 출력하시오.

**요구사항:**
- Serial.begin(9600) 사용
- for 반복문 사용
- "Count: [숫자]" 형식으로 출력
- 10까지 출력 후 다시 0부터 시작

**회로:**
- Arduino만 필요 (LED 불필요)

**예상 출력:**
```
Count: 0
Count: 1
Count: 2
...
Count: 10
Count: 0
Count: 1
...
```

**힌트:**
```c
void loop() {
  for (int i = 0; i <= 10; i++) {
    Serial.print("Count: ");
    Serial.println(i);
    delay(1000);
  }
}
```

**체크리스트:**
□ Serial.begin(9600) 호출  
□ for 반복문 사용  
□ Serial.println() 사용  
□ 1초 간격 유지  

---

## 실습 2: millis()로 LED 깜빡이기 (비차단) ⭐⭐

**학습 목표:**
- `millis()` 함수 사용
- 비차단 타이밍 구현
- Serial로 디버깅

**문제:**
`delay()` 없이 LED를 1초 간격으로 깜빡이게 만드시오.

**요구사항:**
- LED 1개 (Pin 13)
- `millis()` 사용
- `delay()` 사용 금지
- Serial Monitor에 LED 상태 변경 시 출력

**회로:**
```
Arduino Pin 13 → (내장 LED 사용)
또는
Arduino Pin 13 → 220Ω → LED(+) → LED(-) → GND
```

**예상 출력:**
```
LED ON at 1000
LED OFF at 2000
LED ON at 3000
LED OFF at 4000
...
```

**힌트:**
```c
unsigned long previousMillis = 0;
const unsigned long INTERVAL = 1000;
int ledState = LOW;

void loop() {
  unsigned long currentMillis = millis();
  
  if (currentMillis - previousMillis >= INTERVAL) {
    previousMillis = currentMillis;
    ledState = !ledState;
    digitalWrite(13, ledState);
    
    Serial.print("LED ");
    Serial.print(ledState ? "ON" : "OFF");
    Serial.print(" at ");
    Serial.println(currentMillis);
  }
}
```

**체크리스트:**
□ previousMillis 변수 선언  
□ millis() 사용  
□ delay() 사용 안 함  
□ Serial 출력 포함  

---

## 실습 3: 버튼으로 LED 밝기 단계 조절 ⭐⭐

**학습 목표:**
- `analogWrite()` 함수 사용
- 버튼 입력 처리
- 밝기 단계 관리

**문제:**
버튼을 누를 때마다 LED 밝기가 5단계로 변하게 만드시오.

**요구사항:**
- LED 1개 (Pin 9, PWM 핀)
- 버튼 1개 (Pin 2)
- 밝기 5단계: 0 → 64 → 128 → 192 → 255 → 0 (순환)
- Serial Monitor에 현재 밝기 출력

**회로:**
```
Arduino Pin 9 → 220Ω → LED(+) → LED(-) → GND
Arduino Pin 2 → 버튼 → GND
```

**예상 출력:**
```
Brightness: 0
Brightness: 64
Brightness: 128
Brightness: 192
Brightness: 255
Brightness: 0
...
```

**힌트:**
```c
const int BRIGHTNESS_LEVELS[5] = {0, 64, 128, 192, 255};
int currentLevel = 0;
int lastButtonState = HIGH;

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW && lastButtonState == HIGH) {
    currentLevel = (currentLevel + 1) % 5;
    analogWrite(LED_PIN, BRIGHTNESS_LEVELS[currentLevel]);
    
    Serial.print("Brightness: ");
    Serial.println(BRIGHTNESS_LEVELS[currentLevel]);
    
    delay(200);  // 디바운싱
  }
  
  lastButtonState = buttonState;
}
```

**체크리스트:**
□ PWM 핀 (9번) 사용  
□ analogWrite() 사용  
□ 배열로 밝기 관리  
□ 버튼 디바운싱 처리  
□ Serial 출력 포함  

---

## 실습 4: LED 페이드 인/아웃 ⭐⭐

**학습 목표:**
- for 반복문으로 페이드 효과
- `analogWrite()` 활용

**문제:**
LED가 서서히 밝아졌다가 서서히 어두워지기를 반복하게 만드시오.

**요구사항:**
- LED 1개 (Pin 9)
- 페이드 인: 0 → 255 (한 번에 5씩 증가)
- 페이드 아웃: 255 → 0 (한 번에 5씩 감소)
- 각 단계마다 30ms 간격
- Serial Monitor에 현재 밝기 50 단위로 출력

**회로:**
```
Arduino Pin 9 → 220Ω → LED(+) → LED(-) → GND
```

**예상 출력:**
```
Brightness: 0
Brightness: 50
Brightness: 100
Brightness: 150
Brightness: 200
Brightness: 250
Brightness: 200
Brightness: 150
...
```

**힌트:**
```c
void loop() {
  // 페이드 인
  for (int brightness = 0; brightness <= 255; brightness += 5) {
    analogWrite(LED_PIN, brightness);
    
    if (brightness % 50 == 0) {  // 50 단위로 출력
      Serial.print("Brightness: ");
      Serial.println(brightness);
    }
    
    delay(30);
  }
  
  // 페이드 아웃
  for (int brightness = 255; brightness >= 0; brightness -= 5) {
    analogWrite(LED_PIN, brightness);
    
    if (brightness % 50 == 0) {
      Serial.print("Brightness: ");
      Serial.println(brightness);
    }
    
    delay(30);
  }
}
```

**체크리스트:**
□ for 반복문 2개 사용  
□ analogWrite() 사용  
□ 증감량 5로 설정  
□ delay(30) 사용  
□ Serial 출력 (50 단위)  

---

## 실습 5: RGB LED 기본 색상 전환 ⭐⭐⭐

**학습 목표:**
- RGB LED 사용
- 배열로 색상 관리
- 여러 PWM 핀 제어

**문제:**
RGB LED로 6가지 기본 색상을 순서대로 표시하시오.

**요구사항:**
- RGB LED 1개 (Pin 9, 10, 11)
- 색상 순서: 빨강 → 초록 → 파랑 → 노랑 → 자홍 → 청록 → 빨강 (순환)
- 각 색상 1초 유지
- Serial Monitor에 현재 색상 이름 출력

**회로:**
```
Arduino Pin 9 → 220Ω → Red (+)
Arduino Pin 10 → 220Ω → Green (+)
Arduino Pin 11 → 220Ω → Blue (+)
공통 Cathode (-) → GND
```

**예상 출력:**
```
Color: Red
Color: Green
Color: Blue
Color: Yellow
Color: Magenta
Color: Cyan
...
```

**힌트:**
```c
const int RED_PIN = 9;
const int GREEN_PIN = 10;
const int BLUE_PIN = 11;

void setColor(int r, int g, int b) {
  analogWrite(RED_PIN, r);
  analogWrite(GREEN_PIN, g);
  analogWrite(BLUE_PIN, b);
}

void loop() {
  Serial.println("Color: Red");
  setColor(255, 0, 0);
  delay(1000);
  
  Serial.println("Color: Green");
  setColor(0, 255, 0);
  delay(1000);
  
  // 나머지 색상들...
}
```

**체크리스트:**
□ 3개 PWM 핀 사용  
□ setColor() 함수 작성  
□ 6가지 색상 순서대로  
□ 각 저항 220Ω 연결  
□ Serial 출력 포함  

---

## 실습 완료 확인

모든 실습을 완료했다면:
1. 코드가 정상 작동하는지 확인
2. Serial Monitor 출력 확인
3. 회로 연결 재확인
4. Tinkercad 링크 저장 (필요 시)

**추가 도전:**
- 실습 2: 버튼 추가하여 깜빡임 속도 조절
- 실습 3: 밝기 단계를 10단계로 증가
- 실습 4: 비차단 타이밍으로 변경
- 실습 5: 색상 사이를 부드럽게 전환
