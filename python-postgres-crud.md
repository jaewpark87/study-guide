
---

## 📄 초보자를 위한 Python + PostgreSQL 완전 정복 가이드

이 문서는 이전 가이드([Docker + PostgreSQL](docker+postgres.md), [데이터 수정 & 삭제](postgres-update-delete.md))에서 배운 환경을 활용하여, **파이썬(Python) 코드로** 데이터베이스의 **모든 작업**(테이블 만들기 → 데이터 넣기 → 읽기 → 수정 → 삭제 → 테이블 삭제)을 수행하는 방법을 학습합니다.

---

### 🎯 이번에 배울 것

| 순서 | 작업 | SQL | Python 메서드 |
| --- | --- | --- | --- |
| 1 | 테이블 만들기 | `CREATE TABLE` | `cursor.execute()` |
| 2 | 데이터 넣기 | `INSERT INTO` | `cursor.execute()` |
| 3 | 데이터 읽기 | `SELECT` | `cursor.fetchall()` |
| 4 | 데이터 수정 | `UPDATE` | `cursor.execute()` |
| 5 | 데이터 삭제 | `DELETE` | `cursor.execute()` |
| 6 | 테이블 삭제 | `DROP TABLE` | `cursor.execute()` |

---

### 0. 사전 준비

#### **Step 1: Docker PostgreSQL이 실행 중인지 확인**

이전 가이드에서 만든 PostgreSQL 컨테이너가 필요합니다.

```bash
# 컨테이너 상태 확인
docker ps

# 멈춰 있다면 다시 시작
docker start my_postgres
```

#### **Step 2: Python 라이브러리 설치**

파이썬에서 PostgreSQL에 접속하려면 `psycopg2`라는 라이브러리가 필요합니다.

```bash
pip install psycopg2-binary
```

> 💡 **컴퓨터 공학 개념 — 드라이버(Driver)**
>
> `psycopg2`는 PostgreSQL용 **데이터베이스 드라이버**입니다. 드라이버란 프로그래밍 언어와 데이터베이스 사이의 **통역사** 역할을 합니다.
>
> ```
> 파이썬 코드  ←→  psycopg2 (드라이버/통역사)  ←→  PostgreSQL 서버
> ```
>
> 마치 한국어만 아는 사람이 영어권 은행에 갈 때, 통역사가 필요한 것과 같습니다. 각 데이터베이스마다 고유한 드라이버가 있습니다:
> - PostgreSQL → `psycopg2`
> - MySQL → `mysql-connector-python`
> - SQLite → `sqlite3` (파이썬에 기본 포함!)

---

### 1. 데이터베이스 연결하기

모든 작업의 시작은 **연결(Connection)**입니다. 아래 코드를 `db_tutorial.py`라는 파일로 저장하세요.

```python
import psycopg2

# PostgreSQL에 연결 (도커에서 실행 중인 DB에 접속)
conn = psycopg2.connect(
    host="localhost",       # DB 서버 주소 (내 컴퓨터)
    port="5432",            # 포트 번호 (도커에서 설정한 것)
    database="postgres",    # 데이터베이스 이름
    user="postgres",        # 사용자 이름
    password="mysecretpassword"  # 도커에서 설정한 비밀번호
)

print("✅ 데이터베이스 연결 성공!")
```

```bash
python db_tutorial.py
```

`✅ 데이터베이스 연결 성공!`이 출력되면 준비 완료입니다!

> 💡 **컴퓨터 공학 개념 — 클라이언트-서버 모델 (Client-Server Model)**
>
> 지금 여러분의 파이썬 프로그램은 **클라이언트(Client)**, PostgreSQL은 **서버(Server)**입니다.
>
> ```
> [Client: Python]  --- Request --->  [Server: PostgreSQL]
>                   <--- Response ---
> ```
>
> 이것은 인터넷의 기본 구조와 동일합니다! 웹 브라우저(클라이언트)가 네이버(서버)에 페이지를 요청하는 것과 같은 원리입니다. 클라이언트는 "나 이 데이터 줘!" 하고 요청하고, 서버는 데이터를 찾아서 응답합니다.

---

### 2. 커서(Cursor) 만들기

연결만으로는 SQL을 실행할 수 없습니다. **커서(Cursor)**를 만들어야 합니다.

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    port="5432",
    database="postgres",
    user="postgres",
    password="mysecretpassword"
)

# 커서 생성
cur = conn.cursor()
print("✅ 커서 생성 완료!")
```

> 💡 **컴퓨터 공학 개념 — 커서(Cursor)**
>
> 커서는 데이터베이스에서 **SQL 명령을 실행하고 결과를 한 줄씩 읽는 포인터**입니다.
>
> 식당에 비유하면:
> - **Connection** = 식당에 입장하기 (문을 열고 들어감)
> - **Cursor** = 웨이터 호출하기 (주문을 주고, 음식을 받는 통로)
>
> 웨이터(커서) 없이는 주문(SQL)을 전달할 수 없습니다!

---

### 3. 테이블 만들기 (CREATE TABLE)

이제 파이썬으로 테이블을 만들어 보겠습니다. 이전 가이드에서 `psql`에서 직접 실행했던 SQL을 이번에는 **파이썬 코드 안에서** 실행합니다.

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    port="5432",
    database="postgres",
    user="postgres",
    password="mysecretpassword"
)
cur = conn.cursor()

# 기존 테이블이 있으면 삭제 (깨끗한 시작을 위해)
cur.execute("DROP TABLE IF EXISTS products;")

# 새 테이블 만들기
cur.execute("""
    CREATE TABLE products (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        category TEXT NOT NULL,
        price INTEGER NOT NULL,
        stock INTEGER DEFAULT 0
    );
""")

# 변경사항을 DB에 확정 (매우 중요!)
conn.commit()
print("✅ products 테이블 생성 완료!")
```

```bash
python db_tutorial.py
```

| 컬럼 | 설명 | 예시 |
| --- | --- | --- |
| **`id`** | 자동 증가하는 고유 번호 | 1, 2, 3... |
| **`name`** | 상품 이름 | '아메리카노', '라떼' |
| **`category`** | 상품 카테고리 | '커피', '음료' |
| **`price`** | 가격 (원) | 4500, 5000 |
| **`stock`** | 재고 수량 (기본값: 0) | 100, 50 |

> 💡 **컴퓨터 공학 개념 — 커밋(Commit)과 트랜잭션(Transaction)**
>
> `conn.commit()`은 왜 필요할까요?
>
> 데이터베이스는 **트랜잭션(Transaction)**이라는 단위로 작업합니다. 트랜잭션은 "다 성공하거나, 다 실패하거나" 하는 **작업 묶음**입니다.
>
> ```
> BEGIN (시작)
>   └─ SQL 명령 1  ✅
>   └─ SQL 명령 2  ✅
>   └─ SQL 명령 3  ❌ 실패!
> ROLLBACK (전부 되돌리기!)
> ```
>
> `commit()`을 호출해야만 변경사항이 **진짜로 DB에 저장**됩니다. 만약 에러가 나면 `rollback()`으로 전부 되돌릴 수 있습니다. 마치 게임에서 "저장" 버튼을 누르는 것과 같습니다!

---

### 4. 데이터 넣기 (INSERT)

테이블에 상품 데이터를 넣어보겠습니다.

#### **A. 한 줄씩 넣기**

```python
cur.execute("""
    INSERT INTO products (name, category, price, stock)
    VALUES ('아메리카노', '커피', 4500, 100);
""")
conn.commit()
print("✅ 1개 상품 추가 완료!")
```

#### **B. 여러 줄 한꺼번에 넣기**

```python
products_data = [
    ('카페라떼', '커피', 5000, 80),
    ('녹차', '차', 4000, 60),
    ('초코케이크', '디저트', 6500, 30),
    ('아이스티', '차', 3500, 120),
    ('크로와상', '디저트', 4000, 45),
]

cur.executemany("""
    INSERT INTO products (name, category, price, stock)
    VALUES (%s, %s, %s, %s);
""", products_data)

conn.commit()
print(f"✅ {len(products_data)}개 상품 추가 완료!")
```

> ⚠️ **중요 — SQL Injection 방지!**
>
> 위에서 `%s`를 사용한 것을 눈치채셨나요? 이것은 **파라미터 바인딩(Parameter Binding)**이라고 합니다.
>
> ```python
> # ❌ 절대 이렇게 하지 마세요! (SQL Injection 위험)
> name = "아메리카노"
> cur.execute(f"INSERT INTO products (name) VALUES ('{name}');")
>
> # ✅ 항상 이렇게 하세요! (안전)
> name = "아메리카노"
> cur.execute("INSERT INTO products (name) VALUES (%s);", (name,))
> ```
>
> 💡 **컴퓨터 공학 개념 — SQL Injection (SQL 삽입 공격)**
>
> SQL Injection은 해커가 입력값에 SQL 코드를 몰래 끼워 넣어, 데이터를 훔치거나 지우는 공격입니다.
>
> 예를 들어, 사용자가 이름 입력란에 이렇게 입력하면:
> ```
> '; DROP TABLE products; --
> ```
>
> f-string 방식에서는 이렇게 조합됩니다:
> ```sql
> INSERT INTO products (name) VALUES (''; DROP TABLE products; --');
> ```
> → 테이블이 삭제됩니다! 💥
>
> `%s` 파라미터 방식은 입력값을 **순수한 텍스트**로만 취급해서, 이런 공격을 자동으로 막아줍니다.

---

### 5. 데이터 읽기 (SELECT)

저장한 데이터를 읽어보겠습니다. **읽기(SELECT)**는 `commit()`이 필요 없습니다. (데이터를 바꾸는 게 아니니까요!)

#### **A. 전체 데이터 읽기**

```python
cur.execute("SELECT * FROM products;")
rows = cur.fetchall()

print("📋 전체 상품 목록:")
print("-" * 60)
for row in rows:
    print(f"  ID: {row[0]} | 이름: {row[1]} | 카테고리: {row[2]} | 가격: {row[3]}원 | 재고: {row[4]}개")
```

실행 결과:
```
📋 전체 상품 목록:
------------------------------------------------------------
  ID: 1 | 이름: 아메리카노 | 카테고리: 커피 | 가격: 4500원 | 재고: 100개
  ID: 2 | 이름: 카페라떼 | 카테고리: 커피 | 가격: 5000원 | 재고: 80개
  ID: 3 | 이름: 녹차 | 카테고리: 차 | 가격: 4000원 | 재고: 60개
  ID: 4 | 이름: 초코케이크 | 카테고리: 디저트 | 가격: 6500원 | 재고: 30개
  ID: 5 | 이름: 아이스티 | 카테고리: 차 | 가격: 3500원 | 재고: 120개
  ID: 6 | 이름: 크로와상 | 카테고리: 디저트 | 가격: 4000원 | 재고: 45개
```

#### **B. 조건으로 필터링해서 읽기**

```python
cur.execute("SELECT * FROM products WHERE category = %s;", ('커피',))
coffees = cur.fetchall()

print("☕ 커피 카테고리 상품:")
for coffee in coffees:
    print(f"  {coffee[1]} - {coffee[3]}원")
```

실행 결과:
```
☕ 커피 카테고리 상품:
  아메리카노 - 4500원
  카페라떼 - 5000원
```

#### **C. 한 줄만 읽기**

```python
cur.execute("SELECT * FROM products WHERE name = %s;", ('녹차',))
one_product = cur.fetchone()  # fetchall() 대신 fetchone()

if one_product:
    print(f"🍵 찾은 상품: {one_product[1]} ({one_product[3]}원)")
else:
    print("상품을 찾을 수 없습니다.")
```

> 💡 **컴퓨터 공학 개념 — fetch의 종류**
>
> | 메서드 | 동작 | 비유 |
> | --- | --- | --- |
> | `fetchone()` | 결과에서 **1개**만 가져옴 | 뷔페에서 한 접시만 가져오기 |
> | `fetchall()` | 결과 **전부** 가져옴 | 뷔페 음식 전부 한꺼번에 담기 |
> | `fetchmany(n)` | 결과에서 **n개** 가져옴 | 뷔페에서 n접시만 가져오기 |
>
> 데이터가 100만 개라면? `fetchall()`은 메모리를 많이 차지합니다. 이런 경우 `fetchmany(100)`처럼 나눠서 가져오는 것이 효율적입니다. 이것을 **페이징(Paging)**이라고 합니다.

---

### 6. 데이터 수정하기 (UPDATE)

이미 들어있는 데이터의 값을 바꿔보겠습니다.

#### **A. 하나의 상품 가격 수정**

```python
cur.execute("""
    UPDATE products
    SET price = 4000
    WHERE name = %s;
""", ('아메리카노',))

conn.commit()
print("✅ 아메리카노 가격 수정 완료!")
```

#### **B. 조건에 맞는 여러 상품 수정**

```python
# 디저트 카테고리의 재고를 전부 10개씩 추가
cur.execute("""
    UPDATE products
    SET stock = stock + 10
    WHERE category = %s;
""", ('디저트',))

conn.commit()

# 수정된 행의 개수 확인
print(f"✅ {cur.rowcount}개 상품의 재고 수정 완료!")
```

> ⚠️ 이전 가이드에서 배운 것처럼, `WHERE`를 빠뜨리면 **모든 행**이 수정됩니다! 파이썬에서도 마찬가지입니다.

#### **C. 수정 결과 확인**

```python
cur.execute("SELECT * FROM products WHERE category = %s;", ('디저트',))
desserts = cur.fetchall()

print("🍰 디저트 재고 확인:")
for d in desserts:
    print(f"  {d[1]} - 재고: {d[4]}개")
```

> 💡 **컴퓨터 공학 개념 — rowcount**
>
> `cur.rowcount`는 마지막 SQL 명령이 **영향을 준 행의 수**를 알려줍니다.
>
> - `UPDATE` → 수정된 행의 수
> - `DELETE` → 삭제된 행의 수
> - `INSERT` → 추가된 행의 수
>
> 이것을 이용하면 "정말로 수정/삭제됐는지" 프로그래밍적으로 확인할 수 있습니다!

---

### 7. 데이터 삭제하기 (DELETE)

#### **A. 특정 상품 삭제**

```python
# 삭제 전 확인 (좋은 습관!)
cur.execute("SELECT * FROM products WHERE name = %s;", ('아이스티',))
target = cur.fetchone()

if target:
    print(f"🗑️ 삭제할 상품: {target[1]} ({target[3]}원)")

    cur.execute("DELETE FROM products WHERE name = %s;", ('아이스티',))
    conn.commit()
    print(f"✅ 삭제 완료! ({cur.rowcount}개 행 삭제됨)")
else:
    print("삭제할 상품이 없습니다.")
```

#### **B. 조건으로 여러 행 삭제**

```python
# 가격이 4000원 미만인 상품 모두 삭제
cur.execute("SELECT * FROM products WHERE price < %s;", (4000,))
cheap_items = cur.fetchall()

print(f"🗑️ 삭제될 상품 {len(cheap_items)}개:")
for item in cheap_items:
    print(f"  - {item[1]} ({item[3]}원)")

cur.execute("DELETE FROM products WHERE price < %s;", (4000,))
conn.commit()
print(f"✅ 삭제 완료!")
```

> 💡 **컴퓨터 공학 개념 — 소프트 삭제(Soft Delete) vs 하드 삭제(Hard Delete)**
>
> 실무에서는 데이터를 정말로 `DELETE`하는 것을 **하드 삭제(Hard Delete)**라고 합니다. 하지만 많은 서비스에서는 실제로 삭제하지 않고, `is_deleted`라는 컬럼을 `TRUE`로 바꾸는 **소프트 삭제(Soft Delete)**를 사용합니다.
>
> ```sql
> -- 하드 삭제: 데이터가 진짜 사라짐
> DELETE FROM products WHERE id = 1;
>
> -- 소프트 삭제: 데이터는 남아있고 표시만 바꿈
> UPDATE products SET is_deleted = TRUE WHERE id = 1;
> ```
>
> 왜 소프트 삭제를 쓸까요?
> - 실수로 삭제해도 **복구 가능**
> - 삭제 기록을 **추적 가능** (누가 언제 삭제했는지)
> - 법적으로 데이터 **보관 의무**가 있는 경우

---

### 8. 테이블 삭제하기 (DROP TABLE)

실습이 끝나면 테이블을 깔끔하게 정리할 수 있습니다.

```python
cur.execute("DROP TABLE IF EXISTS products;")
conn.commit()
print("✅ products 테이블 삭제 완료!")
```

> `IF EXISTS`를 붙이면 테이블이 없어도 에러가 나지 않습니다. 안전하게 삭제하는 좋은 습관!

> 💡 **컴퓨터 공학 개념 — DROP vs DELETE vs TRUNCATE**
>
> | 명령 | 동작 | 비유 |
> | --- | --- | --- |
> | `DELETE` | 행(데이터)만 삭제, 테이블 구조 유지 | 서랍 안의 물건만 버리기 |
> | `TRUNCATE` | 모든 행을 빠르게 삭제, 테이블 구조 유지 | 서랍을 통째로 비우기 (DELETE보다 빠름) |
> | `DROP` | 테이블 자체를 삭제 | 서랍째로 버리기 |

---

### 9. 연결 정리하기 (항상 해야 합니다!)

작업이 끝나면 커서와 연결을 반드시 닫아야 합니다.

```python
cur.close()
conn.close()
print("✅ 연결 종료!")
```

> 💡 **컴퓨터 공학 개념 — 리소스 관리와 메모리 누수(Memory Leak)**
>
> 데이터베이스 연결은 **한정된 자원(Resource)**입니다. PostgreSQL은 기본적으로 **최대 100개의 동시 연결**만 허용합니다.
>
> `conn.close()`를 하지 않으면:
> - 연결이 계속 열린 채로 남아 있음 → **연결 고갈**
> - 메모리가 해제되지 않음 → **메모리 누수(Memory Leak)**
>
> 이것은 수도꼭지를 틀어놓고 안 잠그는 것과 같습니다! 🚰

---

### 10. 🏆 완성본: 전체 코드 한눈에 보기

지금까지 배운 내용을 하나의 파일로 정리했습니다. `db_tutorial.py`로 저장하고 실행해 보세요!

```python
import psycopg2

# ──────────────────────────────────────
# 1. 연결 및 커서 생성
# ──────────────────────────────────────
conn = psycopg2.connect(
    host="localhost",
    port="5432",
    database="postgres",
    user="postgres",
    password="mysecretpassword"
)
cur = conn.cursor()
print("✅ 데이터베이스 연결 성공!\n")


# ──────────────────────────────────────
# 2. 테이블 만들기 (CREATE)
# ──────────────────────────────────────
cur.execute("DROP TABLE IF EXISTS products;")
cur.execute("""
    CREATE TABLE products (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        category TEXT NOT NULL,
        price INTEGER NOT NULL,
        stock INTEGER DEFAULT 0
    );
""")
conn.commit()
print("✅ products 테이블 생성 완료!\n")


# ──────────────────────────────────────
# 3. 데이터 넣기 (INSERT)
# ──────────────────────────────────────
products_data = [
    ('아메리카노', '커피', 4500, 100),
    ('카페라떼', '커피', 5000, 80),
    ('녹차', '차', 4000, 60),
    ('초코케이크', '디저트', 6500, 30),
    ('아이스티', '차', 3500, 120),
    ('크로와상', '디저트', 4000, 45),
]

cur.executemany("""
    INSERT INTO products (name, category, price, stock)
    VALUES (%s, %s, %s, %s);
""", products_data)

conn.commit()
print(f"✅ {len(products_data)}개 상품 추가 완료!\n")


# ──────────────────────────────────────
# 4. 데이터 읽기 (SELECT)
# ──────────────────────────────────────
cur.execute("SELECT * FROM products;")
rows = cur.fetchall()

print("📋 전체 상품 목록:")
print("-" * 60)
for row in rows:
    print(f"  ID: {row[0]} | {row[1]:6s} | {row[2]:4s} | {row[3]:>6}원 | 재고: {row[4]}개")
print()


# ──────────────────────────────────────
# 5. 데이터 수정하기 (UPDATE)
# ──────────────────────────────────────
cur.execute("""
    UPDATE products SET price = 4000 WHERE name = %s;
""", ('아메리카노',))

cur.execute("""
    UPDATE products SET stock = stock + 10 WHERE category = %s;
""", ('디저트',))

conn.commit()
print("✅ 가격 및 재고 수정 완료!")

# 수정 결과 확인
cur.execute("SELECT * FROM products;")
rows = cur.fetchall()

print("📋 수정 후 상품 목록:")
print("-" * 60)
for row in rows:
    print(f"  ID: {row[0]} | {row[1]:6s} | {row[2]:4s} | {row[3]:>6}원 | 재고: {row[4]}개")
print()


# ──────────────────────────────────────
# 6. 데이터 삭제하기 (DELETE)
# ──────────────────────────────────────
cur.execute("DELETE FROM products WHERE name = %s;", ('아이스티',))
conn.commit()
print(f"✅ 아이스티 삭제 완료! ({cur.rowcount}개 행 삭제됨)")

cur.execute("SELECT * FROM products;")
rows = cur.fetchall()

print("📋 삭제 후 상품 목록:")
print("-" * 60)
for row in rows:
    print(f"  ID: {row[0]} | {row[1]:6s} | {row[2]:4s} | {row[3]:>6}원 | 재고: {row[4]}개")
print()


# ──────────────────────────────────────
# 7. 테이블 삭제하기 (DROP TABLE)
# ──────────────────────────────────────
cur.execute("DROP TABLE IF EXISTS products;")
conn.commit()
print("✅ products 테이블 삭제 완료!\n")


# ──────────────────────────────────────
# 8. 연결 종료
# ──────────────────────────────────────
cur.close()
conn.close()
print("✅ 데이터베이스 연결 종료! 수고하셨습니다! 🎉")
```

---

### 11. 🎁 보너스: `with` 문으로 안전하게 연결 관리하기

위 코드에서 `close()`를 깜빡하면 문제가 생깁니다. 파이썬의 `with` 문을 사용하면 **자동으로 정리**해줍니다.

```python
import psycopg2

# with 문으로 안전하게 연결
with psycopg2.connect(
    host="localhost",
    port="5432",
    database="postgres",
    user="postgres",
    password="mysecretpassword"
) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT * FROM products;")
        rows = cur.fetchall()
        for row in rows:
            print(row)

# 여기에 도달하면 커서와 연결이 자동으로 정리됩니다!
print("✅ 자동으로 안전하게 종료됨!")
```

> 💡 **컴퓨터 공학 개념 — 컨텍스트 매니저(Context Manager)**
>
> `with` 문은 파이썬의 **컨텍스트 매니저(Context Manager)** 패턴입니다.
>
> ```python
> # 이 두 코드는 같은 뜻입니다
>
> # 방법 1: 수동 (실수 가능성 있음)
> f = open("file.txt")
> data = f.read()
> f.close()  # 깜빡할 수 있음!
>
> # 방법 2: with 문 (자동 정리)
> with open("file.txt") as f:
>     data = f.read()
> # close()가 자동 호출됨!
> ```
>
> `with` 블록을 벗어나면 에러가 발생하더라도 **자동으로 리소스를 정리**합니다. 실무에서는 거의 항상 `with` 문을 사용합니다.

---

### 12. 🛠️ 전체 요약 (CRUD Cheat Sheet)

| 작업 | SQL | Python 코드 |
| --- | --- | --- |
| **C**reate (만들기) | `CREATE TABLE ...` | `cur.execute("CREATE TABLE ...")` |
| **R**ead (읽기) | `SELECT * FROM ...` | `cur.execute("SELECT ..."); cur.fetchall()` |
| **U**pdate (수정) | `UPDATE ... SET ...` | `cur.execute("UPDATE ...")` |
| **D**elete (삭제) | `DELETE FROM ... WHERE ...` | `cur.execute("DELETE FROM ...")` |
| 테이블 삭제 | `DROP TABLE ...` | `cur.execute("DROP TABLE ...")` |
| **저장** | `COMMIT` | `conn.commit()` |
| **되돌리기** | `ROLLBACK` | `conn.rollback()` |

> 💡 **컴퓨터 공학 개념 — CRUD**
>
> **CRUD**는 데이터를 다루는 4가지 기본 작업의 머리글자입니다:
>
> - **C**reate (생성) → 새 데이터 만들기
> - **R**ead (읽기) → 기존 데이터 조회
> - **U**pdate (수정) → 기존 데이터 변경
> - **D**elete (삭제) → 기존 데이터 제거
>
> 거의 모든 소프트웨어(웹사이트, 앱, 게임 등)는 이 4가지 작업의 조합으로 이루어져 있습니다. 인스타그램도, 은행 앱도, 게임의 세이브 파일도 결국 CRUD입니다! 이 패턴을 이해하면 어떤 프로그램이든 구조를 파악할 수 있습니다.

---

### 13. 핵심 관리 팁

1. **`%s`를 항상 사용하세요**: f-string으로 SQL을 만들면 해킹(SQL Injection)에 취약합니다.
2. **`commit()`을 잊지 마세요**: `SELECT`을 제외한 `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP` 후에는 반드시 `commit()`.
3. **`with` 문을 사용하세요**: 연결과 커서를 자동으로 정리해줍니다.
4. **삭제 전 SELECT로 확인**: `DELETE` 전에 같은 `WHERE` 조건으로 `SELECT`를 먼저 실행하세요.
5. **에러 처리를 추가하세요**: 실무에서는 `try/except`로 에러를 잡아야 합니다.

    ```python
    try:
        cur.execute("INSERT INTO products (name) VALUES (%s);", ('테스트',))
        conn.commit()
    except Exception as e:
        conn.rollback()  # 에러 시 되돌리기
        print(f"❌ 에러 발생: {e}")
    ```

6. **종료하기**: 프로그램 끝에서 `cur.close()`와 `conn.close()`를 호출하세요. (또는 `with` 문 사용)

---

### 📚 다음 단계

이 가이드를 마쳤다면 아래 주제를 공부해 보세요:

- **ORM (Object-Relational Mapping)**: SQL을 직접 쓰지 않고 파이썬 객체로 DB를 다루는 방법 (SQLAlchemy, Django ORM)
- **Connection Pool**: 연결을 미리 여러 개 만들어 놓고 재활용하는 기법 (실무 필수!)
- **Database Migration**: 테이블 구조 변경을 안전하게 관리하는 방법 (Alembic)

---
