# Arduino Session 1 실습 및 과제 정답

## 실습 1: 깜빡임 속도 변경

### 변형 1: 0.5초 간격

```c
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(500);   // 0.5초
  digitalWrite(13, LOW);
  delay(500);   // 0.5초
}
```

---

### 변형 2: 2초 간격

```c
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(2000);   // 2초
  digitalWrite(13, LOW);
  delay(2000);   // 2초
}
```

---

### 도전 과제: 0.1초 간격

```c
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(100);   // 0.1초
  digitalWrite(13, LOW);
  delay(100);   // 0.1초
}
```

---

## 실습 2: SOS 신호 만들기

```c
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  // S: 짧게 3번 (· · ·)
  for (int i = 0; i < 3; i++) {
    digitalWrite(13, HIGH);
    delay(200);   // 짧은 신호
    digitalWrite(13, LOW);
    delay(200);   // 신호 간격
  }
  
  delay(400);   // 글자 사이 간격
  
  // O: 길게 3번 (― ― ―)
  for (int i = 0; i < 3; i++) {
    digitalWrite(13, HIGH);
    delay(600);   // 긴 신호
    digitalWrite(13, LOW);
    delay(200);   // 신호 간격
  }
  
  delay(400);   // 글자 사이 간격
  
  // S: 짧게 3번 (· · ·)
  for (int i = 0; i < 3; i++) {
    digitalWrite(13, HIGH);
    delay(200);   // 짧은 신호
    digitalWrite(13, LOW);
    delay(200);   // 신호 간격
  }
  
  delay(3000);   // SOS 신호 사이 3초 대기
}
```

**참고**: 반복문을 아직 배우지 않은 학생들은 다음과 같이 작성할 수도 있습니다:

```c
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  // S: 짧게 3번 (· · ·)
  // 첫 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  // 두 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  // 세 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  delay(400);   // 글자 사이 간격
  
  // O: 길게 3번 (― ― ―)
  // 첫 번째
  digitalWrite(13, HIGH);
  delay(600);
  digitalWrite(13, LOW);
  delay(200);
  
  // 두 번째
  digitalWrite(13, HIGH);
  delay(600);
  digitalWrite(13, LOW);
  delay(200);
  
  // 세 번째
  digitalWrite(13, HIGH);
  delay(600);
  digitalWrite(13, LOW);
  delay(200);
  
  delay(400);   // 글자 사이 간격
  
  // S: 짧게 3번 (· · ·)
  // 첫 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  // 두 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  // 세 번째
  digitalWrite(13, HIGH);
  delay(200);
  digitalWrite(13, LOW);
  delay(200);
  
  delay(3000);   // SOS 신호 사이 3초 대기
}
```

---

## 실습 3: 여러 핀 사용하기

```c
void setup() {
  pinMode(12, OUTPUT);   // LED 1
  pinMode(13, OUTPUT);   // LED 2
}

void loop() {
  // LED 1 켜고, LED 2 끄기
  digitalWrite(12, HIGH);
  digitalWrite(13, LOW);
  delay(500);
  
  // LED 1 끄고, LED 2 켜기
  digitalWrite(12, LOW);
  digitalWrite(13, HIGH);
  delay(500);
}
```

---

## 과제 1: 교통 신호등 만들기

### 기본 코드

```c
void setup() {
  pinMode(11, OUTPUT);   // 빨강 LED
  pinMode(12, OUTPUT);   // 노랑 LED
  pinMode(13, OUTPUT);   // 초록 LED
}

void loop() {
  // 빨강 신호: 5초
  digitalWrite(11, HIGH);   // 빨강 켜기
  digitalWrite(12, LOW);    // 노랑 끄기
  digitalWrite(13, LOW);    // 초록 끄기
  delay(5000);              // 5초 대기
  
  // 노랑 신호: 2초
  digitalWrite(11, LOW);    // 빨강 끄기
  digitalWrite(12, HIGH);   // 노랑 켜기
  digitalWrite(13, LOW);    // 초록 끄기
  delay(2000);              // 2초 대기
  
  // 초록 신호: 5초
  digitalWrite(11, LOW);    // 빨강 끄기
  digitalWrite(12, LOW);    // 노랑 끄기
  digitalWrite(13, HIGH);   // 초록 켜기
  delay(5000);              // 5초 대기
}
```

---

### 향상된 코드 (함수 사용)

함수를 배운 학생들은 다음과 같이 작성할 수도 있습니다:

```c
// 전역 상수 정의
const int RED_PIN = 11;
const int YELLOW_PIN = 12;
const int GREEN_PIN = 13;

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(YELLOW_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
}

// 모든 LED를 끄는 함수
void turnOffAll() {
  digitalWrite(RED_PIN, LOW);
  digitalWrite(YELLOW_PIN, LOW);
  digitalWrite(GREEN_PIN, LOW);
}

// 특정 LED만 켜는 함수
void setLight(int pin, int duration) {
  turnOffAll();              // 먼저 모든 LED 끄기
  digitalWrite(pin, HIGH);   // 지정된 LED 켜기
  delay(duration);           // 지정된 시간만큼 대기
}

void loop() {
  setLight(RED_PIN, 5000);      // 빨강 5초
  setLight(YELLOW_PIN, 2000);   // 노랑 2초
  setLight(GREEN_PIN, 5000);    // 초록 5초
}
```

---

## 과제 2 (선택): 자유 LED 패턴 예시

### 예시 1: 경고 알람

```c
const int LED1 = 11;
const int LED2 = 12;
const int LED3 = 13;

void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
}

void loop() {
  // 빠르게 모두 깜빡임 (긴급 신호)
  for (int i = 0; i < 5; i++) {
    digitalWrite(LED1, HIGH);
    digitalWrite(LED2, HIGH);
    digitalWrite(LED3, HIGH);
    delay(100);
    
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, LOW);
    digitalWrite(LED3, LOW);
    delay(100);
  }
  
  delay(1000);   // 1초 대기
  
  // 순차적으로 켜기
  digitalWrite(LED1, HIGH);
  delay(200);
  digitalWrite(LED2, HIGH);
  delay(200);
  digitalWrite(LED3, HIGH);
  delay(500);
  
  // 모두 끄기
  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
  digitalWrite(LED3, LOW);
  delay(500);
}
```

---

### 예시 2: 생일 축하 (간단한 리듬)

```c
const int LED1 = 11;
const int LED2 = 12;
const int LED3 = 13;

void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
}

// "생일 축하합니다" 리듬 표현
void loop() {
  // 생 (짧게)
  digitalWrite(LED1, HIGH);
  delay(300);
  digitalWrite(LED1, LOW);
  delay(100);
  
  // 일 (짧게)
  digitalWrite(LED2, HIGH);
  delay(300);
  digitalWrite(LED2, LOW);
  delay(100);
  
  // 축 (길게)
  digitalWrite(LED3, HIGH);
  delay(600);
  digitalWrite(LED3, LOW);
  delay(100);
  
  // 하 (길게)
  digitalWrite(LED1, HIGH);
  delay(600);
  digitalWrite(LED1, LOW);
  delay(300);
  
  // 합 (짧게)
  digitalWrite(LED2, HIGH);
  delay(300);
  digitalWrite(LED2, LOW);
  delay(100);
  
  // 니 (짧게)
  digitalWrite(LED3, HIGH);
  delay(300);
  digitalWrite(LED3, LOW);
  delay(100);
  
  // 다 (아주 길게 + 모두 켜기)
  digitalWrite(LED1, HIGH);
  digitalWrite(LED2, HIGH);
  digitalWrite(LED3, HIGH);
  delay(1000);
  
  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
  digitalWrite(LED3, LOW);
  delay(2000);   // 다음 반복까지 2초 대기
}
```

---

## 회로 구성 팁

### 기본 브레드보드 연결

**Arduino와 브레드보드 기본 연결:**
```
Arduino GND → 브레드보드 GND 레일 (파란색 - 줄)
```

**중요한 점:**
- Arduino GND 핀은 3개 있음 (위쪽 2개, 아래쪽 1개)
- 어느 GND 핀을 사용해도 동일 (모두 내부 연결됨)
- 브레드보드 GND 레일 1개로 모든 부품의 GND 공유 가능

**브레드보드 색상 관례:**
- 빨간 레일(+): 전원 (5V, 3.3V 등)
- 파란/검정 레일(-): GND (접지)
- **반드시 지킬 것**: 관례를 지키지 않으면 회로 분석이 어렵고 실수 유발

### 교통 신호등 회로 연결

**기본 연결:**
```
Arduino GND → 브레드보드 GND 레일
        ↓
    (공통 GND, 수평 연결)
        ↓    ↓    ↓
    빨강LED  노랑LED  초록LED
        ↑    ↑    ↑
     220Ω  220Ω  220Ω
        ↑    ↑    ↑
    Pin11  Pin12  Pin13
```

**상세 연결:**
```
Arduino Pin 11 → 220Ω 저항 → 빨강 LED(+) → 빨강 LED(-) → 브레드보드 GND 레일
Arduino Pin 12 → 220Ω 저항 → 노랑 LED(+) → 노랑 LED(-) → 브레드보드 GND 레일
Arduino Pin 13 → 220Ω 저항 → 초록 LED(+) → 초록 LED(-) → 브레드보드 GND 레일
Arduino GND → 브레드보드 GND 레일
```

**주의사항**:
- 각 LED마다 별도의 저항 필요 (220Ω)
- 모든 LED의 음극(-)은 공통 GND 레일에 연결
- LED 극성 확인 (긴 다리 = +, 짧은 다리 = -)
- **절대 하지 말 것**: Pin → 직접 GND (단락 발생!)

---

## 디버깅 체크리스트

프로그램이 예상대로 작동하지 않을 때 확인해야 할 사항:

□ 회로 연결 확인
  - 모든 점퍼선이 제대로 연결되었는가?
  - LED 극성이 올바른가?
  - 저항이 연결되어 있는가?

□ 코드 확인
  - `pinMode()`에서 올바른 핀 번호를 사용했는가?
  - `digitalWrite()`에서 올바른 핀 번호를 사용했는가?
  - `delay()` 값이 적절한가?
  - 세미콜론(`;`)을 빠뜨리지 않았는가?

□ Tinkercad 시뮬레이션
  - "Start Simulation" 버튼을 눌렀는가?
  - 에러 메시지가 표시되는가?
  - 시뮬레이션 속도가 너무 느리지 않은가?

---

**참고**: 이 정답 파일은 교육용이며, 학생들이 스스로 문제를 해결한 후 확인 용도로만 사용하세요.
