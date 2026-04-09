# 2장. 웹서버 / WAS 보안 가이드

대상: Apache HTTP Server, Nginx, IIS (Internet Information Services), Apache Tomcat

---

## 목차

- [Apache HTTP Server](#apache-http-server)
- [Nginx](#nginx)
- [IIS](#iis-internet-information-services)
- [Apache Tomcat](#apache-tomcat)

---

## Apache HTTP Server

### AP-01. 서버 버전 정보 숨기기

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -E "ServerTokens|ServerSignature" /etc/httpd/conf/httpd.conf
  curl -I http://localhost/ | grep Server
  ```

- **양호 기준**: `ServerTokens Prod`, `ServerSignature Off`
- **취약 기준**: 서버 버전 및 OS 정보 노출
- **조치 방법**:

  ```httpd.conf
  ServerTokens Prod
  ServerSignature Off
  ```

### AP-02. 디렉토리 인덱싱 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -rn "Options.*Indexes" /etc/httpd/
  # 또는 브라우저로 /icons/ 등 접근 시 파일 목록 노출 여부 확인
  ```

- **양호 기준**: `Options -Indexes` 또는 Indexes 없음
- **취약 기준**: `Options +Indexes` 또는 `Options Indexes`
- **조치 방법**: 각 Directory 블록에서 `Indexes` 옵션 제거

### AP-03. 불필요한 모듈 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  httpd -M 2>&1 | grep -E "status|info|userdir|cgi"
  ```

- **양호 기준**: mod_status, mod_info, mod_userdir, 불필요한 mod_cgi 비활성화
- **취약 기준**: 운영에 불필요한 모듈 활성화

### AP-04. 심볼릭 링크 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -rn "FollowSymLinks\|SymLinksIfOwnerMatch" /etc/httpd/
  ```

- **양호 기준**: `Options -FollowSymLinks`
- **취약 기준**: `Options +FollowSymLinks`

### AP-05. 접근 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -E "CustomLog|LogFormat" /etc/httpd/conf/httpd.conf
  ls -la /var/log/httpd/
  ```

- **양호 기준**: 접근 로그 및 오류 로그 설정, 로그 보관 기간 90일 이상
- **취약 기준**: 로그 미설정 또는 단기 보관

### AP-06. SSL/TLS 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  grep -E "SSLProtocol|SSLCipherSuite|SSLCertificate" /etc/httpd/conf.d/ssl.conf
  openssl s_client -connect <host>:443 -tls1  # SSLv3/TLS 1.0 취약 여부 확인
  ```

- **양호 기준**: TLS 1.2 이상 사용, 취약 Cipher Suite 비활성화
- **취약 기준**: SSLv2/SSLv3/TLS 1.0 허용, RC4·DES·MD5 등 취약 암호화 사용
- **조치 방법**:

  ```ssl.conf
  SSLProtocol -all +TLSv1.2 +TLSv1.3
  SSLCipherSuite HIGH:!aNULL:!MD5:!RC4
  ```

### AP-07. HTTP 보안 헤더 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  curl -I https://<host>/ | grep -E "X-Frame-Options|X-Content-Type|Strict-Transport|Content-Security"
  ```

- **양호 기준**: 주요 보안 헤더 설정
- **조치 방법**:

  ```httpd.conf
  Header always set X-Frame-Options "SAMEORIGIN"
  Header always set X-Content-Type-Options "nosniff"
  Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
  Header always set Content-Security-Policy "default-src 'self'"
  ```

### AP-08. 관리자 인터페이스 접근 제한

- **위험도**: 상
- **점검 내용**: mod_status, server-status 접근 통제
- **점검 방법**:

  ```bash
  curl http://localhost/server-status  # 접근 가능하면 취약
  ```

- **조치 방법**: `/server-status`를 특정 IP만 허용하거나 비활성화

---

## Nginx

### NG-01. 서버 버전 정보 숨기기

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep "server_tokens" /etc/nginx/nginx.conf
  curl -I http://localhost/ | grep Server
  ```

- **양호 기준**: `server_tokens off;`
- **취약 기준**: 서버 버전 정보 노출
- **조치 방법**: `nginx.conf`의 `http` 블록에 `server_tokens off;` 추가

### NG-02. 디렉토리 인덱싱 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -rn "autoindex" /etc/nginx/
  ```

- **양호 기준**: `autoindex off;` (기본값)
- **취약 기준**: `autoindex on;`

### NG-03. SSL/TLS 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  grep -E "ssl_protocols|ssl_ciphers|ssl_prefer_server_ciphers" /etc/nginx/nginx.conf
  ```

- **양호 기준**:
  - `ssl_protocols TLSv1.2 TLSv1.3;`
  - `ssl_prefer_server_ciphers on;`
  - `ssl_ciphers 'HIGH:!aNULL:!MD5:!RC4';`
- **취약 기준**: TLS 1.0/1.1 또는 SSLv3 허용

### NG-04. 요청 크기 제한

- **위험도**: 중
- **점검 내용**: 대용량 요청을 통한 DoS 방지
- **점검 방법**:

  ```bash
  grep "client_max_body_size" /etc/nginx/nginx.conf
  ```

- **양호 기준**: 적절한 크기 제한 설정 (예: `client_max_body_size 10m;`)
- **취약 기준**: 제한 없음 또는 지나치게 큰 값

### NG-05. 액세스 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -E "access_log|error_log" /etc/nginx/nginx.conf
  ```

- **양호 기준**: 접근 및 오류 로그 활성화, 로그 보관 기간 설정
- **취약 기준**: `access_log off;`

### NG-06. HTTP 보안 헤더

- **위험도**: 중
- **조치 방법**:

  ```nginx
  add_header X-Frame-Options "SAMEORIGIN";
  add_header X-Content-Type-Options "nosniff";
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
  add_header X-XSS-Protection "1; mode=block";
  ```

---

## IIS (Internet Information Services)

### IIS-01. 불필요한 HTTP 메소드 비활성화

- **위험도**: 중
- **점검 내용**: PUT, DELETE, TRACE, OPTIONS 등 불필요한 HTTP 메소드 허용 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-WebConfiguration /system.webServer/security/requestFiltering/verbs | Select-Object *
  ```

- **양호 기준**: GET, POST만 허용 (업무에 필요한 경우 추가)
- **취약 기준**: PUT, DELETE, TRACE 등 허용

### IIS-02. 디렉토리 탐색 비활성화

- **위험도**: 중
- **점검 방법** (PowerShell):

  ```powershell
  Get-WebConfigurationProperty /system.webServer/directoryBrowse -PSPath 'IIS:\' -Name enabled
  ```

- **양호 기준**: `False`
- **취약 기준**: `True` (디렉토리 목록 노출)
- **조치 방법**: IIS Manager → 기능 → 디렉토리 탐색 → 비활성화

### IIS-03. 서버 헤더 제거

- **위험도**: 중
- **점검 방법**:

  ```bash
  curl -I http://localhost/ | grep Server
  ```

- **양호 기준**: Server 헤더 없음 또는 버전 정보 없음
- **조치 방법**: `removeServerHeader="true"` 설정 또는 URLRewrite 모듈로 제거

### IIS-04. 요청 필터링 설정

- **위험도**: 중
- **점검 내용**: 파일 확장자, 쿼리 문자열 길이 제한 등
- **점검 방법** (PowerShell):

  ```powershell
  Get-WebConfiguration /system.webServer/security/requestFiltering -PSPath 'IIS:\'
  ```

### IIS-05. SSL/TLS 설정

- **위험도**: 상
- **점검 방법** (PowerShell):

  ```powershell
  Get-Item "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\*"
  ```

- **양호 기준**: TLS 1.2 이상만 활성화

---

## Apache Tomcat

### TC-01. 관리 인터페이스 비활성화

- **위험도**: 상
- **점검 내용**: Tomcat Manager 웹 애플리케이션 비활성화 또는 접근 제한
- **점검 방법**:

  ```bash
  ls /opt/tomcat/webapps/ | grep -E "manager|host-manager"
  curl http://localhost:8080/manager/html  # 접근 가능하면 취약
  ```

- **양호 기준**: Manager 앱 삭제 또는 localhost 전용 접근 제한
- **취약 기준**: 기본 자격증명(tomcat/tomcat)으로 외부 접근 가능

### TC-02. 기본 계정 변경

- **위험도**: 상
- **점검 방법**:

  ```bash
  cat /opt/tomcat/conf/tomcat-users.xml
  ```

- **양호 기준**: 기본 계정 삭제 및 강력한 패스워드로 변경
- **취약 기준**: admin/admin, tomcat/tomcat 등 기본 자격증명 유지

### TC-03. 불필요한 기본 페이지 제거

- **위험도**: 하
- **점검 방법**:

  ```bash
  ls /opt/tomcat/webapps/
  ```

- **양호 기준**: docs, examples, ROOT 등 불필요한 기본 앱 제거
- **취약 기준**: 기본 예제 앱(examples) 존재

### TC-04. TRACE 메소드 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  curl -v -X TRACE http://localhost:8080/
  grep "allowTrace" /opt/tomcat/conf/server.xml
  ```

- **양호 기준**: `allowTrace="false"` (기본값)
- **취약 기준**: TRACE 메소드 허용 (XST 공격 위험)

### TC-05. AJP 커넥터 비활성화

- **위험도**: 상
- **점검 내용**: Apache JServ Protocol(AJP) 커넥터 노출 여부 (Ghostcat CVE-2020-1938)
- **점검 방법**:

  ```bash
  grep -n "AJP\|8009" /opt/tomcat/conf/server.xml
  ss -tlnp | grep 8009
  ```

- **양호 기준**: AJP 커넥터 비활성화 또는 `secretRequired="true"` 설정
- **취약 기준**: AJP 포트(8009) 외부 노출
- **조치 방법**: `server.xml`에서 AJP Connector 주석 처리 또는 `address="localhost"` 설정

### TC-06. 자동 배포 비활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -E "autoDeploy|deployOnStartup" /opt/tomcat/conf/server.xml
  ```

- **양호 기준**: `autoDeploy="false"`, `deployOnStartup="false"`
- **취약 기준**: autoDeploy 활성화 (임의 파일 배포 위험)

### TC-07. 에러 페이지 커스텀화

- **위험도**: 하
- **점검 내용**: 기본 에러 페이지를 통한 정보 노출 방지
- **점검 방법**:

  ```bash
  curl http://localhost:8080/nonexistentpage
  ```

- **양호 기준**: 커스텀 에러 페이지로 내부 정보 숨김
- **취약 기준**: 기본 에러 페이지에서 Tomcat 버전 및 스택 트레이스 노출

### TC-08. SSL/TLS 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  grep -A 10 "SSLEnabled" /opt/tomcat/conf/server.xml
  ```

- **양호 기준**: TLS 1.2 이상, 취약 Cipher 비활성화
- **취약 기준**: SSLv3/TLS 1.0 허용
