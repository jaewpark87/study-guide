
---

## 📄 초보자를 위한 Flask + 데이터베이스 연동 가이드

이 문서는 이전 가이드([Flask 기초](04-python-flask-basics.md))에서 배운 웹 서버에 **PostgreSQL 데이터베이스**를 연결하여, **DB 데이터를 웹 페이지에 표시**하는 방법을 학습합니다.

---

### 🎯 이번에 배울 것

| 순서 | 내용 | 결과 |
| --- | --- | --- |
| 1 | DB에 샘플 데이터 준비 | 메뉴 테이블 만들기 |
| 2 | Flask에서 DB 연결 | 데이터 읽어오기 |
| 3 | 웹 페이지에 데이터 표시 | HTML 템플릿 + DB |
| 4 | JSON API 만들기 | 프로그램용 데이터 제공 |

### 목표

```
브라우저 요청  -->  Flask 서버  -->  PostgreSQL DB  -->  데이터 반환  -->  웹 페이지에 표시
```

---

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

```bash
docker exec -it my_postgres psql -U postgres
```

```sql
DROP TABLE IF EXISTS menu;

CREATE TABLE menu (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price INTEGER NOT NULL,
    description TEXT,
    is_available BOOLEAN DEFAULT TRUE
);

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

SELECT * FROM menu;
```

`\q`로 psql을 종료하세요.

---

### 2. 프로젝트 구조

```
my_cafe_db/
├── app.py
└── templates/
    ├── base.html
    └── menu.html
```

```bash
mkdir -p my_cafe_db/templates
```

---

### 3. Flask 서버 코드 작성

`my_cafe_db/app.py`:

```python
import psycopg2
from flask import Flask, render_template

app = Flask(__name__)


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


@app.route("/")
def home():
    """홈 페이지: 전체 메뉴를 보여줍니다."""
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
        FROM menu WHERE category = %s ORDER BY name;
    """, (category,))
    items = cur.fetchall()

    cur.close()
    conn.close()

    return render_template("menu.html", items=items, category=category)


@app.route("/api/menu")
def api_menu():
    """API 엔드포인트: JSON으로 메뉴 데이터 반환"""
    conn = get_db_connection()
    cur = conn.cursor()

    cur.execute("SELECT id, name, category, price, is_available FROM menu;")
    rows = cur.fetchall()

    cur.close()
    conn.close()

    menu_data = []
    for row in rows:
        menu_data.append({
            "id": row[0], "name": row[1], "category": row[2],
            "price": row[3], "is_available": row[4]
        })

    return {"menu": menu_data}


if __name__ == "__main__":
    app.run(debug=True)
```

> 💡 **컴퓨터 공학 개념 — API (Application Programming Interface)**
>
> `/api/menu`는 **API 엔드포인트**입니다. HTML 대신 **JSON** 데이터를 반환합니다.
>
> - **웹 페이지** (`/`): 사람이 브라우저로 보는 용도
> - **API** (`/api/menu`): 프로그램이 데이터를 가져가는 용도
>
> 모바일 앱, 다른 서버, AI 챗봇 등이 모두 API를 통해 데이터를 주고받습니다.

---

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
        body { font-family: Arial, sans-serif; background: #faf6f1; color: #333; }
        nav { background: #6b4226; padding: 15px 30px; display: flex;
              justify-content: space-between; align-items: center; }
        nav a { color: white; text-decoration: none; font-weight: bold; }
        nav .links a { margin-left: 20px; font-weight: normal; }
        .container { max-width: 800px; margin: 40px auto; padding: 0 20px; }
        .card { background: white; border-radius: 12px; padding: 20px;
                margin: 15px 0; box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
        .card-row { display: flex; justify-content: space-between; align-items: center; }
        .badge { padding: 4px 10px; border-radius: 20px; font-size: 0.8em; font-weight: bold; }
        .badge-available { background: #e6f9e6; color: #2d8a2d; }
        .badge-sold-out { background: #fde8e8; color: #c53030; }
        h1 { color: #6b4226; margin-bottom: 20px; }
        .price { color: #6b4226; font-weight: bold; }
        .description { color: #777; font-size: 0.9em; margin-top: 5px; }
        footer { text-align: center; padding: 30px; color: #aaa; font-size: 0.85em; }
    </style>
</head>
<body>
    <nav>
        <a href="/">☕ 나의 카페</a>
        <div class="links">
            <a href="/">홈</a>
            <a href="/api/menu">API</a>
        </div>
    </nav>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
    <footer>Python + Flask + PostgreSQL 학습 프로젝트</footer>
</body>
</html>
```

#### `my_cafe_db/templates/menu.html` (메뉴 목록)

```html
{% extends "base.html" %}
{% block title %}메뉴{% endblock %}

{% block content %}
<h1>📋 {% if category %}{{ category }} 메뉴{% else %}전체 메뉴{% endif %}</h1>
<p style="color: #888; margin-bottom: 20px;">
    데이터베이스에서 가져온 {{ items|length }}개의 메뉴
</p>

{% for item in items %}
<div class="card">
    <div class="card-row">
        <div>
            <strong>{{ item[1] }}</strong>
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

---

### 5. 실행 및 확인!

```bash
cd my_cafe_db
python app.py
```

| URL | 설명 |
| --- | --- |
| `http://localhost:5000/` | 전체 메뉴 — DB의 모든 메뉴 표시 |
| `http://localhost:5000/menu/커피` | 커피 카테고리만 필터 |
| `http://localhost:5000/menu/디저트` | 디저트 카테고리만 필터 |
| `http://localhost:5000/api/menu` | JSON API — 프로그램용 데이터 |

🎉 **축하합니다!** 데이터베이스의 데이터가 웹 페이지에 표시됩니다!

> 💡 **컴퓨터 공학 개념 — 3-Tier 아키텍처 (3계층 구조)**
>
> 방금 만든 것은 실무에서 사용하는 **3-Tier(3계층) 아키텍처**의 기본 형태입니다!
>
> ```
> +--------------------+
> |   Presentation     |  <- 1계층: 사용자가 보는 부분 (templates/)
> +--------------------+
>           |
>           v
> +--------------------+
> |   Application      |  <- 2계층: 비즈니스 로직 (app.py)
> +--------------------+
>           |
>           v
> +--------------------+
> |   Database         |  <- 3계층: 데이터 저장 (PostgreSQL)
> +--------------------+
> ```
>
> 거의 모든 웹 서비스(네이버, 쿠팡, 인스타그램)가 이 구조를 따릅니다!

---

### 6. 🎁 보너스: DB에 데이터 추가하면 웹에 바로 반영!

```bash
# 다른 터미널에서 실행 (Flask 서버는 계속 켜놔두세요!)
docker exec -it my_postgres psql -U postgres
```

```sql
INSERT INTO menu (name, category, price, description)
VALUES ('딸기스무디', '음료', 6000, '신선한 딸기로 만든 스무디');
```

브라우저에서 `http://localhost:5000`을 **새로고침**하면 딸기스무디가 추가된 것을 볼 수 있습니다!

> 💡 **왜 서버 재시작 없이 반영될까요?**
>
> 매 요청마다 `get_db_connection()`으로 **최신 데이터**를 DB에서 가져오기 때문입니다. DB에 저장하면 프로그램 수정 없이 웹 페이지가 자동 반영됩니다!

---

### 📚 학습 흐름

```
📘 가이드 1: Docker + PostgreSQL     → DB 설치 및 SQL 기초
📘 가이드 2: 데이터 수정 & 삭제       → UPDATE, DELETE
📘 가이드 3: Python + PostgreSQL     → 파이썬으로 CRUD
📘 가이드 4: Flask 기초              → 웹 서버 만들기
📘 가이드 5: Flask + DB (이번!)      → DB 데이터를 웹에 표시
📘 가이드 6: HTTP & Requests         → 다음 가이드!
```

---
