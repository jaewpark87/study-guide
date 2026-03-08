
---

## 📄 초보자를 위한 PostgreSQL 데이터 수정 & 삭제 가이드

이 문서는 이전 가이드에서 배운 Docker + PostgreSQL 환경을 활용하여, 데이터를 **수정(UPDATE)**하고 **삭제(DELETE)**하는 방법을 학습합니다.

---

### 1. 복습: 새 테이블 만들고 데이터 넣기

이전 가이드에서 도커로 PostgreSQL을 실행하고 접속하는 방법을 배웠습니다. 먼저 DB에 접속하세요.

```bash
docker exec -it my_postgres psql -U postgres
```

> 만약 컨테이너가 중지되어 있다면, `docker start my_postgres`로 먼저 시작하세요.

#### **Step 1: `sales` 테이블 만들기**

아래 SQL을 입력하여 판매 기록을 저장할 테이블을 만드세요.

```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    product TEXT NOT NULL,
    quantity INT NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    sold_at DATE DEFAULT CURRENT_DATE
);
```

| 컬럼 | 설명 | 예시 |
| --- | --- | --- |
| **`id`** | 자동 증가하는 고유 번호 | 1, 2, 3... |
| **`product`** | 상품 이름 | '노트북', '키보드' |
| **`quantity`** | 판매 수량 | 5, 10 |
| **`price`** | 단가 (소수점 2자리) | 1500000.00 |
| **`sold_at`** | 판매 날짜 (기본값: 오늘) | 2026-03-05 |

#### **Step 2: 샘플 데이터 넣기**

```sql
INSERT INTO sales (product, quantity, price, sold_at)
VALUES ('노트북', 2, 1500000.00, '2026-01-15'),
       ('키보드', 10, 89000.00, '2026-01-20'),
       ('마우스', 15, 45000.00, '2026-02-01'),
       ('모니터', 3, 350000.00, '2026-02-10'),
       ('USB 케이블', 50, 5000.00, '2026-03-01');
```

#### **Step 3: 데이터 확인하기**

```sql
SELECT * FROM sales;
```

결과가 아래처럼 5개의 행으로 나오면 준비 완료입니다!

```
 id |  product   | quantity |   price    |  sold_at
----+------------+----------+------------+------------
  1 | 노트북     |        2 | 1500000.00 | 2026-01-15
  2 | 키보드     |       10 |   89000.00 | 2026-01-20
  3 | 마우스     |       15 |   45000.00 | 2026-02-01
  4 | 모니터     |        3 |  350000.00 | 2026-02-10
  5 | USB 케이블 |       50 |    5000.00 | 2026-03-01
```

---

### 2. 데이터 수정하기 (UPDATE)

`UPDATE`는 이미 저장된 데이터의 값을 바꿀 때 사용합니다.

#### 📌 기본 문법

```sql
UPDATE 테이블이름
SET 컬럼 = 새로운값
WHERE 조건;
```

> ⚠️ **중요**: `WHERE`를 빠뜨리면 **테이블의 모든 행**이 바뀝니다! 항상 조건을 확인하세요.

#### **A. 하나의 컬럼 수정하기**

키보드의 가격을 79,000원으로 변경하겠습니다.

```sql
UPDATE sales
SET price = 79000.00
WHERE product = '키보드';
```

확인:

```sql
SELECT * FROM sales WHERE product = '키보드';
```

```
 id | product | quantity |  price   |  sold_at
----+---------+----------+----------+------------
  2 | 키보드  |       10 | 79000.00 | 2026-01-20
```

#### **B. 여러 컬럼 동시에 수정하기**

마우스의 수량과 가격을 함께 변경하겠습니다.

```sql
UPDATE sales
SET quantity = 20, price = 42000.00
WHERE product = '마우스';
```

확인:

```sql
SELECT * FROM sales WHERE product = '마우스';
```

```
 id | product | quantity |  price   |  sold_at
----+---------+----------+----------+------------
  3 | 마우스  |       20 | 42000.00 | 2026-02-01
```

#### **C. 조건에 여러 행이 해당되는 경우**

가격이 100,000원 미만인 모든 상품의 수량을 +5 추가하겠습니다.

```sql
UPDATE sales
SET quantity = quantity + 5
WHERE price < 100000;
```

확인:

```sql
SELECT * FROM sales WHERE price < 100000;
```

#### 🛠️ UPDATE 핵심 요약

| 작업 | SQL 예시 |
| --- | --- |
| 단일 컬럼 수정 | `UPDATE sales SET price = 79000 WHERE product = '키보드';` |
| 복수 컬럼 수정 | `UPDATE sales SET quantity = 20, price = 42000 WHERE product = '마우스';` |
| 계산으로 수정 | `UPDATE sales SET quantity = quantity + 5 WHERE price < 100000;` |
| ⚠️ 전체 수정 (위험!) | `UPDATE sales SET price = 0;` ← WHERE 없음 = 전부 변경 |

---

### 3. 데이터 삭제하기 (DELETE)

`DELETE`는 테이블에서 행(row)을 제거할 때 사용합니다.

#### 📌 기본 문법

```sql
DELETE FROM 테이블이름
WHERE 조건;
```

> ⚠️ **중요**: `WHERE`를 빠뜨리면 **테이블의 모든 데이터**가 삭제됩니다!

#### **A. 특정 행 삭제하기**

USB 케이블 판매 기록을 삭제하겠습니다.

```sql
DELETE FROM sales
WHERE product = 'USB 케이블';
```

확인:

```sql
SELECT * FROM sales;
```

이제 4개의 행만 남아 있습니다.

#### **B. 조건으로 여러 행 삭제하기**

2026년 1월에 판매된 기록을 모두 삭제하겠습니다.

```sql
DELETE FROM sales
WHERE sold_at < '2026-02-01';
```

확인:

```sql
SELECT * FROM sales;
```

#### **C. 삭제 전 미리보기 (안전한 습관)**

실수를 방지하기 위해, 삭제 전에 `SELECT`로 어떤 데이터가 삭제될지 먼저 확인하세요.

```sql
-- 먼저 확인
SELECT * FROM sales WHERE price < 50000;

-- 확인 후 삭제
DELETE FROM sales WHERE price < 50000;
```

#### 🛠️ DELETE 핵심 요약

| 작업 | SQL 예시 |
| --- | --- |
| 특정 행 삭제 | `DELETE FROM sales WHERE product = 'USB 케이블';` |
| 조건부 삭제 | `DELETE FROM sales WHERE sold_at < '2026-02-01';` |
| ⚠️ 전체 삭제 (위험!) | `DELETE FROM sales;` ← WHERE 없음 = 전부 삭제 |
| 테이블 자체를 삭제 | `DROP TABLE sales;` ← 테이블 구조까지 완전 제거 |

---

### 4. 핵심 관리 팁

1. **`WHERE`는 안전벨트**: `UPDATE`와 `DELETE`를 쓸 때 항상 `WHERE` 조건을 먼저 작성하세요. 빠뜨리면 전체 데이터가 바뀌거나 사라집니다.
2. **삭제 전 SELECT**: `DELETE` 전에 같은 `WHERE` 조건으로 `SELECT`를 실행해서 삭제될 데이터를 미리 확인하세요.
3. **되돌리기(Rollback)**: 실수했을 때 `BEGIN;`과 `ROLLBACK;`을 활용하면 변경을 취소할 수 있습니다.

    ```sql
    BEGIN;                          -- 트랜잭션 시작
    DELETE FROM sales;              -- 실수로 전체 삭제!
    ROLLBACK;                       -- 되돌리기 → 데이터 복구!
    ```

4. **종료하기**: `\q`를 입력하고 엔터를 치면 DB 접속에서 빠져나옵니다.

---
