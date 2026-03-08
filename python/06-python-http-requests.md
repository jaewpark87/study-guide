
---

## 📄 초보자를 위한 HTTP & Python Requests 가이드

이전 가이드([Flask + DB 연동](05-python-flask-database.md))에서 만든 **Flask API**에서, **파이썬 코드로 데이터를 가져오는 방법**을 배웁니다.

---

### 🎯 이번에 배울 것

| 순서 | 내용 |
| --- | --- |
| 1 | HTTP가 뭔지 이해하기 |
| 2 | `requests`로 데이터 가져오기 |
| 3 | 우리 API에서 메뉴 데이터 가져오기 |
| 4 | 에러 처리하기 |

---

## Part 1: HTTP란?

> 💡 **컴퓨터 공학 개념 — HTTP (HyperText Transfer Protocol)**
>
> **HTTP**는 인터넷에서 데이터를 주고받는 **약속(규칙)**입니다.
>
> ```
> 브라우저/파이썬 (클라이언트)
>        |
>        |  1. "메뉴 데이터 주세요!" (요청/Request)
>        v
>    Flask 서버
>        |
>        |  2. "여기 있어요!" (응답/Response)
>        v
> 브라우저/파이썬
> ```
>
> 브라우저로 네이버에 접속하는 것도, 파이썬으로 API를 호출하는 것도 모두 HTTP입니다!

### HTTP 상태 코드

서버가 응답할 때 **숫자 코드**로 결과를 알려줍니다:

| 코드 | 의미 | 비유 |
| --- | --- | --- |
| **200** | ✅ 성공 | "네, 여기 있습니다!" |
| **404** | 🔍 없는 페이지 | "그런 거 없어요" |
| **500** | 💥 서버 에러 | "서버가 고장났어요" |

> 쉽게 기억하기: **2xx** = 성공 😊, **4xx** = 내 잘못 😤, **5xx** = 서버 잘못 😰

---

## Part 2: Python `requests` 시작하기

### 1. 설치

```bash
pip install requests
```

### 2. 첫 번째 요청 보내기

`request_test.py`:

```python
import requests

# 공개 테스트 서버에 요청 보내기
response = requests.get("https://httpbin.org/get")

print(f"상태 코드: {response.status_code}")  # 200
print(f"성공 여부: {response.ok}")            # True
```

```bash
python request_test.py
```

🎉 파이썬으로 **브라우저 없이** 웹 요청을 보냈습니다!

### 3. JSON 데이터 받기

API는 보통 **JSON** 형식으로 데이터를 줍니다. 파이썬 딕셔너리와 거의 같습니다:

```python
import requests

response = requests.get("https://httpbin.org/get")
data = response.json()  # JSON → 파이썬 딕셔너리로 변환

print(type(data))  # <class 'dict'>
print(data["url"])  # "https://httpbin.org/get"
```

> 💡 **컴퓨터 공학 개념 — JSON**
>
> JSON은 데이터를 주고받을 때 쓰는 **표준 형식**입니다.
>
> | JSON | 파이썬 |
> | --- | --- |
> | `"hello"` | `str` |
> | `42` | `int` |
> | `true` | `True` |
> | `[1, 2]` | `list` |
> | `{"a": 1}` | `dict` |

---

## Part 3: 우리 Flask API에서 데이터 가져오기

### 1. 서버 실행 (터미널 1)

```bash
docker start my_postgres
cd my_cafe_db
python app.py
```

### 2. 데이터 가져오기 (터미널 2)

`get_menu.py`:

```python
import requests

# 우리 Flask API에 요청
response = requests.get("http://localhost:5000/api/menu")

if response.ok:
    data = response.json()
    menu = data["menu"]

    print(f"☕ 총 {len(menu)}개 메뉴\n")

    for item in menu:
        status = "✅" if item["is_available"] else "❌"
        print(f"  {status} {item['name']:10s} {item['price']:>6,}원")
else:
    print(f"❌ 실패! 상태 코드: {response.status_code}")
```

```bash
python get_menu.py
```

실행 결과:
```
☕ 총 9개 메뉴

  ✅ 아메리카노     4,500원
  ✅ 카페라떼       5,000원
  ✅ 바닐라라떼     5,500원
  ✅ 녹차           4,000원
  ...
```

🎉 **파이썬 코드로 API에서 데이터를 가져왔습니다!**

### 3. 가져온 데이터 가공하기

```python
import requests

response = requests.get("http://localhost:5000/api/menu")
menu = response.json()["menu"]

# 판매 중인 메뉴만 필터링
available = [item for item in menu if item["is_available"]]
print(f"판매 중: {len(available)}개")

# 가격 통계
prices = [item["price"] for item in menu]
print(f"최저가: {min(prices):,}원")
print(f"최고가: {max(prices):,}원")
print(f"평균가: {sum(prices) // len(prices):,}원")

# 가격순 정렬
for item in sorted(menu, key=lambda x: x["price"]):
    print(f"  {item['name']} - {item['price']:,}원")
```

> 💡 API에서 가져온 데이터는 결국 **파이썬 리스트와 딕셔너리**입니다. 이전에 배운 `for`문, `if`문, 리스트 컴프리헨션을 그대로 쓸 수 있습니다!

---

## Part 4: 에러 처리 (중요!)

서버가 꺼져 있거나, 응답이 느릴 수 있습니다. 항상 에러를 대비하세요:

```python
import requests

try:
    response = requests.get("http://localhost:5000/api/menu", timeout=5)
    response.raise_for_status()  # 4xx/5xx면 에러 발생

    data = response.json()
    print(f"✅ {len(data['menu'])}개 메뉴를 가져왔습니다!")

except requests.exceptions.ConnectionError:
    print("❌ 서버에 연결할 수 없습니다!")
    print("   → Flask 서버가 실행 중인지 확인하세요.")

except requests.exceptions.Timeout:
    print("⏰ 시간 초과! 서버가 너무 느립니다.")

except requests.exceptions.HTTPError as e:
    print(f"🚫 HTTP 에러: {e}")
```

> 💡 **컴퓨터 공학 개념 — 예외 처리 & Timeout**
>
> - **`try/except`**: 에러가 나도 프로그램이 죽지 않게 보호
> - **`timeout=5`**: 5초 안에 응답 없으면 포기 (없으면 무한 대기!)
>
> 실무에서는 **네트워크 요청에 항상 timeout과 에러 처리**를 넣습니다!

---

## Part 5: 전체 그림

이전 가이드들과 합치면 완전한 시스템이 됩니다:

```
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
| **PostgreSQL** | 데이터 저장 | 가이드 1, 2 |
| **psycopg2** | 파이썬 ↔ DB 연결 | 가이드 3 |
| **Flask** | 웹 서버 + API | 가이드 4, 5 |
| **requests** | API 데이터 가져오기 | 가이드 6 (이번!) |

---

### 🛠️ Cheat Sheet

```python
import requests

# GET 요청
response = requests.get("http://URL", timeout=5)

# 응답 확인
response.status_code   # 200, 404, 500...
response.ok            # True / False
response.json()        # JSON → 딕셔너리
response.text          # 응답 문자열

# 에러 처리
try:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    data = response.json()
except requests.exceptions.ConnectionError:
    print("연결 실패")
except requests.exceptions.Timeout:
    print("시간 초과")
```

---

### 📚 학습 흐름

```
📘 가이드 1: Docker + PostgreSQL     → DB 설치 및 SQL 기초
📘 가이드 2: 데이터 수정 & 삭제       → UPDATE, DELETE
📘 가이드 3: Python + PostgreSQL     → 파이썬으로 CRUD
📘 가이드 4: Flask 기초              → 웹 서버 만들기
📘 가이드 5: Flask + DB 연동         → DB 데이터를 웹에 표시
📘 가이드 6: HTTP & Requests (이번!) → API 데이터 가져오기
```

### 📚 다음 단계

- **POST/PUT/DELETE 요청**: 데이터를 생성/수정/삭제하기
- **REST API 설계**: API URL을 체계적으로 설계하는 방법
- **인증(Authentication)**: API Key, JWT 토큰으로 보안 API 만들기

---
