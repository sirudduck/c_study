# Arduino Week 1 Session 1: Arduino 시작하기 - LED 제어

## 학습 목표
- Arduino 하드웨어와 프로그래밍 환경 이해
- Tinkercad Circuits 시뮬레이터 사용법 습득
- 디지털 출력 제어 (`digitalWrite`) 학습
- C 함수 구조 복습 (`setup()`, `loop()`)

---

## 1. 동기부여: 문제 제시 (10분)

### 문제: LED를 1초 간격으로 깜빡이게 만들어라

**학생들에게 먼저 질문:**
- 프로그램이 LED를 어떻게 제어할 수 있을까?
- "켜기"와 "끄기"를 코드로 어떻게 표현할까?
- 1초 간격은 어떻게 구현할까?

---

## 2. Arduino 소개 (15분)

### 2.1 Arduino란?

**정의**:
- 마이크로컨트롤러 보드 (작은 컴퓨터 + 하드웨어 제어 기능)
- 오픈소스 전자 플랫폼
- 프로그래밍 언어: C/C++ 기반

**용도**:
- LED, 센서, 모터 등 물리적 장치 제어
- 인터랙티브 프로젝트 제작
- 프로토타이핑

**출처**: [Arduino Official Site - What is Arduino?](https://www.arduino.cc/en/Guide/Introduction)

### 2.2 Arduino 프로그램 구조

```c
void setup() {
  // 초기 설정: 프로그램 시작 시 한 번만 실행
  // 핀 모드 설정, 초기값 설정 등
}

void loop() {
  // 반복 실행: setup() 이후 계속 반복
  // 실제 작업 수행
}
```

**수학적 비유:**
- `setup()`: 함수의 정의역과 초기 조건 설정
- `loop()`: 무한 수열처럼 같은 패턴을 계속 반복
  - 수열: a₁, a₂, a₃, ... (무한히 반복)
  - loop: 코드 블록 무한 반복

**출처**: [Arduino Reference - setup() and loop()](https://www.arduino.cc/reference/en/language/structure/sketch/setup/)

---

## 3. Tinkercad Circuits 환경 설정 (20분)

### 3.1 Tinkercad 접속 및 시작

**단계:**
1. 웹브라우저에서 https://www.tinkercad.com 접속
2. 계정 생성 또는 로그인
3. 대시보드에서 "Circuits" 클릭
4. "Create new Circuit" 클릭

**Tinkercad Circuits란?**
- 웹 기반 Arduino 시뮬레이터
- 실제 하드웨어 없이 회로 설계 및 시뮬레이션 가능
- 무료로 사용 가능

**출처**: [Tinkercad Circuits](https://www.tinkercad.com/circuits)

### 3.2 기본 부품 이해

**필요한 부품:**
1. **Arduino Uno R3**: 마이크로컨트롤러 보드
2. **Breadboard (브레드보드)**: 납땜 없이 회로를 구성하는 판
3. **LED (Light Emitting Diode)**: 발광 다이오드
4. **Resistor (저항)**: 220Ω (옴의 법칙 적용)
5. **Jumper Wires (점퍼선)**: 부품 간 연결

### 3.3 회로(Circuit)의 기본 원리

**회로란?**
- Circuit = 순환 경로
- 전류는 **반드시 출발점으로 돌아와야** 흐를 수 있음

```
완전한 회로 (Closed Circuit):
전원(+) → 부품 → GND → 전원(-)
   ↑_________________________↓
      순환 완성 → 전류 흐름 ✓

끊어진 회로 (Open Circuit):
전원(+) → 부품 → (끊김)
      전류 흐름 없음 ✗
```

**물 파이프 비유:**
- 펌프(전원) → 파이프 → 수차(LED) → 다시 펌프로 돌아옴
- 파이프가 끊어지면 물이 흐르지 않음

**출처**: [Electric Circuits - All About Circuits](https://www.allaboutcircuits.com/textbook/direct-current/chpt-1/electric-circuits/)

### 3.4 Arduino 핀 구조

**Arduino Uno R3 핀 종류:**
- **디지털 핀 (0~13)**: 디지털 입출력
- **아날로그 핀 (A0~A5)**: 아날로그 입력
- **전원 핀**: 5V, 3.3V
- **GND 핀**: 3개 (모두 내부적으로 연결됨)

**디지털 핀의 두 가지 모드:**
- **OUTPUT**: 5V 또는 0V를 출력 (LED, 모터 제어)
- **INPUT**: 외부 신호를 읽음 (버튼, 센서)

**출처**: [Arduino Uno Pinout](https://docs.arduino.cc/hardware/uno-rev3)

### 3.5 브레드보드 구조

**브레드보드 구조:**
```
왼쪽                                    오른쪽
+ ─ ─ ─ ─ ─  (전원 레일, 수평 연결)    + ─ ─ ─ ─ ─
- ─ ─ ─ ─ ─  (GND 레일, 수평 연결)     - ─ ─ ─ ─ ─

a │││││  (중앙부, 세로(수직)로 연결)     │││││ j
b │││││                                 │││││ i
c │││││                                 │││││ h
d │││││                                 │││││ g
e │││││                                 │││││ f
─────────────  (중앙 분리 홈)  ─────────────
f │││││                                 │││││ e
g │││││                                 │││││ d
h │││││                                 │││││ c
i │││││                                 │││││ b
j │││││                                 │││││ a

+ ─ ─ ─ ─ ─  (전원 레일)                + ─ ─ ─ ─ ─
- ─ ─ ─ ─ ─  (GND 레일)                 - ─ ─ ─ ─ ─
```

**중요한 점:**
- **좌우 레일은 서로 분리됨**: 왼쪽 + 레일과 오른쪽 + 레일은 연결되지 않음
- **양쪽을 사용하려면**: 점퍼선으로 좌우 레일을 연결해야 함
- **색상 관례**: 빨간색(+) = 전원, 파란색/검정색(-) = GND

**출처**: [Breadboard Tutorial - SparkFun](https://learn.sparkfun.com/tutorials/how-to-use-a-breadboard/all)

---

## 4. 첫 번째 회로 구성 (25분)

### 4.1 회로도

```
Arduino Pin 13 ──→ 저항(220Ω) ──→ LED(+) ──→ LED(-) ──→ Arduino GND
                                    긴 다리     짧은 다리
```

**전류 흐름:**
```
USB 전원 → (Arduino 내부) → Pin 13 (5V) → 저항(220Ω) → LED (빛 발산) 
→ GND → (Arduino 내부) → USB로 돌아옴 (순환 완성)
```

**⚠️ 절대 하지 말아야 할 것:**
```
Pin 13 → 직접 GND 연결 (✗ 단락 발생!)
```
- **단락(Short Circuit)**: 저항 없이 전원이 직접 GND로 연결
- **결과**: 과전류 발생 → Arduino 손상 가능
- **반드시**: Pin → 저항 → LED → GND 순서로 연결

### 4.2 저항이 필요한 이유

**옴의 법칙**: V = IR
- V: 전압 (Voltage)
- I: 전류 (Current)
- R: 저항 (Resistance)

**계산:**
- Arduino 출력 전압: 5V
- LED 정격 전압: 약 2V
- LED 정격 전류: 약 20mA

```
필요한 저항 = (5V - 2V) / 0.02A = 150Ω
안전을 위해 220Ω 사용
```

**출처**: [Ohm's Law - SparkFun](https://learn.sparkfun.com/tutorials/voltage-current-resistance-and-ohms-law/all)

### 4.3 회로 연결 단계

**단계 1: 부품 배치**
1. Tinkercad에서 "Arduino Uno R3" 선택
2. "Breadboard Small" 또는 "Breadboard" 선택 (일반 크기 권장)
3. "LED" 선택 (빨간색)
4. "Resistor" 선택 (220Ω)

**단계 2: Arduino와 브레드보드 기본 연결**
- Arduino GND 핀 → 브레드보드 GND 레일 (파란색 - 줄)
- **Arduino GND 핀은 3개**: 위쪽 2개, 아래쪽 1개 (어느 것을 사용해도 동일)

**왜 브레드보드 GND 레일을 사용하는가?**
```
Arduino GND 핀 (1개만 사용)
        ↓
브레드보드 GND 레일 (━━━━━ 수평 연결)
   ↓    ↓    ↓    ↓
  LED1  LED2  LED3  ... (여러 부품 공유 가능)
```
- **효율성**: Arduino GND 핀 1개로 여러 부품에 GND 제공
- **정리**: 배선이 깔끔해짐
- **확장성**: 나중에 부품 추가가 쉬움

**단계 3: LED 극성 확인**
- **긴 다리(+)**: Anode, 양극 → 신호선 쪽
- **짧은 다리(-)**: Cathode, 음극 → GND 쪽
- **⚠️ 중요**: 극성을 반대로 연결하면 작동하지 않음 (손상되지는 않음)

**단계 4: 회로 연결**
1. 220Ω 저항을 브레드보드 중앙부에 꽂기
   - 한쪽 다리: 행 A
   - 다른쪽 다리: 행 E (중앙 홈 건너편)
2. LED 연결
   - 긴 다리(+): 저항과 같은 행(E) 또는 인접 행
   - 짧은 다리(-): 다른 행(F) - GND 레일로 연결할 위치
3. 점퍼선 연결
   - 빨강 점퍼선: Arduino Pin 13 → 저항(행 A)
   - 검정 점퍼선: LED 짧은다리(행 F) → 브레드보드 GND 레일

### 4.4 연결 확인 체크리스트

□ Arduino Uno R3 배치 완료  
□ Breadboard 배치 완료  
□ 220Ω 저항 연결 (색 코드: 빨강-빨강-갈색)  
□ LED 극성 확인 (긴 다리 = +, 짧은 다리 = -)  
□ Pin 13과 저항이 점퍼선으로 연결  
□ LED 짧은 다리와 GND가 점퍼선으로 연결  
□ 모든 연결 재확인  

---

## 5. 첫 번째 프로그램: Blink (30분)

### 5.1 코드 작성

```c
void setup() {
  pinMode(13, OUTPUT);  // Pin 13을 출력 모드로 설정
}

void loop() {
  digitalWrite(13, HIGH);   // LED 켜기 (5V 출력)
  delay(1000);              // 1000ms = 1초 대기
  digitalWrite(13, LOW);    // LED 끄기 (0V 출력)
  delay(1000);              // 1초 대기
}
```

### 5.2 함수 상세 설명

#### `pinMode(pin, mode)`

**정의**: 디지털 핀의 동작 모드를 설정

**매개변수**:
- `pin` (int): 핀 번호 (0~13)
- `mode`: 
  - `OUTPUT`: 핀이 전압을 출력 (LED, 모터 제어)
  - `INPUT`: 핀이 전압을 읽기 (버튼, 센서)

**예시**:
```c
pinMode(13, OUTPUT);  // Pin 13을 출력으로 설정
pinMode(7, INPUT);    // Pin 7을 입력으로 설정
```

**왜 필요한가?**
- Arduino 핀은 입력/출력 **양방향** 사용 가능
- 사용 목적에 따라 **명시적으로 설정** 필요
- OUTPUT 설정 없이 `digitalWrite()` 사용 시 오작동 가능

**핀 모드별 용도:**
| 모드 | 용도 | 예시 |
|------|------|------|
| OUTPUT | 신호 출력 | LED 켜기, 모터 제어 |
| INPUT | 신호 읽기 | 버튼 누름 감지, 센서 값 읽기 |

**출처**: [Arduino Reference - pinMode()](https://www.arduino.cc/reference/en/language/functions/digital-io/pinmode/)

#### `digitalWrite(pin, value)`

**정의**: 디지털 핀에 HIGH 또는 LOW 값을 출력

**매개변수**:
- `pin` (int): 핀 번호
- `value`:
  - `HIGH`: 5V 출력 → LED 켜짐
  - `LOW`: 0V 출력 → LED 꺼짐

**예시**:
```c
digitalWrite(13, HIGH);  // Pin 13에 5V 출력
digitalWrite(13, LOW);   // Pin 13에 0V 출력
```

**내부 동작**:
- `HIGH`: 핀을 전원(5V)에 연결
- `LOW`: 핀을 GND(0V)에 연결

**출처**: [Arduino Reference - digitalWrite()](https://www.arduino.cc/reference/en/language/functions/digital-io/digitalwrite/)

#### `delay(ms)`

**정의**: 프로그램을 지정된 시간(밀리초) 동안 일시 정지

**매개변수**:
- `ms` (unsigned long): 밀리초 (milliseconds)
  - 1000ms = 1초
  - 500ms = 0.5초

**예시**:
```c
delay(1000);  // 1초 대기
delay(500);   // 0.5초 대기
delay(2000);  // 2초 대기
```

**주의사항**:
- `delay()` 동안 프로그램은 완전히 정지
- 센서 읽기, 버튼 입력 등 모든 작업이 중단됨

**출처**: [Arduino Reference - delay()](https://www.arduino.cc/reference/en/language/functions/time/delay/)

### 5.3 프로그램 실행

**Tinkercad에서 실행:**
1. 코드를 "Code" 창에 입력
2. "Start Simulation" 버튼 클릭
3. LED가 1초 간격으로 깜빡이는지 확인

**디버깅 팁:**
- LED가 켜지지 않으면: 회로 연결 확인
- 깜빡임이 보이지 않으면: delay 시간 확인
- 에러 메시지: 코드 문법 확인

---

## 6. 변형 실습 (30분)

### 실습 1: 깜빡임 속도 변경

**문제**: Blink 프로그램을 수정하여 다양한 속도로 깜빡이게 만들어라

**변형 1**: 0.5초 간격  
**변형 2**: 2초 간격  
**도전 과제**: 0.1초 간격으로 빠르게 깜빡이기

**힌트**: `delay()` 함수의 값을 변경하면 됩니다. (1000ms = 1초)

---

### 실습 2: SOS 신호 만들기

**목표**: 모스 부호로 SOS 신호 구현

**모스 부호**:
- **S**: 짧게 3번 (· · ·)
- **O**: 길게 3번 (― ― ―)
- **S**: 짧게 3번 (· · ·)

**힌트**: 짧은 신호는 200ms, 긴 신호는 600ms로 설정해보세요.

---

### 실습 3: 여러 핀 사용하기

**문제**: 2개의 LED를 번갈아가며 깜빡이게 하라

**회로 수정**:
- LED 1: Pin 12
- LED 2: Pin 13

**힌트**: 두 개의 핀을 `OUTPUT`으로 설정하고, 하나가 `HIGH`일 때 다른 하나는 `LOW`로 설정하세요.

---

## 7. C 문법 복습 (10분)

### 7.1 함수 (Function)

**수학에서의 함수**:
```
f(x) = 2x + 1
입력 x → 처리(2배 후 1 더하기) → 출력 f(x)
```

**C에서의 함수**:
```c
void setup() {
  // void: 반환값이 없음
  // setup: 함수 이름
  // (): 매개변수가 없음
}

int add(int a, int b) {
  // int: 반환 타입
  // add: 함수 이름
  // (int a, int b): 매개변수
  return a + b;
}
```

**비교**:
| 수학 | C 프로그래밍 |
|------|--------------|
| f(x) | 함수 이름 |
| x | 매개변수 (parameter) |
| f(x)의 결과 | 반환값 (return) |

### 7.2 변수 타입 복습

**Arduino에서 자주 사용하는 타입**:
```c
int ledPin = 13;           // 정수 (-32768 ~ 32767)
unsigned int count = 0;    // 양의 정수 (0 ~ 65535)
long bigNumber = 1000000;  // 큰 정수
float voltage = 3.3;       // 실수
```

**출처**: [Arduino Reference - Data Types](https://www.arduino.cc/reference/en/#variables)

---

## 8. 과제 안내 (5분)

### 과제 1: 교통 신호등 만들기 ⭐⭐

**요구사항**:
- 빨강 LED: 5초 켜짐
- 노랑 LED: 2초 켜짐
- 초록 LED: 5초 켜짐
- 위 패턴을 반복

**사용할 핀**:
- 빨강: Pin 11
- 노랑: Pin 12
- 초록: Pin 13

**힌트**: 세 개의 핀을 모두 `OUTPUT`으로 설정하고, 한 번에 하나의 LED만 `HIGH`로 설정하세요.

**제출 방법**:
1. Tinkercad에서 회로 구성 및 코드 작성
2. Tinkercad 링크 공유 (Share → Copy Link)
3. 코드에 주석으로 각 부분 설명 추가

---

### 과제 2 (선택): 자유 LED 패턴 ⭐⭐⭐

**요구사항**:
- 최소 3개의 LED 사용
- 의미 있는 패턴 구현
- 예시: 생일 축하 멜로디 표현, 경고 알람, 크리스마스 트리 등

**평가 기준**:
- 창의성
- 코드의 명확성 (주석 포함)
- 패턴의 완성도

---

## 9. 정리 및 다음 시간 예고 (5분)

### 오늘 배운 내용
✅ Arduino 프로그램 구조 (`setup()`, `loop()`)  
✅ Tinkercad Circuits 사용법  
✅ 디지털 출력 (`digitalWrite`)  
✅ 시간 지연 (`delay`)  
✅ 회로 구성 기초 (LED + 저항)  

### 다음 시간 주제
- 버튼 입력 받기 (`digitalRead`)
- 조건문 활용 (if-else)
- 상호작용하는 LED 제어

---

## 참고 자료

### 공식 문서
- [Arduino Official Reference](https://www.arduino.cc/reference/en/)
- [Arduino Getting Started Guide](https://www.arduino.cc/en/Guide)
- [Tinkercad Circuits Tutorials](https://www.tinkercad.com/learn)

### 추가 학습 자료
- [Arduino Project Hub](https://create.arduino.cc/projecthub)
- [SparkFun Arduino Tutorials](https://learn.sparkfun.com/tutorials/tags/arduino)

---

## 문제 해결 (Troubleshooting)

### 일반적인 문제

**문제 1**: LED가 켜지지 않음
- **확인 사항**:
  - 회로 연결 확인 (Pin 13 → 저항 → LED(+) → LED(-) → GND)
  - LED 극성 확인 (긴 다리가 +)
  - 코드에서 `pinMode(13, OUTPUT)` 설정 확인
  - Arduino GND와 브레드보드 GND 레일 연결 확인

**문제 2**: 코드 업로드 실패
- **확인 사항**:
  - 문법 오류 확인 (세미콜론 `;` 누락?)
  - 중괄호 `{}` 짝 맞는지 확인

**문제 3**: Tinkercad 시뮬레이션이 느림
- **해결책**:
  - 브라우저 새로고침
  - 불필요한 탭 닫기
  - Chrome 또는 Firefox 사용 권장

**문제 4**: 회로가 복잡해지면 헷갈림
- **해결책**:
  - **브레드보드 색상 관례를 항상 지키세요**
  - 빨간 레일(+): 전원만 연결
  - 파란/검정 레일(-): GND만 연결
  - **⚠️ 주의**: 레일 색상은 단순 표시일 뿐이지만, 관례를 지키지 않으면:
    - 회로 분석이 어려워짐
    - 실수로 전원과 GND를 혼동할 위험
    - 단락(short circuit) 발생 가능

**문제 5**: GND 핀이 부족한 것 같음
- **해결책**:
  - Arduino GND 핀은 3개 (모두 동일하게 연결됨)
  - 브레드보드 GND 레일 1개로 여러 부품 연결 가능
  - GND 핀 1개 → 브레드보드 레일 → 모든 부품 공유

---

**수업 준비물 체크리스트**
□ Tinkercad 계정 생성  
□ 인터넷 연결 확인  
□ 노트북/PC 준비  
□ 실습 시간 확보 (최소 1시간)  

**과제 제출 마감**: 다음 수업 시작 전까지
