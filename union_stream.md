# C언어 비유로 배우는 union과 파일 I/O 스트림

> 대상: struct까지 이해한 중급 학습자
> 목표: union과 파일 입출력 stream을 **"왜 필요한가"**와 **"어떻게 쓰는가"** 두 축으로, 비유를 통해 직관적으로 이해하기

---

# 1부. union — "하나의 방, 여러 용도"

## 1-1. struct와 union, 무엇이 다른가

이미 아는 `struct`부터 비유로 다시 잡고 들어가자.

- **struct (구조체) = 칸막이가 있는 서랍장**
  - 양말 칸, 셔츠 칸, 바지 칸이 **따로** 있다.
  - 모든 물건이 **동시에** 자기 자리에 들어가 있다.
  - 서랍장 전체 크기 = 모든 칸의 크기를 **더한 값**.

- **union (공용체) = 칸막이 없는 다용도 방 한 칸**
  - 같은 방을 아침엔 침실, 낮엔 서재, 저녁엔 식당으로 쓴다.
  - 한 순간에는 **딱 한 가지 용도**로만 쓸 수 있다 (침대 펴면 책상은 못 씀).
  - 방 크기 = 가장 큰 가구가 들어갈 만큼 (가장 큰 멤버 크기).

```c
struct Box {       // 서랍장: 각 칸이 따로
    int   i;       // 4바이트 칸
    float f;       // 4바이트 칸
    char  c;       // 1바이트 칸
};                 // 전체 크기 ≈ 12바이트 (정렬 포함)

union Room {       // 다용도 방: 한 공간을 공유
    int   i;       // 같은 공간을
    float f;       // 다르게
    char  c;       // 쓸 뿐
};                 // 전체 크기 = 가장 큰 멤버 = 4바이트
```

> **핵심 한 줄:** struct는 "**그리고(and)**" — i 그리고 f 그리고 c. union은 "**또는(or)**" — i 또는 f 또는 c.

## 1-2. 왜 union이 필요한가 — 세 가지 비유

### 비유 ① 좁은 원룸의 접이식 가구 (메모리 절약)

원룸에 사는 사람은 침대·책상·식탁을 다 놓을 공간이 없다. 그래서 **접이식 소파베드** 하나로 낮엔 소파, 밤엔 침대로 쓴다. 임베디드 기기처럼 메모리가 아주 귀한 환경에서, "어차피 동시에 안 쓰는 값들"을 한 공간에 겹쳐 두면 메모리를 크게 아낄 수 있다.

### 비유 ② 만능 신분 증명 (하나의 값을 여러 시각으로 보기)

같은 사람이 상황에 따라 "주민등록증"으로도, "사원증"으로도, "운전면허증"으로도 제시된다. 사람(메모리)은 하나인데 **보는 관점**이 다르다. union은 똑같은 비트(bit) 묶음을 `int`로도 보고 `float`로도 보고 `byte 배열`로도 볼 수 있게 해준다. 하드웨어 레지스터 해석, 네트워크 패킷 파싱에서 단골로 쓰인다.

```c
// 같은 4바이트를 정수로도, 바이트 4개로도 들여다보기
union View {
    unsigned int  whole;   // 통째로 본 모습
    unsigned char byte[4]; // 한 바이트씩 쪼갠 모습
};
```

### 비유 ③ 택배 상자 + 내용물 라벨 (태그드 유니온)

택배 상자(union)는 같은 크기지만, 안에 책이 들었는지 옷이 들었는지는 **겉의 라벨**을 봐야 안다. union만 쓰면 "지금 어떤 멤버가 유효한지" 알 수 없으므로, 보통 **꼬리표(태그) 역할의 변수**를 struct로 함께 묶는다. 이것을 **태그드 유니온(tagged union)**이라 부른다.

```c
enum Kind { INT_VAL, FLOAT_VAL, STR_VAL };

struct Value {
    enum Kind kind;        // 라벨: 지금 어떤 내용물인가
    union {
        int   i;
        float f;
        char  s[20];
    } data;                // 실제 내용물 (한 번에 하나)
};
```

## 1-3. 어떻게 사용하는가

```c
#include <stdio.h>

union Room {
    int   i;
    float f;
    char  c;
};

int main(void) {
    union Room r;

    r.i = 65;
    printf("정수로: %d\n", r.i);     // 65

    r.f = 3.14f;   // 이 순간 r.i 값은 "덮어써져" 의미 없어짐
    printf("실수로: %.2f\n", r.f);   // 3.14

    // 주의: 여기서 r.i를 읽으면 3.14의 비트를 정수로 잘못 해석한 쓰레기 값
    printf("크기: %zu바이트\n", sizeof(r));  // 4
    return 0;
}
```

**규칙 세 가지**
- 마지막에 **쓴(write) 멤버만** 유효하다. 다른 멤버를 읽으면 비트 재해석 → 대개 의미 없는 값.
- `sizeof(union)`은 **가장 큰 멤버** 크기(+정렬).
- "지금 어떤 멤버가 유효한가"는 union 스스로 모른다 → 태그를 직접 관리해야 안전.

## 1-4. 메모리로 직접 확인하기 (중급 심화)

다용도 방은 "같은 바닥 위에" 가구를 놓는 것이다. union 멤버들은 **모두 같은 시작 주소**를 공유한다.

```c
union Room r;
printf("%p\n", (void*)&r.i);  // 세 주소가
printf("%p\n", (void*)&r.f);  // 모두
printf("%p\n", (void*)&r.c);  // 같다!
```

이 "같은 주소" 성질 때문에 비유 ②(여러 시각으로 보기)가 성립한다.

---

# 2부. 파일 I/O 스트림 — "물이 흐르는 수도관"

## 2-1. 스트림(stream)이란 무엇인가

가장 큰 오해: "파일을 읽는다 = 파일 전체를 한 번에 통째로 가져온다". 실제로는 그렇지 않다.

- **스트림 = 수도관(호스)**
  - 데이터가 **물처럼 한 줄기로 흘러** 들어오거나 나간다.
  - 한 번에 양동이째 붓는 게 아니라, **필요한 만큼 조금씩** 흐른다.
  - 프로그램은 수도꼭지 앞에서 컵으로 받는 사람과 같다.

이 "한 줄기 흐름"이라는 추상화 덕분에, 파일이든 키보드든 네트워크든 **똑같은 방식(`fgetc`, `fputc`, `fread`…)**으로 다룰 수 있다. 수도관 끝에 뭐가 연결됐든 물 받는 동작은 같은 것과 같다.

> 표준 입출력 `stdin`(키보드), `stdout`(화면), `stderr`(오류 화면)도 모두 스트림이다. C는 처음부터 세 개의 수도관을 열어둔 채 프로그램을 시작한다.

## 2-2. 왜 스트림(과 버퍼)이 필요한가

### 비유 ① 댐과 수도 — 직접 퍼오기 vs 수도관 (추상화)

강물을 마시려고 매번 강까지 양동이 들고 가는 사람은 없다. 집까지 **수도관**이 깔려 있고 꼭지만 틀면 된다. 디스크(강)에 직접 접근하는 복잡한 일은 운영체제가 처리하고, 우리는 표준화된 수도꼭지(`FILE*`)만 다룬다.

### 비유 ② 택배 한 번에 모아 배송 — 버퍼(buffer)

편지 한 통 쓸 때마다 우체국에 직접 뛰어가면 비효율적이다. 보통은 **우편함(버퍼)**에 모았다가 한꺼번에 부친다. 디스크 접근은 느리므로, 출력 데이터를 메모리 **버퍼**에 모았다가 가득 차면 한 번에 디스크로 보낸다. 그래서:

- `printf`를 했는데 화면에 바로 안 보일 때가 있다 → 아직 우편함에 쌓여 있는 중.
- `fflush()` = **"지금 당장 우편함 비워서 보내!"** 강제 발송 명령.
- `fclose()` = 수도꼭지를 잠그면서 우편함에 남은 것도 마저 보냄.

### 비유 ③ 책갈피 — 파일 위치 표시자(file position indicator)

스트림에는 **"지금 어디까지 읽었는지" 책갈피**가 꽂혀 있다. 한 글자 읽으면 책갈피가 자동으로 한 칸 뒤로 간다.

- `ftell()` = 지금 책갈피가 몇 페이지인지 확인.
- `fseek()` = 책갈피를 원하는 위치로 옮기기.
- `rewind()` = 책갈피를 맨 앞으로 되돌리기.
- **EOF** = 책의 마지막 장을 넘겼다는 신호 (End Of File).

## 2-3. 어떻게 사용하는가 — 기본 4단계

수도 쓰는 순서와 똑같다: **① 꼭지 열기 → ② 받기/붓기 → ③ (확인) → ④ 꼭지 잠그기**

```c
#include <stdio.h>

int main(void) {
    // ① 열기: fopen(파일이름, 모드)
    FILE *fp = fopen("memo.txt", "w");   // "w" = 쓰기(write)
    if (fp == NULL) {                    // 꼭지가 안 열렸는지 반드시 확인!
        printf("파일 열기 실패\n");
        return 1;
    }

    // ② 붓기: 스트림으로 흘려보내기
    fprintf(fp, "안녕, 스트림!\n");
    fputc('A', fp);

    // ④ 잠그기: 남은 버퍼까지 디스크로 보내고 닫기
    fclose(fp);
    return 0;
}
```

**fopen 모드 = 수도꼭지의 종류**

| 모드 | 비유 | 의미 |
|------|------|------|
| `"r"` | 읽기 전용 꼭지 | 읽기. 파일 없으면 실패 |
| `"w"` | 새 양동이로 교체 | 쓰기. 기존 내용 **싹 비움**, 없으면 새로 만듦 |
| `"a"` | 기존 통에 덧붓기 | 추가(append). 끝에 이어 씀 |
| `"r+"` | 읽고 쓰는 꼭지 | 읽기+쓰기 (파일 있어야 함) |
| `"rb"`,`"wb"` | 정수(원수) 모드 | 바이너리. 텍스트 변환 없이 그대로 |

**읽기 예시 — 한 줄기씩 흘려 받기**

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("memo.txt", "r");
    if (fp == NULL) return 1;

    int ch;
    // fgetc는 한 글자씩, 책의 끝(EOF)까지 흐름을 받는다
    while ((ch = fgetc(fp)) != EOF) {
        putchar(ch);          // 받은 물을 화면 컵에 따름
    }

    fclose(fp);
    return 0;
}
```

> **왜 `int ch`인가?** `fgetc`는 정상 글자(0~255)뿐 아니라 "끝 신호 EOF(보통 -1)"도 돌려준다. `char`로 받으면 EOF와 일반 글자를 구분 못 할 수 있어, 더 넓은 그릇인 `int`로 받는다. (책갈피가 끝장을 넘겼다는 신호를 담을 칸이 필요)

**자주 쓰는 함수 묶음**

| 함수 | 비유 | 용도 |
|------|------|------|
| `fgetc`/`fputc` | 물 한 방울 | 한 글자 |
| `fgets`/`fputs` | 한 컵 | 한 줄(문자열) |
| `fscanf`/`fprintf` | 양식 맞춰 받기/붓기 | 서식 입출력 |
| `fread`/`fwrite` | 통째 펌프질 | 바이너리 블록 |
| `fseek`/`ftell`/`rewind` | 책갈피 | 위치 이동/확인 |
| `feof`/`ferror` | 끝/고장 점검 | 상태 확인 |

---

# 3부. 실습 문제

> 난이도 표시: ★(기초) ★★(응용) ★★★(심화). 정답과 해설은 4부에 있다. **먼저 스스로 풀어 보기!**

## 3-A. union 실습

**문제 A1 ★** — 다음 코드의 출력 `sizeof` 값을 예측하라. 그리고 왜 그런지 "다용도 방" 비유로 설명하라.
```c
union U {
    char  c;      // 1바이트
    int   i;      // 4바이트
    double d;     // 8바이트
};
printf("%zu\n", sizeof(union U));
```

**문제 A2 ★** — 빈칸을 채워 union을 선언하고, `i`에 100을 넣어 출력하라.
```c
union Data {
    int i;
    float f;
};
union Data d;
______ = 100;            // (1) i에 100 대입
printf("%d\n", ______);  // (2) i 출력
```

**문제 A3 ★★** — 아래 코드의 두 번째 `printf` 출력이 "이상한 값"이 나오는 이유를 비유로 설명하라.
```c
union Data { int i; float f; };
union Data d;
d.f = 3.14f;
printf("%.2f\n", d.f);   // 3.14
printf("%d\n", d.i);     // ??? 이상한 정수
```

**문제 A4 ★★** — `enum`과 union을 이용해 "정수 또는 문자열"을 담는 **태그드 유니온** `struct Item`을 설계하라. 그리고 라벨을 보고 알맞게 출력하는 함수 `void print_item(struct Item it)`를 작성하라.

**문제 A5 ★★★** — 다음 union을 이용해 `unsigned int x = 0x12345678;`이 메모리에 어떤 **바이트 순서**로 저장되는지 출력하는 프로그램을 작성하라. 결과로 자신의 컴퓨터가 **리틀 엔디언인지 빅 엔디언인지** 판별하라.
```c
union { unsigned int whole; unsigned char byte[4]; } u;
```
(힌트: 리틀 엔디언이면 `byte[0]`이 `0x78`)

## 3-B. 파일 I/O 스트림 실습

**문제 B1 ★** — `"hello.txt"`에 `Hello, File!`이라는 한 줄을 쓰는 프로그램을 작성하라. fopen 모드는 무엇을 써야 하나? 그 이유를 "양동이 교체 vs 덧붓기" 비유로 답하라.

**문제 B2 ★** — 다음 코드에 **반드시 빠진 두 가지**가 있다. 무엇인가? (힌트: 안전 확인 하나, 마무리 하나)
```c
FILE *fp = fopen("data.txt", "r");
int ch;
while ((ch = fgetc(fp)) != EOF)
    putchar(ch);
```

**문제 B3 ★★** — `"score.txt"`에 학생 3명의 점수를 한 줄에 하나씩 쓰고(예: 90, 85, 77), 다시 파일을 열어 **합계와 평균**을 계산해 출력하는 프로그램을 작성하라. (`fprintf`/`fscanf` 사용)

**문제 B4 ★★** — `printf("로딩중...");` 다음에 5초간 다른 작업을 하는데, 화면에 `로딩중...`이 즉시 안 보이고 한참 뒤 나타났다. **왜 그런가**를 "우편함" 비유로 설명하고, **한 줄을 추가**해 즉시 보이게 고쳐라.

**문제 B5 ★★** — 파일을 한 줄씩 읽어 **줄 번호를 붙여** 화면에 출력하는 프로그램을 작성하라. (`fgets` 사용, 예: `1: 첫째 줄`)

**문제 B6 ★★★** — `fseek`, `ftell`을 사용해, 어떤 텍스트 파일의 **크기(바이트 수)**를 한 글자도 읽지 않고 구하는 함수 `long file_size(const char *name)`를 작성하라. (힌트: 책갈피를 끝으로 보낸 뒤 위치를 읽기)

**문제 B7 ★★★** — `fwrite`/`fread`로 `struct Student { char name[20]; int age; }` 배열 3개를 바이너리 파일에 저장하고 다시 읽어와 출력하라. 텍스트 모드(`"w"`)가 아니라 바이너리 모드(`"wb"`/`"rb"`)를 써야 하는 이유를 한 줄로 설명하라.

---

# 4부. 정답 및 해설

## 4-A. union 정답

**A1.** `8`. 다용도 방의 크기는 **가장 큰 가구(double, 8바이트)**가 들어갈 만큼이어야 한다. char나 int를 넣어도 방은 8바이트로 유지된다. (정렬상 8의 배수가 되어야 하므로 정확히 8)

**A2.** `(1) d.i` , `(2) d.i`
```c
d.i = 100;
printf("%d\n", d.i);   // 100
```

**A3.** `d.f = 3.14f`로 그 4바이트에는 3.14의 **부동소수점 비트 패턴**이 들어 있다. 그런데 `d.i`로 읽으면 같은 비트를 **정수로 재해석**한다. "같은 신분증을 다른 안경으로 본 셈"이라 전혀 다른(이상한) 숫자가 나온다. union은 마지막에 쓴 멤버만 유효하다.

**A4.**
```c
#include <stdio.h>
#include <string.h>

enum Kind { K_INT, K_STR };

struct Item {
    enum Kind kind;            // 라벨
    union {
        int  i;
        char s[20];
    } data;
};

void print_item(struct Item it) {
    if (it.kind == K_INT)
        printf("정수: %d\n", it.data.i);
    else
        printf("문자열: %s\n", it.data.s);
}

int main(void) {
    struct Item a = { K_INT };  a.data.i = 42;
    struct Item b = { K_STR };  strcpy(b.data.s, "hello");
    print_item(a);              // 정수: 42
    print_item(b);              // 문자열: hello
    return 0;
}
```

**A5.**
```c
#include <stdio.h>

int main(void) {
    union { unsigned int whole; unsigned char byte[4]; } u;
    u.whole = 0x12345678;
    for (int k = 0; k < 4; k++)
        printf("byte[%d] = 0x%02X\n", k, u.byte[k]);

    if (u.byte[0] == 0x78)
        printf("=> 리틀 엔디언\n");
    else
        printf("=> 빅 엔디언\n");
    return 0;
}
```
대부분의 PC(x86/x64)는 **리틀 엔디언**이라 `byte[0]=0x78`이 나온다. 같은 4바이트를 "통째 정수"와 "바이트 배열"이라는 두 시각으로 본 것이 핵심(비유 ②).

## 4-B. 파일 I/O 정답

**B1.** 모드 `"w"`. 새 메모를 쓰는 것이므로 기존 내용을 비우고 새 양동이로 교체하는 `"w"`가 적절하다(`"a"`는 기존 내용 끝에 덧붓기).
```c
#include <stdio.h>
int main(void) {
    FILE *fp = fopen("hello.txt", "w");
    if (fp == NULL) return 1;
    fprintf(fp, "Hello, File!\n");
    fclose(fp);
    return 0;
}
```

**B2.** ① `fopen` 결과가 `NULL`인지 **여는 데 성공했는지 확인**이 빠졌다(꼭지가 안 열렸을 수 있음). ② `fclose(fp)`로 **닫기**가 빠졌다(수도꼭지 안 잠금 → 자원 누수). 고친 코드:
```c
FILE *fp = fopen("data.txt", "r");
if (fp == NULL) return 1;        // 추가 ①
int ch;
while ((ch = fgetc(fp)) != EOF)
    putchar(ch);
fclose(fp);                      // 추가 ②
```

**B3.**
```c
#include <stdio.h>
int main(void) {
    int scores[3] = {90, 85, 77};

    FILE *fp = fopen("score.txt", "w");
    if (fp == NULL) return 1;
    for (int k = 0; k < 3; k++)
        fprintf(fp, "%d\n", scores[k]);
    fclose(fp);

    fp = fopen("score.txt", "r");
    if (fp == NULL) return 1;
    int v, sum = 0, n = 0;
    while (fscanf(fp, "%d", &v) == 1) {   // 한 정수씩 흘려 받기
        sum += v;
        n++;
    }
    fclose(fp);

    printf("합계 = %d, 평균 = %.2f\n", sum, (double)sum / n);
    return 0;
}
```

**B4.** `printf`는 출력을 **버퍼(우편함)**에 모아두고, 줄바꿈(`\n`)이 없으면 바로 보내지 않는다. 그래서 화면에 즉시 안 나타난다. `fflush(stdout);`을 추가하면 "지금 당장 우편함 비워라" → 즉시 표시된다. (또는 문자열 끝에 `\n` 추가)
```c
printf("로딩중...");
fflush(stdout);   // 추가: 강제 발송
```

**B5.**
```c
#include <stdio.h>
int main(void) {
    FILE *fp = fopen("input.txt", "r");
    if (fp == NULL) return 1;
    char line[256];
    int no = 1;
    while (fgets(line, sizeof(line), fp) != NULL)
        printf("%d: %s", no++, line);   // fgets는 \n까지 읽으므로 별도 줄바꿈 불필요
    fclose(fp);
    return 0;
}
```

**B6.**
```c
#include <stdio.h>
long file_size(const char *name) {
    FILE *fp = fopen(name, "rb");
    if (fp == NULL) return -1;
    fseek(fp, 0, SEEK_END);   // 책갈피를 맨 끝으로
    long size = ftell(fp);    // 끝의 위치 = 전체 바이트 수
    fclose(fp);
    return size;
}
int main(void) {
    printf("크기: %ld 바이트\n", file_size("hello.txt"));
    return 0;
}
```
`SEEK_END`로 책갈피를 끝까지 보낸 뒤 `ftell`로 그 위치(=바이트 수)를 읽는다. 한 글자도 실제로 읽지 않는다.

**B7.**
```c
#include <stdio.h>
struct Student { char name[20]; int age; };

int main(void) {
    struct Student arr[3] = {
        {"Kim", 20}, {"Lee", 22}, {"Park", 19}
    };

    FILE *fp = fopen("students.dat", "wb");   // 바이너리 쓰기
    if (fp == NULL) return 1;
    fwrite(arr, sizeof(struct Student), 3, fp);  // 구조체 3개 통째로
    fclose(fp);

    struct Student in[3];
    fp = fopen("students.dat", "rb");          // 바이너리 읽기
    if (fp == NULL) return 1;
    fread(in, sizeof(struct Student), 3, fp);
    fclose(fp);

    for (int k = 0; k < 3; k++)
        printf("%s, %d\n", in[k].name, in[k].age);
    return 0;
}
```
바이너리 모드를 쓰는 이유: 구조체에는 사람이 읽는 글자가 아닌 **원시 바이트(int 4바이트 등)**가 들어 있다. 텍스트 모드는 줄바꿈 문자 변환 등을 하므로 바이트가 **오염**될 수 있다. 원수(原水) 그대로 흘려야 하므로 `"wb"`/`"rb"`를 쓴다.

---

# 5부. 한 장 요약 (복습 카드)

**union**
- struct = "그리고(서랍장)", union = "또는(다용도 방)"
- 크기 = 가장 큰 멤버. 모든 멤버가 같은 주소를 공유.
- 마지막에 쓴 멤버만 유효 → 안전하게 쓰려면 **태그(라벨)**를 함께.
- 쓰임새: 메모리 절약, 같은 비트를 여러 시각으로 보기(엔디언/패킷 파싱).

**파일 I/O 스트림**
- 스트림 = 수도관. 데이터는 통째가 아니라 **흐름**으로 오간다.
- 순서: `fopen`(열기) → 읽기/쓰기 → `fclose`(닫기). 열고 나면 **NULL 검사** 필수.
- 버퍼 = 우편함. `fflush`로 강제 발송, `fclose`가 마저 비움.
- 책갈피 = 위치 표시자. `fseek`/`ftell`/`rewind`, 끝 신호는 `EOF`.
- 텍스트는 사람용, 구조체 등 원시 데이터는 **바이너리 모드(`"wb"`/`"rb"`)**.
