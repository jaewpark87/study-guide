
---

## 📄 초보 맥 유저를 위한 터미널 입문 가이드

---

### 🤔 왜 터미널을 배워야 할까?

마우스로 클릭하면 되는데 왜 굳이 **검은 화면에 글자를 타이핑**해야 할까요?

#### 1. 💨 더 빠르다

```
마우스: Finder 열기 → 폴더 클릭 → 폴더 클릭 → 폴더 클릭 → 파일 찾기
터미널: cd project && ls    (2초면 끝!)
```

#### 2. 🔁 반복 작업을 자동화할 수 있다

```bash
# 100개 폴더를 한 줄로 만들기
mkdir folder_{001..100}

# 마우스로? 100번 우클릭 → 새 폴더... 😱
```

#### 3. 💼 개발자의 필수 도구

| 도구 | 터미널 필요? |
| --- | --- |
| Git (코드 관리) | ✅ |
| Docker (서버 실행) | ✅ |
| Python, Node.js | ✅ |
| 서버 접속 (SSH) | ✅ |

> 개발자 채용 공고에 **"Linux/터미널 경험"**이 항상 있는 이유입니다!

#### 4. 🖥️ GUI가 없는 환경

실제 서버(AWS, Google Cloud 등)에는 **모니터도 마우스도 없습니다**. 터미널만으로 모든 것을 제어해야 합니다!

> 💡 **컴퓨터 공학 개념 — GUI vs CLI**
>
> | | GUI | CLI |
> | --- | --- | --- |
> | 풀네임 | Graphical User Interface | Command Line Interface |
> | 조작 방법 | 마우스 클릭 | 키보드 타이핑 |
> | 예시 | Finder, 탐색기 | 터미널, PowerShell |
> | 장점 | 직관적, 배우기 쉬움 | 빠르고, 자동화 가능 |
> | 실무에서 | 일반 사용자 | 개발자, 시스템 관리자 |
>
> 처음엔 CLI가 어렵게 느껴지지만, 익숙해지면 GUI보다 **훨씬 빠르고 강력**합니다!

---

### 🎯 이번에 배울 것

| 파트 | 내용 |
| --- | --- |
| 1 | 터미널 열기 & 기본 개념 |
| 2 | 폴더 이동하기 (pwd, ls, cd) |
| 3 | 파일 & 폴더 만들고 관리하기 |

---

## Part 1: 터미널 열기

### 터미널 실행 방법

`Command(⌘) + Space` → "Terminal" 입력 → Enter

터미널이 열리면 이런 화면이 보입니다:

```
username@MacBook ~ %
```

이것이 **프롬프트(Prompt)**입니다. "명령어를 입력하세요!"라는 뜻이에요.

> 💡 **컴퓨터 공학 개념 — Shell이란?**
>
> **Shell**은 사용자와 운영체제(OS) 사이의 **통역사**입니다.
>
> ```
> 사용자 (당신)
>    |
>    |  "ls" 입력
>    v
>  Shell (zsh) ← 통역사!
>    |
>    |  OS에게 "파일 목록 보여줘" 전달
>    v
>  운영체제 (macOS)
>    |
>    |  파일 목록 반환
>    v
>  화면에 출력
> ```
>
> macOS의 기본 Shell은 **zsh** (Z Shell)입니다.
> Linux에서는 보통 **bash** (Bourne Again Shell)를 사용합니다.
> Shell은 여러 종류가 있지만, 기본 명령어는 거의 같습니다!

### 첫 번째 명령어!

```bash
echo "안녕, 터미널!"
```

결과:
```
안녕, 터미널!
```

`echo`는 **앵무새** 🦜 입니다. 뒤에 쓴 글자를 그대로 따라해요!

---

## Part 2: 폴더 이동하기 (내비게이션) 🗺️

### `pwd` — 지금 어디에 있지? (Print Working Directory)

```bash
pwd
```
```
/Users/username
```

> 비유: GPS에서 **현재 위치** 보기 📍

> 💡 **컴퓨터 공학 개념 — 파일 시스템 (File System)**
>
> 컴퓨터의 모든 파일과 폴더는 **나무(트리) 구조**로 정리되어 있습니다:
>
> ```
> / (루트 - 가장 꼭대기)
> ├── Users/
> │   └── username/        ← 여기가 홈 폴더 (~)
> │       ├── Desktop/
> │       ├── Documents/
> │       ├── Downloads/
> │       └── .zshrc        ← 숨김 파일 (.으로 시작)
> ├── Applications/
> └── System/
> ```
>
> - **루트(`/`)**: 나무의 뿌리 (모든 것의 시작)
> - **홈(`~`)**: 내 개인 공간 (`/Users/username`)
> - **경로(Path)**: 파일의 주소 (`/Users/username/Desktop/file.txt`)

### `ls` — 여기에 뭐가 있지? (List)

```bash
ls
```
```
Desktop    Documents    Downloads    Music    Pictures
```

```bash
ls -l     # 상세 정보 (크기, 날짜, 권한)
ls -a     # 숨김 파일까지 (.으로 시작하는 파일)
ls -la    # 둘 다!
```

> 비유: 방 안에서 **주위를 둘러보기** 👀

> 💡 **컴퓨터 공학 개념 — 옵션과 플래그 (Flags)**
>
> `-l`, `-a` 같은 것을 **플래그(Flag)** 또는 **옵션(Option)**이라고 합니다.
>
> ```
> ls  -l  -a   /Desktop
> ──  ──  ──   ────────
> 명령어 옵션1 옵션2  인자(argument)
> ```
>
> 대부분의 명령어는 `명령어 [옵션] [인자]` 구조입니다.
> **`-`(짧은 옵션)** 과 **`--`(긴 옵션)** 두 가지 형태가 있어요:
> - `ls -a` = `ls --all` (같은 뜻!)

### `cd` — 다른 폴더로 이동! (Change Directory)

```bash
cd Desktop       # 바탕화면으로 이동
pwd              # → /Users/username/Desktop

cd ..            # 한 단계 위로 (상위 폴더)
cd ~             # 홈 폴더로 바로 이동
cd -             # 이전 폴더로 돌아가기 (뒤로가기!)
```

> 비유: Finder에서 **폴더를 더블클릭**하는 것 = `cd 폴더이름`

### 경로 기호 정리

| 기호 | 의미 | 예시 |
| --- | --- | --- |
| `.` | 현재 폴더 | `./file.txt` |
| `..` | 상위 폴더 | `cd ..` |
| `~` | 홈 폴더 | `cd ~` |
| `/` | 최상위 (루트) | `cd /` |

> 💡 **컴퓨터 공학 개념 — 절대 경로 vs 상대 경로**
>
> | 종류 | 설명 | 예시 |
> | --- | --- | --- |
> | **절대 경로** | `/`부터 시작하는 전체 주소 | `/Users/username/Desktop/file.txt` |
> | **상대 경로** | 현재 위치 기준 | `./Desktop/file.txt` 또는 `../Documents/` |
>
> 비유: "서울시 강남구 역삼동 123번지" = 절대 경로, "옆 건물 2층" = 상대 경로

### 🎮 연습해보기!

```bash
cd Desktop       # 바탕화면으로
ls               # 뭐가 있는지 보기
cd ..            # 다시 위로
cd Documents     # 문서 폴더로
pwd              # 어디에 있는지 확인
cd ~             # 홈으로 복귀!
```

---

## Part 3: 파일 & 폴더 관리하기 📁

### `mkdir` — 새 폴더 만들기 (Make Directory)

```bash
mkdir my_project

# 중첩 폴더 한번에 만들기 (-p = parents)
mkdir -p my_project/src/components
```

### `touch` — 빈 파일 만들기

```bash
touch hello.txt
touch index.html style.css app.js    # 여러 개 한번에!
```

### 🎮 연습: 프로젝트 폴더 만들기

```bash
cd Desktop
mkdir -p my_website/css my_website/js
cd my_website
touch index.html css/style.css js/app.js
ls -R    # -R = 하위 폴더까지 전부 보기
```

### `cp` — 파일 복사하기 (Copy)

```bash
cp hello.txt hello_backup.txt              # 파일 복사
cp -r my_website my_website_backup         # 폴더 복사 (-r = recursive)
```

### `mv` — 이동 또는 이름 바꾸기 (Move)

```bash
mv hello.txt Documents/          # 파일 이동
mv old_name.txt new_name.txt     # 이름 바꾸기 (같은 폴더에서!)
```

> 💡 `mv`는 **이삿짐 트럭** 🚚 입니다!
> - 다른 폴더로 보내면 = **이동**
> - 같은 폴더에서 쓰면 = **이름 변경**

### `rm` — 파일 삭제하기 (Remove)

```bash
rm hello.txt               # 파일 삭제
rm -r my_website_backup    # 폴더 삭제 (-r)
rm -i important_file.txt   # 확인 후 삭제 (-i = interactive)
```

> ⚠️ **`rm`은 휴지통을 거치지 않습니다!** 삭제하면 복구가 매우 어렵습니다.
> 처음에는 `rm -i`로 확인하면서 삭제하는 습관을 들이세요!

> 💡 **컴퓨터 공학 개념 — 재귀(Recursive)**
>
> `-r` 옵션은 **재귀(Recursive)**를 뜻합니다. "폴더 안에 폴더가 있으면, 그 안의 폴더도, 또 그 안의 폴더도... 전부 다 처리해!"라는 의미입니다.
>
> ```
> my_project/           ← rm -r은 이 모든 것을 삭제
> ├── src/
> │   ├── app.js
> │   └── utils.js
> └── README.md
> ```
>
> **재귀**는 프로그래밍에서 자주 나오는 개념입니다. "자기 자신을 반복 호출"하는 것이에요!

---

### 🛠️ Cheat Sheet

```
📂 이동
  pwd          현재 위치
  ls           폴더 내용 보기
  ls -la       상세 + 숨김파일
  cd 폴더      폴더로 이동
  cd ..        상위 폴더
  cd ~         홈 폴더

📁 파일/폴더 관리
  mkdir 이름    폴더 만들기
  touch 이름    빈 파일 만들기
  cp A B       복사 (-r로 폴더)
  mv A B       이동/이름변경
  rm 파일       삭제 (주의!)
  rm -r 폴더    폴더 삭제
```

---

### 📚 다음 가이드

👉 [파일 보기, 검색, 꿀팁](02-shell-tips.md) — cat, grep, 파이프, 단축키!

---
