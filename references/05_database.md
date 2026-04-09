# 2장. 데이터베이스 보안 가이드

대상: MySQL, MS-SQL, Redis, Elasticsearch, MongoDB, PostgreSQL, Cubrid, CouchDB, SQLite, Tibero, InfluxDB, Oracle

---

## 목차

- [MySQL](#mysql)
- [MS-SQL (MSSQL)](#ms-sql-mssql)
- [Redis](#redis)
- [Elasticsearch](#elasticsearch)
- [MongoDB](#mongodb)
- [PostgreSQL](#postgresql)
- [Cubrid](#cubrid)
- [CouchDB](#couchdb)
- [SQLite](#sqlite)
- [Tibero](#tibero)
- [InfluxDB](#influxdb)
- [Oracle](#oracle)

---

## MySQL

### MY-01. root 계정 원격 접속 제한

- **위험도**: 상
- **점검 내용**: root 계정이 원격에서 접속 가능한지 여부
- **점검 방법**:

  ```sql
  SELECT user, host FROM mysql.user WHERE user='root';
  ```

- **양호 기준**: root 계정의 host가 'localhost' 또는 '127.0.0.1'만 허용
- **취약 기준**: root 계정의 host가 '%' (모든 IP 허용)
- **조치 방법**:

  ```sql
  UPDATE mysql.user SET host='localhost' WHERE user='root' AND host='%';
  FLUSH PRIVILEGES;
  ```

### MY-02. 익명 계정 제거

- **위험도**: 상
- **점검 내용**: 패스워드 없는 익명 계정 존재 여부
- **점검 방법**:

  ```sql
  SELECT user, host FROM mysql.user WHERE user='';
  ```

- **양호 기준**: 익명 계정 없음
- **취약 기준**: 익명 계정 존재
- **조치 방법**:

  ```sql
  DELETE FROM mysql.user WHERE user='';
  FLUSH PRIVILEGES;
  ```

### MY-03. 패스워드 정책 설정

- **위험도**: 상
- **점검 방법**:

  ```sql
  SHOW VARIABLES LIKE 'validate_password%';
  ```

- **양호 기준**: `validate_password_policy=MEDIUM` 이상, 최소 길이 8자
- **조치 방법**:

  ```sql
  SET GLOBAL validate_password_policy=MEDIUM;
  SET GLOBAL validate_password_length=8;
  ```

### MY-04. 파일 시스템 접근 제한

- **위험도**: 상
- **점검 내용**: FILE 권한 부여 현황 (LOAD DATA INFILE, SELECT INTO OUTFILE 남용 방지)
- **점검 방법**:

  ```sql
  SELECT user, host FROM mysql.user WHERE File_priv='Y';
  ```

- **양호 기준**: FILE 권한 최소화
- **취약 기준**: 일반 계정에 FILE 권한 부여

### MY-05. 불필요한 권한 제거

- **위험도**: 중
- **점검 내용**: 사용자별 과도한 권한 부여 여부
- **점검 방법**:

  ```sql
  SHOW GRANTS FOR '<username>'@'<host>';
  SELECT * FROM mysql.user WHERE Super_priv='Y';
  ```

- **양호 기준**: 최소 권한 원칙 적용
- **취약 기준**: 일반 사용자에게 SUPER, GRANT OPTION 등 과도한 권한

### MY-06. 로그 활성화

- **위험도**: 중
- **점검 방법**:

  ```sql
  SHOW VARIABLES LIKE '%log%';
  ```

- **양호 기준**: general_log, slow_query_log 설정, binlog 활성화
- **조치 방법**: `/etc/my.cnf`에서 로그 설정 추가

---

## MS-SQL (MSSQL)

### MS-01. SA 계정 관리

- **위험도**: 상
- **점검 내용**: SA (System Administrator) 계정 비활성화 또는 이름 변경
- **점검 방법** (T-SQL):

  ```sql
  SELECT name, is_disabled FROM sys.sql_logins WHERE name='sa';
  ```

- **양호 기준**: SA 계정 비활성화 또는 이름 변경
- **취약 기준**: SA 계정 활성화 상태로 사용
- **조치 방법**:

  ```sql
  ALTER LOGIN sa DISABLE;
  -- 또는
  ALTER LOGIN sa WITH NAME = [newname];
  ```

### MS-02. 인증 모드 설정

- **위험도**: 상
- **점검 내용**: SQL Server 인증 모드 (SQL Server + Windows 혼합 모드 vs. Windows 전용)
- **점검 방법**: SQL Server Management Studio → 서버 속성 → 보안 탭
- **양호 기준**: Windows 인증 모드만 사용 (가능한 경우)
- **취약 기준**: SQL Server 인증 모드 사용 + 약한 패스워드

### MS-03. xp_cmdshell 비활성화

- **위험도**: 상
- **점검 내용**: 운영체제 명령 실행 가능한 xp_cmdshell 프로시저 비활성화
- **점검 방법** (T-SQL):

  ```sql
  SELECT name, value_in_use FROM sys.configurations WHERE name = 'xp_cmdshell';
  ```

- **양호 기준**: `value_in_use = 0` (비활성화)
- **취약 기준**: `value_in_use = 1` (활성화)
- **조치 방법**:

  ```sql
  EXEC sp_configure 'xp_cmdshell', 0;
  RECONFIGURE;
  ```

### MS-04. 원격 접속 포트 제한

- **위험도**: 상
- **점검 내용**: SQL Server 포트(기본 1433)에 대한 방화벽 접근 통제
- **양호 기준**: DB 서버 접근 허용 IP 대역 제한, 기본 포트(1433) 변경
- **취약 기준**: 인터넷에 직접 노출

### MS-05. 감사 설정

- **위험도**: 중
- **점검 방법** (T-SQL):

  ```sql
  SELECT * FROM sys.server_audits;
  SELECT * FROM sys.server_audit_specifications;
  ```

- **양호 기준**: 로그인 실패, DDL 변경, 권한 변경 등 감사 설정
- **취약 기준**: 감사 미설정

---

## Redis

### RD-01. 인증 설정

- **위험도**: 상
- **점검 내용**: requirepass 설정으로 인증 활성화 여부
- **점검 방법**:

  ```bash
  cat /etc/redis/redis.conf | grep requirepass
  redis-cli -h <host> -p <port> ping  # 인증 없이 응답하면 취약
  ```

- **양호 기준**: `requirepass` 설정으로 강력한 패스워드 인증
- **취약 기준**: requirepass 미설정 (인증 없이 접근 가능)
- **조치 방법**: `/etc/redis/redis.conf`에서 `requirepass <strongpassword>` 추가

### RD-02. 바인드 주소 제한

- **위험도**: 상
- **점검 내용**: Redis가 외부 인터페이스에 바인딩 여부
- **점검 방법**:

  ```bash
  cat /etc/redis/redis.conf | grep "^bind"
  ss -tlnp | grep 6379
  ```

- **양호 기준**: `bind 127.0.0.1` (로컬호스트만 바인딩)
- **취약 기준**: `bind 0.0.0.0` 또는 퍼블릭 IP 바인딩
- **조치 방법**: `bind 127.0.0.1`로 설정 변경

### RD-03. 보호 모드 활성화

- **위험도**: 상
- **점검 방법**:

  ```bash
  cat /etc/redis/redis.conf | grep "protected-mode"
  ```

- **양호 기준**: `protected-mode yes`
- **취약 기준**: `protected-mode no`

### RD-04. 위험 명령어 비활성화

- **위험도**: 상
- **점검 내용**: FLUSHALL, CONFIG, DEBUG 등 위험 명령어 비활성화(rename-command)
- **점검 방법**:

  ```bash
  cat /etc/redis/redis.conf | grep rename-command
  ```

- **양호 기준**: 위험 명령어 rename 또는 disable 처리
- **조치 방법**:

  ```redis.conf
  rename-command FLUSHALL ""
  rename-command CONFIG ""
  rename-command DEBUG ""
  ```

### RD-05. TLS 설정

- **위험도**: 중
- **점검 내용**: Redis 통신 암호화 여부 (Redis 6.0+)
- **점검 방법**:

  ```bash
  cat /etc/redis/redis.conf | grep "^tls"
  ```

- **양호 기준**: TLS 활성화 및 인증서 설정
- **취약 기준**: TLS 미사용 (평문 통신)

---

## Elasticsearch

### ES-01. 보안 기능 활성화

- **위험도**: 상
- **점검 내용**: Elasticsearch Security(xpack.security) 활성화 여부
- **점검 방법**:

  ```bash
  cat /etc/elasticsearch/elasticsearch.yml | grep "xpack.security"
  curl -X GET http://localhost:9200/_security/user  # 인증 없이 접근되면 취약
  ```

- **양호 기준**: `xpack.security.enabled: true`
- **취약 기준**: 인증 없이 접근 가능
- **조치 방법**:

  ```yaml
  # elasticsearch.yml
  xpack.security.enabled: true
  xpack.security.http.ssl.enabled: true
  ```

### ES-02. 네트워크 바인딩 제한

- **위험도**: 상
- **점검 내용**: 외부 IP에 노출 여부
- **점검 방법**:

  ```bash
  cat /etc/elasticsearch/elasticsearch.yml | grep "network.host"
  ss -tlnp | grep 9200
  ```

- **양호 기준**: `network.host: localhost` 또는 내부 IP만 바인딩
- **취약 기준**: `network.host: 0.0.0.0` (인터넷 노출)

### ES-03. 사용자별 권한 설정

- **위험도**: 상
- **점검 내용**: Elasticsearch 사용자 역할 및 권한 설정
- **점검 방법**:

  ```bash
  curl -u admin:password -X GET http://localhost:9200/_security/role
  ```

- **양호 기준**: 최소 권한 원칙 적용, 역할별 인덱스 접근 제한
- **취약 기준**: 모든 사용자에게 superuser 역할 부여

---

## MongoDB

### MG-01. 인증 활성화

- **위험도**: 상
- **점검 내용**: MongoDB 인증 설정 여부
- **점검 방법**:

  ```bash
  cat /etc/mongod.conf | grep "authorization"
  mongo --eval "db.adminCommand({getCmdLineOpts:1})"
  ```

- **양호 기준**: `authorization: enabled`
- **취약 기준**: 인증 없이 접근 가능
- **조치 방법**: `/etc/mongod.conf`에서 `authorization: enabled` 설정

### MG-02. 바인딩 IP 제한

- **위험도**: 상
- **점검 방법**:

  ```bash
  cat /etc/mongod.conf | grep "bindIp"
  ```

- **양호 기준**: `bindIp: 127.0.0.1` 또는 내부 IP
- **취약 기준**: `bindIp: 0.0.0.0`

### MG-03. TLS/SSL 활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  cat /etc/mongod.conf | grep -A 10 "net:"
  ```

- **양호 기준**: TLS 활성화, 인증서 설정
- **취약 기준**: 평문 통신

---

## PostgreSQL

### PG-01. pg_hba.conf 설정

- **위험도**: 상
- **점검 내용**: 호스트 기반 인증 설정 적절성
- **점검 방법**:

  ```bash
  cat /etc/postgresql/<version>/main/pg_hba.conf
  # 또는
  cat $(psql -U postgres -t -c "SHOW hba_file;")
  ```

- **양호 기준**: `trust` 인증 방식 미사용, 허가된 IP만 접근 허용
- **취약 기준**: `host all all 0.0.0.0/0 trust` (패스워드 없이 모든 IP 접근)
- **조치 방법**: `trust`를 `md5` 또는 `scram-sha-256`으로 변경

### PG-02. postgres 슈퍼유저 패스워드

- **위험도**: 상
- **점검 방법**:

  ```sql
  SELECT usename, passwd FROM pg_shadow WHERE usename='postgres';
  ```

- **양호 기준**: 강력한 패스워드 설정
- **취약 기준**: 패스워드 미설정 또는 기본 패스워드

### PG-03. 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  psql -U postgres -c "SHOW log_connections; SHOW log_disconnections; SHOW log_duration;"
  ```

- **양호 기준**: 연결/연결 해제 로깅, DDL 로깅 활성화

---

## Oracle

### OR-01. 기본 계정 비활성화

- **위험도**: 상
- **점검 내용**: SCOTT, SYS, SYSTEM 등 기본 계정 관리
- **점검 방법**:

  ```sql
  SELECT username, account_status FROM dba_users WHERE username IN ('SCOTT','DBSNMP','OUTLN');
  ```

- **양호 기준**: 불필요한 기본 계정 LOCKED 또는 EXPIRED 상태
- **취약 기준**: 기본 패스워드로 계정 활성화

### OR-02. 원격 OS 인증 비활성화

- **위험도**: 상
- **점검 방법**:

  ```sql
  SHOW PARAMETER remote_os_authent;
  ```

- **양호 기준**: `remote_os_authent=FALSE`
- **취약 기준**: `remote_os_authent=TRUE`

### OR-03. 감사 기능 활성화

- **위험도**: 중
- **점검 방법**:

  ```sql
  SHOW PARAMETER audit_trail;
  SELECT * FROM dba_audit_policies;
  ```

- **양호 기준**: `audit_trail=DB` 또는 `OS` 설정, 주요 작업 감사 설정
- **취약 기준**: 감사 기능 비활성화

---

## Redis, Elasticsearch, MongoDB, PostgreSQL, Cubrid, CouchDB, SQLite, Tibero, InfluxDB

> 위 주요 DB 외 나머지 DB의 경우, 다음 공통 원칙을 적용하라:
>
> 1. **인증**: 반드시 강력한 패스워드 또는 인증서 기반 인증 활성화
> 2. **네트워크 제한**: 로컬호스트 또는 허가된 IP 대역만 접근 허용
> 3. **암호화**: 저장 데이터 및 전송 데이터 암호화 적용
> 4. **최소 권한**: 애플리케이션 계정에 필요한 최소 권한만 부여
> 5. **로그**: 보안 이벤트(접속, 오류, 권한 변경) 로깅 및 보관
>
> 특정 DB에 대한 세부 점검 항목이 필요하면 해당 제품명을 명시하여 요청하라.
