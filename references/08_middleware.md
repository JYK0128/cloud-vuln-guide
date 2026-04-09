# 2장. 미들웨어 / 런타임 보안 가이드

대상: PHP, RabbitMQ, Node.js

---

## 목차

- [PHP](#php)
- [RabbitMQ](#rabbitmq)
- [Node.js](#nodejs)

---

## PHP

### PHP-01. 에러 출력 비활성화 (운영 환경)

- **위험도**: 중
- **점검 내용**: PHP 에러 메시지가 브라우저에 직접 출력되면 시스템 구조 노출 위험
- **점검 방법**:

  ```bash
  php -i | grep -E "display_errors|log_errors|error_reporting"
  grep -E "display_errors|log_errors" /etc/php.ini
  ```

- **양호 기준**: `display_errors = Off`, `log_errors = On`
- **취약 기준**: `display_errors = On`
- **조치 방법**:

  ```ini
  display_errors = Off
  log_errors = On
  error_log = /var/log/php_errors.log
  ```

### PHP-02. 원격 파일 포함 비활성화

- **위험도**: 상
- **점검 내용**: 원격 URL 파일 포함(Remote File Inclusion) 취약점 방지
- **점검 방법**:

  ```bash
  php -i | grep -E "allow_url_fopen|allow_url_include"
  ```

- **양호 기준**: `allow_url_include = Off`
- **취약 기준**: `allow_url_include = On` (RFI 공격 가능)
- **조치 방법**: `php.ini`에서 `allow_url_include = Off` 설정

### PHP-03. expose_php 비활성화

- **위험도**: 하
- **점검 내용**: HTTP 응답 헤더에 PHP 버전 정보 노출 여부
- **점검 방법**:

  ```bash
  curl -I http://localhost/ | grep X-Powered-By
  php -i | grep expose_php
  ```

- **양호 기준**: `expose_php = Off`
- **취약 기준**: 응답 헤더에 `X-Powered-By: PHP/x.x.x` 노출

### PHP-04. 파일 업로드 보안

- **위험도**: 상
- **점검 내용**: 파일 업로드 크기 제한 및 위험 확장자 필터링
- **점검 방법**:

  ```bash
  php -i | grep -E "file_uploads|upload_max_filesize|post_max_size"
  ```

- **양호 기준**: `file_uploads = On` 시 적절한 크기 제한, 서버 측 확장자 검증
- **조치 방법**:

  ```ini
  upload_max_filesize = 10M
  post_max_size = 12M
  ```

### PHP-05. open_basedir 설정

- **위험도**: 상
- **점검 내용**: PHP가 접근 가능한 파일 시스템 경로 제한
- **점검 방법**:

  ```bash
  php -i | grep open_basedir
  ```

- **양호 기준**: `open_basedir = /var/www/html:/tmp` 등 필요한 경로만 지정
- **취약 기준**: `open_basedir` 미설정 (전체 파일 시스템 접근 가능)

### PHP-06. disable_functions 설정

- **위험도**: 상
- **점검 내용**: 위험한 PHP 내장 함수 비활성화
- **점검 방법**:

  ```bash
  php -i | grep disable_functions
  ```

- **양호 기준**: `disable_functions = exec,passthru,shell_exec,system,proc_open,popen`
- **취약 기준**: disable_functions 미설정 또는 빈 값

### PHP-07. 세션 보안 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  php -i | grep -E "session.cookie_secure|session.cookie_httponly|session.use_strict_mode"
  ```

- **양호 기준**:
  - `session.cookie_secure = On` (HTTPS 환경)
  - `session.cookie_httponly = On`
  - `session.use_strict_mode = On`
- **취약 기준**: HttpOnly/Secure 플래그 없는 세션 쿠키

---

## RabbitMQ

### RMQ-01. 기본 계정 변경

- **위험도**: 상
- **점검 내용**: 기본 관리자 계정(guest/guest) 삭제 또는 패스워드 변경
- **점검 방법**:

  ```bash
  rabbitmqctl list_users
  # 또는 Management UI: http://<host>:15672 에서 guest로 로그인 시도
  ```

- **양호 기준**: guest 계정 삭제 또는 localhost 전용으로 제한
- **취약 기준**: guest/guest 계정으로 원격 접근 가능
- **조치 방법**:

  ```bash
  rabbitmqctl delete_user guest
  rabbitmqctl add_user admin <strongpassword>
  rabbitmqctl set_user_tags admin administrator
  ```

### RMQ-02. TLS 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  cat /etc/rabbitmq/rabbitmq.conf | grep -E "ssl|tls"
  rabbitmq-diagnostics status | grep listeners
  ```

- **양호 기준**: AMQPS(5671) 사용, Management UI HTTPS 설정
- **취약 기준**: 평문 AMQP(5672) 사용

### RMQ-03. 관리 플러그인 접근 제한

- **위험도**: 상
- **점검 내용**: RabbitMQ Management Plugin 웹 UI 접근 통제
- **점검 방법**:

  ```bash
  ss -tlnp | grep 15672
  cat /etc/rabbitmq/rabbitmq.conf | grep management
  ```

- **양호 기준**: Management UI 접근을 허가된 IP 대역만 허용
- **취약 기준**: 인터넷에 Management UI(15672) 노출

### RMQ-04. 가상 호스트(vhost) 분리

- **위험도**: 중
- **점검 내용**: 애플리케이션별 vhost 분리 및 권한 최소화
- **점검 방법**:

  ```bash
  rabbitmqctl list_vhosts
  rabbitmqctl list_permissions -p /
  ```

- **양호 기준**: 애플리케이션별 별도 vhost 사용, 최소 권한 부여
- **취약 기준**: 모든 사용자가 기본 vhost(/)에 full 권한 보유

### RMQ-05. 리소스 제한 설정

- **위험도**: 중
- **점검 내용**: 메모리 및 디스크 사용량 알람 설정
- **점검 방법**:

  ```bash
  rabbitmqctl status | grep -E "memory|disk"
  cat /etc/rabbitmq/rabbitmq.conf | grep -E "memory_high_watermark|disk_free_limit"
  ```

- **양호 기준**: 메모리 watermark 및 디스크 여유 공간 최소값 설정
- **취약 기준**: 리소스 제한 미설정 (무제한 사용으로 DoS 위험)

---

## Node.js

### ND-01. 의존성 취약점 점검

- **위험도**: 상
- **점검 내용**: npm 패키지 취약점 존재 여부
- **점검 방법**:

  ```bash
  npm audit
  npm audit --json | jq '.metadata.vulnerabilities'
  ```

- **양호 기준**: High/Critical 취약점 없음
- **취약 기준**: High 또는 Critical 취약점 포함 패키지 존재
- **조치 방법**: `npm audit fix` 또는 취약 패키지 업데이트

### ND-02. 환경 변수를 통한 시크릿 관리

- **위험도**: 상
- **점검 내용**: 소스 코드 및 설정 파일에 하드코딩된 시크릿(패스워드, API 키) 여부
- **점검 방법**:

  ```bash
  grep -rn "password\|secret\|api_key\|token" --include="*.js" --include="*.json" ./ | grep -v node_modules
  ```

- **양호 기준**: 환경 변수 또는 시크릿 관리 도구(Vault, AWS Secrets Manager) 사용
- **취약 기준**: 소스 코드에 하드코딩된 자격증명

### ND-03. HTTPS 강제 적용

- **위험도**: 상
- **점검 방법**:

  ```bash
  # HTTP 리다이렉트 확인
  curl -I http://<host>:<port>/
  # 또는 소스 코드 확인
  grep -rn "listen.*80\|http.createServer" --include="*.js" ./
  ```

- **양호 기준**: HTTP 요청을 HTTPS로 리다이렉트, HSTS 헤더 설정
- **취약 기준**: HTTP 상태로 운영

### ND-04. 보안 HTTP 헤더 설정 (Helmet.js)

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -rn "helmet" package.json
  curl -I https://<host>/ | grep -E "X-Frame-Options|Strict-Transport|X-Content-Type"
  ```

- **양호 기준**: Helmet.js 또는 동등한 보안 헤더 미들웨어 적용
- **취약 기준**: 보안 HTTP 헤더 미적용

### ND-05. 에러 메시지 정보 노출 방지

- **위험도**: 중
- **점검 내용**: 운영 환경에서 스택 트레이스 등 내부 정보 노출 차단
- **점검 방법**:

  ```bash
  # 에러 발생 시 응답 확인
  curl http://<host>/error-trigger
  grep -n "NODE_ENV\|stack" --include="*.js" ./ -r
  ```

- **양호 기준**: `NODE_ENV=production` 설정, 에러 핸들러에서 스택 트레이스 미노출
- **취약 기준**: 운영 환경에서 스택 트레이스 그대로 클라이언트에 노출

### ND-06. Rate Limiting 설정

- **위험도**: 중
- **점검 내용**: API 요청 속도 제한으로 Brute Force 및 DoS 방지
- **점검 방법**:

  ```bash
  grep -rn "rateLimit\|express-rate-limit" package.json
  ```

- **양호 기준**: Rate Limiting 미들웨어(express-rate-limit 등) 적용
- **취약 기준**: Rate Limiting 없음 (무제한 API 요청 가능)

### ND-07. 입력값 검증 및 SQL 인젝션 방지

- **위험도**: 상
- **점검 내용**: 사용자 입력값에 대한 sanitization 및 parameterized query 사용 여부
- **점검 방법**:

  ```bash
  # ORM/쿼리 빌더 사용 여부 확인
  grep -rn "query\|execute\|raw" --include="*.js" ./ | grep -v node_modules | head -20
  ```

- **양호 기준**: ORM (Sequelize, TypeORM 등) 또는 parameterized query 사용
- **취약 기준**: 문자열 연결로 SQL 쿼리 생성 (SQL Injection 위험)

### ND-08. 디버그 모드 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  env | grep -E "NODE_ENV|DEBUG"
  cat .env | grep -E "NODE_ENV|DEBUG"
  ```

- **양호 기준**: `NODE_ENV=production`, DEBUG 변수 미설정
- **취약 기준**: `NODE_ENV=development` 또는 `DEBUG=*` 운영 환경 적용
