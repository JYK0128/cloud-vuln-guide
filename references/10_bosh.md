# 2장. PaaS 플랫폼 (BOSH) 보안 가이드

대상: BOSH Director, BOSH UAA (User Account and Authentication)

---

## 목차

- [BOSH Director](#bosh-director)
- [BOSH UAA](#bosh-uaa)

---

## BOSH Director

### BD-01. BOSH Director 인증 설정

- **위험도**: 상
- **점검 내용**: BOSH Director API 접근 인증 설정 여부
- **점검 방법**:

  ```bash
  bosh env
  # 응답에 인증 정보 확인
  cat ~/bosh-deploy/credentials.yml | grep -E "admin_password|director_ssl"
  ```

- **양호 기준**: UAA 기반 인증 사용, 강력한 패스워드 설정
- **취약 기준**: 기본 자격증명 유지 또는 약한 패스워드

### BD-02. Director 네트워크 접근 제한

- **위험도**: 상
- **점검 내용**: BOSH Director API(25555), Director Agent 포트 외부 노출 여부
- **점검 방법**:

  ```bash
  ss -tlnp | grep -E "25555|4222|6868"
  # IaaS 보안 그룹 확인
  bosh cloud-config | grep -A 5 "networks:"
  ```

- **양호 기준**: Jump(Bastion) 서버 또는 VPN을 통해서만 Director 접근
- **취약 기준**: Director API가 인터넷에 직접 노출

### BD-03. BOSH Agent 통신 암호화

- **위험도**: 상
- **점검 내용**: Director와 Agent 간 통신 TLS 설정 여부
- **점검 방법**:

  ```bash
  cat /var/vcap/bosh/settings.json | grep -E "cert|tls"
  bosh manifest -d <deployment> | grep -E "nats_tls|director_ssl"
  ```

- **양호 기준**: NATS 및 API 통신 모두 TLS 암호화
- **취약 기준**: 평문 NATS 통신

### BD-04. 배포 매니페스트 시크릿 관리

- **위험도**: 상
- **점검 내용**: 매니페스트 파일에 하드코딩된 자격증명 여부
- **점검 방법**:

  ```bash
  grep -rn "password\|secret\|key" ~/bosh-deploy/ | grep -v "#"
  ```

- **양호 기준**: CredHub 또는 Vault를 통한 시크릿 관리 (변수 참조: `((variable-name))`)
- **취약 기준**: 매니페스트 파일에 패스워드·키 하드코딩

### BD-05. CredHub 통합

- **위험도**: 중
- **점검 내용**: BOSH Director와 CredHub(시크릿 관리) 통합 여부
- **점검 방법**:

  ```bash
  bosh env | grep credhub
  credhub api
  ```

- **양호 기준**: CredHub 통합 및 자격증명 암호화 저장
- **취약 기준**: CredHub 미사용, 자격증명 평문 저장

### BD-06. 감사 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  cat /var/vcap/sys/log/director/director.stdout.log | grep -E "audit|action"
  ```

- **양호 기준**: 배포·삭제·접속 등 주요 작업 감사 로그 기록
- **취약 기준**: 감사 로그 미기록

### BD-07. 네트워크 정책 (Cloud Config)

- **위험도**: 중
- **점검 방법**:

  ```bash
  bosh cloud-config
  ```

- **양호 기준**: 네트워크 격리 설정, VM 간 불필요한 통신 차단
- **취약 기준**: 모든 VM이 동일 네트워크에 배치, 접근 제어 없음

---

## BOSH UAA

### UAA-01. UAA 관리자 계정 보안

- **위험도**: 상
- **점검 내용**: UAA admin 계정 패스워드 복잡도 및 기본값 변경 여부
- **점검 방법**:

  ```bash
  uaac target https://<uaa-host>:8443
  uaac token client get admin -s <admin-secret>
  uaac users | grep admin
  ```

- **양호 기준**: 강력한 admin 패스워드 설정, 기본값 변경
- **취약 기준**: 기본 admin secret 유지

### UAA-02. TLS 인증서 설정

- **위험도**: 상
- **점검 내용**: UAA HTTPS 인증서 유효성 및 TLS 설정
- **점검 방법**:

  ```bash
  echo | openssl s_client -connect <uaa-host>:8443 2>/dev/null | openssl x509 -noout -dates
  ```

- **양호 기준**: 유효한 TLS 인증서, TLS 1.2 이상 사용
- **취약 기준**: 만료된 인증서, 취약한 TLS 버전 허용

### UAA-03. 토큰 만료 시간 설정

- **위험도**: 중
- **점검 내용**: Access Token 및 Refresh Token 만료 시간 적절성
- **점검 방법**:

  ```bash
  uaac client get cf | grep -E "access_token|refresh_token"
  cat /var/vcap/jobs/uaa/config/uaa.yml | grep -E "accessTokenValidity|refreshTokenValidity"
  ```

- **양호 기준**: Access Token 1시간 이내, Refresh Token 24시간 이내
- **취약 기준**: 토큰 만료 시간 없음 또는 과도하게 길게 설정

### UAA-04. 클라이언트 권한 최소화

- **위험도**: 중
- **점검 내용**: UAA 클라이언트 권한(scope, authorities) 최소화
- **점검 방법**:

  ```bash
  uaac clients
  uaac client get <client-id>
  ```

- **양호 기준**: 각 클라이언트에 필요한 최소 scope·authorities만 부여
- **취약 기준**: `uaa.admin` 또는 `*.admin` scope가 불필요한 클라이언트에 부여

### UAA-05. 패스워드 정책 설정

- **위험도**: 중
- **점검 내용**: UAA 사용자 패스워드 복잡도 정책 설정
- **점검 방법**:

  ```bash
  cat /var/vcap/jobs/uaa/config/uaa.yml | grep -A 10 "passwordPolicy"
  ```

- **양호 기준**: 최소 길이 8자 이상, 복잡도 요건 설정, 잠금 정책 적용
- **취약 기준**: 패스워드 정책 미설정

### UAA-06. 외부 LDAP/SAML 통합 보안

- **위험도**: 중
- **점검 내용**: 외부 LDAP 또는 SAML IdP 연동 시 보안 설정
- **점검 방법**:

  ```bash
  cat /var/vcap/jobs/uaa/config/uaa.yml | grep -E "ldap:|saml:"
  ```

- **양호 기준**: LDAP over SSL(LDAPS), SAML 서명 검증 활성화
- **취약 기준**: 평문 LDAP 사용, SAML 서명 검증 비활성화
