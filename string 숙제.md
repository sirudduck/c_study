# C언어 string.h 응용 실습

## 학습 목표

- `strtok_s`를 활용한 구분자 기반 문자열 파싱
- `strtol`, `strtof`를 활용한 문자열 → 숫자 변환
- 구조체 배열과 파일 입출력의 결합

---

## 예제 코드: 학생 데이터 파일 파서

```c
#define _CRT_SECURE_NO_WARNINGS
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

### 입력 파일 형식 (`students.txt`)

```
3
Alice,20240001,97.5
Bob,20240002,88.0
Charlie,20240003,73.0
```

### 핵심 함수 정리

| 함수 | 역할 |
|------|------|
| `strtok_s(str, delim, &ctx)` | 구분자 기준으로 문자열을 토큰으로 분리 |
| `strtol(str, NULL, 10)` | 문자열 → `long` 정수 변환 |
| `strtof(str, NULL)` | 문자열 → `float` 실수 변환 |
| `strcspn(str, "\n")` | 줄바꿈 위치 반환 (제거에 활용) |
| `strncpy(dst, src, n)` | 최대 n글자 복사 (버퍼 오버플로 방지) |

---

## 숙제 문제 (5선)

> 위 예제 코드를 기반으로, 직접 수정·확장하는 방식의 문제입니다.

---

### Q1. 평균 점수 출력

`print_students()` 호출 후, 전체 학생의 **평균 점수**를 계산해 출력하라.

```
평균 점수: 82.4
```

---

### Q2. 최고 점수 학생 찾기

점수가 가장 높은 학생의 이름과 점수를 출력하라.

```
최고 점수: Alice (97.5)
```

---

### Q3. 이름으로 검색

`main()`에서 이름을 입력받아, 해당 학생의 정보를 출력하라.
없으면 `"해당 학생 없음"` 출력. `strcmp` 사용.

```
검색할 이름: Bob
Bob | 20240002 | 88.0
```

---

### Q4. 점수 오름차순 정렬 후 출력

학생 배열을 점수 기준 오름차순으로 정렬한 뒤 `print_students()`로 출력하라.
(`qsort` 또는 버블 정렬 사용)

---

### Q5. 파일로 저장

현재 배열 데이터를 `load_students()`가 읽을 수 있는 **동일한 형식**으로 `output.txt`에 저장하는 함수 `save_students()`를 작성하라.

```c
void save_students(const char *filename, Student *arr, int n);
```

저장 결과 예시 (`output.txt`):

```
3
Alice,20240001,97.5
Bob,20240002,88.0
Charlie,20240003,73.0
```

---

> **풀이 순서 추천:** Q1 → Q2 → Q3 → Q4 → Q5
> Q4에서 정렬한 결과를 Q5에서 파일로 저장하면 자연스럽게 이어집니다.
