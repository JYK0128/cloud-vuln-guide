# 2장. 네트워크 장비 보안 가이드

대상: 라우터, 스위치, 방화벽 등 네트워크 장비 (Cisco, Juniper, Huawei 등)

---

## 목차

- [공통 점검 항목](#공통-점검-항목)
- [라우터 / 스위치 전용 점검 항목](#라우터--스위치-전용-점검-항목)

---

## 공통 점검 항목

### NW-01. 기본 계정 및 패스워드 변경

- **위험도**: 상
- **점검 내용**: 장비 기본 관리자 계정 및 패스워드 변경 여부
- **점검 방법**:

  ```ios
  # Cisco IOS
  show running-config | include username
  # 또는 기본 패스워드(cisco/admin/password)로 로그인 시도
  ```

- **양호 기준**: 기본 계정 삭제 및 강력한 패스워드 설정
- **취약 기준**: 기본 패스워드 유지 (cisco/admin 등)
- **조치 방법**:

  ```ios
  # Cisco IOS
  username admin privilege 15 secret <strongpassword>
  no username cisco
  ```

### NW-02. 패스워드 암호화 저장

- **위험도**: 상
- **점검 내용**: 설정 파일 내 패스워드 평문 저장 여부
- **점검 방법**:

  ```ios
  # Cisco IOS
  show running-config | include enable password
  show running-config | include password 7
  ```

- **양호 기준**: `service password-encryption` 적용 또는 `enable secret` (MD5/SHA) 사용
- **취약 기준**: `enable password` 평문 설정
- **조치 방법**:

  ```ios
  service password-encryption
  enable secret <strongpassword>
  ```

### NW-03. 원격 접속 프로토콜 보안

- **위험도**: 상
- **점검 내용**: Telnet 비활성화, SSH v2 사용 여부
- **점검 방법**:

  ```ios
  # Cisco IOS
  show line vty 0 4
  show running-config | include transport input
  show ip ssh
  ```

- **양호 기준**: `transport input ssh`, SSH version 2 사용
- **취약 기준**: Telnet 허용 또는 SSH version 1 사용
- **조치 방법**:

  ```ios
  ip ssh version 2
  line vty 0 4
   transport input ssh
  ```

### NW-04. 접속 실패 임계값 설정

- **위험도**: 상
- **점검 내용**: 로그인 실패 임계값 및 잠금 설정
- **점검 방법**:

  ```ios
  # Cisco IOS
  show running-config | include login block-for
  show running-config | include security authentication
  ```

- **양호 기준**: `login block-for 30 attempts 5 within 60` 등 임계값 설정
- **취약 기준**: 로그인 실패 제한 없음
- **조치 방법**:

  ```ios
  login block-for 30 attempts 5 within 60
  login on-failure log
  ```

### NW-05. 불필요한 서비스 비활성화

- **위험도**: 중
- **점검 내용**: SNMP v1/v2c, Finger, HTTP, CDP 등 불필요/취약 서비스 비활성화
- **점검 방법**:

  ```ios
  # Cisco IOS
  show running-config | include ^service
  show running-config | include ^snmp
  show ip http server status
  ```

- **양호 기준**: 불필요한 서비스 비활성화, SNMP v3 사용
- **취약 기준**: SNMP v1/v2c(community string: public/private), HTTP 서버 활성화
- **조치 방법**:

  ```ios
  no service finger
  no ip http server
  no cdp run
  no snmp-server community public ro
  ```

### NW-06. 관리 접속 IP 제한 (ACL)

- **위험도**: 상
- **점검 내용**: 관리 인터페이스(SSH, SNMP, HTTP) 접근 허용 IP 제한
- **점검 방법**:

  ```ios
  # Cisco IOS
  show ip access-lists
  show running-config | section line vty
  ```

- **양호 기준**: 허가된 관리 IP 대역만 접근 허용 ACL 적용
- **취약 기준**: 모든 IP에서 관리 접속 허용
- **조치 방법**:

  ```ios
  ip access-list standard MGMT-ACL
   permit 192.168.1.0 0.0.0.255
   deny any log
  line vty 0 15
   access-class MGMT-ACL in
  ```

### NW-07. 로그 및 감사 설정

- **위험도**: 중
- **점검 내용**: Syslog 서버 설정, 로그 레벨 설정
- **점검 방법**:

  ```ios
  # Cisco IOS
  show logging
  show running-config | include logging
  ```

- **양호 기준**: 외부 Syslog 서버로 로그 전송, 로그 레벨 informational 이상
- **취약 기준**: 로그 미설정 또는 로컬 버퍼만 사용
- **조치 방법**:

  ```ios
  logging <syslog-server-ip>
  logging trap informational
  logging on
  ```

### NW-08. NTP 설정

- **위험도**: 중
- **점검 내용**: NTP(Network Time Protocol) 서버 설정 및 인증
- **점검 방법**:

  ```ios
  # Cisco IOS
  show ntp status
  show running-config | include ntp
  ```

- **양호 기준**: NTP 서버 설정, NTP 인증 적용
- **취약 기준**: NTP 미설정 (로그 타임스탬프 신뢰 불가)
- **조치 방법**:

  ```ios
  ntp authentication-key 1 md5 <authkey>
  ntp authenticate
  ntp trusted-key 1
  ntp server <ntp-server-ip> key 1
  ```

### NW-09. SNMP 보안 설정

- **위험도**: 상
- **점검 내용**: SNMP community string 및 버전 설정
- **점검 방법**:

  ```ios
  # Cisco IOS
  show running-config | include snmp
  ```

- **양호 기준**: SNMP v3 사용(authPriv 수준), 기본 community string(public/private) 제거
- **취약 기준**: SNMP v1/v2c + 기본 community string 사용
- **조치 방법**:

  ```ios
  no snmp-server community public
  no snmp-server community private
  snmp-server group SNMPV3GROUP v3 priv
  snmp-server user snmpv3user SNMPV3GROUP v3 auth sha <authpass> priv aes 128 <privpass>
  ```

### NW-10. 배너 메시지 설정

- **위험도**: 하
- **점검 내용**: 접속 전 법적 경고 배너 메시지 설정(무단 접속 금지)
- **점검 방법**:

  ```ios
  show running-config | include banner
  ```

- **양호 기준**: `banner login` 또는 `banner motd`로 법적 경고 메시지 설정
- **취약 기준**: 배너 미설정 또는 시스템 정보 노출 배너

---

## 라우터 / 스위치 전용 점검 항목

### NW-SW-01. 미사용 포트 shutdown

- **위험도**: 중
- **점검 내용**: 사용하지 않는 스위치 포트 비활성화
- **점검 방법**:

  ```ios
  # Cisco IOS
  show interfaces status
  ```

- **양호 기준**: 미사용 포트 `shutdown` 상태
- **취약 기준**: 미사용 포트 활성화 상태
- **조치 방법**:

  ```ios
  interface range GigabitEthernet 0/1-24
   shutdown
  ```

### NW-SW-02. VLAN 설정 및 분리

- **위험도**: 중
- **점검 내용**: 관리 VLAN, 사용자 VLAN, DMZ 등 역할별 분리 여부
- **점검 방법**:

  ```ios
  show vlan brief
  show running-config | section vlan
  ```

- **양호 기준**: 역할별 VLAN 분리, Native VLAN을 기본값(1)에서 변경
- **취약 기준**: 모든 호스트가 동일 VLAN 사용

### NW-SW-03. 스패닝 트리 보안

- **위험도**: 중
- **점검 내용**: STP PortFast, BPDU Guard, Root Guard 설정
- **점검 방법**:

  ```ios
  show spanning-tree summary
  show running-config | include portfast\|bpduguard\|guard root
  ```

- **양호 기준**: 엔드포인트 포트에 PortFast+BPDU Guard, 루트 포트에 Root Guard
- **취약 기준**: BPDU Guard 없이 PortFast 적용 (STP 공격 위험)

### NW-RT-01. 라우팅 프로토콜 인증

- **위험도**: 중
- **점검 내용**: OSPF, BGP, EIGRP 등 라우팅 프로토콜 인증 설정
- **점검 방법**:

  ```ios
  show running-config | section router ospf
  show running-config | section router bgp
  ```

- **양호 기준**: MD5 또는 SHA 인증 설정
- **취약 기준**: 라우팅 프로토콜 인증 없음 (경로 삽입 공격 위험)

### NW-RT-02. IP Source Guard / DHCP Snooping

- **위험도**: 중
- **점검 내용**: IP 스푸핑 방지 기능 설정
- **점검 방법**:

  ```ios
  show ip dhcp snooping
  show ip source binding
  ```

- **양호 기준**: DHCP Snooping + IP Source Guard 활성화
- **취약 기준**: IP 스푸핑 방지 기능 없음
