# FreeRADIUS + MySQL 기반 AAA 인증 시스템 구축 가이드

## 1. 개요

본 문서는 FreeRADIUS와 MySQL(MariaDB)을 연동하여
중앙 인증(AAA) 시스템을 구축하는 방법을 정리한 가이드입니다.

본 프로젝트에서는 해당 구조를 기반으로
네트워크 장비의 인증 및 접근 통제를 구현하였습니다.

---

## 2. 구성 개요

* FreeRADIUS Server
* MySQL(MariaDB)
* Network Devices (Router / Switch)
* AAA(Authentication, Authorization, Accounting)

---

## 3. 패키지 설치

```bash
dnf install freeradius freeradius-mysql freeradius-utils mariadb-server -y
```

---

## 4. DB 설정

### 4.1 DB 접속

```bash
mysql -u root -p
```

### 4.2 DB 및 사용자 생성

```sql
CREATE DATABASE radius;

CREATE USER 'radius'@'localhost' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';

FLUSH PRIVILEGES;
```

---

## 5. RADIUS 테이블 생성

```bash
mysql -u root -p radius < /etc/raddb/mods-config/sql/main/mysql/schema.sql
```

---

## 6. RADIUS SQL 연동 설정

```bash
vi /etc/raddb/mods-enabled/sql
```

다음 항목 수정:

```
* driver = "rlm_sql_mysql"
* server = "localhost"
* login = "radius"
* password = "password"
* radius_db = "radius"
```

---

## 7. 사용자 계정 추가

### 7.1 평문 방식

```sql
INSERT INTO radcheck (username, attribute, op, value)
VALUES ('testuser', 'Cleartext-Password', ':=', '1234');
````

---

### 7.2 암호화 방식 (SHA-512)

#### 해시 생성

```bash
openssl passwd -6
```

#### DB 저장

```sql
INSERT INTO radcheck (username, attribute, op, value)
VALUES ('testuser', 'Crypt-Password', ':=', '$6$xxxxxxxx...');
```

---

## 8. 테스트

```bash
radtest testuser 1234 localhost 0 testing123
```

정상 결과:

```
Access-Accept
```

---

## 9. 트러블슈팅

### 9.1 DB 암호화 인증 문제

* 현상
  비밀번호를 암호화하여 DB에 저장한 이후 인증 실패 발생

* 원인
  FreeRADIUS는 저장된 비밀번호와 입력값을 비교하는 방식으로 동작하나,
  해시 방식이 일치하지 않아 비교가 불가능한 상태

* 해결
  SHA-512($6$) 기반 해시를 적용하고,
  RADIUS에서 해당 값을 인식하도록 설정 수정

---

## 10. 참고 사항

* Cleartext-Password: 평문 비교 방식
* Crypt-Password: 해시 기반 비교 방식
* 인증(Authentication)과 권한(Authorization)은 별도로 동작함
