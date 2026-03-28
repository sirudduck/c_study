# Session 09 — 문자열 처리 Part 3: 파일 I/O 연계

---

## 1. 지금까지 배운 것들을 연결하기

Part 1과 Part 2에서 배운 내용을 정리하면 이렇다.

```
fgets          →  파일에서 한 줄을 통째로 읽는다
strtok_s       →  읽어온 줄을 구분자로 잘라낸다
strtol / strtof →  잘라낸 조각을 숫자로 변환한다
```

이 세 가지를 조합하면 **파일에서 구조체 데이터를 안전하게 읽어올 수 있다.**

---

## 2. 왜 fscanf를 현업에서 피하는가?

파일 I/O 시간에 `fscanf`로 파일을 읽는 방법을 배웠다.  
그런데 현업에서는 `fscanf`를 잘 사용하지 않는다.

아래 예시를 보자.

**`students.txt`**
```
3
홍 길동,20240001,88.5
이 순신,20240002,95.0
강 감찬,20240003,72.0
```

이름에 공백이 포함되어 있다. (성과 이름 사이 공백)

```c
// fscanf로 시도
fscanf(fp, "%s %d %f", name, &id, &score);
```

**실제 결과:**
```
이름: [홍]       학번: 0      점수: 0.0   ← 틀림
이름: [길동,20240001,88.5]   ← 완전히 엉망
```

`fscanf`의 `%s`는 **공백을 구분자**로 사용하기 때문에  
`"홍 길동"` 에서 `"홍"` 만 읽고 멈춘다.  
이후 데이터가 완전히 틀어진다.

**현업에서 `fscanf`를 피하는 3가지 이유:**

| 이유 | 설명 |
|------|------|
| 공백 처리 불가 | `%s`는 공백에서 무조건 멈춤 |
| 에러 복구 어려움 | 파싱 실패 시 파일 커서 위치를 예측하기 어려움 |
| 버퍼 오버플로우 | `%s`에 길이 제한 없으면 배열을 넘칠 수 있음 |

---

## 3. fgets + strtok_s 조합으로 해결하기

### 3.1 전체 흐름

```
fgets로 한 줄 읽기
       ↓
strtok_s로 쉼표 기준으로 자르기
       ↓
각 조각을 strtol / strtof로 숫자 변환
       ↓
구조체에 저장
```

### 3.2 파일 포맷 — CSV (쉼표로 구분)

```
students.txt
─────────────────────
3
홍 길동,20240001,88.5
이 순신,20240002,95.0
강 감찬,20240003,72.0
```

첫 줄은 학생 수, 나머지 줄은 `이름,학번,점수` 형식이다.  
이름에 공백이 있어도 쉼표로 구분하므로 문제없다.

### 3.3 전체 코드

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_STUDENTS 100
#define LINE_SIZE    100

typedef struct {
    char  name[20];
    int   id;
    float score;
} Student;

// 파일에서 학생 데이터 읽기
int load_students(const char *filename, Student *arr) {
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        printf("불러오기 실패: 파일이 없습니다.\n");
        return 0;
    }

    char line[LINE_SIZE];
    char *context = NULL;
    char *token;

    // 첫 줄: 학생 수 읽기
    if (fgets(line, sizeof(line), fp) == NULL) {
        fclose(fp);
        return 0;
    }
    int n = (int)strtol(line, NULL, 10);

    // 나머지 줄: 학생 데이터 읽기
    for (int i = 0; i < n; i++) {
        if (fgets(line, sizeof(line), fp) == NULL) break;

        // 줄바꿈 문자 제거
        line[strcspn(line, "\n")] = '\0';

        context = NULL;

        // 이름 (쉼표 전까지)
        token = strtok_s(line, ",", &context);
        if (token == NULL) continue;
        strncpy(arr[i].name, token, sizeof(arr[i].name) - 1);
        arr[i].name[sizeof(arr[i].name) - 1] = '\0';

        // 학번
        token = strtok_s(NULL, ",", &context);
        if (token == NULL) continue;
        arr[i].id = (int)strtol(token, NULL, 10);

        // 점수
        token = strtok_s(NULL, ",", &context);
        if (token == NULL) continue;
        arr[i].score = strtof(token, NULL);
    }

    fclose(fp);
    printf("%d명 불러오기 완료 ← %s\n", n, filename);
    return n;
}

void print_students(Student *arr, int n) {
    printf("\n%-12s %10s %6s\n", "이름", "학번", "점수");
    printf("─────────────────────────────\n");
    for (int i = 0; i < n; i++) {
        printf("%-12s %10d %6.1f\n", arr[i].name, arr[i].id, arr[i].score);
    }
    printf("\n");
}

int main(void) {
    Student arr[MAX_STUDENTS];
    int n = load_students("students.txt", arr);
    print_students(arr, n);
    return 0;
}
```

**실행 결과:**
```
3명 불러오기 완료 ← students.txt

이름             학번   점수
─────────────────────────────
홍 길동      20240001   88.5
이 순신      20240002   95.0
강 감찬      20240003   72.0
```

이름에 공백이 있어도 정확하게 읽힌다.

---

## 4. 코드 핵심 포인트 설명

### 4.1 줄바꿈 문자 제거

```c
line[strcspn(line, "\n")] = '\0';
```

`fgets`는 줄바꿈 문자(`\n`)까지 포함해서 읽는다.  
그대로 두면 마지막 토큰에 `\n`이 붙어서 비교나 출력이 틀어진다.

`strcspn(line, "\n")` 은 `line`에서 `'\n'`이 처음 나오는 위치(인덱스)를 반환한다.  
그 위치를 `'\0'`으로 덮어써서 줄바꿈을 제거한다.

```
"홍 길동,20240001,88.5\n"
                       ↑
              이 위치를 '\0' 으로 바꿈
```

### 4.2 strncpy — 안전한 문자열 복사

```c
strncpy(arr[i].name, token, sizeof(arr[i].name) - 1);
arr[i].name[sizeof(arr[i].name) - 1] = '\0';
```

`strcpy` 대신 `strncpy`를 사용하면 최대 복사 길이를 지정할 수 있다.  
배열 크기를 초과하는 이름이 들어와도 안전하다.

마지막 줄은 `strncpy`가 길이 초과 시 `'\0'`을 붙이지 않는 경우를 대비한 안전장치다.

### 4.3 context를 루프마다 NULL로 초기화

```c
context = NULL;
token = strtok_s(line, ",", &context);
```

`strtok_s`는 `context`에 현재 파싱 위치를 저장한다.  
새 줄을 파싱할 때마다 `context = NULL`로 초기화해야  
이전 줄의 파싱 상태가 섞이지 않는다.

---

## 5. fscanf vs fgets + strtok_s 비교

| 항목 | fscanf | fgets + strtok_s |
|------|--------|-----------------|
| 공백 포함 문자열 | 처리 불가 | 처리 가능 |
| 에러 복구 | 어려움 | 줄 단위로 처리 가능 |
| 버퍼 오버플로우 | 위험 | `sizeof` 지정으로 안전 |
| 코드 복잡도 | 단순 | 다소 복잡 |
| 현업 사용 | 간단한 경우에만 | 표준적 방식 |

---

## 6. 전체 핵심 정리

```
텍스트 파일 읽기 — 현업 표준 흐름
──────────────────────────────────────────────────────
1. fgets(line, sizeof(line), fp)
   → 한 줄을 통째로, 안전하게 읽는다

2. line[strcspn(line, "\n")] = '\0'
   → 줄바꿈 문자를 제거한다

3. strtok_s(line, ",", &context)
   strtok_s(NULL, ",", &context)  ...반복...
   → 쉼표 기준으로 토큰을 하나씩 꺼낸다

4. strtol(token, NULL, 10)
   strtof(token, NULL)
   → 토큰을 숫자로 변환한다

5. strncpy(dest, token, sizeof(dest) - 1)
   → 문자열 토큰을 구조체에 안전하게 복사한다
──────────────────────────────────────────────────────
```

---

*참고: ISO/IEC 9899:2011 (C11) §7.21 Input/output \<stdio.h\>, §7.24 String handling \<string.h\>, §7.22.1 Numeric conversion functions \<stdlib.h\>*
