# Session 09 — 문자열 처리 Part 2: 문자열 파싱

---

## 1. 문자열 파싱이란?

**파싱(Parsing)** 이란 문자열에서 필요한 정보를 **잘라내어 꺼내는** 작업이다.

예를 들어 아래와 같은 한 줄의 텍스트가 있다고 하자.

```
홍길동,20240001,88.5
```

여기서 이름, 학번, 점수를 각각 꺼내야 한다.  
이 작업이 파싱이다.

> **비유:**  
> 냉장고에서 꺼낸 반찬통 안에 여러 재료가 쉼표로 구분되어 담겨 있다.  
> 파싱은 그 반찬통을 열고 재료를 하나씩 꺼내 각자의 접시에 담는 과정이다.

---

## 2. strtok — 문자열을 구분자로 자르기

### 2.1 기본 사용법

```c
char *strtok(char *str, const char *delim);
```

| 인자 | 의미 |
|------|------|
| `str` | 자를 문자열 (첫 호출에만 전달, 이후는 `NULL`) |
| `delim` | 구분자 문자열 (예: `","`, `" "`, `",;"`) |
| 반환값 | 잘라낸 조각의 시작 포인터, 더 없으면 `NULL` |

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char line[100] = "홍길동,20240001,88.5";

    char *token = strtok(line, ",");    // 첫 번째 조각
    while (token != NULL) {
        printf("[%s]\n", token);
        token = strtok(NULL, ",");      // 두 번째부터는 NULL 전달
    }
    return 0;
}
```

**실행 결과:**
```
[홍길동]
[20240001]
[88.5]
```

### 2.2 strtok는 내부적으로 어떻게 동작하는가?

> **비유:**  
> 요리사가 긴 소시지를 칼로 자른다고 생각해보자.  
> 첫 번째 자를 때는 소시지를 직접 건네준다.  
> 두 번째부터는 "아까 그 소시지 계속 잘라줘" 라고만 하면 된다. (`NULL` 전달)  
> 요리사가 어디까지 잘랐는지 **스스로 기억**하고 있기 때문이다.

`strtok`는 구분자를 찾으면 그 자리를 `'\0'`으로 **덮어써서** 문자열을 자른다.

```
원본:   '홍','길','동',',','2','0','2','4',...
자른 후: '홍','길','동','\0','2','0','2','4',...
                          ↑
                     ',' 를 '\0' 으로 덮어씀
```

### 2.3 strtok의 한계 — 왜 현업에서 주의해서 쓰는가?

**한계 1: 원본 문자열을 수정한다**

```c
char line[100] = "홍길동,20240001,88.5";
strtok(line, ",");
printf("%s\n", line);   // "홍길동" (원본이 바뀌어 있음)
```

`strtok`를 쓰고 나면 원본 문자열이 손상된다.  
원본이 필요하다면 반드시 **복사본**을 만들어서 사용해야 한다.

**한계 2: 중첩 루프에서 사용할 수 없다**

`strtok`는 내부적으로 **정적 변수(static variable)** 에 현재 위치를 저장한다.  
두 개의 문자열을 동시에 자르려고 하면 **서로 충돌**한다.

```c
// 잘못된 사용 예:
char line1[50] = "A,B,C";
char line2[50] = "X,Y,Z";

char *t1 = strtok(line1, ",");
char *t2 = strtok(line2, ",");   // 이 순간 line1 파싱 상태가 사라짐
```

이 문제를 해결한 것이 `strtok_s` (MSVC) 와 `strtok_r` (POSIX) 이다.

---

## 3. strtok_s / strtok_r — 더 안전한 버전

### 3.1 차이점

| 함수 | 사용 환경 | 헤더 |
|------|----------|------|
| `strtok` | 표준 C (모든 환경) | `<string.h>` |
| `strtok_s` | MSVC (Visual Studio) | `<string.h>` |
| `strtok_r` | POSIX (Linux, macOS, GCC) | `<string.h>` |

`strtok_s`와 `strtok_r`은 현재 위치를 **외부 변수**에 저장한다.  
덕분에 여러 문자열을 동시에 파싱할 수 있다.

### 3.2 strtok_s (Visual Studio 기준)

```c
char *strtok_s(char *str, const char *delim, char **context);
```

세 번째 인자 `context`가 추가되었다.  
현재 파싱 위치를 이 포인터에 저장하므로, 각 문자열마다 별도의 `context`를 쓰면 된다.

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char line[100] = "홍길동,20240001,88.5";

    char *context = NULL;   // 파싱 위치를 기억할 변수
    char *token = strtok_s(line, ",", &context);

    while (token != NULL) {
        printf("[%s]\n", token);
        token = strtok_s(NULL, ",", &context);
    }
    return 0;
}
```

**실행 결과:**
```
[홍길동]
[20240001]
[88.5]
```

### 3.3 strtok_r (Linux / GCC 기준)

```c
char *strtok_r(char *str, const char *delim, char **saveptr);
```

`strtok_s`와 구조가 동일하다. 세 번째 인자 이름만 `saveptr`로 다르다.

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char line[100] = "홍길동,20240001,88.5";

    char *saveptr = NULL;
    char *token = strtok_r(line, ",", &saveptr);

    while (token != NULL) {
        printf("[%s]\n", token);
        token = strtok_r(NULL, ",", &saveptr);
    }
    return 0;
}
```

### 3.4 환경별 선택 정리

```
Visual Studio (MSVC) 사용 시  →  strtok_s
Linux / GCC 사용 시           →  strtok_r
이식성이 필요한 코드           →  조건부 컴파일로 분기
```

```c
// 이식성이 필요한 경우 (참고용)
#ifdef _MSC_VER
    char *token = strtok_s(line, ",", &context);
#else
    char *token = strtok_r(line, ",", &saveptr);
#endif
```

> 수업에서는 **Visual Studio 환경**이므로 `strtok_s`를 기준으로 사용한다.

---

## 4. 문자열을 숫자로 변환 — strtol / strtof

`strtok`로 잘라낸 조각은 모두 **문자열**이다.  
`"20240001"` 은 숫자처럼 생겼지만 실제로는 글자들의 배열이다.  
이것을 실제 숫자로 변환해야 한다.

### 4.1 atoi / atof — 간단하지만 위험한 방법

```c
#include <stdlib.h>

int   n = atoi("20240001");    // 문자열 → int
float f = atof("88.5");        // 문자열 → float (double 반환)
```

간단하지만 **변환 실패를 감지할 수 없다.**

```c
int n = atoi("abc");   // 0을 반환하지만, 오류인지 정말 0인지 구분 불가
```

### 4.2 strtol / strtof — 안전한 변환

```c
#include <stdlib.h>

long  strtol (const char *str, char **endptr, int base);
float strtof (const char *str, char **endptr);
```

| 인자 | 의미 |
|------|------|
| `str` | 변환할 문자열 |
| `endptr` | 변환을 멈춘 위치를 저장하는 포인터 (`NULL` 전달 시 무시) |
| `base` | 진법 (10진수이면 `10`) |

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char id_str[20]    = "20240001";
    char score_str[20] = "88.5";

    char *endptr;

    long  id    = strtol(id_str, &endptr, 10);
    float score = strtof(score_str, NULL);

    printf("학번: %ld\n", id);       // 20240001
    printf("점수: %.1f\n", score);   // 88.5
    return 0;
}
```

**변환 실패 감지:**

```c
char *end;
long id = strtol("abc", &end, 10);

if (end == "abc") {
    printf("변환 실패: 숫자가 아닙니다.\n");
}
```

`endptr`가 원본 문자열 시작과 같다면 → 변환된 글자가 하나도 없다는 뜻이다.

---

## 5. 핵심 정리 (Part 2)

```
파싱 함수 요약
─────────────────────────────────────────────────────
strtok(str, delim)               표준, 단순, 중첩 불가
strtok_s(str, delim, &context)   MSVC 전용, 안전
strtok_r(str, delim, &saveptr)   POSIX 전용, 안전

strtok 주의사항:
  1. 원본 문자열을 수정한다 → 복사본 사용 권장
  2. 두 번째 호출부터 첫 인자는 NULL
  3. 중첩 루프에서는 strtok_s / strtok_r 사용

숫자 변환 요약
─────────────────────────────────────────────────────
atoi(str)              문자열 → int    (오류 감지 불가)
atof(str)              문자열 → double (오류 감지 불가)
strtol(str, &end, 10)  문자열 → long   (오류 감지 가능)
strtof(str, &end)      문자열 → float  (오류 감지 가능)
─────────────────────────────────────────────────────
```

---

*참고: ISO/IEC 9899:2011 (C11) §7.24 String handling \<string.h\>, §7.22.1 Numeric conversion functions \<stdlib.h\>*
