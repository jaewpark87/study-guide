
---

## 📄 초보자를 위한 Python 웹 서버 만들기 (Flask 기초)

이 문서는 이전 가이드([Docker + PostgreSQL](01-docker-postgres.md), [데이터 수정 & 삭제](02-sql-update-delete.md), [Python CRUD](03-python-database.md))에서 배운 내용을 바탕으로, **파이썬으로 웹 서버를 만드는 방법**을 학습합니다.

---

### 🎯 이번에 배울 것

| 파트 | 내용 | 결과 |
| --- | --- | --- |
| **Part 1** | 가장 간단한 웹 서버 만들기 | 브라우저에서 "Hello World" 보기 |
| **Part 2** | 여러 페이지(URL) 만들기 | 각 URL마다 다른 응답 |
| **Part 3** | HTML 페이지 보여주기 | 예쁜 웹 페이지 만들기 |

---

## Part 1: 가장 간단한 웹 서버 만들기

### 1. Flask 설치하기

```bash
pip install flask
```

> 💡 **컴퓨터 공학 개념 — 웹 프레임워크(Web Framework)**
>
> **Flask**는 파이썬 **웹 프레임워크**입니다. 프레임워크란 "자주 쓰는 기능을 미리 만들어 놓은 도구 상자"입니다.
>
> 웹 서버를 밑바닥부터 만들려면 네트워크 통신, HTTP 파싱, 보안 처리 등 수천 줄의 코드가 필요합니다. Flask는 이런 복잡한 부분을 대신 처리하고, 여러분은 **"이 URL에 이 내용을 보여줘"**라는 핵심 로직에만 집중할 수 있습니다.
>
> | 프레임워크 | 특징 | 비유 |
> | --- | --- | --- |
> | **Flask** | 가볍고 간단 (학습용에 최고) | 레고 블록 (필요한 것만 조립) |
> | **Django** | 기능이 풍부함 (실무에서 많이 사용) | 완성된 가구 세트 |
> | **FastAPI** | 빠르고 현대적 (API 서버에 특화) | 스포츠카 |

### 2. 첫 번째 웹 서버 만들기

`app.py` 파일을 만들고 아래 코드를 입력하세요. **단 5줄**이면 됩니다!

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "안녕하세요! 나의 첫 번째 웹 서버입니다! 🎉"

if __name__ == "__main__":
    app.run(debug=True)
```

### 3. 서버 실행하기

```bash
python app.py
```

터미널에 아래 메시지가 보이면 성공입니다:

```
 * Running on http://127.0.0.1:5000
```

### 4. 브라우저에서 확인하기

웹 브라우저(Chrome, Safari 등)를 열고 주소창에 입력하세요:

```
http://localhost:5000
```

🎉 **"안녕하세요! 나의 첫 번째 웹 서버입니다!"**가 보이면 성공입니다!

> 💡 **컴퓨터 공학 개념 — 웹 서버의 동작 원리**
>
> 방금 무슨 일이 일어난 걸까요? 단계별로 살펴보겠습니다:
>
> ```
> 1. 브라우저 주소창에 "localhost:5000" 입력
>        ↓
> 2. 브라우저가 HTTP 요청(Request)을 보냄
>    "GET / HTTP/1.1" → "/ 페이지를 주세요!"
>        ↓
> 3. Flask 서버가 요청을 받음
>    @app.route("/")에 해당하는 함수 실행
>        ↓
> 4. home() 함수가 문자열을 반환(return)
>        ↓
> 5. Flask가 HTTP 응답(Response)으로 보냄
>        ↓
> 6. 브라우저가 응답을 받아 화면에 표시
> ```
>
> 이전 가이드에서 배운 **클라이언트-서버 모델**과 같은 원리입니다!
> - **클라이언트** = 웹 브라우저 (Chrome)
> - **서버** = 여러분의 Flask 프로그램

> 💡 **컴퓨터 공학 개념 — localhost와 포트(Port)**
>
> - **`localhost`** (= `127.0.0.1`): "내 컴퓨터 자신"을 가리키는 특별한 주소입니다.
> - **`5000`**: **포트 번호**입니다.
>
> IP 주소가 "건물 주소"라면, 포트 번호는 "몇 호실"입니다:
>
> | 포트 | 용도 |
> | --- | --- |
> | `80` | HTTP (웹사이트) |
> | `443` | HTTPS (보안 웹사이트) |
> | `5432` | PostgreSQL (이전 가이드에서 사용!) |
> | `5000` | Flask 기본 포트 |

> 💡 **컴퓨터 공학 개념 — 코드 설명**
>
> ```python
> @app.route("/")       # 데코레이터: "/" URL로 접속하면
> def home():           # 이 함수를 실행하고
>     return "Hello!"   # 이 텍스트를 브라우저에 보여줘
> ```
>
> `@app.route("/")`는 파이썬의 **데코레이터(Decorator)**라는 문법입니다. "이 함수를 `/` 경로(URL)에 연결해줘"라는 뜻입니다.

---

## Part 2: 여러 페이지(URL) 만들기

하나의 웹 서버에 여러 URL(페이지)을 추가할 수 있습니다.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "🏠 홈페이지입니다!"

@app.route("/about")
def about():
    return "ℹ️ 이 사이트는 파이썬 학습용입니다."

@app.route("/menu")
def menu():
    return "☕ 메뉴: 아메리카노, 카페라떼, 녹차"

@app.route("/hello/<name>")
def hello(name):
    return f"👋 안녕하세요, {name}님!"

if __name__ == "__main__":
    app.run(debug=True)
```

서버를 다시 시작하고 (`Ctrl+C`로 종료 후 `python app.py`) 각 URL을 방문해 보세요:

| URL | 결과 |
| --- | --- |
| `http://localhost:5000/` | 🏠 홈페이지입니다! |
| `http://localhost:5000/about` | ℹ️ 이 사이트는 파이썬 학습용입니다. |
| `http://localhost:5000/menu` | ☕ 메뉴: 아메리카노, 카페라떼, 녹차 |
| `http://localhost:5000/hello/철수` | 👋 안녕하세요, 철수님! |

> 💡 **컴퓨터 공학 개념 — 라우팅(Routing)**
>
> URL에 따라 다른 함수를 실행하는 것을 **라우팅(Routing)**이라고 합니다. `<name>` 처럼 꺾쇠(`< >`)로 감싸면 **동적 라우트**가 됩니다. URL의 해당 부분이 변수가 되어 함수의 인자로 전달됩니다.

> 💡 **컴퓨터 공학 개념 — HTTP 메서드(Method)**
>
> | HTTP 메서드 | 용도 | CRUD |
> | --- | --- | --- |
> | **GET** | 데이터를 **읽기** | Read |
> | **POST** | 데이터를 **보내기** | Create |
> | **PUT** | 데이터를 **수정** | Update |
> | **DELETE** | 데이터를 **삭제** | Delete |
>
> 브라우저 주소창에 URL을 입력하면 항상 **GET** 요청이 보내집니다.

---

## Part 3: HTML 페이지 보여주기

지금까지는 단순 텍스트만 보여줬습니다. 이제 **진짜 웹 페이지(HTML)**를 만들어 보겠습니다.

### 1. HTML을 직접 반환하기

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>나의 카페</title>
        <style>
            body { font-family: Arial; background: #f5f0eb; text-align: center; padding: 50px; }
            h1 { color: #6b4226; }
            .menu-item { background: white; border-radius: 10px; padding: 15px;
                         margin: 10px auto; max-width: 300px; }
        </style>
    </head>
    <body>
        <h1>☕ 나의 카페</h1>
        <div class="menu-item">🫘 아메리카노 - 4,500원</div>
        <div class="menu-item">🥛 카페라떼 - 5,000원</div>
        <div class="menu-item">🍵 녹차 - 4,000원</div>
    </body>
    </html>
    """

if __name__ == "__main__":
    app.run(debug=True)
```

> 💡 **컴퓨터 공학 개념 — HTML, CSS, JavaScript (웹의 3대 요소)**
>
> | 기술 | 역할 | 비유 |
> | --- | --- | --- |
> | **HTML** | 페이지의 **구조와 내용** | 건물의 뼈대 |
> | **CSS** | 페이지의 **디자인** | 건물의 인테리어 |
> | **JavaScript** | 페이지의 **동작** | 건물의 전기/수도 |

### 2. 템플릿 파일 분리하기 (깔끔한 방법)

HTML 코드를 파이썬 파일 안에 넣으면 지저분해집니다. Flask는 **템플릿(Template)** 기능을 제공합니다.

#### Step 1: 폴더 구조 만들기

```
my_cafe/
├── app.py
└── templates/
    └── home.html
```

#### Step 2: HTML 템플릿 파일 만들기 (`my_cafe/templates/home.html`)

```html
<!DOCTYPE html>
<html>
<head>
    <title>나의 카페</title>
    <style>
        body { font-family: Arial; background: #f5f0eb; text-align: center; padding: 50px; }
        h1 { color: #6b4226; }
        .menu-item { background: white; border-radius: 10px; padding: 15px;
                     margin: 10px auto; max-width: 300px; }
    </style>
</head>
<body>
    <h1>☕ 나의 카페</h1>
    {% for item in menu %}
    <div class="menu-item">{{ item.name }} - {{ item.price }}원</div>
    {% endfor %}
    <p>총 <strong>{{ menu|length }}개</strong>의 메뉴가 있습니다.</p>
</body>
</html>
```

#### Step 3: Flask에서 템플릿 사용하기 (`my_cafe/app.py`)

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    menu = [
        {"name": "🫘 아메리카노", "price": "4,500"},
        {"name": "🥛 카페라떼", "price": "5,000"},
        {"name": "🍵 녹차", "price": "4,000"},
    ]
    return render_template("home.html", menu=menu)

if __name__ == "__main__":
    app.run(debug=True)
```

> 💡 **컴퓨터 공학 개념 — 템플릿 엔진(Template Engine)**
>
> `{{ item.name }}`이나 `{% for %}` 같은 문법은 **Jinja2**라는 템플릿 엔진의 문법입니다.
>
> | Jinja2 문법 | 의미 | 예시 |
> | --- | --- | --- |
> | `{{ 변수 }}` | 변수의 값 출력 | `{{ item.name }}` → 아메리카노 |
> | `{% for %}` | 반복문 | 메뉴 목록을 하나씩 표시 |
> | `{% if %}` | 조건문 | 특정 조건일 때만 표시 |
>
> 템플릿 엔진의 장점은 **파이썬(로직)과 HTML(디자인)을 분리**할 수 있다는 것입니다. 이것을 **관심사의 분리(Separation of Concerns)**라고 합니다.

---

### 🛠️ 전체 요약

| 개념 | 설명 |
| --- | --- |
| **Flask** | 파이썬 웹 프레임워크 |
| **라우팅** | URL과 함수를 연결 (`@app.route`) |
| **동적 라우트** | URL의 일부를 변수로 사용 (`<name>`) |
| **템플릿** | HTML과 파이썬 데이터를 결합 (Jinja2) |

---

### 📚 학습 흐름

```
📘 가이드 1: Docker + PostgreSQL     → DB 설치 및 SQL 기초
📘 가이드 2: 데이터 수정 & 삭제       → UPDATE, DELETE
📘 가이드 3: Python + PostgreSQL     → 파이썬으로 CRUD
📘 가이드 4: Flask 기초 (이번!)       → 웹 서버 만들기
📘 가이드 5: Flask + DB 연동          → 다음 가이드!
```

---
