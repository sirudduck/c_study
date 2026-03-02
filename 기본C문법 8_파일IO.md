# Session 08 - 파일 입출력 (File I/O)

## 1. 문제 먼저 생각하기

### 요리사 비유로 다시 생각해보자

지금까지 배운 내용을 주방 비유로 정리하면 이렇다.

```
CPU   = 요리사       (실제로 일을 처리)
RAM   = 조리대       (지금 당장 쓰는 재료를 올려두는 곳)
SSD   = 냉장고       (재료를 장기간 보관하는 곳)
```

지금까지 만든 프로그램은 **조리대(RAM) 위에서만 작업**했다.

```c
int main(void) {
    int score = 95;            // 조리대 위에 재료를 꺼냄
    printf("%d\n", score);
    return 0;
}
// 프로그램 종료 = 퇴근
// 퇴근하면 조리대 위의 재료는 모두 치워짐 → 데이터 사라짐
```

요리사가 **퇴근(프로그램 종료)** 하면 조리대(RAM) 위의 재료는 모두 치워진다.  
내일 출근해도 어제 꺼내 뒀던 재료는 없다.

> **핵심 질문:** 퇴근해도 재료가 남아있으려면 어떻게 해야 할까?

답은 간단하다. **냉장고(SSD)에 넣어두면 된다.**  
냉장고는 전기가 꺼져도 재료가 사라지지 않는다.

```
조리대(RAM)                    냉장고(SSD)
┌──────────────┐               ┌──────────────┐
│  score = 95  │  →  파일 저장 │  score.txt   │
│  (임시 보관) │               │  (영구 보관) │
└──────────────┘               └──────────────┘
  퇴근하면 사라짐                 전원 꺼져도 유지
```

C에서 냉장고(SSD)에 재료(데이터)를 넣고 꺼내는 작업이 바로 **파일 입출력**이다.

---

## 2. 파일이란 무엇인가?

**파일(File)** 은 냉장고(SSD) 안에 있는 **반찬통** 이라고 생각하면 된다.

```
냉장고(SSD) 안
┌────────────────────────────────────┐
│  📦 students.txt   (학생 성적 통)  │
│  📦 config.txt     (설정값 통)     │
│  📦 photo.jpg      (사진 통)       │
└────────────────────────────────────┘
```

반찬통(파일)을 사용하는 순서는 항상 동일하다.

```
1. 냉장고에서 반찬통 꺼내기   (fopen  — 파일 열기)
2. 재료를 넣거나 꺼내기       (fprintf, fread 등 — 읽기/쓰기)
3. 반찬통 뚜껑 닫고 냉장고에 다시 넣기  (fclose — 파일 닫기)
```

> **중요:** 반찬통 뚜껑을 닫지 않으면(fclose 생략) 내용물이 흘러서 **데이터가 손실**될 수 있다.  
> 반드시 열고 → 작업하고 → **닫는** 순서를 지켜야 한다.

---

## 3. 파일 열기와 닫기

### 3.1 fopen — 냉장고에서 반찬통 꺼내기

```c
#include <stdio.h>

FILE *fp = fopen("파일이름", "모드");
```

`fopen`은 파일을 열고, 그 파일을 가리키는 **포인터(`FILE *`)** 를 반환한다.  
반찬통의 **손잡이**를 받아오는 것이라고 생각하면 된다.  
파일 열기에 실패하면 `NULL`(손잡이 없음)을 반환한다.

### 3.2 모드(mode) — 반찬통을 어떤 용도로 꺼내느냐

| 모드 | 의미 | 파일이 없을 때 |
|------|------|--------------|
| `"r"` | 읽기 전용 (꺼내기만) | 실패 (`NULL`) |
| `"w"` | 쓰기 전용 (새로 담기) | 새 반찬통 생성 |
| `"a"` | 추가 쓰기 (기존 내용 유지하고 뒤에 추가) | 새 반찬통 생성 |
| `"rb"` | 이진 읽기 | 실패 (`NULL`) |
| `"wb"` | 이진 쓰기 | 새 반찬통 생성 |

> **주의:** `"w"` 모드는 기존 반찬통의 내용을 **모두 비우고** 시작한다.  
> 기존 내용을 유지하고 뒤에 추가하려면 `"a"` 모드를 사용해야 한다.

### 3.3 fclose — 뚜껑 닫고 냉장고에 다시 넣기

```c
fclose(fp);
```

파일 작업이 끝나면 반드시 닫아야 한다.  
닫지 않으면 쓰기 버퍼(임시로 모아둔 데이터)가 SSD에 반영되지 않아 **데이터가 손실**된다.

### 3.4 열기 실패 처리 — 냉장고에 없는 반찬통을 꺼내려 할 때

```c
FILE *fp = fopen("students.txt", "r");
if (fp == NULL) {   // 반찬통을 찾지 못했을 때
    printf("파일 열기 실패\n");
    return 1;
}
// 정상 처리
fclose(fp);
```

`NULL` 체크를 빠뜨리면, 존재하지 않는 손잡이로 작업하려다 **프로그램이 충돌**한다.  
파일을 열고 나면 항상 `NULL` 체크를 먼저 하는 습관을 들이자.

---

## 4. 텍스트 파일 쓰기 — fprintf

`fprintf`는 `printf`와 사용법이 거의 동일하다.  
`printf`가 화면에 출력한다면, `fprintf`는 **반찬통(파일)에 적어 넣는다.**

```
printf("홍길동")   →  화면(모니터)에 출력
fprintf(fp, "홍길동")  →  파일에 기록
```

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("students.txt", "w");   // 반찬통 꺼내기 (쓰기 모드)
    if (fp == NULL) {
        printf("파일 열기 실패\n");
        return 1;
    }

    fprintf(fp, "홍길동 20240001 88.5\n");
    fprintf(fp, "이순신 20240002 95.0\n");
    fprintf(fp, "강감찬 20240003 72.0\n");

    fclose(fp);   // 뚜껑 닫기
    printf("저장 완료\n");
    return 0;
}
```

실행 후 `students.txt` 파일 내용 (메모장으로 열면 보임):
```
홍길동 20240001 88.5
이순신 20240002 95.0
강감찬 20240003 72.0
```

---

## 5. 텍스트 파일 읽기 — fscanf

`fscanf`는 `scanf`와 사용법이 거의 동일하다.  
`scanf`가 키보드로 입력받는다면, `fscanf`는 **반찬통(파일)에서 꺼낸다.**

```
scanf("%s", name)        →  키보드에서 입력 받기
fscanf(fp, "%s", name)   →  파일에서 읽어 오기
```

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("students.txt", "r");   // 반찬통 꺼내기 (읽기 모드)
    if (fp == NULL) {
        printf("파일 열기 실패\n");
        return 1;
    }

    char  name[20];
    int   id;
    float score;

    // EOF: 반찬통이 빌 때까지 계속 꺼내기
    while (fscanf(fp, "%s %d %f", name, &id, &score) != EOF) {
        printf("이름: %s, 학번: %d, 점수: %.1f\n", name, id, score);
    }

    fclose(fp);   // 뚜껑 닫기
    return 0;
}
```

**실행 결과:**
```
이름: 홍길동, 학번: 20240001, 점수: 88.5
이름: 이순신, 학번: 20240002, 점수: 95.0
이름: 강감찬, 학번: 20240003, 점수: 72.0
```

> **EOF(End Of File)란?**  
> 반찬통 바닥에 붙어 있는 **"이 이상 없음"** 표시다.  
> `fscanf`가 더 읽을 내용이 없으면 EOF를 반환하고, 이를 루프 종료 조건으로 사용한다.

---

## 6. 텍스트 방식 vs 이진(Binary) 방식

지금까지 사용한 `fprintf`/`fscanf`는 **텍스트(Text) 방식**이다.  
재료(데이터)를 냉장고에 넣을 때 **레시피 카드에 글자로 적어서** 넣는 방식이다.

C에는 또 다른 방식인 **이진(Binary) 방식**이 있다.  
재료를 글자로 변환하지 않고, **재료 자체를 진공포장해서** 그대로 넣는 방식이다.

```
텍스트 방식으로 float 88.5 저장:
→ 레시피 카드에 "88.5"라고 적음  (문자 4개 = 4 bytes)
→ 사람이 메모장으로 열어서 읽을 수 있음

이진 방식으로 float 88.5 저장:
→ float 값 88.5를 진공포장 그대로 저장  (float = 4 bytes)
→ 메모장으로 열면 깨진 문자가 보임 (사람이 읽을 수 없음)
```

| 항목 | 텍스트 방식 (레시피 카드) | 이진 방식 (진공포장) |
|------|--------------------------|---------------------|
| **저장 형태** | 사람이 읽을 수 있는 문자 | 메모리 내용 그대로 |
| **파일 크기** | 상대적으로 큼 | 상대적으로 작음 |
| **속도** | 변환 과정이 있어 느림 | 빠름 |
| **함수** | `fprintf`, `fscanf` | `fwrite`, `fread` |
| **모드** | `"r"`, `"w"` | `"rb"`, `"wb"` |
| **용도** | 설정 파일, 로그, 사람이 편집할 데이터 | 구조체 저장, 대용량 데이터 |

---

## 7. 이진 파일 쓰기 — fwrite

```c
size_t fwrite(const void *ptr, size_t size, size_t count, FILE *fp);
```

| 인자 | 의미 | 비유 |
|------|------|------|
| `ptr` | 저장할 데이터의 주소 | 진공포장할 재료의 위치 |
| `size` | 데이터 하나의 크기 (bytes) | 재료 하나의 크기 |
| `count` | 저장할 개수 | 포장할 개수 |
| `fp` | 파일 포인터 | 반찬통 손잡이 |

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("data.bin", "wb");   // 이진 쓰기 모드
    if (fp == NULL) {
        printf("파일 열기 실패\n");
        return 1;
    }

    int values[5] = {10, 20, 30, 40, 50};

    // int 5개를 진공포장해서 반찬통에 넣기
    fwrite(values, sizeof(int), 5, fp);

    fclose(fp);
    printf("저장 완료\n");
    return 0;
}
```

---

## 8. 이진 파일 읽기 — fread

```c
size_t fread(void *ptr, size_t size, size_t count, FILE *fp);
```

`fwrite`와 인자 구조가 동일하다.  
반찬통에서 진공포장된 재료를 꺼내 조리대(메모리)에 올려놓는 것이다.

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("data.bin", "rb");   // 이진 읽기 모드
    if (fp == NULL) {
        printf("파일 열기 실패\n");
        return 1;
    }

    int values[5];

    // 반찬통에서 진공포장된 int 5개를 꺼내 조리대에 올리기
    fread(values, sizeof(int), 5, fp);

    for (int i = 0; i < 5; i++) {
        printf("values[%d] = %d\n", i, values[i]);
    }

    fclose(fp);
    return 0;
}
```

**실행 결과:**
```
values[0] = 10
values[1] = 20
values[2] = 30
values[3] = 40
values[4] = 50
```

---

## 9. 구조체 데이터를 파일로 저장하고 불러오기

지금까지 배운 내용을 모두 합친다.  
학생 성적 구조체를 파일로 저장하고, 다시 불러오는 프로그램이다.

### 9.1 텍스트 방식 — 레시피 카드에 적어서 냉장고에 넣기

```c
#include <stdio.h>
#include <string.h>

#define MAX_STUDENTS 100

typedef struct {
    char  name[20];
    int   id;
    float score;
} Student;

// 학생 배열을 텍스트 파일로 저장 (레시피 카드에 적기)
void save_students(const char *filename, Student *arr, int n) {
    FILE *fp = fopen(filename, "w");
    if (fp == NULL) {
        printf("저장 실패: 파일을 열 수 없습니다.\n");
        return;
    }

    fprintf(fp, "%d\n", n);   // 첫 줄에 학생 수 기록 (나중에 몇 명인지 알기 위해)
    for (int i = 0; i < n; i++) {
        fprintf(fp, "%s %d %.1f\n", arr[i].name, arr[i].id, arr[i].score);
    }

    fclose(fp);
    printf("%d명 저장 완료 → %s\n", n, filename);
}

// 텍스트 파일에서 학생 배열로 불러오기 (레시피 카드 읽기)
int load_students(const char *filename, Student *arr) {
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        printf("불러오기 실패: 파일이 없습니다.\n");
        return 0;
    }

    int n = 0;
    fscanf(fp, "%d", &n);   // 첫 줄에서 학생 수 읽기

    for (int i = 0; i < n; i++) {
        fscanf(fp, "%s %d %f", arr[i].name, &arr[i].id, &arr[i].score);
    }

    fclose(fp);
    printf("%d명 불러오기 완료 ← %s\n", n, filename);
    return n;
}

void print_students(Student *arr, int n) {
    printf("\n%-10s %10s %6s\n", "이름", "학번", "점수");
    printf("─────────────────────────────\n");
    for (int i = 0; i < n; i++) {
        printf("%-10s %10d %6.1f\n", arr[i].name, arr[i].id, arr[i].score);
    }
    printf("\n");
}

int main(void) {
    Student class[MAX_STUDENTS] = {
        {"홍길동", 20240001, 88.5f},
        {"이순신", 20240002, 95.0f},
        {"강감찬", 20240003, 72.0f}
    };
    int n = 3;

    save_students("students.txt", class, n);

    Student loaded[MAX_STUDENTS];
    int loaded_n = load_students("students.txt", loaded);

    print_students(loaded, loaded_n);
    return 0;
}
```

**실행 결과:**
```
3명 저장 완료 → students.txt
3명 불러오기 완료 ← students.txt

이름            학번   점수
─────────────────────────────
홍길동      20240001   88.5
이순신      20240002   95.0
강감찬      20240003   72.0
```

**저장된 `students.txt`를 메모장으로 열면:**
```
3
홍길동 20240001 88.5
이순신 20240002 95.0
강감찬 20240003 72.0
```

사람이 직접 읽고 수정할 수 있다.

---

### 9.2 이진 방식 — 진공포장째로 냉장고에 넣기

구조체를 글자로 변환하지 않고, **조리대(RAM)에 있는 모양 그대로** 냉장고(SSD)에 넣는다.  
코드가 훨씬 간단하고, 구조체 멤버가 늘어나도 코드를 바꿀 필요가 없다.

```c
#include <stdio.h>

#define MAX_STUDENTS 100

typedef struct {
    char  name[20];
    int   id;
    float score;
} Student;

// 이진 파일로 저장 (진공포장째로 냉장고에 넣기)
void save_binary(const char *filename, Student *arr, int n) {
    FILE *fp = fopen(filename, "wb");
    if (fp == NULL) {
        printf("저장 실패\n");
        return;
    }

    fwrite(&n,   sizeof(int),     1, fp);   // 학생 수 저장
    fwrite(arr,  sizeof(Student), n, fp);   // 구조체 배열 저장

    fclose(fp);
    printf("%d명 이진 저장 완료 → %s\n", n, filename);
}

// 이진 파일에서 불러오기 (냉장고에서 꺼내 조리대에 올리기)
int load_binary(const char *filename, Student *arr) {
    FILE *fp = fopen(filename, "rb");
    if (fp == NULL) {
        printf("불러오기 실패\n");
        return 0;
    }

    int n = 0;
    fread(&n,   sizeof(int),     1, fp);   // 학생 수 읽기
    fread(arr,  sizeof(Student), n, fp);   // 구조체 배열 읽기

    fclose(fp);
    printf("%d명 이진 불러오기 완료 ← %s\n", n, filename);
    return n;
}

int main(void) {
    Student class[MAX_STUDENTS] = {
        {"홍길동", 20240001, 88.5f},
        {"이순신", 20240002, 95.0f},
        {"강감찬", 20240003, 72.0f}
    };
    int n = 3;

    save_binary("students.bin", class, n);

    Student loaded[MAX_STUDENTS];
    int loaded_n = load_binary("students.bin", loaded);

    for (int i = 0; i < loaded_n; i++) {
        printf("[%d] %s - %.1f점\n", loaded[i].id, loaded[i].name, loaded[i].score);
    }
    return 0;
}
```

**실행 결과:**
```
3명 이진 저장 완료 → students.bin
3명 이진 불러오기 완료 ← students.bin
[20240001] 홍길동 - 88.5점
[20240002] 이순신 - 95.0점
[20240003] 강감찬 - 72.0점
```

> **`students.bin`을 메모장으로 열면 깨진 문자가 보인다.**  
> 진공포장된 상태이기 때문에 사람이 직접 읽을 수 없다. 이는 정상이다.  
> 반드시 같은 구조체 타입으로 `fread`해야 올바르게 꺼낼 수 있다.

---

## 10. 텍스트 방식 vs 이진 방식 — 언제 어떤 걸 쓰나?

| 상황 | 권장 방식 | 이유 |
|------|---------|------|
| 설정 파일, 로그 파일 | 텍스트 | 사람이 메모장으로 직접 열고 수정 가능 |
| 대량 데이터 저장 | 이진 | 빠르고 파일 크기가 작음 |
| 구조체 배열 저장 | 이진 | 코드가 간단, 변환 오류 없음 |
| 다른 프로그램과 공유 | 텍스트 | 호환성이 높음 |

---

## 11. 핵심 정리

```
냉장고(SSD) 작업 흐름
─────────────────────────────────────────────────
fopen   →  냉장고에서 반찬통 꺼내기
fclose  →  뚜껑 닫고 냉장고에 다시 넣기

fprintf →  레시피 카드에 글자로 적어서 넣기
fscanf  →  레시피 카드를 읽어서 조리대에 올리기

fwrite  →  재료를 진공포장째로 넣기
fread   →  진공포장을 꺼내 조리대에 올리기
─────────────────────────────────────────────────
```

**파일 처리 필수 원칙:**
1. `fopen` 직후 반드시 `NULL` 체크 — 반찬통이 냉장고에 있는지 확인
2. 작업 후 반드시 `fclose` 호출 — 뚜껑을 닫아야 내용이 확실히 저장됨
3. 이진 방식은 저장과 불러오기에 **동일한 구조체 타입** 사용 — 같은 크기의 포장지여야 꺼낼 수 있음

---

## 12. 연습 문제

### 문제 1 (기본)
아래 내용을 `memo.txt`에 저장하는 프로그램을 작성하라.
```
오늘 할 일:
1. C 언어 파일 입출력 공부
2. 연습 문제 풀기
```

---

### 문제 2 (중급)
아래 구조체를 사용하여, 사용자로부터 학생 정보를 3명 입력받고  
텍스트 파일 `grades.txt`에 저장한 뒤, 다시 불러와서 출력하는 프로그램을 작성하라.

```c
typedef struct {
    char  name[20];
    int   id;
    float score;
} Student;
```

---

### 문제 3 (심화)
문제 2를 이진 방식(`fwrite`/`fread`)으로 다시 작성하라.  
저장 파일 이름은 `grades.bin`으로 한다.  
텍스트 방식과 이진 방식의 **파일 크기 차이**를 `sizeof`를 활용하여 예측하고,  
실제 파일 크기와 비교하라.

> 힌트: 텍스트 방식에서 `20240001`은 숫자 8개를 문자로 적으므로 8 bytes,  
> 이진 방식에서는 `int` 그대로 저장하므로 4 bytes다.  
> 냉장고에 넣을 때 **레시피 카드에 옮겨 적는 것**과 **진공포장째 넣는 것**의 크기 차이와 같다.

---

*참고: ISO/IEC 9899:2011 (C11) §7.21 Input/output \<stdio.h\>*
