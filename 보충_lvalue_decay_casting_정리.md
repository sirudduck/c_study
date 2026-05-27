# 보충자료 — lvalue · modifiable lvalue · Decay · Casting 핵심 정리

> **언제 보면 좋은가:** 시험 전날 5분 훑기용 / 서술형·구두 질문 대비
> **다루는 4개 용어:** `lvalue`, `modifiable lvalue`, `Array Decay`, `Casting`
> **선수 학습:** [10주차_시험대비_포인터_보충강의.md](./10주차_시험대비_포인터_보충강의.md)

---

## 0. 왜 이 4개 용어를 알아야 하나?

이 4개를 명확히 구분하지 못하면 다음 질문에 답할 수 없다.

- "왜 `int a[5], b[5]; b = a;`는 컴파일 에러인가?"
- "왜 `sizeof(arr)`은 20인데 함수에 넘긴 뒤 `sizeof(a)`는 8인가?"
- "`int *p = arr;`과 `int *p = (int*)arr;`은 무엇이 다른가?"
- "왜 `&arr`과 `arr`은 주소 숫자는 같은데 타입이 다른가?"

시험에서 단순 코드 문제가 아닌 **개념 서술형**으로 출제될 가능성이 높은 부분이다.

---

## 1. 한 장으로 보는 4가지 핵심 용어

| 용어 | 한국어 | 한 줄 정의 |
|------|--------|-----------|
| **lvalue** | (좌변값) | 메모리에 자리(주소)가 있는 표현식 → `&` 적용 가능 |
| **modifiable lvalue** | 변경 가능한 lvalue | lvalue 중에서 **값을 바꿀 수 있는 것** → 대입의 좌변 가능 |
| **Decay** | 붕괴·퇴화 | 배열·함수가 **자동으로 포인터로 변환**되는 묵시적 변환 |
| **Casting** | (명시적) 형변환 | 프로그래머가 **`(타입)`을 직접 써서** 강제하는 명시적 변환 |

---

## Part 1. lvalue와 rvalue

### 1.1 어원

- **lvalue** = **L**eft value → 옛날엔 "대입문 좌변에 올 수 있는 값"
- **rvalue** = **R**ight value → "우변에만 올 수 있는 값"

### 1.2 현대 C 표준의 정의 (§6.3.2.1)

> **lvalue** = 객체(메모리 공간)를 지칭하는 표현식.
> 즉, **&로 주소를 얻을 수 있는 것**.

### 1.3 구분 예시

```c
int x = 10;

x          // lvalue ✓  →  &x  가능
10         // rvalue   →  &10 컴파일 에러
x + 1      // rvalue   →  &(x+1) 컴파일 에러
*p         // lvalue ✓  →  포인터가 가리키는 객체 자체
arr[i]     // lvalue ✓  →  배열의 i번 원소 자체
```

> 🍳 **주방 비유**
> - lvalue = 식탁 위에 **이름표 달린 그릇** (자리가 있고 가리킬 수 있음)
> - rvalue = 그릇에서 떠 올린 **수증기** (한순간 존재하지만 자리는 없음)

### 1.4 검증 코드

```c
#include <stdio.h>
int main(void) {
    int x = 10;
    int *p = &x;            // OK
    // int *q = &42;        // ❌ 42는 rvalue
    // int *r = &(x + 1);   // ❌ x+1은 rvalue
    printf("x=%d, *p=%d\n", x, *p);
    return 0;
}
```

---

## Part 2. modifiable lvalue

### 2.1 정의

> **lvalue이면서 다음 4가지 조건을 모두 만족하는 것** (§6.3.2.1 p.1)
>
> ① 배열 타입이 아닐 것
> ② 불완전 타입(incomplete type)이 아닐 것
> ③ `const`가 붙지 않을 것
> ④ 구조체/공용체일 경우 `const` 멤버를 포함하지 않을 것

핵심: **"메모리에 자리가 있고(lvalue) + 그 값을 바꿀 수도 있는(modifiable) 표현식"**.

### 2.2 핵심 표

| 표현식 | lvalue? | modifiable? | 좌변에 올 수 있나? | 차단 이유 |
|--------|---------|-------------|---------------------|-----------|
| `int x; → x` | O | O | `x = 1;` ✓ | — |
| `const int y; → y` | O | **X** | `y = 1;` ❌ | const |
| `int arr[5]; → arr` | O | **X** | `arr = ...;` ❌ | **배열 타입** |
| `int arr[5]; → arr[0]` | O | O | `arr[0] = 1;` ✓ | (원소는 OK) |
| `"hello"` (문자열 리터럴) | O | **X** | 변경 ❌ | 배열 + 읽기전용 |
| `42` | X (rvalue) | — | ❌ | lvalue 자체가 아님 |
| `x + 1` | X (rvalue) | — | ❌ | lvalue 자체가 아님 |

### 2.3 ★ 시험 직결 — "왜 배열은 대입 못 하는가?"

```c
int a[5], b[5];
b = a;          // ❌ 컴파일 에러
```

**모범 답안:**
> 배열 표현식은 lvalue이지만 **modifiable lvalue가 아니다**(C 표준 §6.3.2.1).
> 대입 연산자의 좌변은 modifiable lvalue여야 하므로(§6.5.16),
> 좌변에 배열을 둘 수 없어 컴파일 에러가 발생한다.

### 2.4 ★ 유일한 우회 방법 — 구조체로 감싸기

```c
typedef struct { int data[5]; } Arr5;
Arr5 a = {{1, 2, 3, 4, 5}}, b;
b = a;          // ✓ OK — 구조체 변수는 modifiable lvalue
```

C에서 구조체 변수 자체는 modifiable lvalue이므로 대입 가능. 컴파일러가 멤버 단위로 복사 코드를 생성한다.

---

## Part 3. 변환(Conversion)의 전체 그림

### 3.1 분류 트리

```
변환(Conversion)
│
├── 묵시적(Implicit) — 컴파일러가 자동 수행
│   ├── Array-to-Pointer Decay     ← 배열→포인터 ★ 이게 우리가 다룬 'decay'
│   ├── Function-to-Pointer Decay  ← 함수→함수포인터
│   ├── 산술 변환 (Arithmetic)      ← int + double → double
│   ├── 정수 승격 (Integer Promotion) ← char → int
│   └── lvalue → rvalue 변환
│
└── 명시적(Explicit) = Casting — 프로그래머가 직접
    ├── C 스타일:  (타입)값
    └── (C++ 추가: static_cast, reinterpret_cast 등)
```

### 3.2 핵심 포인트

- **Decay**는 **묵시적 변환의 한 특수 케이스** (배열/함수에만 적용)
- **Casting**은 **명시적 변환의 전체** (거의 모든 타입 간 가능)
- 둘 다 "변환"이지만 **누가 시작하느냐**가 다르다

---

## Part 4. Decay (배열의 자동 포인터 변환)

### 4.1 정의 (§6.3.2.1 paragraph 3)

> 배열 표현식이 사용되면, **대부분의 컨텍스트에서 자동으로 "첫 원소를 가리키는 포인터"로 변환**된다.

영어 단어 "decay"의 원의는 "**부패하다, 정보를 잃다**". 변환 과정에서 **크기 정보가 사라지기 때문에** 붙은 이름이다.

```
변환 전: int arr[5]              ← 타입에 "5개"라는 크기 정보 있음
        ↓ decay (정보 손실)
변환 후: int *                   ← 단순한 포인터, 크기 정보 사라짐
```

### 4.2 ★ Decay가 일어나지 **않는** 3가지 예외 컨텍스트

이게 시험에서 가장 자주 노리는 부분.

| # | 컨텍스트 | 예시 | 그때의 타입 |
|---|----------|------|-------------|
| ① | **`sizeof` 의 피연산자** | `sizeof(arr)` | 배열 그대로 → 전체 바이트 반환 |
| ② | **`&` 의 피연산자** | `&arr` | `int (*)[5]` 배열 전체의 주소 |
| ③ | **문자열 리터럴이 배열 초기화** | `char s[] = "abc";` | 배열 그대로 복사 |

### 4.3 ★ 결정적 증거 — sizeof 차이

```c
#include <stdio.h>

void print_size(int a[]) {                       // 매개변수에서 자동 decay
    printf("함수 안 sizeof(a) = %lu\n", sizeof(a));    // 8 ← 포인터 크기
}

int main(void) {
    int arr[5] = {1, 2, 3, 4, 5};
    int *p = arr;                                // 여기서도 decay

    printf("sizeof(arr) = %lu\n", sizeof(arr));   // 20 ← 배열 (decay 안 됨)
    printf("sizeof(p)   = %lu\n", sizeof(p));     // 8  ← 포인터
    print_size(arr);                              // 8 (decay됨)
    return 0;
}
```

**실행 결과:**
```
sizeof(arr) = 20
sizeof(p)   = 8
함수 안 sizeof(a) = 8
```

> ⚠️ gcc가 친절하게 다음 경고도 띄움:
> `warning: 'sizeof' on array function parameter 'a' will return size of 'int *'`
> → 컴파일러 본인이 "이거 decay된 거야"라고 알려주는 셈.

### 4.4 ★ "왜 `arr`과 `&arr`의 숫자는 같은데 타입은 다른가?"

```c
int arr[5];
printf("%p\n", (void*)arr);    // 첫 원소 주소
printf("%p\n", (void*)&arr);   // 배열 전체의 주소 (숫자는 같음)
```

- `arr` 단독 → decay → `int *` 타입 (첫 원소 주소)
- `&arr` → decay 안 됨(예외 ②) → `int (*)[5]` 타입 (배열 5개 통째 주소)
- 숫자(주소값)는 같지만 **타입이 다르다**
- 그래서 `arr + 1`은 4바이트 이동, `&arr + 1`은 **20바이트 이동**

```c
int arr[5];
printf("arr + 1  - arr  = %ld\n", (char*)(arr + 1)  - (char*)arr);   // 4
printf("&arr + 1 - &arr = %ld\n", (char*)(&arr + 1) - (char*)&arr);  // 20
```

> 🍳 **주방 비유**
> - `arr` = "**한 칸의 문 손잡이** 위치" (가까운 위치로 잡힘)
> - `&arr` = "**도시락 통 전체**의 위치" (같은 좌표지만 의미가 통째)
> - +1을 했을 때 한 명은 다음 칸으로, 한 명은 다음 도시락으로 점프

---

## Part 5. Casting (명시적 형변환)

### 5.1 정의

> 프로그래머가 `(타입)값` 문법으로 **의도적으로** 타입을 바꾸는 것.

```c
double d = 3.14;
int    n = (int)d;        // 명시적 cast → n = 3

int   *p = (int*)0x1000;  // 정수를 포인터로 강제 변환
char  *q = (char*)p;      // int* → char* 강제 변환
```

### 5.2 ★ "묵시적 변환"과 다른 점

- **묵시적 변환:** 컴파일러가 알아서 (`int n = 3.14;` → 경고는 가능)
- **명시적 cast:** 프로그래머가 명시적으로 (`int n = (int)3.14;` → 경고 없음)

명시적 cast는 컴파일러에게 **"내가 알고 한 일이다, 경고 끄세요"**라고 통보하는 효과도 있다.

### 5.3 ★ Cast의 위험성

```c
double d = 3.14;
int *p = (int*)&d;       // 컴파일은 통과
printf("%d\n", *p);       // 의미 없는 비트 패턴이 int로 해석됨
```

- C 컴파일러는 cast를 거의 무조건 허용 → 프로그래머가 **결과를 보장**해야 함
- 부주의한 cast는 메모리 정렬(alignment) 위반, undefined behavior로 직결

---

## Part 6. Decay vs Casting — 결정적 차이 5가지

| 항목 | Decay | Casting |
|------|-------|---------|
| **발동 주체** | 컴파일러 자동 | 프로그래머 직접 |
| **표기 방식** | 코드에 안 보임 | `(타입)값` 명시 |
| **표준 분류** | 묵시적 변환 | 명시적 변환 |
| **적용 범위** | 배열→포인터, 함수→함수포인터 (좁음) | 거의 모든 타입 (넓음) |
| **안전성** | 표준이 보장 (안전) | 프로그래머 책임 (위험 가능) |

### 동일해 보이지만 의미가 다른 두 줄

```c
int arr[5];
int *p1 = arr;          // ① decay (묵시적, 컴파일러가 했음)
int *p2 = (int*)arr;    // ② cast  (명시적, 내가 했음)
```

- 두 줄의 결과(`p1`, `p2` 값)는 **완전히 동일**
- 그러나 ②는 **불필요한 cast** — 정적 분석 도구(예: clang-tidy)가 경고할 수 있음
- 실무에서는 ①을 쓴다. ②는 "고의로 한 일임을 강조"하는 경우에만.

> 🍳 **주방 비유 — 에스컬레이터 vs 계단**
> - **Decay** = 에스컬레이터에 발 디디면 자동으로 올라감 (의식 안 해도 일어남)
> - **Casting** = 계단을 한 발씩 직접 디뎌 올라감 (내가 의지를 가지고 함)

---

## Part 7. 실습 — 직접 컴파일해서 확인

다음 코드를 그대로 컴파일(`gcc concept_test.c -o concept_test`)하고 실행해 결과를 손으로 옮겨 적어보자.

```c
#include <stdio.h>

void show_size(int a[]) {
    printf("함수 안: sizeof(a) = %lu\n", sizeof(a));
}

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;
    const int c = 100;

    // (1) lvalue / modifiable lvalue 테스트
    printf("--- Part 1: lvalue ---\n");
    int *pa = &arr[0];               // arr[0]은 modifiable lvalue
    int *pc = (int*)&c;              // c는 lvalue지만 const → modifiable 아님
    printf("arr[0] 주소: %p\n", (void*)pa);
    printf("c 의 주소  : %p\n", (void*)pc);

    // (2) decay 증거
    printf("\n--- Part 2: decay ---\n");
    printf("sizeof(arr) = %lu (decay 안됨, 배열 전체)\n", sizeof(arr));
    printf("sizeof(p)   = %lu (포인터)\n", sizeof(p));
    show_size(arr);                  // 매개변수는 decay됨

    // (3) &arr vs arr 타입 차이
    printf("\n--- Part 3: &arr vs arr ---\n");
    printf("arr        주소: %p\n", (void*)arr);
    printf("&arr       주소: %p (숫자는 같음)\n", (void*)&arr);
    printf("(arr + 1)  주소: %p (4바이트 차이)\n",  (void*)(arr + 1));
    printf("(&arr + 1) 주소: %p (20바이트 차이!)\n", (void*)(&arr + 1));

    // (4) decay vs casting
    printf("\n--- Part 4: decay vs cast ---\n");
    int *p1 = arr;                   // 묵시적 decay
    int *p2 = (int*)arr;             // 명시적 cast (결과는 같음)
    printf("p1[0]=%d, p2[0]=%d\n", p1[0], p2[0]);

    return 0;
}
```

**예상 출력 (64-bit Linux/gcc 기준):**
```
--- Part 1: lvalue ---
arr[0] 주소: 0x7ffe...
c 의 주소  : 0x7ffe...

--- Part 2: decay ---
sizeof(arr) = 20 (decay 안됨, 배열 전체)
sizeof(p)   = 8 (포인터)
함수 안: sizeof(a) = 8

--- Part 3: &arr vs arr ---
arr        주소: 0x7ffe...
&arr       주소: 0x7ffe... (숫자는 같음)
(arr + 1)  주소: 0x7ffe... (4바이트 차이)
(&arr + 1) 주소: 0x7ffe... (20바이트 차이!)

--- Part 4: decay vs cast ---
p1[0]=10, p2[0]=10
```

---

## Part 8. 시험 단골 함정 Top 5

### 함정 1 — 배열 통째 대입
```c
int a[5], b[5];
b = a;       // ❌ 배열은 modifiable lvalue 아님
```

### 함정 2 — 함수 안에서 sizeof
```c
void f(int a[]) {
    sizeof(a);   // ❌ 8 (decay됨). 원래 배열 크기 못 가져옴
}
```
→ 함수에 배열을 넘기면 크기도 따로 넘겨야 한다 (`f(int a[], int n)` 패턴).

### 함정 3 — `arr` vs `&arr` 산술
```c
int arr[5];
arr + 1   // 4바이트 이동
&arr + 1  // 20바이트 이동
```

### 함정 4 — const 변수 좌변
```c
const int c = 10;
c = 20;      // ❌ modifiable lvalue 아님
```

### 함정 5 — 문자열 리터럴 변경
```c
char *s = "hello";
s[0] = 'H';  // ❌ 문자열 리터럴은 modifiable 아님 → 런타임 크래시
char s[] = "hello";   // ✓ 이건 modifiable (배열 초기화)
s[0] = 'H';
```

---

## Part 9. 시험 1분 전 — 핵심 5개 문장

> **① lvalue = 메모리에 자리가 있는 표현식.** &로 주소를 얻을 수 있다.
>
> **② modifiable lvalue = lvalue 중 값을 바꿀 수 있는 것.** const도 배열도 modifiable 아님.
>
> **③ Decay = 배열·함수가 자동으로 포인터로 변환되는 묵시적 변환.** 크기 정보를 잃는다.
>
> **④ Casting = 프로그래머가 `(타입)`을 명시해 강제하는 명시적 변환.** 결과 책임은 프로그래머.
>
> **⑤ Decay는 sizeof, &, 문자열 리터럴 초기화 3가지 컨텍스트에서는 일어나지 않는다.**

---

## Part 10. 서술형·구두 질문 모범 답안

### Q1. "`int a[5], b[5]; b = a;`는 왜 컴파일 에러인가?"
> A. 배열 표현식은 lvalue이지만 **modifiable lvalue가 아니다**(C 표준 §6.3.2.1).
> 대입 연산자의 좌변은 modifiable lvalue여야 하므로(§6.5.16),
> 좌변에 배열을 둘 수 없다.

### Q2. "배열을 함수로 전달할 때 일어나는 현상을 설명하고, 그로 인한 부작용을 기술하시오."
> A. 배열은 함수 매개변수로 전달될 때 **첫 원소를 가리키는 포인터로 자동 변환(Array-to-Pointer Decay)**된다.
> 이때 배열의 **크기 정보가 사라지므로**, 함수 안에서 `sizeof`는 배열 크기가 아닌
> **포인터의 크기(8바이트, 64-bit)**를 반환한다.
> 따라서 함수는 배열의 크기를 별도 인자로 받아야 한다.

### Q3. "`int *p = arr;`과 `int *p = (int*)arr;`의 차이를 설명하시오."
> A. 결과는 동일하다(둘 다 `arr`의 첫 원소를 가리키게 됨).
> 전자는 **묵시적 decay**, 후자는 **명시적 casting**이다.
> 후자는 불필요한 cast이며, 가독성을 해치고 정적 분석 도구의 경고 대상이 될 수 있다.
> 실무에서는 전자(decay)를 사용한다.

### Q4. "`arr`과 `&arr`은 같은 주소값을 출력하는데, 두 표현은 무엇이 다른가?"
> A. 주소값(숫자)은 같지만 **타입이 다르다**.
> `arr`은 decay되어 `int *` 타입(첫 원소 주소),
> `&arr`은 decay되지 않고 `int (*)[5]` 타입(배열 5개 통째 주소).
> 따라서 `arr + 1`은 `sizeof(int)`(4바이트)만큼 이동하지만,
> `&arr + 1`은 `sizeof(int[5])`(20바이트)만큼 이동한다.

---

## 참고 — 표준 조항 출처

- **ISO/IEC 9899:2018 §6.3.2.1** — Lvalues, arrays, and function designators (lvalue·modifiable lvalue 정의, array decay)
- **ISO/IEC 9899:2018 §6.5.16** — Assignment operators (좌변은 modifiable lvalue여야 함)
- **ISO/IEC 9899:2018 §6.3** — Conversions (변환 전체 분류)
- **ISO/IEC 9899:2018 §6.5.4** — Cast operators (명시적 형변환)

> ⚠️ 이 자료는 ISO C 표준 본문을 학생 학습용으로 풀어 쓴 것이다. 정확한 원문 표현은 위 조항을 직접 확인할 것.

---

**관련 자료:**
- [Array_Decay_가이드.md (기존 저장소)](https://github.com/sirudduck/c_study/blob/main/Array_Decay_%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [09주차_시험대비_배열_보충강의.md](./09주차_시험대비_배열_보충강의.md)
- [10주차_시험대비_포인터_보충강의.md](./10주차_시험대비_포인터_보충강의.md)
- [모의기말고사_해설지.md](./모의기말고사_해설지.md)
