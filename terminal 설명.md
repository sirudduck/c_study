# C 프로그래밍 학습을 위한 터미널 명령어 레퍼런스

## 1. 파일 시스템 탐색 명령어

### 1.1 디렉토리 이동 및 확인

#### `pwd` (Print Working Directory)
현재 작업 디렉토리의 절대 경로를 출력합니다.

```bash
$ pwd
/home/username/projects
```

**옵션:**
- `-L`: 심볼릭 링크를 따라가서 표시 (기본값)
- `-P`: 실제 물리적 경로 표시

**사용 예:**
```bash
$ pwd          # 현재 위치 확인
$ pwd -P       # 심볼릭 링크의 실제 경로 확인
```

#### `cd` (Change Directory)
디렉토리를 변경합니다.

```bash
$ cd /home/username/projects    # 절대 경로로 이동
$ cd projects                   # 상대 경로로 이동
$ cd ..                         # 상위 디렉토리로 이동
$ cd ~                          # 홈 디렉토리로 이동
$ cd -                          # 이전 디렉토리로 이동
```

**특수 디렉토리 기호:**
- `.`: 현재 디렉토리
- `..`: 상위 디렉토리
- `~`: 홈 디렉토리 (`/home/username`)
- `-`: 이전 디렉토리

#### `ls` (List)
디렉토리 내용을 나열합니다.

```bash
$ ls                    # 현재 디렉토리 파일 목록
$ ls -l                 # 상세 정보 포함 (long format)
$ ls -a                 # 숨김 파일 포함 (all)
$ ls -la                # 상세 정보 + 숨김 파일
$ ls -lh                # 파일 크기를 읽기 쉽게 (human-readable)
$ ls -lt                # 수정 시간 순 정렬
$ ls -ltr               # 수정 시간 역순 정렬
$ ls *.c                # .c 파일만 나열
```

**주요 옵션:**
- `-l`: 상세 정보 (권한, 소유자, 크기, 날짜)
- `-a`: 숨김 파일 포함 (`.`로 시작하는 파일)
- `-h`: 사람이 읽기 쉬운 크기 표시 (KB, MB)
- `-t`: 수정 시간 순 정렬
- `-r`: 역순 정렬
- `-R`: 재귀적으로 하위 디렉토리까지 표시

**출력 예:**
```bash
$ ls -lh
total 24K
-rw-r--r-- 1 user group  156 Jan  4 10:30 hello.c
-rwxr-xr-x 1 user group  16K Jan  4 10:31 hello
drwxr-xr-x 2 user group 4.0K Jan  4 10:00 src
```

### 1.2 디렉토리 생성 및 삭제

#### `mkdir` (Make Directory)
새 디렉토리를 생성합니다.

```bash
$ mkdir projects                # 디렉토리 생성
$ mkdir -p projects/c/week1     # 중간 경로 자동 생성
$ mkdir dir1 dir2 dir3          # 여러 디렉토리 동시 생성
```

**주요 옵션:**
- `-p`: 중간 경로가 없으면 자동 생성 (parents)
- `-m`: 권한 지정 (예: `-m 755`)

#### `rmdir` (Remove Directory)
빈 디렉토리를 삭제합니다.

```bash
$ rmdir empty_directory         # 빈 디렉토리만 삭제
$ rmdir -p a/b/c               # 빈 디렉토리 경로 전체 삭제
```

**주의:** `rmdir`은 빈 디렉토리만 삭제 가능합니다.

### 1.3 파일 생성 및 삭제

#### `touch`
빈 파일을 생성하거나 파일의 타임스탬프를 갱신합니다.

```bash
$ touch file.txt               # 빈 파일 생성
$ touch file1.c file2.c        # 여러 파일 동시 생성
$ touch existing_file.c        # 기존 파일의 수정 시간 갱신
```

#### `rm` (Remove)
파일이나 디렉토리를 삭제합니다.

```bash
$ rm file.txt                  # 파일 삭제
$ rm file1.c file2.c           # 여러 파일 삭제
$ rm *.o                       # 모든 .o 파일 삭제
$ rm -r directory              # 디렉토리와 내용물 삭제 (recursive)
$ rm -rf directory             # 강제 삭제 (force)
$ rm -i file.c                 # 삭제 전 확인 (interactive)
```

**주요 옵션:**
- `-r` 또는 `-R`: 디렉토리 재귀 삭제
- `-f`: 강제 삭제 (확인 없이)
- `-i`: 삭제 전 확인

**⚠️ 주의:** `rm -rf`는 복구 불가능하므로 신중하게 사용!

### 1.4 파일 복사 및 이동

#### `cp` (Copy)
파일이나 디렉토리를 복사합니다.

```bash
$ cp source.c destination.c         # 파일 복사
$ cp source.c backup/               # 디렉토리로 복사
$ cp *.c backup/                    # 여러 파일 복사
$ cp -r src_dir dest_dir           # 디렉토리 전체 복사
$ cp -i source.c dest.c            # 덮어쓰기 전 확인
$ cp -u source.c dest.c            # 최신 파일만 복사 (update)
```

**주요 옵션:**
- `-r` 또는 `-R`: 디렉토리 재귀 복사
- `-i`: 덮어쓰기 전 확인
- `-u`: 소스가 대상보다 최신이거나 대상이 없을 때만 복사
- `-v`: 복사 과정 표시 (verbose)

#### `mv` (Move)
파일이나 디렉토리를 이동하거나 이름을 변경합니다.

```bash
$ mv old_name.c new_name.c         # 파일 이름 변경
$ mv file.c directory/             # 파일 이동
$ mv *.c src/                      # 여러 파일 이동
$ mv old_dir new_dir               # 디렉토리 이름 변경
$ mv -i source.c dest.c            # 덮어쓰기 전 확인
```

**주요 옵션:**
- `-i`: 덮어쓰기 전 확인
- `-f`: 강제 이동 (확인 없이)
- `-u`: 소스가 대상보다 최신일 때만 이동

## 2. 파일 내용 확인 명령어

### 2.1 텍스트 파일 보기

#### `cat` (Concatenate)
파일 내용을 출력합니다.

```bash
$ cat file.c                       # 파일 내용 출력
$ cat file1.c file2.c              # 여러 파일 연결 출력
$ cat file1.c file2.c > merged.c   # 여러 파일 병합
$ cat -n file.c                    # 줄 번호 포함 출력
```

**주요 옵션:**
- `-n`: 줄 번호 표시
- `-b`: 빈 줄 제외하고 줄 번호 표시
- `-A`: 모든 특수 문자 표시

#### `less`
파일을 페이지 단위로 봅니다 (스크롤 가능).

```bash
$ less file.c                      # 파일 보기
$ less +F file.log                 # 실시간 추적 (tail -f와 유사)
```

**less 내부 명령:**
- `Space` 또는 `f`: 다음 페이지
- `b`: 이전 페이지
- `/pattern`: 문자열 검색
- `n`: 다음 검색 결과
- `N`: 이전 검색 결과
- `q`: 종료
- `G`: 파일 끝으로 이동
- `g`: 파일 처음으로 이동

#### `head`
파일의 첫 부분을 출력합니다 (기본 10줄).

```bash
$ head file.c                      # 첫 10줄 출력
$ head -n 20 file.c               # 첫 20줄 출력
$ head -n 5 *.c                   # 각 파일의 첫 5줄
```

#### `tail`
파일의 끝 부분을 출력합니다 (기본 10줄).

```bash
$ tail file.c                      # 마지막 10줄 출력
$ tail -n 20 file.c               # 마지막 20줄 출력
$ tail -f output.log              # 실시간 추적 (로그 모니터링)
```

**주요 옵션:**
- `-n N`: N줄 출력
- `-f`: 파일 끝을 실시간으로 추적 (follow)

### 2.2 파일 검색

#### `grep` (Global Regular Expression Print)
파일에서 패턴을 검색합니다.

```bash
$ grep "main" file.c               # "main" 문자열 검색
$ grep -n "printf" file.c          # 줄 번호 포함
$ grep -i "MAIN" file.c            # 대소문자 무시
$ grep -r "include" .              # 현재 디렉토리에서 재귀 검색
$ grep -v "comment" file.c         # 패턴이 없는 줄 출력
$ grep "^int" file.c               # 'int'로 시작하는 줄
$ grep "return$" file.c            # 'return'으로 끝나는 줄
$ grep -c "printf" file.c          # 매칭된 줄 수 출력
```

**주요 옵션:**
- `-n`: 줄 번호 표시
- `-i`: 대소문자 무시
- `-r` 또는 `-R`: 재귀 검색
- `-v`: 패턴이 없는 줄 출력 (invert)
- `-c`: 매칭된 줄 수만 출력
- `-l`: 매칭된 파일 이름만 출력
- `-w`: 단어 단위로 매칭

**정규 표현식:**
- `^`: 줄의 시작
- `$`: 줄의 끝
- `.`: 임의의 한 문자
- `*`: 0번 이상 반복
- `[abc]`: a, b, c 중 하나

#### `find`
파일 시스템에서 파일을 검색합니다.

```bash
$ find . -name "*.c"               # .c 파일 찾기
$ find . -name "main.c"            # 특정 파일 찾기
$ find . -type f -name "*.c"       # 파일만 검색 (-type f)
$ find . -type d -name "src"       # 디렉토리만 검색 (-type d)
$ find . -name "*.c" -mtime -7     # 7일 내 수정된 .c 파일
$ find . -name "*.o" -delete       # .o 파일 찾아서 삭제
$ find . -name "*.c" -exec grep "main" {} \;  # 찾은 파일에서 "main" 검색
```

**주요 옵션:**
- `-name`: 파일 이름으로 검색
- `-type f`: 파일만
- `-type d`: 디렉토리만
- `-mtime -N`: N일 내 수정된 파일
- `-size +Nk`: N킬로바이트보다 큰 파일
- `-exec command {} \;`: 찾은 파일에 명령 실행

## 3. 텍스트 편집기

### 3.1 nano
간단하고 초보자 친화적인 텍스트 편집기입니다.

```bash
$ nano file.c                      # 파일 편집
$ nano                             # 새 파일 생성
```

**주요 단축키:**
- `Ctrl + O`: 저장 (Write Out)
- `Ctrl + X`: 종료
- `Ctrl + K`: 줄 잘라내기
- `Ctrl + U`: 붙여넣기
- `Ctrl + W`: 검색
- `Ctrl + G`: 도움말

### 3.2 vim
강력한 텍스트 편집기 (학습 곡선이 있음).

```bash
$ vim file.c                       # 파일 편집
$ vim +10 file.c                   # 10번째 줄에서 시작
```

**기본 사용법:**
```
# 명령 모드 (기본)
i          # 입력 모드로 전환 (Insert)
Esc        # 명령 모드로 복귀
:w         # 저장 (Write)
:q         # 종료 (Quit)
:wq        # 저장 후 종료
:q!        # 저장 없이 강제 종료
dd         # 현재 줄 삭제
yy         # 현재 줄 복사
p          # 붙여넣기
/pattern   # 검색
```

### 3.3 VS Code (터미널에서 실행)

```bash
$ code file.c                      # VS Code로 파일 열기
$ code .                           # 현재 디렉토리를 프로젝트로 열기
$ code -n .                        # 새 창에서 열기
```

## 4. C 컴파일 및 실행 명령어

### 4.1 GCC (GNU Compiler Collection)

#### 기본 컴파일
```bash
$ gcc hello.c                      # a.out 생성
$ gcc hello.c -o hello            # hello 실행 파일 생성
$ gcc -c file.c                    # file.o 목적 파일만 생성 (링크 안 함)
```

#### 경고 및 디버깅 옵션
```bash
$ gcc -Wall hello.c -o hello       # 모든 경고 표시
$ gcc -Wextra hello.c -o hello     # 추가 경고 표시
$ gcc -Werror hello.c -o hello     # 경고를 에러로 처리
$ gcc -g hello.c -o hello          # 디버깅 정보 포함
$ gcc -g3 hello.c -o hello         # 최대 디버깅 정보
```

#### 최적화 옵션
```bash
$ gcc -O0 hello.c -o hello         # 최적화 없음 (기본값)
$ gcc -O1 hello.c -o hello         # 기본 최적화
$ gcc -O2 hello.c -o hello         # 추천 최적화
$ gcc -O3 hello.c -o hello         # 공격적 최적화
$ gcc -Os hello.c -o hello         # 크기 최적화
```

#### C 표준 지정
```bash
$ gcc -std=c89 hello.c -o hello    # C89/C90 표준
$ gcc -std=c99 hello.c -o hello    # C99 표준
$ gcc -std=c11 hello.c -o hello    # C11 표준
$ gcc -std=c17 hello.c -o hello    # C17 표준 (최신)
```

#### 여러 파일 컴파일
```bash
$ gcc main.c utils.c -o program    # 여러 소스 파일 컴파일
$ gcc *.c -o program               # 모든 .c 파일 컴파일
$ gcc main.c -c                    # main.o 생성
$ gcc utils.c -c                   # utils.o 생성
$ gcc main.o utils.o -o program    # 목적 파일 링크
```

#### 라이브러리 링크
```bash
$ gcc main.c -o program -lm        # 수학 라이브러리 링크
$ gcc main.c -o program -lpthread  # pthread 라이브러리
$ gcc main.c -L/path/to/lib -lmylib  # 커스텀 라이브러리
```

#### 헤더 파일 경로 지정
```bash
$ gcc -I./include main.c -o program  # include 디렉토리 추가
$ gcc -I./inc1 -I./inc2 main.c -o program  # 여러 경로
```

#### 전처리만 수행
```bash
$ gcc -E hello.c                   # 전처리 결과 출력
$ gcc -E hello.c > hello.i         # 전처리 결과를 파일로 저장
```

#### 어셈블리 코드 생성
```bash
$ gcc -S hello.c                   # hello.s 어셈블리 파일 생성
$ gcc -S -masm=intel hello.c       # Intel 형식 어셈블리
```

**GCC 주요 옵션 요약:**
- `-o <file>`: 출력 파일 이름 지정
- `-c`: 컴파일만 (링크 안 함)
- `-Wall`: 모든 경고
- `-Wextra`: 추가 경고
- `-Werror`: 경고를 에러로
- `-g`: 디버깅 정보 포함
- `-O0`, `-O1`, `-O2`, `-O3`: 최적화 수준
- `-std=<표준>`: C 표준 지정
- `-I<dir>`: 헤더 파일 경로
- `-L<dir>`: 라이브러리 경로
- `-l<lib>`: 라이브러리 링크
- `-E`: 전처리만
- `-S`: 어셈블리 생성

### 4.2 프로그램 실행

```bash
$ ./hello                          # 실행 파일 실행
$ ./program arg1 arg2              # 인자와 함께 실행
$ ./program < input.txt            # 표준 입력 리다이렉션
$ ./program > output.txt           # 표준 출력 리다이렉션
$ ./program 2> error.txt           # 표준 에러 리다이렉션
$ ./program < input.txt > output.txt 2> error.txt  # 모두 리다이렉션
```

### 4.3 make
빌드 자동화 도구입니다.

```bash
$ make                             # 기본 타겟 빌드 (Makefile 사용)
$ make clean                       # clean 타겟 실행
$ make program                     # 특정 타겟 빌드
$ make -f my_makefile             # 다른 Makefile 사용
```

**간단한 Makefile 예:**
```makefile
# Makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11

program: main.c utils.c
	$(CC) $(CFLAGS) main.c utils.c -o program

clean:
	rm -f program *.o
```

사용:
```bash
$ make          # program 빌드
$ make clean    # 생성된 파일 삭제
```

## 5. 디버깅 도구

### 5.1 GDB (GNU Debugger)

```bash
$ gcc -g program.c -o program      # 디버깅 정보 포함 컴파일
$ gdb program                       # GDB 시작
```

**GDB 기본 명령:**
```gdb
(gdb) run                          # 프로그램 실행
(gdb) run arg1 arg2                # 인자와 함께 실행
(gdb) break main                   # main 함수에 중단점 설정
(gdb) break 10                     # 10번째 줄에 중단점
(gdb) list                         # 소스 코드 보기
(gdb) next                         # 다음 줄 실행 (함수 건너뜀)
(gdb) step                         # 다음 줄 실행 (함수 진입)
(gdb) continue                     # 계속 실행
(gdb) print variable               # 변수 값 출력
(gdb) info locals                  # 지역 변수 출력
(gdb) backtrace                    # 호출 스택 출력
(gdb) quit                         # GDB 종료
```

### 5.2 Valgrind
메모리 누수 및 오류 검사 도구입니다.

```bash
$ gcc -g program.c -o program      # 디버깅 정보 포함
$ valgrind ./program               # 메모리 검사 실행
$ valgrind --leak-check=full ./program  # 상세 메모리 누수 검사
$ valgrind --track-origins=yes ./program  # 초기화되지 않은 값 추적
```

## 6. 파일 권한 및 정보

### 6.1 파일 권한

#### `chmod` (Change Mode)
파일 권한을 변경합니다.

```bash
$ chmod +x program                 # 실행 권한 추가
$ chmod -x program                 # 실행 권한 제거
$ chmod 755 program                # rwxr-xr-x
$ chmod 644 file.c                 # rw-r--r--
$ chmod u+x program                # 소유자에게 실행 권한
$ chmod go-w file.c                # 그룹, 기타에서 쓰기 권한 제거
```

**권한 표기법:**
```
숫자 표기:
7 = rwx (읽기 + 쓰기 + 실행)
6 = rw- (읽기 + 쓰기)
5 = r-x (읽기 + 실행)
4 = r-- (읽기)

기호 표기:
u = user (소유자)
g = group (그룹)
o = others (기타)
a = all (모두)

+ = 권한 추가
- = 권한 제거
= = 권한 설정
```

### 6.2 파일 정보

#### `file`
파일 타입을 확인합니다.

```bash
$ file program                     # 실행 파일 정보
$ file hello.c                     # 텍스트 파일 정보
$ file image.png                   # 이미지 파일 정보
```

출력 예:
```
program: ELF 64-bit LSB executable
hello.c: C source, ASCII text
image.png: PNG image data, 800 x 600
```

#### `wc` (Word Count)
줄, 단어, 문자 수를 셉니다.

```bash
$ wc file.c                        # 줄, 단어, 바이트 수
$ wc -l file.c                     # 줄 수만
$ wc -w file.c                     # 단어 수만
$ wc -c file.c                     # 바이트 수만
$ wc -L file.c                     # 가장 긴 줄의 길이
```

#### `du` (Disk Usage)
디스크 사용량을 확인합니다.

```bash
$ du -h file.c                     # 사람이 읽기 쉽게
$ du -sh directory                 # 디렉토리 전체 크기
$ du -ah directory                 # 모든 파일 크기
```

#### `df` (Disk Free)
디스크 여유 공간을 확인합니다.

```bash
$ df -h                            # 사람이 읽기 쉽게
$ df -h .                          # 현재 파일 시스템
```

## 7. 프로세스 관리

### 7.1 프로세스 확인

#### `ps` (Process Status)
실행 중인 프로세스를 표시합니다.

```bash
$ ps                               # 현재 터미널의 프로세스
$ ps aux                           # 모든 프로세스 상세 정보
$ ps aux | grep gcc                # gcc 프로세스만
$ ps -ef                           # 모든 프로세스 (다른 형식)
```

#### `top`
실시간 프로세스 모니터링입니다.

```bash
$ top                              # CPU, 메모리 사용률 실시간 표시
```

**top 내부 명령:**
- `q`: 종료
- `k`: 프로세스 종료 (PID 입력)
- `P`: CPU 사용률 순 정렬
- `M`: 메모리 사용률 순 정렬

### 7.2 프로세스 제어

#### `kill`
프로세스를 종료합니다.

```bash
$ kill 1234                        # PID 1234 프로세스에 SIGTERM
$ kill -9 1234                     # 강제 종료 (SIGKILL)
$ killall program_name             # 이름으로 프로세스 종료
```

**주요 시그널:**
- `SIGTERM (15)`: 정상 종료 요청 (기본값)
- `SIGKILL (9)`: 강제 종료
- `SIGINT (2)`: 인터럽트 (Ctrl+C)

#### 백그라운드 실행
```bash
$ ./long_program &                 # 백그라운드 실행
$ jobs                             # 백그라운드 작업 목록
$ fg %1                            # 작업 1을 포어그라운드로
$ bg %1                            # 작업 1을 백그라운드로
$ nohup ./program &                # 로그아웃 후에도 실행
```

## 8. 입출력 리다이렉션 및 파이프

### 8.1 리다이렉션

```bash
# 출력 리다이렉션
$ ./program > output.txt           # stdout을 파일로 (덮어쓰기)
$ ./program >> output.txt          # stdout을 파일로 (추가)
$ ./program 2> error.txt           # stderr을 파일로
$ ./program &> all.txt             # stdout과 stderr 모두
$ ./program > output.txt 2>&1      # stdout과 stderr를 같은 파일로

# 입력 리다이렉션
$ ./program < input.txt            # stdin을 파일에서

# 조합
$ ./program < input.txt > output.txt 2> error.txt
```

**파일 디스크립터:**
- `0`: 표준 입력 (stdin)
- `1`: 표준 출력 (stdout)
- `2`: 표준 에러 (stderr)

### 8.2 파이프 (Pipeline)

```bash
$ cat file.c | grep "main"         # cat 출력을 grep 입력으로
$ ls -l | wc -l                    # 파일 개수 세기
$ gcc program.c 2>&1 | tee compile.log  # 컴파일 에러를 화면과 파일로
$ cat *.c | grep -c "printf"       # 모든 .c 파일에서 printf 횟수
```

#### `tee`
출력을 화면과 파일 모두에 보냅니다.

```bash
$ ./program | tee output.txt       # 화면과 파일 모두 출력
$ ./program | tee -a output.txt    # 파일에 추가
```

## 9. 압축 및 아카이브

### 9.1 tar (Tape Archive)

```bash
# 압축 생성
$ tar -czf project.tar.gz project/     # gzip 압축
$ tar -cjf project.tar.bz2 project/    # bzip2 압축
$ tar -czvf project.tar.gz *.c *.h     # 특정 파일만

# 압축 해제
$ tar -xzf project.tar.gz              # gzip 압축 해제
$ tar -xjf project.tar.bz2             # bzip2 압축 해제
$ tar -xzvf project.tar.gz -C /dest    # 특정 디렉토리에

# 내용 보기
$ tar -tzf project.tar.gz              # 압축 파일 내용 보기
```

**주요 옵션:**
- `-c`: 생성 (create)
- `-x`: 추출 (extract)
- `-t`: 목록 보기 (list)
- `-z`: gzip 압축/해제
- `-j`: bzip2 압축/해제
- `-v`: 상세 출력 (verbose)
- `-f`: 파일 이름 지정

### 9.2 gzip / gunzip

```bash
$ gzip file.c                      # file.c.gz 생성 (원본 삭제)
$ gzip -k file.c                   # 원본 유지
$ gunzip file.c.gz                 # 압축 해제
$ gzip -d file.c.gz                # 압축 해제 (동일)
```

## 10. 시스템 정보

### 10.1 시스템 확인

```bash
$ uname -a                         # 시스템 전체 정보
$ uname -s                         # 운영체제 이름
$ uname -r                         # 커널 버전
$ uname -m                         # 아키텍처 (x86_64 등)

$ date                             # 현재 날짜 및 시간
$ uptime                           # 시스템 가동 시간
$ whoami                           # 현재 사용자 이름
$ hostname                         # 호스트 이름

$ which gcc                        # gcc 실행 파일 위치
$ whereis gcc                      # gcc 관련 파일 위치
```

### 10.2 환경 변수

```bash
$ echo $PATH                       # PATH 환경 변수 출력
$ echo $HOME                       # 홈 디렉토리
$ echo $USER                       # 사용자 이름
$ echo $SHELL                      # 현재 셸

$ env                              # 모든 환경 변수
$ export VAR=value                 # 환경 변수 설정
$ export PATH=$PATH:/new/path      # PATH에 경로 추가
```

## 11. 도움말

### 11.1 man (Manual)
명령어 매뉴얼을 봅니다.

```bash
$ man gcc                          # gcc 매뉴얼
$ man ls                           # ls 매뉴얼
$ man 3 printf                     # C 라이브러리 함수 printf
$ man -k keyword                   # 키워드로 검색
```

**man 페이지 섹션:**
- 1: 사용자 명령
- 2: 시스템 콜
- 3: C 라이브러리 함수
- 7: 기타

### 11.2 기타 도움말

```bash
$ gcc --help                       # gcc 도움말
$ command --help                   # 명령어 도움말
$ info gcc                         # info 문서 (상세)
```

## 12. 기타 유용한 명령어

### 12.1 diff
파일 차이를 비교합니다.

```bash
$ diff file1.c file2.c             # 차이점 출력
$ diff -u file1.c file2.c          # unified 형식
$ diff -y file1.c file2.c          # 나란히 비교
```

### 12.2 sort
텍스트를 정렬합니다.

```bash
$ sort file.txt                    # 알파벳순 정렬
$ sort -n numbers.txt              # 숫자순 정렬
$ sort -r file.txt                 # 역순 정렬
$ sort -u file.txt                 # 중복 제거 정렬
```

### 12.3 uniq
중복된 줄을 제거합니다.

```bash
$ sort file.txt | uniq             # 중복 제거
$ sort file.txt | uniq -c          # 중복 횟수 포함
$ sort file.txt | uniq -d          # 중복된 줄만
```

### 12.4 echo
텍스트를 출력합니다.

```bash
$ echo "Hello, World!"             # 문자열 출력
$ echo $PATH                       # 변수 값 출력
$ echo "text" > file.txt           # 파일로 출력
$ echo "more text" >> file.txt     # 파일에 추가
```

### 12.5 history
명령어 히스토리를 봅니다.

```bash
$ history                          # 명령어 히스토리
$ history 10                       # 최근 10개
$ !100                             # 100번 명령 재실행
$ !!                               # 직전 명령 재실행
$ !gcc                             # gcc로 시작하는 최근 명령
```

### 12.6 clear
터미널 화면을 지웁니다.

```bash
$ clear                            # 화면 지우기
```
또는 `Ctrl + L` 단축키 사용

### 12.7 alias
명령어 별칭을 만듭니다.

```bash
$ alias ll='ls -la'                # ll 별칭 생성
$ alias compile='gcc -Wall -Wextra -std=c11'
$ unalias ll                       # 별칭 제거
```

## 13. C 프로그래밍 워크플로우 예제

### 13.1 기본 워크플로우

```bash
# 1. 프로젝트 디렉토리 생성
$ mkdir my_project
$ cd my_project

# 2. 소스 파일 작성
$ nano hello.c
# 또는
$ vim hello.c
# 또는
$ code hello.c

# 3. 컴파일
$ gcc -Wall -Wextra -std=c11 hello.c -o hello

# 4. 실행
$ ./hello

# 5. 에러 발생 시 디버깅
$ gcc -g -Wall hello.c -o hello
$ gdb hello
```

### 13.2 여러 파일 프로젝트

```bash
# 디렉토리 구조 생성
$ mkdir -p project/{src,include,obj,bin}
$ cd project

# 소스 파일 작성
$ nano src/main.c
$ nano src/utils.c
$ nano include/utils.h

# 개별 컴파일
$ gcc -c -I./include src/main.c -o obj/main.o
$ gcc -c -I./include src/utils.c -o obj/utils.o

# 링크
$ gcc obj/main.o obj/utils.o -o bin/program

# 또는 한 번에
$ gcc -I./include src/*.c -o bin/program

# 실행
$ ./bin/program
```

### 13.3 Makefile 사용

```bash
# Makefile 작성
$ nano Makefile

# 빌드
$ make

# 클린
$ make clean

# 재빌드
$ make clean && make
```

## 14. 참고 자료

### 14.1 공식 문서
- GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/
- GCC Manual: https://gcc.gnu.org/onlinedocs/
- GDB Manual: https://sourceware.org/gdb/documentation/
- Make Manual: https://www.gnu.org/software/make/manual/

### 14.2 표준
- POSIX.1-2017 (IEEE Std 1003.1-2017)
- ISO/IEC 9899:2018 (C17 Standard)

### 14.3 추가 학습 자료
- Linux man pages: `man <command>`
- tldr pages: 간단한 예제 중심 도움말 (https://tldr.sh/)

---

**문서 정보**
- 작성 목적: C 프로그래밍 학습을 위한 터미널 명령어 완전 레퍼런스
- 근거: GNU Documentation, POSIX Standards, Linux man pages
- 대상: C 프로그래밍을 배우는 대학생
