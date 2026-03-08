
---

## 📄 초보자를 위한 Docker & PostgreSQL 통합 가이드

이 문서는 도커의 기본 원리부터 실제 데이터베이스를 구축하고 데이터를 다루는 SQL까지 한 번에 학습할 수 있도록 설계되었습니다.

---

### 1. 도커(Docker) 이해하기: "소프트웨어 도시락"

도커는 프로그램을 실행하는 데 필요한 환경(OS, 라이브러리 등)을 하나로 묶어 어디서든 똑같이 실행되게 만드는 기술입니다.

#### 🛠️ 꼭 알아야 할 도커 명령어

| 명령어 | 용도 | 비유 |
| --- | --- | --- |
| **`docker pull`** | 이미지(설계도) 다운로드 | 밀키트 주문하기 |
| **`docker run`** | 컨테이너 생성 및 실행 | 요리 시작하기 |
| **`docker ps`** | 실행 중인 컨테이너 확인 | 가스레인지 위 냄비 확인 |
| **`docker stop`** | 컨테이너 중지 | 불 끄기 |
| **`docker rm`** | 컨테이너 삭제 | 설거지(상자 버리기) |
| **`docker logs`** | 내부 상태/에러 확인 | 요리 과정 지켜보기 |

---

### 2. PostgreSQL 데이터베이스 설치 및 실행

이제 도커를 사용해 1분 만에 데이터베이스 서버를 만들어 보겠습니다.

#### **Step 1: 실행 명령어 입력**

터미널(또는 CMD)을 열고 아래 명령어를 복사해 넣으세요.

```bash
docker run --name my_postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres

```

* **`--name my_postgres`**: 내 DB 상자의 이름입니다.
* **`-e POSTGRES_PASSWORD=...`**: 접속 비밀번호를 설정합니다.
* **`-p 5432:5432`**: 내 컴퓨터와 도커 내부의 통로를 연결합니다.
* **`-d`**: 백그라운드 모드(터미널을 꺼도 계속 실행)입니다.

#### **Step 2: 데이터베이스 접속**

실행된 상자 안으로 들어가서 SQL 명령을 내릴 준비를 합니다.

```bash
docker exec -it my_postgres psql -U postgres

```

---

### 3. 실습: SQL로 데이터 주무르기

접속이 완료되면 `postgres=#`이라는 표시가 뜹니다. 이제 아래 명령어를 순서대로 입력하세요.

#### **A. 테이블 만들기 (Create)**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

```

#### **B. 데이터 집어넣기 (Insert)**

```sql
INSERT INTO users (name, email) 
VALUES ('홍길동', 'hong@example.com'), 
       ('김철수', 'kim@example.com');

```

#### **C. 데이터 꺼내보기 (Select)**

```sql
SELECT * FROM users;

```

---

### 4. 핵심 관리 팁

1. **종료하기**: `\q`를 입력하고 엔터를 치면 DB 접속에서 빠져나옵니다.
2. **데이터 보관**: 도커 컨테이너를 삭제(`rm`)하면 데이터도 사라집니다. 실무에서는 `-v` 옵션을 사용해 데이터를 내 컴퓨터 하드디스크에 따로 저장(Volume)합니다.
3. **다시 시작**: 컴퓨터를 껐다 켰다면 `docker start my_postgres` 명령어로 다시 깨울 수 있습니다.

---
