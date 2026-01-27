# Arduino Week 1 Session 2: 버튼 입력과 조건문

## 학습 목표
- 디지털 입력 (`digitalRead`) 이해
- 버튼(Pushbutton) 회로 구성
- Pull-up 저항 개념 학습
- 조건문(if-else) 복습 및 활용
- INPUT과 OUTPUT의 차이 이해

---

## 1. 동기부여: 문제 제시 (10분)

### 문제: 버튼을 누르면 LED가 켜지고, 떼면 꺼지게 만들어라

**학생들에게 먼저 질문:**
- 프로그램이 버튼이 눌렸는지 어떻게 알 수 있을까?
- 버튼 상태에 따라 LED 제어를 어떻게 구현할까?
- "누름"과 "안 누름"을 코드로 어떻게 구별할까?

---

## 2. 복습: INPUT vs OUTPUT (15분)

### 2.1 지난 시간 복습

**디지털 핀의 두 가지 모드:**

| 모드 | 역할 | 비유 | 함수 | 예시 |
|------|------|------|------|------|
| OUTPUT | 전압 **공급** | 펌프 (물을 내보냄) | `digitalWrite()` | LED 제어 |
| INPUT | 전압 **읽기** | 수압계 (압력 측정) | `digitalRead()` | 버튼 감지 |

**핵심 차이:**
- **OUTPUT**: Arduino가 능동적으로 전압을 결정 (HIGH/LOW 설정)
- **INPUT**: 외부 신호를 수동적으로 읽음 (HIGH/LOW 측정)

**출처**: [Arduino Digital Pins](https://www.arduino.cc/en/Tutorial/Foundations/DigitalPins)

### 2.2 전압과 전류의 차이

**전압 (Voltage):**
- 전기적 압력
- 단위: V (볼트)
- 병렬 연결 시 **동일하게 유지**

**전류 (Current):**
- 전기의 흐름
- 단위: A (암페어)
- 병렬 연결 시 **나뉘어짐**

**물탱크 비유:**
```
    5m 높이 물탱크 (5V)
         │
    ┌────┴────┐
    │         │
  LED1      LED2
```
- 두 LED 모두 **5V 압력(전압)** 받음
- 물의 양(전류)은 나뉘어 흐름
- USB는 최대 500mA 전류 공급 가능

**출처**: [Voltage, Current, Resistance - SparkFun](https://learn.sparkfun.com/tutorials/voltage-current-resistance-and-ohms-law/all)

---

## 3. 버튼(Pushbutton) 이해 (20분)

### 3.1 버튼의 구조

**버튼 다리 배치:**
```
1a ─┬─ 1b  (항상 연결)
    │
  [버튼]
    │
2a ─┬─ 2b  (항상 연결)

버튼 누르면: 1과 2가 연결됨
```

**내부 연결:**
- **1a와 1b**: 항상 전기적으로 연결
- **2a와 2b**: 항상 전기적으로 연결
- **버튼 누르기 전**: 1과 2는 **분리**
- **버튼 누른 후**: 1과 2가 **연결**

**올바른 연결 방법:**
- Pin 2 → **1**a (또는 1b)
- GND → **2**a (또는 2b)
- **핵심**: 숫자만 다르게 (1과 2), 문자는 상관없음

**잘못된 연결:**
- Pin 2 → 1a, GND → 1b (✗ 같은 1)
- 이미 연결된 상태이므로 버튼 기능 없음

**출처**: [Button Tutorial - Arduino](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Button)

### 3.2 Floating 상태 문제

**문제 상황:**
```
Pin 2 ─── 버튼 (안 누름) ─── GND
     ?
```

버튼을 누르지 않았을 때:
- Pin 2가 어디에도 연결되지 않음 (Floating)
- 전압이 불확실 (2V? 3V? 4V?)
- 주변 노이즈에 영향받음
- HIGH/LOW가 랜덤하게 읽힘

**비유:**
- 줄에 매달리지 않은 풍선이 이리저리 떠다니는 것과 같음

---

## 4. Pull-up 저항 (25분)

### 4.1 Pull-up 저항의 원리

**회로 구조:**
```
        5V (HIGH)
         │
        10kΩ (내부 저항)
         │
Pin 2 ───┼─── 버튼 ─── GND (LOW)
```

**동작 원리:**

**버튼 안 눌렀을 때:**
```
        5V
         │
        10kΩ
         │
Pin 2 ───┤  (버튼 열림)
         
         × (연결 끊김)
         
        GND
```
- Pin 2는 저항을 통해 **5V에 연결** (Pull-up)
- GND로 가는 경로가 차단됨
- 전류가 거의 흐르지 않음
- **Pin 2 = HIGH (5V)**

**버튼 눌렀을 때:**
```
        5V
         │
        10kΩ
         │
Pin 2 ───┼─── 버튼(닫힘) ─── GND
         │
         └→ 전류 흐름
```
- Pin 2가 GND에 **직접 연결**
- 전류가 흐름: 5V → 저항 → Pin 2 → 버튼 → GND
- **Pin 2 = LOW (0V)**

**출처**: [Pull-up Resistors - SparkFun](https://learn.sparkfun.com/tutorials/pull-up-resistors/all)

### 4.2 물 파이프 비유

**버튼 안 눌렀을 때:**
```
    물탱크 (5V, 높은 수압)
        │
     좁은 파이프 (저항)
        │
Pin 2 ──┤   × (밸브 닫힘)
        
    나갈 곳 없음
```
- 물이 Pin 2까지 차오름
- 나갈 곳이 없어서 **수압이 높음**
- Pin 2 수압 = 5V (HIGH)

**버튼 눌렀을 때:**
```
    물탱크 (5V)
        │
     좁은 파이프
        │
Pin 2 ──┼── 밸브(열림) ── 땅 (GND)
        │
        └→ 물 흐름
```
- 물이 땅으로 흘러감
- Pin 2 수압이 낮아짐
- Pin 2 수압 = 0V (LOW)

### 4.3 Arduino 내부 Pull-up 저항

**Arduino의 편리한 기능:**
- 모든 디지털 핀에 **내부 Pull-up 저항** 내장 (약 20kΩ~50kΩ)
- `INPUT_PULLUP` 모드로 자동 활성화
- 외부 저항 불필요!

**코드 예시:**
```c
pinMode(2, INPUT_PULLUP);  // 내부 Pull-up 활성화
```

**출처**: [INPUT_PULLUP - Arduino Reference](https://www.arduino.cc/en/Tutorial/DigitalInputPullup)

---

## 5. 첫 번째 버튼 회로 (25분)

### 5.1 회로 구성

**필요한 부품:**
- Arduino Uno R3 (기존)
- Breadboard (기존)
- Pushbutton 1개 (추가)
- LED 1개 (기존 Pin 13)
- 220Ω 저항 1개 (기존)
- 점퍼선 2개 (추가)

**회로도:**
```
Arduino Pin 2 → 버튼 (1a 또는 1b)
버튼 (2a 또는 2b) → 브레드보드 GND 레일

기존 LED 회로 유지:
Arduino Pin 13 → 220Ω → LED(+) → LED(-) → GND 레일
Arduino GND → GND 레일
```

**연결 단계:**
1. Tinkercad에서 "Pushbutton" 검색 후 추가
2. 버튼을 브레드보드 중앙부에 배치
   - 중앙 홈을 가로지르도록 배치 (1과 2가 좌우로 분리)
3. Pin 2 → 버튼 1a (빨강/노랑 점퍼선)
4. 버튼 2a → 브레드보드 GND 레일 (검정 점퍼선)
5. 기존 LED 회로는 그대로 유지

### 5.2 회로 확인 체크리스트

□ 버튼이 브레드보드 중앙 홈을 가로지름  
□ Pin 2와 버튼의 1a(또는 1b) 연결  
□ 버튼의 2a(또는 2b)와 GND 레일 연결  
□ 버튼 다리 위치 확인 (1과 2가 다른 쪽)  
□ 기존 LED 회로 정상 작동  

---

## 6. 버튼 제어 코드 (30분)

### 6.1 기본 코드

```c
void setup() {
  pinMode(2, INPUT_PULLUP);   // 버튼 핀 (내부 Pull-up)
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

### 6.2 함수 상세 설명

#### `digitalRead(pin)`

**정의**: 디지털 핀의 상태(HIGH 또는 LOW)를 읽음

**매개변수**:
- `pin` (int): 읽을 핀 번호

**반환값**:
- `HIGH` (1): 핀 전압이 약 3V 이상
- `LOW` (0): 핀 전압이 약 2V 이하

**예시**:
```c
int buttonState = digitalRead(2);
if (buttonState == LOW) {
  // 버튼이 눌린 상태
}
```

**출처**: [Arduino Reference - digitalRead()](https://www.arduino.cc/reference/en/language/functions/digital-io/digitalread/)

#### `INPUT_PULLUP` 모드

**설정:**
```c
pinMode(2, INPUT_PULLUP);
```

**효과:**
- 내부 Pull-up 저항 활성화 (약 20kΩ~50kΩ)
- 버튼 안 눌렀을 때: HIGH
- 버튼 눌렀을 때: LOW

**주의:**
- `INPUT_PULLUP` 사용 시 로직이 **반전**됨
- 버튼 누름 = LOW (일반적인 직관과 반대)

### 6.3 조건문(if-else) 복습

**기본 구조:**
```c
if (조건) {
  // 조건이 참(true)일 때 실행
} else {
  // 조건이 거짓(false)일 때 실행
}
```

**수학적 비교:**
```
조건문: if (x > 0)
수학: x > 0일 때와 x ≤ 0일 때로 경우 나누기
```

**비교 연산자:**
| 연산자 | 의미 | 예시 |
|--------|------|------|
| `==` | 같다 | `digitalRead(2) == LOW` |
| `!=` | 다르다 | `digitalRead(2) != HIGH` |
| `>` | 크다 | `x > 5` |
| `<` | 작다 | `x < 10` |
| `>=` | 크거나 같다 | `x >= 0` |
| `<=` | 작거나 같다 | `x <= 100` |

**출처**: [Arduino Reference - if-else](https://www.arduino.cc/reference/en/language/structure/control-structure/if/)

---

## 7. 변형 실습 (25분)

### 실습 1: 토글 동작 구현

**문제**: 버튼을 누를 때마다 LED 상태가 토글(켜짐↔꺼짐)되게 만들어라

**힌트**: 
- 변수를 사용하여 LED 상태를 저장
- 버튼을 눌렀다가 뗄 때 상태 변경
- 이전 버튼 상태와 현재 버튼 상태를 비교

---

### 실습 2: 2개 버튼으로 2개 LED 제어

**문제**: 버튼 2개와 LED 2개를 사용하여 각각 독립적으로 제어하라

**추가 부품**:
- 버튼 1개 (Pin 3)
- LED 1개 (Pin 12)
- 220Ω 저항 1개

**힌트**: 
- 두 버튼을 각각 `INPUT_PULLUP`으로 설정
- 각 버튼에 대해 독립적인 `if` 문 사용

---

### 실습 3: 버튼 길게 누르기 감지

**문제**: 버튼을 2초 이상 누르고 있으면 LED가 켜지게 만들어라

**힌트**: 
- `millis()` 함수 사용
- 버튼이 눌린 시간을 측정
- 2000ms 이상이면 LED 켜기

---

## 8. C 문법 복습 (10분)

### 8.1 논리 연산자

**여러 조건 결합:**

```c
// AND (그리고): 둘 다 참일 때
if (digitalRead(2) == LOW && digitalRead(3) == LOW) {
  // 버튼 2개를 모두 누를 때
}

// OR (또는): 하나라도 참일 때
if (digitalRead(2) == LOW || digitalRead(3) == LOW) {
  // 버튼 하나라도 누를 때
}

// NOT (부정): 반대
if (!(digitalRead(2) == LOW)) {
  // 버튼을 누르지 않을 때
}
```

**수학적 대응:**
- AND (`&&`): 교집합 (∩)
- OR (`||`): 합집합 (∪)
- NOT (`!`): 여집합

**출처**: [Arduino Reference - Boolean Operators](https://www.arduino.cc/reference/en/language/structure/boolean-operators/boolean/)

### 8.2 변수와 상태 저장

```c
int ledState = LOW;      // LED 현재 상태
int buttonState;         // 버튼 현재 상태
int lastButtonState = HIGH;  // 버튼 이전 상태

void loop() {
  buttonState = digitalRead(2);
  
  // 버튼 상태가 변했는지 확인
  if (buttonState != lastButtonState) {
    // 상태 변화 처리
  }
  
  lastButtonState = buttonState;  // 이전 상태 업데이트
}
```

---

## 9. 과제 안내 (5분)

### 과제 1: 안전 확인 시스템 ⭐⭐

**요구사항**:
- 버튼 2개 사용 (Pin 2, Pin 3)
- LED 1개 (Pin 13)
- **두 버튼을 동시에** 눌러야 LED가 켜짐
- 하나라도 떼면 LED가 꺼짐

**힌트**: 논리 연산자 AND (`&&`) 사용

**제출 방법**:
1. Tinkercad에서 회로 구성 및 코드 작성
2. Tinkercad 링크 공유
3. 코드에 주석으로 동작 원리 설명

---

### 과제 2: 카운터 시스템 ⭐⭐⭐

**요구사항**:
- 버튼 1개 (Pin 2)
- LED 3개 (Pin 11, 12, 13)
- 버튼을 누를 때마다 카운트 증가
- LED로 카운트 수를 이진수로 표시
  - 0: 모두 꺼짐
  - 1: LED1만 켜짐 (001)
  - 2: LED2만 켜짐 (010)
  - 3: LED1, LED2 켜짐 (011)
  - ...
  - 7: 모두 켜짐 (111)
  - 8: 다시 0으로

**힌트**: 
- 변수를 사용하여 카운트 저장
- 버튼 누름 감지 (이전 상태와 비교)
- 비트 연산 또는 조건문으로 LED 제어

---

## 10. 정리 및 다음 시간 예고 (5분)

### 오늘 배운 내용
✅ 디지털 입력 (`digitalRead`)  
✅ 버튼 회로 구성  
✅ Pull-up 저항 개념  
✅ INPUT_PULLUP 모드  
✅ 조건문 활용  
✅ INPUT과 OUTPUT의 차이  

### 다음 시간 주제
- 반복 패턴 제어
- `millis()` 함수로 정밀한 시간 제어
- `delay()` 없이 LED 깜빡이기
- for/while 루프 복습

---

## 참고 자료

### 공식 문서
- [Arduino Digital Pins](https://www.arduino.cc/en/Tutorial/Foundations/DigitalPins)
- [Arduino digitalRead()](https://www.arduino.cc/reference/en/language/functions/digital-io/digitalread/)
- [Arduino INPUT_PULLUP](https://www.arduino.cc/en/Tutorial/DigitalInputPullup)
- [Pull-up Resistors - SparkFun](https://learn.sparkfun.com/tutorials/pull-up-resistors/all)

### 추가 학습 자료
- [Button Tutorial - Arduino](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Button)
- [Debouncing Buttons](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Debounce)

---

## 문제 해결 (Troubleshooting)

### 일반적인 문제

**문제 1**: 버튼을 누르지 않아도 LED가 켜짐/꺼짐을 반복
- **원인**: Floating 상태 (Pull-up 저항 미사용)
- **해결**: `pinMode(2, INPUT_PULLUP)` 확인
- **확인 방법**: `INPUT` 대신 `INPUT_PULLUP` 사용했는지 체크

**문제 2**: 버튼을 눌러도 아무 반응 없음
- **확인 사항**:
  - 버튼 연결 확인 (1과 2가 다른 쪽에 연결되었는지)
  - 버튼이 브레드보드 중앙 홈을 가로지르는지
  - GND 레일 연결 확인
  - 코드에서 올바른 핀 번호 사용

**문제 3**: 버튼이 여러 번 눌린 것처럼 동작 (채터링)
- **원인**: 기계적 접점의 물리적 진동
- **해결**: 디바운싱(Debouncing) 구현
  ```c
  delay(50);  // 간단한 디바운싱
  ```
- **참고**: [Debounce Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/Debounce)

**문제 4**: INPUT_PULLUP을 사용했는데 로직이 반대로 동작
- **정상 동작**: INPUT_PULLUP 사용 시
  - 버튼 안 누름 = HIGH
  - 버튼 누름 = LOW
- **확인**: 조건문에서 `== LOW`로 버튼 누름 확인

**문제 5**: 여러 버튼 사용 시 하나만 동작
- **확인 사항**:
  - 각 버튼이 서로 다른 핀에 연결되었는지
  - 모든 버튼이 공통 GND 레일에 연결되었는지
  - 각 버튼에 대해 `pinMode()` 설정했는지

---

**수업 준비물 체크리스트**
□ Session 1 회로 유지  
□ Pushbutton 추가 준비  
□ 점퍼선 여분 확보  
□ Tinkercad 접속 확인  

**과제 제출 마감**: 다음 수업 시작 전까지
