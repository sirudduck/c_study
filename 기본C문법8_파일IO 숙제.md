# Session 09 - 숙제: 파일 입출력 + 구조체

---

## 문제 1 ⭐

아래 구조체를 정의하고, 변수 1개에 값을 직접 대입한 뒤  
`book.txt`에 저장하라. 저장 후 파일을 다시 열어 읽어서 화면에 출력하라.

```c
typedef struct {
    char  title[40];   // 책 제목
    char  author[20];  // 저자
    int   year;        // 출판 연도
} Book;
```

**예상 출력:**
```
제목: The C Programming Language
저자: Kernighan
연도: 1988
```

> 힌트: `fprintf`로 저장할 때와 `fscanf`로 읽을 때 형식 문자열을 동일하게 맞춰야 한다.

**점검 포인트:**
- `fopen` 직후 `NULL` 체크를 했는가
- 저장과 읽기에 각각 `fclose`를 호출했는가

---

## 문제 2 ⭐⭐

아래 구조체 배열을 선언하고 초기값을 코드 안에 작성하라.  
배열 전체를 `weather.txt`에 저장한 뒤, 파일을 다시 열어 읽어서 출력하라.

```c
typedef struct {
    char  city[20];    // 도시명
    float temp;        // 기온 (°C)
    int   humidity;    // 습도 (%)
} Weather;
```

**초기 데이터:**

| 도시    | 기온  | 습도 |
|---------|-------|------|
| Seoul   | -2.5  | 40   |
| Busan   |  5.3  | 60   |
| Jeju    |  8.1  | 75   |

**예상 출력:**
```
Seoul -2.5 40
Busan 5.3 60
Jeju 8.1 75
```

> 힌트: 저장 시 각 구조체를 한 줄로 쓰면 `fscanf`로 읽기 쉽다.

**점검 포인트:**
- `for` 반복문으로 배열 전체를 저장/읽기했는가
- 읽기 시 루프 종료 조건으로 `fscanf`의 반환값을 활용했는가

---

## 문제 3 ⭐⭐⭐

문제 2에서 만든 `weather.txt`에 새 도시 데이터 1건을 **추가**하라.  
추가 후 파일 전체를 다시 읽어 모든 도시의 **평균 기온**을 계산하여 출력하라.

추가할 데이터: `Incheon`, 기온 `-3.2`, 습도 `45`

**예상 출력:**
```
Seoul -2.5 40
Busan 5.3 60
Jeju 8.1 75
Incheon -3.2 45

평균 기온: 1.93°C
```

> 힌트: 기존 파일 내용을 지우지 않고 추가하려면 어떤 `fopen` 모드를 써야 할까?

**점검 포인트:**
- `"a"` 모드(append)로 파일을 열었는가
- 평균 계산 함수에 `Weather *arr`과 `int n`을 포인터로 전달했는가

---

## 문제 4 ⭐⭐⭐⭐

아래 구조체를 이진 방식(`fwrite` / `fread`)으로 파일에 저장하고 다시 읽어 출력하라.  
파일 이름은 `scores.bin`으로 한다.

```c
typedef struct {
    char  name[20];   // 선수 이름
    int   rank;       // 순위
    float score;      // 점수
} Player;
```

**초기 데이터:**

| 이름    | 순위 | 점수  |
|---------|------|-------|
| Alice   | 1    | 98.5  |
| Bob     | 2    | 95.2  |
| Carol   | 3    | 91.0  |

저장 직전과 읽기 직후에 아래 두 값을 각각 출력하라.
- `sizeof(Player)` — 구조체 1개의 크기
- 실제로 저장된 데이터의 총 예상 크기 (`sizeof(Player) * 항목 수`)

**예상 출력 (일부):**
```
sizeof(Player) = 28 bytes
예상 파일 크기 = 84 bytes

[1위] Alice - 98.5점
[2위] Bob - 95.2점
[3위] Carol - 91.0점
```

> 힌트: `fwrite(arr, sizeof(Player), 3, fp)` 한 번으로 배열 전체를 저장할 수 있다.

**점검 포인트:**
- `fopen` 모드에 `"b"`(binary)가 포함되었는가 (`"wb"`, `"rb"`)
- `fread`의 반환값(실제로 읽은 항목 수)을 출력 루프의 상한으로 사용했는가

---

## 문제 5 ⭐⭐⭐⭐⭐

아래 구조체와 함수 원형을 활용하여 프로그램을 완성하라.

```c
typedef struct {
    char  name[20];   // 상품명
    int   quantity;   // 재고 수량
    float price;      // 단가 (원)
} Item;

// 구조체 배열을 텍스트 파일로 저장
void save_items(const char *filename, Item *arr, int n);

// 텍스트 파일을 읽어 구조체 배열에 저장, 읽은 항목 수 반환
int load_items(const char *filename, Item *arr, int max);

// 단가가 가장 높은 항목의 포인터 반환
Item *find_most_expensive(Item *arr, int n);

// 전체 재고 금액 계산 (단가 × 수량의 합계)
float calc_total(Item *arr, int n);
```

**요구사항:**
1. 아래 초기 데이터를 `inventory.txt`에 저장한다.

| 상품명   | 수량 | 단가    |
|----------|------|---------|
| Keyboard | 10   | 55000.0 |
| Mouse    | 25   | 32000.0 |
| Monitor  | 5    | 320000.0|

2. `inventory.txt`를 다시 읽어 구조체 배열에 불러온다.
3. `find_most_expensive`로 가장 비싼 상품을 출력한다.
4. `calc_total`로 전체 재고 금액을 출력한다.

**예상 출력:**
```
=== 재고 목록 ===
Keyboard  | 수량: 10 | 단가: 55000원
Mouse     | 수량: 25 | 단가: 32000원
Monitor   | 수량:  5 | 단가: 320000원

가장 비싼 상품: Monitor (320000원)
전체 재고 금액: 2,950,000원
```

> 힌트: `calc_total`은 `Item *arr`을 받아 `quantity * price`를 누적한다.  
> `find_most_expensive`는 포인터를 반환하므로 `main`에서 `->` 연산자로 접근한다.

**점검 포인트:**
- 4개 함수가 모두 구현되었는가
- `find_most_expensive`가 `Item *`를 반환하고, `main`에서 `->` 로 멤버에 접근하는가
- `load_items`에서 `fscanf`의 반환값으로 루프 종료 조건을 처리했는가
- `fopen` 후 `NULL` 체크 및 `fclose` 호출이 모든 파일 작업에 포함되었는가

---

*참고: ISO/IEC 9899:2011 (C11) §6.7.2.1 Structure and union specifiers, §7.21 Input/output \<stdio.h\>*
