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

## 1-5. 태그(tag)를 활용한 완성 예시 — "라벨 보고 꺼내는 만능 상자"

비유 ③에서 본 택배 상자를 실제로 만들어 보자. 핵심은 **"넣을 때 라벨도 같이 붙이고, 꺼낼 때 라벨부터 확인한다"**는 약속이다. 이 약속만 지키면 union의 위험("엉뚱한 멤버를 읽어 쓰레기 값")이 사라진다.

```c
#include <stdio.h>
#include <string.h>

// 1) 라벨(태그) 정의 — 지금 상자에 무엇이 들었는가
enum Type { T_INT, T_DOUBLE, T_STRING };

// 2) 라벨 + 내용물(union)을 struct로 묶기 = 태그드 유니온
struct Variant {
    enum Type tag;        // 라벨
    union {
        int    i;
        double d;
        char   s[32];
    } u;                  // 한 번에 하나만 유효
};

// 3) 넣을 때: 값과 라벨을 "항상 함께" 설정하는 생성 함수
struct Variant make_int(int v)       { struct Variant x; x.tag = T_INT;    x.u.i = v; return x; }
struct Variant make_double(double v) { struct Variant x; x.tag = T_DOUBLE; x.u.d = v; return x; }
struct Variant make_string(const char *v){ struct Variant x; x.tag = T_STRING; strcpy(x.u.s, v); return x; }

// 4) 꺼낼 때: 라벨부터 확인하고 알맞은 멤버만 읽기
void print_variant(struct Variant x) {
    switch (x.tag) {                       // 반드시 tag로 분기!
        case T_INT:    printf("[정수]   %d\n", x.u.i);    break;
        case T_DOUBLE: printf("[실수]   %.2f\n", x.u.d);  break;
        case T_STRING: printf("[문자열] %s\n", x.u.s);    break;
    }
}

int main(void) {
    // 하나의 배열에 서로 다른 타입을 담을 수 있다 (표현력!)
    struct Variant box[3] = {
        make_int(42),
        make_double(3.14),
        make_string("hello")
    };
    for (int k = 0; k < 3; k++)
        print_variant(box[k]);
    return 0;
}
```

출력:
```
[정수]   42
[실수]   3.14
[문자열] hello
```

> **왜 이게 강력한가:** `box`는 한 종류의 타입 배열인데도 **정수·실수·문자열을 한꺼번에** 담는다. JSON 값, 인터프리터의 변수 값, 설정 파일 항목처럼 "타입이 정해지지 않은 값"을 다룰 때 쓰는 바로 그 패턴이다. 메모리 절약이 아니라 **표현력** 때문에 쓴다는 점에 주목.

**태그드 유니온 3대 규칙**
- 넣을 때 **값과 태그를 함께** 설정한다 (생성 함수로 강제하면 실수 방지).
- 꺼낼 때 **항상 태그부터 확인**하고 그 멤버만 읽는다 (`switch`가 정석).
- 태그와 실제 내용물이 어긋나면 → 비유 ③의 "라벨 잘못 붙은 택배" → 버그. 이 일관성은 **프로그래머가 책임**진다.

## 1-6. "요즘 union 안 쓰지 않나?" — 흔한 오해

union을 "메모리 부족하던 옛날 도구"로만 알면 "요즘 안 쓴다"가 맞는 말처럼 들린다. 하지만 **메모리 절약은 union의 여러 용도 중 하나일 뿐**이다. 용도가 바뀌었을 뿐 union은 건재하다.

**왜 앱·웹 개발에선 거의 안 보이나**
- 고수준 언어(JS·Python·Java)엔 union이 없거나 다른 형태로 추상화됨.
- 메모리가 흔해져 "공간 겹쳐쓰기" 목적의 실효성↓.
- 그래서 일반 응용 개발 현장에선 보기 어려운 게 맞다.

**그런데도 union이 지금 살아있는 영역 (메모리 절약과 무관한 이유)**
- **타입 재해석(type punning)**: 같은 비트를 int↔float↔byte로 보기 → 엔디언 변환, 직렬화, IEEE754 비트 조작. 그래픽스·네트워킹·코덱에서 상시 사용. (1-2 비유 ②, A5 문제가 이것)
- **태그드 유니온(합타입)**: "정수 또는 문자열 또는 …"을 한 변수에 → 인터프리터·컴파일러의 AST 노드, JSON 파서, 동적 타입 값(예: Lua의 `TValue`). 메모리가 아니라 **표현력** 때문에 쓴다. (1-5가 이것)
- **임베디드/MCU**: 하드웨어 레지스터를 비트필드 struct와 union으로 겹쳐 "통째 32비트로도, 비트별로도" 접근. 데이터시트 매핑의 정석. → 질문의 "극한 임베디드"가 정확히 여기.
- **OS·시스템 API**: 표준에도 박혀 있음 — `epoll_data`, `union sigval` 등.

**게임 엔진(Unreal/Unity) 맥락**
- 언리얼 엔진 소스에도 union이 등장(색상·벡터의 비트 접근, variant 류 패턴).
- 다만 C++에선 날(raw) union보다 **`std::variant`(타입 안전한 태그드 유니온)**로 대체되는 추세. 개념은 동일, 안전성만 강화.

> **결론:** "메모리 부족 시대용 도구"라는 인식은 낡았다. 오늘날 union은 **시스템·임베디드·언어 런타임** 계층에서 *비트 재해석*과 *합타입 표현* 용도로 쓰인다. 응용 레벨에서 안 보이는 건 자연스러운 일.
>
> ※ `std::variant`로의 대체 등 "추세" 서술은 일반적 기술 동향 기반이므로, 특정 코드베이스 적용 시엔 해당 프로젝트 컨벤션 확인이 필요함.

---

# 2부. 파일 I/O 스트림 — "음악 플레이어처럼"

## 2-1. 스트림(stream)이란 무엇인가

가장 큰 오해: "파일을 읽는다 = 파일 전체를 한 번에 통째로 가져온다". 실제로는 그렇지 않다.

- **스트림 = 음악 플레이어의 재생**
  - 곡 전체를 다 내려받은 뒤 트는 게 아니라, **재생되며 흘러나온다**. (유튜브·스포티파이의 "스트리밍"이 바로 이 단어)
  - 프로그램은 그 소리를 **앞에서부터 차례로** 듣는 청취자와 같다.

이 "재생되며 흐른다"는 추상화 덕분에, 파일이든 키보드든 네트워크든 **똑같은 방식(`fgetc`, `fputc`, `fread`…)**으로 다룰 수 있다. 무엇을 재생하든 "듣는 동작"은 똑같은 셈이다.

> 표준 입출력 `stdin`(키보드), `stdout`(화면), `stderr`(오류 화면)도 모두 스트림이다. C는 처음부터 세 개의 플레이어를 켜둔 채 프로그램을 시작한다.

## 2-2. 하나의 그림으로 보기 — "음악 플레이어"

여러 비유를 따로 외우면 헷갈린다. **모든 것을 하나의 음악 플레이어** 그림 안에 넣어 보자. 등장인물은 단 하나, "플레이어"이고 나머지는 그 부품일 뿐이다.

```
[디스크 = 저장된 음원]──버퍼링──플레이어(FILE*)──▶ 프로그램(스피커)
 │◀─────────────── 재생 바(타임라인) ───────────────▶│
                         ● 재생 위치 = 파일 위치 표시자
```

세 가지 부품을 이 그림 안에서 이해하면 된다.

**① 재생 = 스트림 (왜 필요한가: 추상화)**
- 곡 전체를 통째로 메모리에 올리지 않고, **재생하며 필요한 만큼** 흘려보낸다.
- 음원에 직접 접근하는 복잡한 일은 OS가 처리하고, 우리는 표준 플레이어(`FILE*`)만 다룬다.
- 그래서 파일·키보드·네트워크가 뭐든 **재생(읽기) 동작은 똑같다**.

**② 버퍼링 = 버퍼 (왜 필요한가: 효율)**
- 디스크는 느리다. 그래서 데이터를 조금씩 **미리 받아 모아두는데**, 이게 바로 누구나 아는 그 **"버퍼링"**이다.
- 쓰기도 마찬가지: `printf` 한 게 화면에 바로 안 보이면 → 아직 **버퍼에 모이는 중**.
- `fflush()` = "지금 당장 **버퍼에 모인 것을 내보내라**!"
- `fclose()` = 플레이어를 끄면서 버퍼에 남은 것도 마저 내보냄.

**③ 재생 위치 = 파일 위치 표시자 (지금 어디를 듣고 있나)**
- 타임라인 위에 **재생 위치(커서)**가 있다. 한 글자(한 음) 읽을 때마다 저절로 앞으로 간다.
- **음원은 사라지지 않으므로** 앞뒤로 자유롭게 옮길 수 있다 — 이게 물 비유로는 안 되던 부분.
- `ftell()` = 지금 **재생 위치(몇 초 지점)** 확인.
- `fseek()` = 타임라인의 **원하는 지점으로 점프**(드래그).
- `rewind()` = **맨 처음으로 되감기**(⏮).
- **EOF** = 곡의 끝, 더 들을 것이 없다는 신호 (End Of File).

> 즉 **재생(흐름) · 버퍼링(버퍼) · 재생 위치** 세 부품만 기억하면 된다. 물·택배·책갈피처럼 서로 다른 세계가 아니라, **한 플레이어의 세 부분**이다.

## 2-3. 어떻게 사용하는가 — 기본 4단계

플레이어 쓰는 순서와 똑같다: **① 곡 열기 → ② 재생/녹음 → ③ (확인) → ④ 닫기**

```c
#include <stdio.h>

int main(void) {
    // ① 열기: fopen(파일이름, 모드)
    FILE *fp = fopen("memo.txt", "w");   // "w" = 쓰기(녹음)
    if (fp == NULL) {                    // 곡이 안 열렸는지 반드시 확인!
        printf("파일 열기 실패\n");
        return 1;
    }

    // ② 녹음: 스트림으로 내보내기
    fprintf(fp, "안녕, 스트림!\n");
    fputc('A', fp);

    // ④ 닫기: 버퍼에 남은 것까지 저장하고 닫기
    fclose(fp);
    return 0;
}
```

**fopen 모드 = 재생/녹음 방식**

| 모드 | 비유 | 의미 |
|------|------|------|
| `"r"` | 재생 전용 | 읽기. 파일 없으면 실패 |
| `"w"` | 새 트랙 녹음 | 쓰기. 기존 내용 **싹 지우고** 처음부터, 없으면 새로 만듦 |
| `"a"` | 이어 녹음 | 추가(append). 끝에 이어 씀 |
| `"r+"` | 재생+녹음 | 읽기+쓰기 (파일 있어야 함) |
| `"rb"`,`"wb"` | 원본(raw) 모드 | 바이너리. 텍스트 변환 없이 그대로 |

**읽기 예시 — 한 음씩 재생하며 받기**

```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("memo.txt", "r");
    if (fp == NULL) return 1;

    int ch;
    // fgetc는 한 글자씩, 곡 끝(EOF)까지 재생하며 받는다
    while ((ch = fgetc(fp)) != EOF) {
        putchar(ch);          // 재생된 소리를 스피커(화면)로
    }

    fclose(fp);
    return 0;
}
```

> **왜 `int ch`인가?** `fgetc`는 정상 글자(0~255)뿐 아니라 "곡 끝 신호 EOF(보통 -1)"도 돌려준다. `char`로 받으면 EOF와 일반 글자를 구분 못 할 수 있어, 더 넓은 그릇인 `int`로 받는다. (곡이 끝났다는 신호를 담을 칸이 필요)

**자주 쓰는 함수 묶음**

| 함수 | 비유 | 용도 |
|------|------|------|
| `fgetc`/`fputc` | 음 하나 | 한 글자 |
| `fgets`/`fputs` | 한 소절 | 한 줄(문자열) |
| `fscanf`/`fprintf` | 양식 맞춰 듣기/녹음 | 서식 입출력 |
| `fread`/`fwrite` | 구간 통째 | 바이너리 블록 |
| `fseek`/`ftell`/`rewind` | 재생 위치 | 위치 이동/확인 |
| `fflush` | 버퍼 즉시 내보내기 | 버퍼 강제 출력 |
| `feof`/`ferror` | 곡 끝/오류 점검 | 상태 확인 |

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

**문제 B1 ★** — `"hello.txt"`에 `Hello, File!`이라는 한 줄을 쓰는 프로그램을 작성하라. fopen 모드는 무엇을 써야 하나? 그 이유를 "새 트랙 녹음 vs 이어 녹음" 비유로 답하라.

**문제 B2 ★** — 다음 코드에 **반드시 빠진 두 가지**가 있다. 무엇인가? (힌트: 안전 확인 하나, 마무리 하나)
```c
FILE *fp = fopen("data.txt", "r");
int ch;
while ((ch = fgetc(fp)) != EOF)
    putchar(ch);
```

**문제 B3 ★★** — `"score.txt"`에 학생 3명의 점수를 한 줄에 하나씩 쓰고(예: 90, 85, 77), 다시 파일을 열어 **합계와 평균**을 계산해 출력하는 프로그램을 작성하라. (`fprintf`/`fscanf` 사용)

**문제 B4 ★★** — `printf("로딩중...");` 다음에 5초간 다른 작업을 하는데, 화면에 `로딩중...`이 즉시 안 보이고 한참 뒤 나타났다. **왜 그런가**를 "버퍼링" 비유로 설명하고, **한 줄을 추가**해 즉시 보이게 고쳐라.

**문제 B5 ★★** — 파일을 한 줄씩 읽어 **줄 번호를 붙여** 화면에 출력하는 프로그램을 작성하라. (`fgets` 사용, 예: `1: 첫째 줄`)

**문제 B6 ★★★** — `fseek`, `ftell`을 사용해, 어떤 텍스트 파일의 **크기(바이트 수)**를 한 글자도 읽지 않고 구하는 함수 `long file_size(const char *name)`를 작성하라. (힌트: 재생 위치를 맨 끝으로 보낸 뒤 그 위치를 읽기)

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

**B1.** 모드 `"w"`. 새 메모를 쓰는 것이므로 기존 내용을 지우고 처음부터 새로 녹음하는 `"w"`가 적절하다(`"a"`는 기존 내용 끝에 이어 녹음).
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

**B2.** ① `fopen` 결과가 `NULL`인지 **여는 데 성공했는지 확인**이 빠졌다(곡이 안 열렸을 수 있음). ② `fclose(fp)`로 **닫기**가 빠졌다(플레이어 안 닫음 → 자원 누수). 고친 코드:
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

**B4.** `printf`는 출력을 **버퍼에 모아두고(버퍼링)**, 줄바꿈(`\n`)이 없으면 바로 내보내지 않는다. 그래서 화면에 즉시 안 나타난다. `fflush(stdout);`을 추가하면 "지금 당장 버퍼를 내보내라" → 즉시 표시된다. (또는 문자열 끝에 `\n` 추가)
```c
printf("로딩중...");
fflush(stdout);   // 추가: 버퍼를 즉시 내보내기
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
    fseek(fp, 0, SEEK_END);   // 재생 위치를 맨 끝으로
    long size = ftell(fp);    // 끝의 위치 = 전체 바이트 수
    fclose(fp);
    return size;
}
int main(void) {
    printf("크기: %ld 바이트\n", file_size("hello.txt"));
    return 0;
}
```
`SEEK_END`로 재생 위치를 곡 끝까지 보낸 뒤 `ftell`로 그 위치(=바이트 수)를 읽는다. 한 글자도 실제로 읽지 않는다.

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
바이너리 모드를 쓰는 이유: 구조체에는 사람이 읽는 글자가 아닌 **원시 바이트(int 4바이트 등)**가 들어 있다. 텍스트 모드는 줄바꿈 문자 변환 등을 하므로 바이트가 **오염**될 수 있다. 원본을 가공 없이 그대로 저장·재생해야 하므로 `"wb"`/`"rb"`를 쓴다.

---

# 5부. 한 장 요약 (복습 카드)

**union**
- struct = "그리고(서랍장)", union = "또는(다용도 방)"
- 크기 = 가장 큰 멤버. 모든 멤버가 같은 주소를 공유.
- 마지막에 쓴 멤버만 유효 → 안전하게 쓰려면 **태그(라벨)**를 함께.
- 쓰임새: (옛날) 메모리 절약 / (지금) 같은 비트를 여러 시각으로 보기(엔디언·패킷 파싱), 태그드 유니온으로 "여러 타입을 한 변수에" → 인터프리터·JSON·임베디드 레지스터.

**파일 I/O 스트림 — "음악 플레이어" 하나로 기억**
- **재생(stream)**: 데이터는 통째가 아니라 **재생되며 흐른다**. `fopen`(곡 열기) → 읽기/쓰기 → `fclose`(닫기). 열고 나면 **NULL 검사** 필수.
- **버퍼링(buffer)**: 효율 위해 미리 모았다 내보냄. `fflush`로 즉시 내보내기, `fclose`가 마저 내보냄.
- **재생 위치(위치 표시자)**: `fseek`(점프)/`ftell`(현재 위치)/`rewind`(처음으로), 곡 끝 신호는 `EOF`.
- 텍스트는 사람용, 구조체 등 원시 데이터는 **바이너리 모드(`"wb"`/`"rb"`)**.
