# Array Decay (배열 붕괴) 완전 가이드

## 학습 목표
- Array Decay의 정확한 의미 이해
- Decay가 발생하는 상황과 발생하지 않는 상황 구분
- 정보 손실의 의미와 그 영향 파악

---

## 1. Decay란 무엇인가?

### 1.1 사전적 의미
- **decay**: 부패하다, 쇠퇴하다, (품질이) 떨어지다

### 1.2 C 언어에서의 의미

**Array Decay (배열 붕괴/배열 퇴화)**
- 배열 타입이 **포인터 타입으로 암묵적으로 변환**되는 현상
- 배열의 "완전한 타입 정보"가 "단순한 포인터"로 **퇴화**되는 것

```c
int arr[5];  // 타입: int [5] (크기 정보 포함)
              ↓ decay
int *ptr;    // 타입: int * (크기 정보 상실)
```

---

## 2. 왜 "Decay"라는 용어를 사용하나?

배열이 포인터로 변환되면 **중요한 정보가 손실**되기 때문입니다.

### 2.1 정보 손실 비교

```
변환 전: int numbers[5]
┌─────────────────────────────┐
│ 알고 있는 정보:              │
│ • 타입: int                  │
│ • 크기: 5개                  │
│ • 전체 크기: 20바이트        │
│ • 연속된 메모리 블록         │
│ • 배열의 시작 주소           │
└─────────────────────────────┘

         ↓ decay

변환 후: int *ptr
┌─────────────────────────────┐
│ 알고 있는 정보:              │
│ • 타입: int                  │
│ • 첫 번째 원소의 주소        │
│                              │
│ 손실된 정보:                 │
│ ✗ 크기: ??? (모름!)         │
│ ✗ 전체 크기: ??? (모름!)    │
│ ✗ 어디까지가 배열인지 모름   │
└─────────────────────────────┘
```

---

## 3. Decay 발생 예제

### 3.1 기본 예제

```c
#include <stdio.h>

void show_decay(int arr[]) {
    // arr는 이미 decay되어 포인터가 됨
    printf("함수 내부:\n");
    printf("  arr의 타입: int* (포인터)\n");
    printf("  sizeof(arr) = %zu\n", sizeof(arr));  // 8바이트 (포인터 크기)
    printf("  배열 크기? 알 수 없음!\n");
}

int main(void) {
    int numbers[5] = {10, 20, 30, 40, 50};
    
    printf("main 함수:\n");
    printf("  numbers의 타입: int[5] (배열)\n");
    printf("  sizeof(numbers) = %zu\n", sizeof(numbers));  // 20바이트
    printf("  배열 크기: 5개\n\n");
    
    // 함수로 전달 시 decay 발생
    show_decay(numbers);
    
    return 0;
}
```

**실행 결과:**
```
main 함수:
  numbers의 타입: int[5] (배열)
  sizeof(numbers) = 20
  배열 크기: 5개

함수 내부:
  arr의 타입: int* (포인터)
  sizeof(arr) = 8
  배열 크기? 알 수 없음!
```

### 3.2 상세 예제

```c
#include <stdio.h>

int main(void) {
    int numbers[5] = {10, 20, 30, 40, 50};
    
    printf("=== Decay 발생 예시 ===\n\n");
    
    // 1. 포인터 변수에 할당
    int *ptr = numbers;  // decay 발생
    printf("1. int *ptr = numbers;\n");
    printf("   → numbers가 포인터로 decay됨\n");
    printf("   → 크기 정보 상실\n\n");
    
    // 2. 포인터 산술
    printf("2. numbers + 1\n");
    printf("   → 포인터 연산을 위해 decay 발생\n");
    printf("   → 값: %d\n\n", *(numbers + 1));
    
    // 3. 배열 인덱스 접근
    printf("3. numbers[2]\n");
    printf("   → 내부적으로 *(numbers + 2)로 변환\n");
    printf("   → decay 발생\n");
    printf("   → 값: %d\n\n", numbers[2]);
    
    // 4. 함수 매개변수로 전달
    printf("4. 함수에 배열 전달\n");
    printf("   → 함수 매개변수에서 decay 발생\n");
    printf("   → int arr[]는 int *arr로 해석됨\n\n");
    
    printf("=== Decay가 일어나지 않는 경우 ===\n\n");
    
    // sizeof 연산자
    printf("1. sizeof(numbers)\n");
    printf("   → sizeof는 예외: decay 안됨\n");
    printf("   → 결과: %zu 바이트 (배열 전체 크기)\n\n", sizeof(numbers));
    
    // & 연산자
    printf("2. &numbers\n");
    printf("   → 주소 연산자도 예외: decay 안됨\n");
    printf("   → 배열 전체의 주소를 반환\n\n");
    
    // 문자열 리터럴
    printf("3. char str[] = \"hello\";\n");
    printf("   → 초기화 시에는 decay 안됨\n");
    printf("   → 배열에 문자열 복사\n");
    
    return 0;
}
```

**실행 결과:**
```
=== Decay 발생 예시 ===

1. int *ptr = numbers;
   → numbers가 포인터로 decay됨
   → 크기 정보 상실

2. numbers + 1
   → 포인터 연산을 위해 decay 발생
   → 값: 20

3. numbers[2]
   → 내부적으로 *(numbers + 2)로 변환
   → decay 발생
   → 값: 30

4. 함수에 배열 전달
   → 함수 매개변수에서 decay 발생
   → int arr[]는 int *arr로 해석됨

=== Decay가 일어나지 않는 경우 ===

1. sizeof(numbers)
   → sizeof는 예외: decay 안됨
   → 결과: 20 바이트 (배열 전체 크기)

2. &numbers
   → 주소 연산자도 예외: decay 안됨
   → 배열 전체의 주소를 반환

3. char str[] = "hello";
   → 초기화 시에는 decay 안됨
   → 배열에 문자열 복사
```

---

## 4. C 표준 근거

### 4.1 공식 용어

ISO C 표준에서는 "decay" 대신 **"conversion"(변환)**이라는 공식 용어를 사용합니다.

**ISO/IEC 9899:2018 Section 6.3.2.1 paragraph 3:**
> "Except when it is the operand of the sizeof operator, the _Alignof operator, or the unary & operator, or is a string literal used to initialize an array, an expression that has type 'array of type' **is converted to** an expression with type 'pointer to type'"

**번역:**
> "sizeof 연산자, _Alignof 연산자, 단항 & 연산자의 피연산자이거나, 배열을 초기화하는 문자열 리터럴인 경우를 제외하고, 'type의 배열' 타입을 가진 표현식은 'type에 대한 포인터' 타입을 가진 표현식으로 **변환된다**"

### 4.2 Decay가 발생하지 않는 예외 상황

1. **sizeof 연산자의 피연산자**
   ```c
   int arr[5];
   sizeof(arr);  // decay 안됨, 20 반환
   ```

2. **& 연산자의 피연산자**
   ```c
   int arr[5];
   &arr;  // decay 안됨, 배열 전체의 주소
   ```

3. **배열 초기화용 문자열 리터럴**
   ```c
   char str[] = "hello";  // decay 안됨
   ```

---

## 5. 실전 비교

### 5.1 타입 정보 비교

```c
#include <stdio.h>

void function_param(int arr[100]) {
    // arr의 실제 타입: int*
    // 100은 무시됨!
    printf("sizeof(arr) in function = %zu\n", sizeof(arr));
}

int main(void) {
    int local_array[100];
    
    // 지역 배열
    printf("=== 지역 배열 ===\n");
    printf("타입: int[100]\n");
    printf("sizeof(local_array) = %zu\n", sizeof(local_array));
    printf("크기 정보: 유지됨\n\n");
    
    // 함수 매개변수
    printf("=== 함수 매개변수 ===\n");
    printf("선언: int arr[100]\n");
    printf("실제 타입: int*\n");
    function_param(local_array);
    printf("크기 정보: 손실됨\n");
    
    return 0;
}
```

**실행 결과:**
```
=== 지역 배열 ===
타입: int[100]
sizeof(local_array) = 400
크기 정보: 유지됨

=== 함수 매개변수 ===
선언: int arr[100]
실제 타입: int*
sizeof(arr) in function = 8
크기 정보: 손실됨
```

### 5.2 메모리 레이아웃

```
지역 배열 선언: int numbers[5] = {10, 20, 30, 40, 50};

메모리:
┌────────┬────────┬────────┬────────┬────────┐
│   10   │   20   │   30   │   40   │   50   │
├────────┼────────┼────────┼────────┼────────┤
│0x1000  │0x1004  │0x1008  │0x100C  │0x1010  │
└────────┴────────┴────────┴────────┴────────┘
    ↑
    │
numbers (배열 이름) - 전체 정보 보유:
• 타입: int[5]
• 크기: 5개
• 메모리: 20바이트


함수 호출: func(numbers)
              ↓
         decay 발생!

함수 내부: int *arr

arr (포인터) - 제한된 정보만 보유:
    ↓
┌────────┐
│0x1000  │ (8바이트 포인터 변수)
└────────┘
    │
    └──→ [10][20][30][40][50]
         어디까지가 배열인지 모름!
```

---

## 6. 용어 정리

### 6.1 한국어 번역 옵션

| 영어 | 한국어 옵션 | 설명 | 추천도 |
|------|-------------|------|--------|
| Array Decay | 배열 붕괴 | 가장 직역에 가까움 | ★★★ |
| | 배열 퇴화 | 정보 손실 의미 강조 | ★★★ |
| | 배열-포인터 변환 | 정확하지만 길다 | ★★ |
| | 배열 디케이 | 영어 발음 그대로 | ★★ |

**추천:**
- 정식 문서: "배열이 포인터로 변환됨"
- 구두 설명: "배열 decay" 또는 "배열 붕괴"

### 6.2 관련 용어

| 용어 | 의미 |
|------|------|
| Implicit Conversion | 암묵적 변환 (자동 변환) |
| Type Decay | 타입 퇴화 (정보 손실을 동반한 타입 변환) |
| Pointer Decay | 포인터로의 붕괴 |

---

## 7. 핵심 요약

### 7.1 Decay의 본질

```
Decay = 배열 → 포인터 자동 변환 (정보 손실 발생)

손실되는 정보:
❌ 배열 크기 (몇 개의 원소?)
❌ 전체 메모리 크기 (몇 바이트?)
❌ "배열"이라는 타입 정보

남는 정보:
✅ 첫 번째 원소의 주소
✅ 원소의 타입 (int, char 등)
```

### 7.2 Decay 발생 여부 정리

| 상황 | Decay 발생? | 이유 |
|------|-------------|------|
| `int *p = arr` | ✅ 발생 | 포인터 할당 |
| `arr[i]` | ✅ 발생 | `*(arr+i)`로 변환 |
| `arr + 1` | ✅ 발생 | 포인터 산술 |
| `func(arr)` | ✅ 발생 | 함수 매개변수 전달 |
| `sizeof(arr)` | ❌ 발생 안함 | sizeof 예외 |
| `&arr` | ❌ 발생 안함 | 주소 연산자 예외 |
| `char s[] = "hi"` | ❌ 발생 안함 | 초기화 예외 |

### 7.3 실무적 의미

**왜 중요한가?**
1. 배열을 함수에 전달하면 크기 정보를 잃어버림
2. 따라서 크기를 별도의 매개변수로 전달해야 함
3. sizeof()로 함수 내에서 배열 크기를 알 수 없음
4. 포인터 산술 시 경계를 넘지 않도록 주의 필요

**안전한 코딩:**
```c
// ✅ 올바른 방법
void process(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        // arr[i] 처리
    }
}

int main(void) {
    int data[100];
    int size = sizeof(data) / sizeof(data[0]);  // main에서 계산
    process(data, size);  // 크기와 함께 전달
}
```

```c
// ❌ 잘못된 방법
void process(int arr[]) {
    int size = sizeof(arr) / sizeof(arr[0]);  // 잘못된 크기!
    // ...
}
```

---

## 8. 연습 문제

### 문제 1: Decay 판별
다음 중 decay가 발생하는 경우를 모두 고르세요:
```c
int arr[10];
int *p;

A. p = arr;
B. sizeof(arr);
C. &arr;
D. arr[5];
E. arr + 3;
```

### 문제 2: 코드 분석
다음 코드의 출력을 예측하세요:
```c
#include <stdio.h>

void func(int a[20]) {
    printf("%zu\n", sizeof(a));
}

int main(void) {
    int arr[20];
    printf("%zu\n", sizeof(arr));
    func(arr);
    return 0;
}
```

### 문제 3: 디버깅
다음 코드의 문제점을 찾고 수정하세요:
```c
void print_all(int arr[]) {
    int size = sizeof(arr) / sizeof(arr[0]);
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
}
```

---

## 참고 자료

- **ISO/IEC 9899:2018** (C18 Standard) Section 6.3.2.1
- **C Reference**: https://en.cppreference.com/w/c/language/conversion
- **Kernighan & Ritchie**, "The C Programming Language", Section 5.3
