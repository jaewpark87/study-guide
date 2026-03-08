
---

## 📄 초보자를 위한 HTTP 이해 & Python Requests 가이드

이 문서는 이전 가이드([Python 웹 서버](python-web-server.md))에서 만든 **Flask API**를 활용하여, **HTTP의 기본 원리**를 이해하고 **파이썬 `requests` 라이브러리**로 API에서 데이터를 가져오는 방법을 학습합니다.

---

### 🎯 이번에 배울 것

| 순서 | 내용 | 결과 |
| --- | --- | --- |
| 1 | HTTP가 뭔지 이해하기 | 인터넷이 어떻게 작동하는지 알기 |
| 2 | `requests` 라이브러리 설치 및 첫 요청 | 파이썬으로 웹 데이터 가져오기 |
| 3 | 우리가 만든 API에서 데이터 가져오기 | Flask API ↔ Python 연동 |
| 4 | 가져온 데이터 가공하고 활용하기 | 실전 활용 |

---

## Part 1: HTTP 완전 이해하기

### HTTP란 무엇인가?

> 💡 **컴퓨터 공학 개념 — HTTP (HyperText Transfer Protocol)**
>
> **HTTP**는 인터넷에서 데이터를 주고받는 **약속(규칙)**입니다.
>
> "Protocol(프로토콜)"은 **통신 규칙**이라는 뜻입니다. 마치 전화 통화에도 규칙이 있듯이요:
>
> ```
> 전화 규칙:
>   1. 전화를 건다 (연결)
>   2. "여보세요?" (인사)
>   3. 용건을 말한다 (요청)
>   4. 상대방이 대답한다 (응답)
>   5. "끊을게요~" (종료)
>
> HTTP 규칙:
>   1. 서버에 연결한다 (TCP 연결)
>   2. 요청을 보낸다 (HTTP Request)
>   3. 서버가 응답한다 (HTTP Response)
>   4. 연결이 끊어진다 (종료)
> ```
>
> 여러분이 네이버에 접속할 때, 크롬 브라우저는 네이버 서버에 **HTTP 규칙에 맞게** "메인 페이지를 주세요!" 라고 요청하고, 네이버는 **HTTP 규칙에 맞게** 페이지를 보내줍니다.

### HTTP 요청(Request)의 구조

HTTP 요청은 **편지**와 비슷합니다:

```
+----------------------------------------------+
| HTTP Request (편지)                           |
+----------------------------------------------+
| 받는 곳:    localhost:5000/api/menu   (URL)    |
| 요청 종류:  GET                      (Method) |
+----------------------------------------------+
| 추가 정보 (Headers):                          |
|   보내는 사람:    Python-requests/2.31         |
|   받고 싶은 형식: application/json             |
+----------------------------------------------+
| 내용물 (Body):                                |
|   (GET 요청은 보통 비어있음)                    |
+----------------------------------------------+
```

> 💡 **컴퓨터 공학 개념 — HTTP 메서드 (이전 가이드 복습 + 심화)**
>
> | 메서드 | 용도 | CRUD | 비유 |
> | --- | --- | --- | --- |
> | **GET** | 데이터 **조회** | Read | "이 책 보여주세요" |
> | **POST** | 데이터 **생성** | Create | "새 책 등록해주세요" |
> | **PUT** | 데이터 **전체 수정** | Update | "이 책 정보 전체를 바꿔주세요" |
> | **PATCH** | 데이터 **일부 수정** | Update | "이 책 가격만 바꿔주세요" |
> | **DELETE** | 데이터 **삭제** | Delete | "이 책 삭제해주세요" |
>
> 이번 가이드에서는 주로 **GET** 요청을 다룹니다. 브라우저 주소창에 URL을 입력하는 것 = GET 요청을 보내는 것!

### HTTP 응답(Response)의 구조

서버가 보내는 응답도 구조가 있습니다:

```
+----------------------------------------------+
| HTTP Response (답장)                          |
+----------------------------------------------+
| 상태 코드: 200 OK  ("성공했어요!")              |
+----------------------------------------------+
| 추가 정보 (Headers):                          |
|   콘텐츠 종류: application/json                |
|   콘텐츠 길이: 542 bytes                       |
+----------------------------------------------+
| 내용물 (Body):                                |
|   {"menu": [{"name": "아메리카노"}]}           |
+----------------------------------------------+
```

> 💡 **컴퓨터 공학 개념 — HTTP 상태 코드 (Status Code)**
>
> 상태 코드는 **3자리 숫자**로, 요청이 어떻게 처리됐는지 알려줍니다.
>
> | 코드 | 의미 | 비유 | 언제? |
> | --- | --- | --- | --- |
> | **200** | ✅ 성공 | "네, 여기 있습니다!" | 정상적으로 처리됨 |
> | **201** | ✅ 생성 성공 | "새로 만들었어요!" | POST로 데이터 생성 성공 |
> | **301** | ↪️ 이사했어요 | "새 주소로 가세요" | URL이 바뀜 |
> | **400** | ❌ 잘못된 요청 | "무슨 말인지 모르겠어요" | 요청 형식이 잘못됨 |
> | **403** | 🚫 접근 금지 | "권한이 없어요" | 인증은 됐지만 권한 없음 |
> | **404** | 🔍 없는 페이지 | "그런 거 없어요" | URL이 존재하지 않음 |
> | **500** | 💥 서버 에러 | "서버가 고장났어요" | 서버 내부 오류 |
>
> **기억하는 쉬운 방법:**
> - **2xx** = 😊 성공! (다 좋아요)
> - **3xx** = 🔄 다른 곳으로 (리다이렉트)
> - **4xx** = 😤 클라이언트 잘못 (당신이 잘못했어요)
> - **5xx** = 😰 서버 잘못 (우리가 잘못했어요)
>
> 모두가 경험했을 **"404 Not Found"**는 존재하지 않는 URL에 접속했을 때 나옵니다!

---

## Part 2: Python `requests` 시작하기

### 1. 설치하기

```bash
pip install requests
```

> 💡 **`requests`는 왜 필요할까요?**
>
> 파이썬에는 HTTP 요청을 보내는 기본 모듈(`urllib`)이 있지만, 사용하기 복잡합니다. `requests`는 이것을 **매우 쉽게** 만들어 줍니다.
>
> ```python
> # urllib (기본 모듈) — 복잡함 😵
> import urllib.request
> import json
> req = urllib.request.Request("http://example.com/api")
> with urllib.request.urlopen(req) as response:
>     data = json.loads(response.read().decode())
>
> # requests 라이브러리 — 간단함 😊
> import requests
> data = requests.get("http://example.com/api").json()
> ```

### 2. 첫 번째 요청 보내기

먼저 공개된 웹사이트로 연습해 보겠습니다. `request_tutorial.py` 파일을 만드세요:

```python
import requests

# 공개 API에 GET 요청 보내기
response = requests.get("https://httpbin.org/get")

# 응답 확인
print(f"상태 코드: {response.status_code}")
print(f"성공 여부: {response.ok}")
print(f"콘텐츠 타입: {response.headers['Content-Type']}")
print()
print("응답 내용:")
print(response.text)
```

```bash
python request_tutorial.py
```

실행 결과:
```
상태 코드: 200
성공 여부: True
콘텐츠 타입: application/json
```

🎉 방금 파이썬으로 **HTTP GET 요청**을 보내고 **응답을 받았습니다!** 브라우저 없이, 코드로!

> 💡 **컴퓨터 공학 개념 — httpbin.org**
>
> `httpbin.org`는 HTTP 요청을 테스트할 수 있는 **무료 공개 서비스**입니다. 보낸 요청 정보를 그대로 돌려보내 줍니다. 마치 거울처럼요! 개발자들이 HTTP 통신을 테스트할 때 자주 사용합니다.

### 3. Response 객체 이해하기

`requests.get()`이 반환하는 `response` 객체에는 다양한 정보가 들어 있습니다:

```python
import requests

response = requests.get("https://httpbin.org/get")

# 자주 쓰는 속성들
print(f"상태 코드:     {response.status_code}")    # 200
print(f"성공 여부:     {response.ok}")              # True (200~299면 True)
print(f"텍스트 응답:   {response.text[:50]}...")     # 응답 본문 (문자열)
print(f"인코딩:       {response.encoding}")         # utf-8
print(f"응답 시간:     {response.elapsed}")          # 걸린 시간
```

| 속성/메서드 | 타입 | 설명 |
| --- | --- | --- |
| `.status_code` | int | HTTP 상태 코드 (200, 404 등) |
| `.ok` | bool | 상태 코드가 200~299이면 `True` |
| `.text` | str | 응답 본문 (문자열) |
| `.json()` | dict/list | 응답 본문을 JSON으로 파싱 |
| `.headers` | dict | 응답 헤더 정보 |
| `.elapsed` | timedelta | 요청~응답 소요 시간 |
| `.url` | str | 실제 요청된 URL |

---

## Part 3: 우리가 만든 API에서 데이터 가져오기

### 1. Flask 서버 실행하기

이전 가이드에서 만든 카페 서버를 실행하세요. **터미널을 2개** 사용합니다:

**터미널 1 — 서버 실행:**

```bash
# Docker PostgreSQL 시작
docker start my_postgres

# Flask 서버 시작
cd my_cafe_db
python app.py
```

서버가 `http://localhost:5000`에서 실행 중인지 확인하세요.

**터미널 2 — requests 코드 실행** (다른 터미널을 열어주세요!)

### 2. API에서 전체 메뉴 가져오기

`get_menu.py` 파일을 만드세요:

```python
import requests

# 우리 Flask 서버의 API에 GET 요청 보내기
response = requests.get("http://localhost:5000/api/menu")

# 상태 코드 확인
print(f"상태 코드: {response.status_code}")

if response.ok:
    # JSON 데이터를 파이썬 딕셔너리로 변환
    data = response.json()
    menu = data["menu"]

    print(f"\n☕ 총 {len(menu)}개의 메뉴를 가져왔습니다!\n")
    print("-" * 50)

    for item in menu:
        status = "✅ 판매중" if item["is_available"] else "❌ 품절"
        print(f"  {item['name']:10s} | {item['price']:>7,}원 | {status}")

    print("-" * 50)
else:
    print(f"❌ 요청 실패! 상태 코드: {response.status_code}")
```

```bash
python get_menu.py
```

실행 결과:
```
상태 코드: 200

☕ 총 9개의 메뉴를 가져왔습니다!

--------------------------------------------------
  아메리카노    |   4,500원 | ✅ 판매중
  카페라떼      |   5,000원 | ✅ 판매중
  바닐라라떼    |   5,500원 | ✅ 판매중
  녹차          |   4,000원 | ✅ 판매중
  캐모마일      |   4,000원 | ✅ 판매중
  아이스티      |   3,500원 | ❌ 품절
  초코케이크    |   6,500원 | ✅ 판매중
  크로와상      |   4,000원 | ✅ 판매중
  치즈케이크    |   7,000원 | ❌ 품절
--------------------------------------------------
```

🎉 **파이썬 코드로 Flask API에서 데이터를 가져왔습니다!**

> 💡 **컴퓨터 공학 개념 — JSON (JavaScript Object Notation)**
>
> API에서 받은 데이터는 **JSON** 형식입니다. JSON은 데이터를 주고받을 때 가장 많이 쓰는 **표준 형식**입니다.
>
> ```json
> {
>   "menu": [
>     {
>       "id": 1,
>       "name": "아메리카노",
>       "price": 4500,
>       "is_available": true
>     }
>   ]
> }
> ```
>
> JSON이 인기 있는 이유:
> - **사람이 읽기 쉬움** (위처럼 깔끔한 구조)
> - **프로그래밍 언어와 호환** (파이썬 딕셔너리와 거의 같음!)
> - **가벼움** (XML보다 데이터 크기가 작음)
>
> | JSON 타입 | 파이썬 타입 | 예시 |
> | --- | --- | --- |
> | `string` | `str` | `"아메리카노"` |
> | `number` | `int`, `float` | `4500`, `3.14` |
> | `boolean` | `bool` | `true` → `True` |
> | `null` | `None` | `null` → `None` |
> | `array` | `list` | `[1, 2, 3]` |
> | `object` | `dict` | `{"key": "value"}` |
>
> `response.json()`을 호출하면 JSON이 자동으로 파이썬 딕셔너리/리스트로 변환됩니다!

### 3. 데이터 필터링하고 가공하기

가져온 데이터를 파이썬으로 자유롭게 가공할 수 있습니다:

```python
import requests

response = requests.get("http://localhost:5000/api/menu")
data = response.json()
menu = data["menu"]


# ── A. 판매 중인 메뉴만 필터링 ──
available = [item for item in menu if item["is_available"]]
print(f"📋 판매 중인 메뉴: {len(available)}개")
for item in available:
    print(f"  ✅ {item['name']} - {item['price']:,}원")


# ── B. 카테고리별 분류 ──
print("\n📂 카테고리별 분류:")
categories = {}
for item in menu:
    cat = item["category"]
    if cat not in categories:
        categories[cat] = []
    categories[cat].append(item)

for cat, items in categories.items():
    print(f"\n  [{cat}]")
    for item in items:
        print(f"    - {item['name']} ({item['price']:,}원)")


# ── C. 통계 계산 ──
prices = [item["price"] for item in menu]
print(f"\n📊 가격 통계:")
print(f"  최저가: {min(prices):,}원")
print(f"  최고가: {max(prices):,}원")
print(f"  평균가: {sum(prices) // len(prices):,}원")


# ── D. 가격 순으로 정렬 ──
print(f"\n💰 가격 낮은 순:")
sorted_menu = sorted(menu, key=lambda x: x["price"])
for item in sorted_menu:
    print(f"  {item['name']:10s} - {item['price']:>7,}원")
```

> 💡 **핵심 포인트**: API에서 가져온 데이터는 결국 파이썬 **리스트와 딕셔너리**입니다. 이전에 배운 파이썬 문법(for문, if문, 리스트 컴프리헨션 등)을 그대로 활용할 수 있습니다!

---

## Part 4: 에러 처리와 안전한 요청

### 1. 서버가 꺼져있을 때

서버가 꺼져 있으면 에러가 발생합니다. 이를 안전하게 처리해 봅시다:

```python
import requests

try:
    response = requests.get("http://localhost:5000/api/menu", timeout=5)
    response.raise_for_status()  # 4xx, 5xx 에러면 예외 발생

    data = response.json()
    print(f"✅ {len(data['menu'])}개 메뉴를 가져왔습니다!")

except requests.exceptions.ConnectionError:
    print("❌ 서버에 연결할 수 없습니다!")
    print("   → Flask 서버가 실행 중인지 확인하세요.")
    print("   → 'python app.py'로 서버를 시작하세요.")

except requests.exceptions.Timeout:
    print("⏰ 요청 시간이 초과되었습니다!")
    print("   → 서버가 너무 바쁘거나 응답이 느립니다.")

except requests.exceptions.HTTPError as e:
    print(f"🚫 HTTP 에러 발생: {e}")

except requests.exceptions.JSONDecodeError:
    print("📄 응답이 JSON 형식이 아닙니다!")
    print(f"   → 받은 내용: {response.text[:100]}")
```

> 💡 **컴퓨터 공학 개념 — 예외 처리 (Exception Handling)**
>
> 네트워크 통신에서는 **항상 에러가 발생할 수 있습니다**:
> - 서버가 꺼져 있을 수 있음
> - 인터넷이 끊길 수 있음
> - 서버가 너무 느릴 수 있음
> - 응답 형식이 예상과 다를 수 있음
>
> 이런 상황을 대비하지 않으면 프로그램이 **크래시(Crash)**됩니다. `try/except`로 에러를 **잡아서(catch)** 적절히 처리하는 것을 **예외 처리**라고 합니다.
>
> ```python
> try:
>     # 위험한 작업 (에러가 날 수 있는 코드)
>     response = requests.get(url)
> except 에러종류:
>     # 에러가 났을 때 대처 방법
>     print("에러 처리!")
> ```
>
> 실무에서는 예외 처리 없이 네트워크 요청을 보내면 **코드 리뷰에서 반려**됩니다! 🙅

### 2. timeout (시간 제한)

```python
# 5초 안에 응답이 없으면 포기
response = requests.get("http://localhost:5000/api/menu", timeout=5)
```

> 💡 **컴퓨터 공학 개념 — Timeout (타임아웃)**
>
> `timeout`이 없으면, 서버가 응답하지 않을 때 **프로그램이 영원히 기다립니다**. 이것을 **행(Hang)**이라고 합니다.
>
> | timeout 설정 | 동작 |
> | --- | --- |
> | `timeout=5` | 5초 안에 응답 없으면 에러 |
> | `timeout=(3, 10)` | 연결에 3초, 응답에 10초 |
> | timeout 없음 | 무한 대기 (위험!) |
>
> 실무에서는 **항상 timeout을 설정**합니다!

---

## Part 5: 요청에 추가 정보 보내기

### 1. 쿼리 파라미터 (Query Parameters)

URL 뒤에 `?key=value`를 붙여서 추가 정보를 보낼 수 있습니다:

```python
import requests

# 방법 1: URL에 직접 쓰기
response = requests.get("https://httpbin.org/get?name=홍길동&age=25")

# 방법 2: params 딕셔너리 사용 (더 깔끔!)
params = {
    "name": "홍길동",
    "age": 25
}
response = requests.get("https://httpbin.org/get", params=params)

print(f"실제 요청 URL: {response.url}")
# → https://httpbin.org/get?name=홍길동&age=25

data = response.json()
print(f"서버가 받은 파라미터: {data['args']}")
```

> 💡 **컴퓨터 공학 개념 — URL의 구조**
>
> URL은 여러 부분으로 나뉩니다:
>
> ```
> https://www.example.com:443/menu/커피?sort=price&limit=10#top
> ──┬──   ──────┬───────  ─┬─ ───┬──── ─────────┬───────── ─┬──
>   │          │          │     │              │            │
> 스킴     호스트(도메인)  포트  경로(Path)  쿼리 파라미터   프래그먼트
> (https)  (어디로?)     (몇호?) (어떤페이지?) (추가조건)    (페이지내위치)
> ```
>
> | 부분 | 예시 | 쓰이는 곳 |
> | --- | --- | --- |
> | 스킴 | `https://` | 통신 방식 (http vs https) |
> | 호스트 | `www.example.com` | 서버 주소 |
> | 포트 | `:443` | 서버의 문 번호 (생략 가능) |
> | 경로 | `/menu/커피` | 어떤 페이지/리소스 |
> | 쿼리 | `?sort=price&limit=10` | 추가 조건/필터 |
>
> 쿠팡에서 "노트북"을 검색하면 URL이 이렇게 바뀝니다:
> ```
> https://www.coupang.com/np/search?q=노트북&channel=user
> ```
> `q=노트북`이 바로 **쿼리 파라미터**입니다!

### 2. 헤더(Headers) 설정하기

```python
import requests

headers = {
    "User-Agent": "MyCafeApp/1.0",     # 내 앱 이름
    "Accept": "application/json",       # JSON으로 응답해주세요
}

response = requests.get(
    "http://localhost:5000/api/menu",
    headers=headers,
    timeout=5
)

print(response.json())
```

> 💡 **컴퓨터 공학 개념 — HTTP 헤더(Headers)**
>
> 헤더는 요청/응답에 대한 **부가 정보**입니다. 택배에 비유하면:
>
> - **URL** = 배송 주소 (어디로 보낼지)
> - **Body** = 택배 상자 안의 내용물
> - **Headers** = 택배 상자에 붙은 라벨 (취급주의, 냉장보관, 보내는 사람 등)
>
> 자주 사용되는 헤더:
>
> | 헤더 | 역할 | 예시 |
> | --- | --- | --- |
> | `User-Agent` | 요청자 정보 | Chrome/120, Python-requests |
> | `Accept` | 원하는 응답 형식 | application/json |
> | `Content-Type` | 보내는 데이터 형식 | application/json |
> | `Authorization` | 인증 정보 | Bearer eyJhbGciOi... |

---

## Part 6: 실전 활용 — 카페 메뉴 분석 프로그램

지금까지 배운 것을 모두 합쳐서, 카페 메뉴를 분석하는 프로그램을 만들어 보겠습니다!

`cafe_analyzer.py`:

```python
import requests


def get_menu_data():
    """Flask API에서 메뉴 데이터를 가져옵니다."""
    try:
        response = requests.get(
            "http://localhost:5000/api/menu",
            timeout=5
        )
        response.raise_for_status()
        return response.json()["menu"]

    except requests.exceptions.ConnectionError:
        print("❌ 서버에 연결할 수 없습니다! Flask 서버가 실행 중인지 확인하세요.")
        return None
    except requests.exceptions.Timeout:
        print("⏰ 요청 시간 초과!")
        return None
    except Exception as e:
        print(f"❌ 에러 발생: {e}")
        return None


def show_all_menu(menu):
    """전체 메뉴를 보여줍니다."""
    print("\n" + "=" * 55)
    print("  ☕ 나의 카페 — 전체 메뉴")
    print("=" * 55)

    for item in menu:
        status = "✅" if item["is_available"] else "❌"
        print(f"  {status} {item['name']:10s}  {item['category']:6s}  {item['price']:>7,}원")

    print("=" * 55)
    print(f"  총 {len(menu)}개 메뉴\n")


def show_available_only(menu):
    """판매 중인 메뉴만 보여줍니다."""
    available = [m for m in menu if m["is_available"]]

    print(f"\n📋 판매 중인 메뉴 ({len(available)}개):")
    for item in available:
        print(f"  ✅ {item['name']} - {item['price']:,}원")


def show_by_category(menu):
    """카테고리별로 분류해서 보여줍니다."""
    categories = {}
    for item in menu:
        cat = item["category"]
        if cat not in categories:
            categories[cat] = []
        categories[cat].append(item)

    print("\n📂 카테고리별 메뉴:")
    for cat, items in categories.items():
        total = sum(item["price"] for item in items)
        print(f"\n  [{cat}] ({len(items)}개, 평균 {total // len(items):,}원)")
        for item in items:
            status = "✅" if item["is_available"] else "❌"
            print(f"    {status} {item['name']} - {item['price']:,}원")


def show_statistics(menu):
    """메뉴 통계를 보여줍니다."""
    prices = [item["price"] for item in menu]
    available_count = sum(1 for item in menu if item["is_available"])

    print("\n📊 메뉴 통계:")
    print(f"  총 메뉴 수:    {len(menu)}개")
    print(f"  판매 중:       {available_count}개")
    print(f"  품절:          {len(menu) - available_count}개")
    print(f"  최저가:        {min(prices):,}원")
    print(f"  최고가:        {max(prices):,}원")
    print(f"  평균가:        {sum(prices) // len(prices):,}원")

    most_expensive = max(menu, key=lambda x: x["price"])
    cheapest = min(menu, key=lambda x: x["price"])
    print(f"  가장 비싼 메뉴: {most_expensive['name']} ({most_expensive['price']:,}원)")
    print(f"  가장 싼 메뉴:   {cheapest['name']} ({cheapest['price']:,}원)")


def main():
    print("🔄 서버에서 메뉴 데이터를 가져오는 중...")
    menu = get_menu_data()

    if menu is None:
        return

    print(f"✅ {len(menu)}개 메뉴 데이터를 가져왔습니다!")

    while True:
        print("\n┌─────────────────────────────┐")
        print("│   ☕ 카페 메뉴 분석 프로그램  │")
        print("├─────────────────────────────┤")
        print("│  1. 전체 메뉴 보기           │")
        print("│  2. 판매 중인 메뉴만 보기     │")
        print("│  3. 카테고리별 보기           │")
        print("│  4. 통계 보기                │")
        print("│  5. 데이터 새로고침           │")
        print("│  0. 종료                     │")
        print("└─────────────────────────────┘")

        choice = input("\n선택하세요: ").strip()

        if choice == "1":
            show_all_menu(menu)
        elif choice == "2":
            show_available_only(menu)
        elif choice == "3":
            show_by_category(menu)
        elif choice == "4":
            show_statistics(menu)
        elif choice == "5":
            print("🔄 데이터 새로고침 중...")
            new_menu = get_menu_data()
            if new_menu:
                menu = new_menu
                print(f"✅ {len(menu)}개 메뉴 데이터를 다시 가져왔습니다!")
        elif choice == "0":
            print("👋 프로그램을 종료합니다!")
            break
        else:
            print("⚠️ 잘못된 입력입니다. 0~5 사이의 숫자를 입력하세요.")


if __name__ == "__main__":
    main()
```

```bash
python cafe_analyzer.py
```

> 💡 **컴퓨터 공학 개념 — 관심사의 분리 (Separation of Concerns)**
>
> 위 코드에서 각 함수가 **하나의 역할만** 담당합니다:
>
> | 함수 | 역할 |
> | --- | --- |
> | `get_menu_data()` | 데이터 가져오기 (네트워크) |
> | `show_all_menu()` | 전체 표시 (화면 출력) |
> | `show_statistics()` | 통계 계산 (로직) |
> | `main()` | 사용자 입력 처리 (제어 흐름) |
>
> 이렇게 역할을 나누면:
> - **수정이 쉬움**: 통계 계산을 바꾸려면 `show_statistics()`만 고치면 됨
> - **재사용 가능**: `get_menu_data()`를 다른 프로그램에서도 쓸 수 있음
> - **테스트가 쉬움**: 각 함수를 따로 테스트할 수 있음
>
> 이것은 소프트웨어 공학의 **핵심 원칙** 중 하나입니다!

---

## Part 7: 전체 그림 — 데이터의 여행

이번 가이드와 이전 가이드들을 합치면, 완전한 시스템이 만들어집니다:

```
                     ── 전체 시스템 구조 ──

  +------------------+          +------------------+
  |  Python          |  -HTTP-> |  Flask           |
  |  requests        |  <JSON-  |  Web Server      |
  |  (Client)        |          |  (app.py)        |
  +------------------+          +--------+---------+
                                         |
  +------------------+                   | SQL
  |  Web Browser     |  <-- HTML --+     |
  |  (Chrome)        |             |     v
  +------------------+          +--+---------------+
                                |  PostgreSQL      |
                                |  Database        |
                                |  (Docker)        |
                                +------------------+
```

| 구성 요소 | 역할 | 배운 가이드 |
| --- | --- | --- |
| **PostgreSQL** | 데이터 저장 및 관리 | 가이드 1, 2 |
| **psycopg2** | 파이썬 ↔ DB 연결 드라이버 | 가이드 3 |
| **Flask** | 웹 서버 + API 제공 | 가이드 4 |
| **requests** | API에서 데이터 가져오기 | 가이드 5 (이번!) |

> 💡 **컴퓨터 공학 개념 — 클라이언트의 종류**
>
> 우리 Flask 서버에 접속하는 **클라이언트**는 여러 종류가 있습니다:
>
> | 클라이언트 | 접속 URL | 받는 내용 |
> | --- | --- | --- |
> | **웹 브라우저** | `/menu` | HTML (예쁜 페이지) |
> | **Python requests** | `/api/menu` | JSON (데이터) |
> | **모바일 앱** | `/api/menu` | JSON (데이터) |
> | **다른 서버** | `/api/menu` | JSON (데이터) |
>
> 이것이 **API**의 진짜 가치입니다! 하나의 서버가 **다양한 클라이언트**에게 서비스를 제공할 수 있습니다. 카카오톡이 PC, 모바일, 웹에서 모두 작동하는 것도 같은 원리입니다!

---

### 🛠️ 전체 요약 (Cheat Sheet)

```python
import requests

# ── GET 요청 ──
response = requests.get("http://URL", timeout=5)

# ── 응답 확인 ──
response.status_code   # 200, 404, 500...
response.ok            # True / False
response.json()        # JSON → 딕셔너리
response.text          # 응답 문자열

# ── 파라미터 전달 ──
requests.get(url, params={"key": "value"})

# ── 헤더 설정 ──
requests.get(url, headers={"Authorization": "Bearer token"})

# ── 에러 처리 ──
try:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    data = response.json()
except requests.exceptions.ConnectionError:
    print("연결 실패")
except requests.exceptions.Timeout:
    print("시간 초과")
except requests.exceptions.HTTPError:
    print("HTTP 에러")
```

---

### 📚 학습 흐름

```
📘 가이드 1: Docker + PostgreSQL     → DB 설치 및 SQL 기초
📘 가이드 2: 데이터 수정 & 삭제       → UPDATE, DELETE
📘 가이드 3: Python + PostgreSQL     → 파이썬으로 CRUD
📘 가이드 4: Python 웹 서버          → Flask + DB 연동
📘 가이드 5: HTTP & Requests (이번!) → API 데이터 가져오기
```

### 📚 다음 단계

- **POST/PUT/DELETE 요청**: `requests.post()`, `requests.put()`으로 데이터를 생성/수정/삭제하기
- **REST API 설계**: API URL을 체계적으로 설계하는 방법
- **인증(Authentication)**: API Key, JWT 토큰으로 보안 API 만들기
- **비동기 요청(Async)**: 여러 API를 동시에 호출하는 방법 (`aiohttp`)

---
