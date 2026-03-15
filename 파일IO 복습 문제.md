# Session 09 - File I/O 복습 문제 ⭐⭐⭐

## 문제: 학생 성적 파일 변환기

### 구조체 정의

```c
typedef struct {
    char  name[32];
    int   student_id;
    float grade;
} Student;
```

---

### 요구사항

1. `Student` 구조체 배열 3개를 `students.txt`에 텍스트 형식으로 저장한다.
2. `students.txt`를 읽어 `students.bin`에 바이너리 형식으로 변환 저장한다.
3. `students.bin`을 읽어 모든 학생 정보를 아래 형식으로 출력한다.

```
Name: Alice  ID: 20210001  Grade: 95.50
Name: Bob    ID: 20210002  Grade: 88.00
Name: Carol  ID: 20210003  Grade: 72.30
```

> **주의**: 모든 `fopen()` 호출 후 반드시 NULL 체크를 수행할 것.

---

### 스켈레톤 코드

```c
#include <stdio.h>

typedef struct {
    char  name[32];
    int   student_id;
    float grade;
} Student;

void write_text(const char *filename, Student *students, int count) {
    FILE *fp = fopen(filename, "w");
    // TODO: NULL 체크

    for (int i = 0; i < count; i++) {
        // TODO: fprintf로 name, student_id, grade 저장
    }

    fclose(fp);
}

void text_to_binary(const char *src, const char *dst) {
    FILE *fin  = fopen(src, "r");
    FILE *fout = fopen(dst, "wb");
    // TODO: NULL 체크 (fin, fout 각각)

    Student s;
    while (/* TODO: fscanf로 한 명씩 읽기 */) {
        // TODO: fwrite로 바이너리 저장
    }

    fclose(fin);
    fclose(fout);
}

void read_binary(const char *filename) {
    FILE *fp = fopen(filename, "rb");
    // TODO: NULL 체크

    Student s;
    while (/* TODO: fread로 한 명씩 읽기 */) {
        // TODO: 출력
    }

    fclose(fp);
}

int main(void) {
    Student students[3] = {
        {"Alice", 20210001, 95.5f},
        {"Bob",   20210002, 88.0f},
        {"Carol", 20210003, 72.3f}
    };

    write_text("students.txt", students, 3);
    text_to_binary("students.txt", "students.bin");
    read_binary("students.bin");

    return 0;
}
```

---

### 핵심 개념 정리

| 함수 | 모드 | 용도 |
|------|------|------|
| `fprintf(fp, fmt, ...)` | 텍스트 쓰기 | 형식 문자열로 파일에 저장 |
| `fscanf(fp, fmt, ...)` | 텍스트 읽기 | 형식 문자열로 파일에서 읽기 |
| `fwrite(ptr, size, n, fp)` | 바이너리 쓰기 | 메모리 블록을 그대로 저장 |
| `fread(ptr, size, n, fp)` | 바이너리 읽기 | 파일에서 메모리 블록으로 읽기 |

> `fwrite` / `fread`의 반환값은 실제로 처리된 **항목 수**이다. 루프 종료 조건에 반드시 활용할 것.
