
---

## 📄 초보자를 위한 Python 웹 서버 + 데이터베이스 연동 가이드

이 문서는 이전 가이드들([Docker + PostgreSQL](docker+postgres.md), [데이터 수정 & 삭제](postgres-update-delete.md), [Python CRUD](python-postgres-crud.md))에서 배운 내용을 바탕으로, **파이썬으로 웹 서버를 만들고**, 나아가 **데이터베이스의 데이터를 웹 페이지에 보여주는 방법**을 학습합니다.

---

### 🎯 이번에 배울 것

| 파트 | 내용 | 결과 |
| --- | --- | --- |
| **Part 1** | 가장 간단한 웹 서버 만들기 | 브라우저에서 "Hello World" 보기 |
| **Part 2** | 여러 페이지(URL) 만들기 | 각 URL마다 다른 응답 |
| **Part 3** | HTML 페이지 보여주기 | 예쁜 웹 페이지 만들기 |
| **Part 4** | 데이터베이스 연동 | DB 데이터를 웹 페이지에 표시 |

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
> - **`localhost`** (= `127.0.0.1`): "내 컴퓨터 자신"을 가리키는 특별한 주소입니다. 다른 컴퓨터가 아니라 **지금 이 컴퓨터**를 뜻합니다.
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
> `@app.route("/")`는 파이썬의 **데코레이터(Decorator)**라는 문법입니다. "이 함수를 `/` 경로(URL)에 연결해줘"라는 뜻입니다. 마치 식당에서 "1번 테이블 주문은 이 요리사가 담당해!"라고 배정하는 것과 같습니다.

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
> URL에 따라 다른 함수를 실행하는 것을 **라우팅(Routing)**이라고 합니다.
>
> ```
> URL 경로              →    실행될 함수
> ─────────────────          ──────────
> /                    →    home()
> /about               →    about()
> /menu                →    menu()
> /hello/<name>        →    hello(name)
> ```
>
> `<name>` 처럼 꺾쇠(`< >`)로 감싸면 **동적 라우트**가 됩니다. URL의 해당 부분이 변수가 되어 함수의 인자로 전달됩니다. 마치 택배 주소에서 "OO동 XX호"처럼, URL의 일부가 변하는 것입니다!

> 💡 **컴퓨터 공학 개념 — HTTP 메서드(Method)**
>
> 브라우저가 서버에 요청할 때, **어떤 종류의 요청**인지도 함께 보냅니다:
>
> | HTTP 메서드 | 용도 | 비유 |
> | --- | --- | --- |
> | **GET** | 데이터를 **읽기** (페이지 보기) | 메뉴판 보여주세요 |
> | **POST** | 데이터를 **보내기** (회원가입, 글쓰기) | 주문합니다 |
> | **PUT** | 데이터를 **수정** | 주문 변경합니다 |
> | **DELETE** | 데이터를 **삭제** | 주문 취소합니다 |
>
> 이전 가이드의 **CRUD**와 대응됩니다!
> - **C**reate → POST
> - **R**ead → GET
> - **U**pdate → PUT
> - **D**elete → DELETE
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
            body {
                font-family: Arial, sans-serif;
                background-color: #f5f0eb;
                text-align: center;
                padding: 50px;
            }
            h1 { color: #6b4226; }
            .menu-item {
                background: white;
                border-radius: 10px;
                padding: 15px;
                margin: 10px auto;
                max-width: 300px;
                box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            }
        </style>
    </head>
    <body>
        <h1>☕ 나의 카페</h1>
        <p>따뜻한 커피 한 잔의 여유</p>
        <div class="menu-item">🫘 아메리카노 - 4,500원</div>
        <div class="menu-item">🥛 카페라떼 - 5,000원</div>
        <div class="menu-item">🍵 녹차 - 4,000원</div>
    </body>
    </html>
    """

if __name__ == "__main__":
    app.run(debug=True)
```

브라우저에서 `http://localhost:5000`에 접속하면, 예쁜 카페 페이지가 보입니다!

> 💡 **컴퓨터 공학 개념 — HTML, CSS, JavaScript (웹의 3대 요소)**
>
> 웹 페이지는 3가지 기술로 만들어집니다:
>
> | 기술 | 역할 | 비유 |
> | --- | --- | --- |
> | **HTML** | 페이지의 **구조와 내용** | 건물의 뼈대 (벽, 기둥) |
> | **CSS** | 페이지의 **디자인** | 건물의 인테리어 (페인트, 가구) |
> | **JavaScript** | 페이지의 **동작** | 건물의 전기/수도 (버튼 클릭, 애니메이션) |
>
> 지금 위 코드에서 `<style>` 안에 있는 것이 CSS입니다. HTML로 내용을 만들고, CSS로 예쁘게 꾸민 것입니다!

### 2. 템플릿 파일 분리하기 (깔끔한 방법)

HTML 코드를 파이썬 파일 안에 넣으면 코드가 지저분해집니다. Flask는 **템플릿(Template)** 기능을 제공합니다.

#### Step 1: 폴더 구조 만들기

```
my_cafe/
├── app.py
└── templates/
    └── home.html
```

```bash
mkdir -p my_cafe/templates
```

#### Step 2: HTML 템플릿 파일 만들기

`my_cafe/templates/home.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>나의 카페</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f0eb;
            text-align: center;
            padding: 50px;
        }
        h1 { color: #6b4226; font-size: 2.5em; }
        p { color: #8b6914; font-size: 1.2em; }
        .menu-list {
            max-width: 400px;
            margin: 30px auto;
        }
        .menu-item {
            background: white;
            border-radius: 10px;
            padding: 20px;
            margin: 10px 0;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .menu-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .item-name { font-weight: bold; color: #333; }
        .item-price { color: #6b4226; font-weight: bold; }
    </style>
</head>
<body>
    <h1>☕ 나의 카페</h1>
    <p>따뜻한 커피 한 잔의 여유</p>

    <div class="menu-list">
        {% for item in menu %}
        <div class="menu-item">
            <span class="item-name">{{ item.name }}</span>
            <span class="item-price">{{ item.price }}원</span>
        </div>
        {% endfor %}
    </div>

    <p>총 <strong>{{ menu|length }}개</strong>의 메뉴가 있습니다.</p>
</body>
</html>
```

#### Step 3: Flask에서 템플릿 사용하기

`my_cafe/app.py`:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    # 메뉴 데이터 (나중에 DB에서 가져올 것!)
    menu = [
        {"name": "🫘 아메리카노", "price": "4,500"},
        {"name": "🥛 카페라떼", "price": "5,000"},
        {"name": "🍵 녹차", "price": "4,000"},
        {"name": "🍰 초코케이크", "price": "6,500"},
    ]
    return render_template("home.html", menu=menu)

if __name__ == "__main__":
    app.run(debug=True)
```

```bash
cd my_cafe
python app.py
```

> 💡 **컴퓨터 공학 개념 — 템플릿 엔진(Template Engine)**
>
> `{{ item.name }}`이나 `{% for %}` 같은 문법을 눈치채셨나요? 이것은 **Jinja2**라는 템플릿 엔진의 문법입니다.
>
> ```
> Python 데이터  +  HTML 템플릿  →  완성된 웹 페이지
>   (menu 리스트)    (home.html)     (브라우저에 보여지는 것)
> ```
>
> | Jinja2 문법 | 의미 | 예시 |
> | --- | --- | --- |
> | `{{ 변수 }}` | 변수의 값 출력 | `{{ item.name }}` → 아메리카노 |
> | `{% for %}` | 반복문 | 메뉴 목록을 하나씩 표시 |
> | `{% if %}` | 조건문 | 특정 조건일 때만 표시 |
> | `{{ 리스트\|length }}` | 필터 (가공) | 리스트의 길이 출력 |
>
> 템플릿 엔진의 장점은 **파이썬(로직)과 HTML(디자인)을 분리**할 수 있다는 것입니다. 이것을 **관심사의 분리(Separation of Concerns)**라고 합니다. 요리사(파이썬)는 음식을 만들고, 서빙 직원(HTML)은 손님에게 예쁘게 내놓는 것처럼요!

---

## Part 4: 데이터베이스 연동하기 🔥

드디어 핵심입니다! 이전 가이드에서 배운 **PostgreSQL 데이터베이스**와 **Flask 웹 서버**를 연결합니다.

### 목표

```
브라우저 요청  →  Flask 서버  →  PostgreSQL DB  →  데이터 반환  →  웹 페이지에 표시
```

### 1. 사전 준비

#### Step 1: Docker PostgreSQL 실행 확인

```bash
docker start my_postgres
```

#### Step 2: 필요한 라이브러리 설치

```bash
pip install flask psycopg2-binary
```

#### Step 3: 데이터베이스에 샘플 데이터 넣기

먼저 `psql`로 접속해서 데이터를 준비합니다:

```bash
docker exec -it my_postgres psql -U postgres
```

```sql
-- 기존 테이블 정리
DROP TABLE IF EXISTS menu;

-- 메뉴 테이블 만들기
CREATE TABLE menu (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price INTEGER NOT NULL,
    description TEXT,
    is_available BOOLEAN DEFAULT TRUE
);

-- 샘플 데이터 넣기
INSERT INTO menu (name, category, price, description, is_available)
VALUES
    ('아메리카노', '커피', 4500, '깊고 진한 에스프레소의 풍미', TRUE),
    ('카페라떼', '커피', 5000, '부드러운 우유와 에스프레소의 조화', TRUE),
    ('바닐라라떼', '커피', 5500, '달콤한 바닐라 시럽이 들어간 라떼', TRUE),
    ('녹차', '차', 4000, '제주산 녹차잎을 우려낸 건강한 차', TRUE),
    ('캐모마일', '차', 4000, '은은한 꽃향기의 허브티', TRUE),
    ('아이스티', '차', 3500, '시원하고 상큼한 복숭아 아이스티', FALSE),
    ('초코케이크', '디저트', 6500, '진한 벨기에 초콜릿 케이크', TRUE),
    ('크로와상', '디저트', 4000, '바삭한 프랑스식 크로와상', TRUE),
    ('치즈케이크', '디저트', 7000, '뉴욕 스타일 크림치즈 케이크', FALSE);

-- 확인
SELECT * FROM menu;
```

`\q`로 psql을 종료하세요.

### 2. 프로젝트 구조 만들기

```
my_cafe_db/
├── app.py
└── templates/
    ├── base.html
    ├── home.html
    ├── menu.html
    └── category.html
```

```bash
mkdir -p my_cafe_db/templates
```

### 3. 데이터베이스 연결 + Flask 서버 작성

`my_cafe_db/app.py`:

```python
import psycopg2
from flask import Flask, render_template

app = Flask(__name__)


# ──────────────────────────────────────
# 데이터베이스 연결 함수
# ──────────────────────────────────────
def get_db_connection():
    """PostgreSQL에 연결하고 connection 객체를 반환합니다."""
    conn = psycopg2.connect(
        host="localhost",
        port="5432",
        database="postgres",
        user="postgres",
        password="mysecretpassword"
    )
    return conn


# ──────────────────────────────────────
# 라우트 (URL → 함수 연결)
# ──────────────────────────────────────

@app.route("/")
def home():
    """홈 페이지: 메뉴 개수와 카테고리 요약을 보여줍니다."""
    conn = get_db_connection()
    cur = conn.cursor()

    # 전체 메뉴 수
    cur.execute("SELECT COUNT(*) FROM menu;")
    total = cur.fetchone()[0]

    # 카테고리별 메뉴 수
    cur.execute("""
        SELECT category, COUNT(*) as cnt
        FROM menu
        GROUP BY category
        ORDER BY category;
    """)
    categories = cur.fetchall()

    cur.close()
    conn.close()

    return render_template("home.html", total=total, categories=categories)


@app.route("/menu")
def menu_list():
    """전체 메뉴 목록 페이지"""
    conn = get_db_connection()
    cur = conn.cursor()

    cur.execute("""
        SELECT id, name, category, price, description, is_available
        FROM menu
        ORDER BY category, name;
    """)
    items = cur.fetchall()

    cur.close()
    conn.close()

    return render_template("menu.html", items=items)


@app.route("/menu/<category>")
def menu_by_category(category):
    """카테고리별 메뉴 페이지"""
    conn = get_db_connection()
    cur = conn.cursor()

    cur.execute("""
        SELECT id, name, category, price, description, is_available
        FROM menu
        WHERE category = %s
        ORDER BY name;
    """, (category,))
    items = cur.fetchall()

    cur.close()
    conn.close()

    return render_template("category.html", category=category, items=items)


@app.route("/api/menu")
def api_menu():
    """API 엔드포인트: JSON으로 메뉴 데이터 반환"""
    conn = get_db_connection()
    cur = conn.cursor()

    cur.execute("SELECT id, name, category, price, is_available FROM menu;")
    rows = cur.fetchall()

    cur.close()
    conn.close()

    # 딕셔너리 리스트로 변환
    menu_data = []
    for row in rows:
        menu_data.append({
            "id": row[0],
            "name": row[1],
            "category": row[2],
            "price": row[3],
            "is_available": row[4]
        })

    return {"menu": menu_data}


if __name__ == "__main__":
    app.run(debug=True)
```

> 💡 **컴퓨터 공학 개념 — API (Application Programming Interface)**
>
> 마지막 라우트 `/api/menu`는 **API 엔드포인트**입니다. HTML 페이지 대신 **JSON** 데이터를 반환합니다.
>
> ```json
> {
>   "menu": [
>     {"id": 1, "name": "아메리카노", "price": 4500, ...},
>     {"id": 2, "name": "카페라떼", "price": 5000, ...}
>   ]
> }
> ```
>
> **왜 API가 필요할까요?**
>
> - **웹 페이지**: 사람이 브라우저로 보는 용도
> - **API**: 프로그램이 데이터를 가져가는 용도
>
> 예를 들어 카카오맵 앱은 네이버 지도 **웹사이트**를 열지 않습니다. 대신 지도 **API**에서 데이터만 가져옵니다. 모바일 앱, 다른 서버, AI 챗봇 등이 모두 API를 통해 데이터를 주고받습니다.

### 4. HTML 템플릿 파일 만들기

#### `my_cafe_db/templates/base.html` (공통 레이아웃)

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>☕ 나의 카페 - {% block title %}{% endblock %}</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Apple SD Gothic Neo', 'Malgun Gothic', Arial, sans-serif;
            background-color: #faf6f1;
            color: #333;
        }

        /* 네비게이션 바 */
        nav {
            background: linear-gradient(135deg, #6b4226, #8b6914);
            padding: 15px 30px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        nav .logo { color: white; font-size: 1.5em; font-weight: bold; text-decoration: none; }
        nav .links a {
            color: rgba(255,255,255,0.9);
            text-decoration: none;
            margin-left: 20px;
            font-size: 1em;
        }
        nav .links a:hover { color: white; text-decoration: underline; }

        /* 메인 콘텐츠 */
        .container {
            max-width: 800px;
            margin: 40px auto;
            padding: 0 20px;
        }

        /* 카드 스타일 */
        .card {
            background: white;
            border-radius: 12px;
            padding: 20px 25px;
            margin: 15px 0;
            box-shadow: 0 2px 10px rgba(0,0,0,0.08);
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .card:hover {
            transform: translateY(-3px);
            box-shadow: 0 4px 15px rgba(0,0,0,0.12);
        }
        .card-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        /* 상태 배지 */
        .badge {
            padding: 4px 10px;
            border-radius: 20px;
            font-size: 0.8em;
            font-weight: bold;
        }
        .badge-available { background: #e6f9e6; color: #2d8a2d; }
        .badge-sold-out { background: #fde8e8; color: #c53030; }

        /* 카테고리 링크 */
        .category-link {
            display: inline-block;
            background: white;
            border-radius: 10px;
            padding: 15px 25px;
            margin: 8px;
            text-decoration: none;
            color: #6b4226;
            font-weight: bold;
            box-shadow: 0 2px 5px rgba(0,0,0,0.08);
            transition: all 0.2s;
        }
        .category-link:hover {
            background: #6b4226;
            color: white;
        }

        h1 { color: #6b4226; margin-bottom: 10px; }
        h2 { color: #8b6914; margin-bottom: 20px; }
        p { line-height: 1.6; }
        .subtitle { color: #888; margin-bottom: 30px; }
        .price { color: #6b4226; font-weight: bold; font-size: 1.1em; }
        .description { color: #777; font-size: 0.9em; margin-top: 5px; }

        footer {
            text-align: center;
            padding: 30px;
            color: #aaa;
            font-size: 0.85em;
        }
    </style>
</head>
<body>
    <nav>
        <a href="/" class="logo">☕ 나의 카페</a>
        <div class="links">
            <a href="/">홈</a>
            <a href="/menu">전체 메뉴</a>
            <a href="/api/menu">API</a>
        </div>
    </nav>

    <div class="container">
        {% block content %}{% endblock %}
    </div>

    <footer>
        Python + Flask + PostgreSQL 학습 프로젝트
    </footer>
</body>
</html>
```

#### `my_cafe_db/templates/home.html` (홈 페이지)

```html
{% extends "base.html" %}
{% block title %}홈{% endblock %}

{% block content %}
<h1>☕ 나의 카페에 오신 걸 환영합니다!</h1>
<p class="subtitle">데이터베이스에서 가져온 실시간 메뉴 정보</p>

<div class="card">
    <h2>📊 메뉴 현황</h2>
    <p>현재 총 <strong>{{ total }}개</strong>의 메뉴가 등록되어 있습니다.</p>
</div>

<h2>📂 카테고리별 보기</h2>
<div style="text-align: center;">
    {% for cat in categories %}
    <a href="/menu/{{ cat[0] }}" class="category-link">
        {% if cat[0] == '커피' %}☕{% elif cat[0] == '차' %}🍵{% else %}🍰{% endif %}
        {{ cat[0] }} ({{ cat[1] }}개)
    </a>
    {% endfor %}
</div>
{% endblock %}
```

#### `my_cafe_db/templates/menu.html` (전체 메뉴)

```html
{% extends "base.html" %}
{% block title %}전체 메뉴{% endblock %}

{% block content %}
<h1>📋 전체 메뉴</h1>
<p class="subtitle">데이터베이스에서 가져온 {{ items|length }}개의 메뉴</p>

{% for item in items %}
<div class="card">
    <div class="card-row">
        <div>
            <strong style="font-size: 1.1em;">{{ item[1] }}</strong>
            <span style="color: #aaa; margin-left: 8px;">{{ item[2] }}</span>
        </div>
        <div style="display: flex; align-items: center; gap: 10px;">
            <span class="price">{{ "{:,}".format(item[3]) }}원</span>
            {% if item[5] %}
                <span class="badge badge-available">판매중</span>
            {% else %}
                <span class="badge badge-sold-out">품절</span>
            {% endif %}
        </div>
    </div>
    {% if item[4] %}
    <p class="description">{{ item[4] }}</p>
    {% endif %}
</div>
{% endfor %}
{% endblock %}
```

#### `my_cafe_db/templates/category.html` (카테고리별 메뉴)

```html
{% extends "base.html" %}
{% block title %}{{ category }}{% endblock %}

{% block content %}
<h1>
    {% if category == '커피' %}☕{% elif category == '차' %}🍵{% else %}🍰{% endif %}
    {{ category }} 메뉴
</h1>
<p class="subtitle">{{ items|length }}개의 {{ category }} 메뉴가 있습니다</p>

{% if items %}
    {% for item in items %}
    <div class="card">
        <div class="card-row">
            <strong>{{ item[1] }}</strong>
            <div style="display: flex; align-items: center; gap: 10px;">
                <span class="price">{{ "{:,}".format(item[3]) }}원</span>
                {% if item[5] %}
                    <span class="badge badge-available">판매중</span>
                {% else %}
                    <span class="badge badge-sold-out">품절</span>
                {% endif %}
            </div>
        </div>
        {% if item[4] %}
        <p class="description">{{ item[4] }}</p>
        {% endif %}
    </div>
    {% endfor %}
{% else %}
    <div class="card">
        <p>이 카테고리에 등록된 메뉴가 없습니다.</p>
    </div>
{% endif %}

<p style="margin-top: 20px;"><a href="/menu">← 전체 메뉴로 돌아가기</a></p>
{% endblock %}
```

### 5. 실행 및 확인!

```bash
cd my_cafe_db
python app.py
```

이제 브라우저에서 아래 URL들을 방문해 보세요:

| URL | 설명 |
| --- | --- |
| `http://localhost:5000/` | 홈 — DB에서 카테고리 요약 표시 |
| `http://localhost:5000/menu` | 전체 메뉴 — DB의 모든 메뉴 표시 |
| `http://localhost:5000/menu/커피` | 커피 카테고리만 필터 |
| `http://localhost:5000/menu/디저트` | 디저트 카테고리만 필터 |
| `http://localhost:5000/api/menu` | JSON API — 프로그램용 데이터 |

🎉 **축하합니다!** 데이터베이스의 데이터가 웹 페이지에 표시됩니다!

> 💡 **컴퓨터 공학 개념 — 3-Tier 아키텍처 (3계층 구조)**
>
> 방금 만든 것은 실무에서 사용하는 **3-Tier(3계층) 아키텍처**의 기본 형태입니다!
>
> ```
> ┌──────────────────┐
> │   Presentation   │  ← 1계층: 사용자가 보는 부분
> │  (HTML/CSS/JS)   │     templates/ 폴더의 HTML 파일들
> └────────┬─────────┘
>          │
> ┌────────▼─────────┐
> │   Application    │  ← 2계층: 비즈니스 로직
> │    (Flask/Python) │     app.py의 라우트 함수들
> └────────┬─────────┘
>          │
> ┌────────▼─────────┐
> │     Database     │  ← 3계층: 데이터 저장
> │   (PostgreSQL)   │     Docker로 실행 중인 DB
> └──────────────────┘
> ```
>
> 거의 모든 웹 서비스(네이버, 쿠팡, 인스타그램)가 이 구조를 따릅니다!
>
> | 계층 | 이 프로젝트에서 | 실무 예시 |
> | --- | --- | --- |
> | Presentation | `templates/*.html` | React, Vue.js |
> | Application | `app.py` (Flask) | Django, Spring |
> | Database | PostgreSQL (Docker) | PostgreSQL, MySQL, MongoDB |

> 💡 **컴퓨터 공학 개념 — 데이터의 여행 (Request-Response 사이클)**
>
> 사용자가 `/menu/커피`에 접속하면, 데이터는 이렇게 여행합니다:
>
> ```
> 1. 브라우저 → Flask: "GET /menu/커피 주세요!"
>        ↓
> 2. Flask → @app.route("/menu/<category>") 찾기
>        ↓
> 3. Flask → PostgreSQL: "SELECT * FROM menu WHERE category = '커피'"
>        ↓
> 4. PostgreSQL → Flask: [{아메리카노, 4500}, {카페라떼, 5000}, ...]
>        ↓
> 5. Flask → Jinja2 템플릿에 데이터 전달
>        ↓
> 6. Jinja2 → 완성된 HTML 생성
>        ↓
> 7. Flask → 브라우저: 완성된 HTML 전송
>        ↓
> 8. 브라우저 → 화면에 예쁜 웹 페이지 표시!
> ```

---

### 6. 🎁 보너스: DB에 데이터 추가하면 웹에 바로 반영!

데이터베이스에 새 메뉴를 추가하면, **서버를 재시작할 필요 없이** 웹 페이지에 바로 반영됩니다!

```bash
# 다른 터미널에서 실행 (Flask 서버는 계속 켜놔두세요!)
docker exec -it my_postgres psql -U postgres
```

```sql
INSERT INTO menu (name, category, price, description)
VALUES ('딸기스무디', '음료', 6000, '신선한 딸기로 만든 스무디');
```

브라우저에서 `http://localhost:5000/menu`를 **새로고침**하면 딸기스무디가 추가된 것을 볼 수 있습니다!

> 💡 **왜 서버 재시작 없이 반영될까요?**
>
> 매 요청마다 `get_db_connection()`으로 **최신 데이터**를 DB에서 가져오기 때문입니다. 이것이 데이터베이스의 큰 장점입니다!
>
> - **파일에 저장**: 파일을 수정하면 프로그램도 수정해야 할 수 있음
> - **DB에 저장**: 데이터만 바꾸면 프로그램은 그대로, 웹 페이지는 자동 반영!

---

### 7. 🛠️ 전체 요약

#### 이번 가이드에서 배운 것

| 개념 | 설명 |
| --- | --- |
| **Flask** | 파이썬 웹 프레임워크 (웹 서버 만들기) |
| **라우팅** | URL과 함수를 연결 (`@app.route`) |
| **템플릿** | HTML과 파이썬 데이터를 결합 (Jinja2) |
| **DB 연동** | Flask에서 PostgreSQL 데이터를 읽어 웹 페이지에 표시 |
| **API** | JSON 형태로 데이터를 반환하는 엔드포인트 |
| **3-Tier 구조** | Presentation → Application → Database |

#### 학습 흐름 정리

```
📘 가이드 1: Docker + PostgreSQL     → DB 설치 및 SQL 기초
📘 가이드 2: 데이터 수정 & 삭제       → UPDATE, DELETE
📘 가이드 3: Python + PostgreSQL     → 파이썬으로 CRUD
📘 가이드 4: Python 웹 서버 (이번!)   → Flask + DB 연동 → 웹 페이지에 표시
```

---

### 📚 다음 단계

이 가이드를 마쳤다면 아래 주제를 공부해 보세요:

- **폼(Form) 처리**: 웹 페이지에서 사용자 입력을 받아 DB에 저장하기 (POST 요청)
- **ORM (SQLAlchemy)**: SQL을 직접 쓰지 않고 파이썬 객체로 DB를 다루는 방법
- **사용자 인증**: 로그인/회원가입 구현하기
- **배포(Deploy)**: 만든 사이트를 인터넷에 공개하기 (Heroku, AWS, Docker)

---
