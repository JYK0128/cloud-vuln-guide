# 2장. 하이퍼바이저 보안 가이드

대상: KVM, XenServer, ESXi, Hyper-V

---

## 목차

- [공통 점검 항목](#공통-점검-항목)
- [KVM 전용 점검 항목](#kvm-전용-점검-항목)
- [XenServer 전용 점검 항목](#xenserver-전용-점검-항목)
- [ESXi 전용 점검 항목](#esxi-전용-점검-항목)
- [Hyper-V 전용 점검 항목](#hyper-v-전용-점검-항목)

---

## 공통 점검 항목

### H-01. 계정 관리

- **위험도**: 상
- **점검 내용**: 하이퍼바이저 관리 계정의 기본 패스워드 변경 여부 및 불필요한 계정 존재 여부 확인
- **양호 기준**: 기본 계정 삭제 또는 패스워드 변경, 불필요 계정 없음
- **취약 기준**: 기본 패스워드 유지 또는 불필요한 계정 존재
- **조치 방법**: 기본 계정 삭제, 패스워드 정책 적용(8자 이상, 복잡도 충족)

### H-02. 패스워드 정책

- **위험도**: 상
- **점검 내용**: 패스워드 최소 길이, 복잡도, 만료 기간 설정 여부
- **양호 기준**: 패스워드 최소 8자 이상, 복잡도 정책 적용, 90일 이내 만료
- **취약 기준**: 패스워드 정책 미적용
- **조치 방법**: 운영체제 또는 하이퍼바이저 관리 도구에서 패스워드 정책 적용

### H-03. 관리자 접근 통제

- **위험도**: 상
- **점검 내용**: 하이퍼바이저 관리 인터페이스(웹 콘솔, SSH 등)에 대한 접근 통제 설정
- **양호 기준**: 허가된 IP에서만 접근 가능, 관리 인터페이스 접근 로깅 활성화
- **취약 기준**: 모든 IP에서 관리 인터페이스 접근 허용
- **조치 방법**: 방화벽 규칙 또는 ACL로 관리 IP 대역만 허용

### H-04. 보안 패치 관리

- **위험도**: 상
- **점검 내용**: 하이퍼바이저 소프트웨어 최신 보안 패치 적용 여부
- **양호 기준**: 최신 보안 패치 적용
- **취약 기준**: 알려진 취약점이 있는 구버전 사용
- **조치 방법**: 벤더사 보안 공지 모니터링 및 정기 패치 적용

### H-05. 로깅 및 감사

- **위험도**: 중
- **점검 내용**: 관리자 접속 및 주요 이벤트에 대한 로그 기록 활성화 여부
- **양호 기준**: 로그 기록 활성화, 로그 보관 기간 설정(최소 3개월)
- **취약 기준**: 로그 미기록 또는 단기 보관
- **조치 방법**: 감사 로그 활성화 및 보관 정책 수립

### H-06. 네트워크 분리

- **위험도**: 상
- **점검 내용**: 관리 네트워크와 서비스 네트워크 분리 여부
- **양호 기준**: 관리 네트워크와 서비스/VM 네트워크 논리적 분리
- **취약 기준**: 관리 네트워크와 서비스 네트워크 미분리
- **조치 방법**: VLAN 또는 물리적 네트워크 분리 적용

### H-07. VM 간 통신 제어

- **위험도**: 중
- **점검 내용**: 동일 하이퍼바이저 상의 VM 간 불필요한 통신 허용 여부
- **양호 기준**: VM 간 통신 필요한 경우만 허용
- **취약 기준**: 모든 VM 간 통신 허용
- **조치 방법**: 가상 스위치 보안 정책 적용, vSwitch 설정에서 Promiscuous Mode 비활성화

### H-08. 스냅샷 관리

- **위험도**: 하
- **점검 내용**: 오래된 스냅샷 존재 여부 및 스냅샷 접근 통제
- **양호 기준**: 불필요한 스냅샷 삭제, 스냅샷 접근 권한 최소화
- **취약 기준**: 오래된 스냅샷 다수 존재, 접근 통제 미흡
- **조치 방법**: 스냅샷 정책 수립 및 불필요한 스냅샷 정리

---

## KVM 전용 점검 항목

### KVM-01. libvirt 데몬 보안 설정

- **위험도**: 상
- **점검 내용**: libvirtd 접근 제어 및 TLS 인증 설정 여부
- **점검 방법**:

  ```bash
  cat /etc/libvirt/libvirtd.conf | grep -E "tls_|auth_"
  systemctl status libvirtd
  ```

- **양호 기준**: TLS 활성화, 인증서 기반 인증 사용
- **취약 기준**: TCP 소켓에 인증 없이 접근 허용
- **조치 방법**: `/etc/libvirt/libvirtd.conf` 에서 TLS 설정 및 인증 강화

### KVM-02. QEMU 보안 설정

- **위험도**: 상
- **점검 내용**: QEMU 프로세스 권한 및 seccomp 필터 설정
- **점검 방법**:

  ```bash
  cat /etc/libvirt/qemu.conf | grep -E "user|group|seccomp"
  ```

- **양호 기준**: QEMU가 비특권 사용자로 실행, seccomp 활성화
- **취약 기준**: QEMU가 root 권한으로 실행
- **조치 방법**: `/etc/libvirt/qemu.conf`에서 `user` 및 `group` 설정을 비특권 계정으로 변경

### KVM-03. 가상 네트워크 격리

- **위험도**: 중
- **점검 내용**: VM 네트워크 브리지 설정 및 격리 상태 확인
- **점검 방법**:

  ```bash
  virsh net-list --all
  virsh net-info <network-name>
  brctl show
  ```

- **양호 기준**: VM별 별도 가상 네트워크 구성, 불필요한 브리지 없음
- **취약 기준**: 모든 VM이 동일한 브리지 네트워크 사용

---

## XenServer 전용 점검 항목

### XEN-01. RBAC(역할 기반 접근 제어) 설정

- **위험도**: 상
- **점검 내용**: XenServer RBAC 정책 설정 및 최소 권한 원칙 적용 여부
- **점검 방법**:

  ```bash
  xe role-list
  xe subject-list
  ```

- **양호 기준**: 역할별 최소 권한 부여, Pool Admin 계정 최소화
- **취약 기준**: 모든 사용자에게 Pool Admin 권한 부여

### XEN-02. Xen Security Module(XSM) 활성화

- **위험도**: 중
- **점검 내용**: XSM/FLASK 보안 모듈 활성화 여부
- **점검 방법**:

  ```bash
  xl info | grep xsm
  ```

- **양호 기준**: XSM 활성화
- **취약 기준**: XSM 비활성화

---

## ESXi 전용 점검 항목

### ESXI-01. ESXi Shell 및 SSH 비활성화

- **위험도**: 상
- **점검 내용**: 불필요한 ESXi Shell 및 SSH 서비스 실행 여부
- **점검 방법**: vSphere Client → 호스트 → 구성 → 보안 프로필 → 서비스 확인
- **양호 기준**: ESXi Shell 및 SSH 비활성화(운영 중 불필요 시)
- **취약 기준**: ESXi Shell 및 SSH 항시 활성화
- **조치 방법**: vSphere Client에서 해당 서비스 중지 및 시작 유형을 '수동'으로 변경

### ESXI-02. Lockdown Mode 활성화

- **위험도**: 상
- **점검 내용**: ESXi Lockdown Mode를 통한 직접 접근 제한 여부
- **점검 방법**: vSphere Client → 호스트 → 구성 → 보안 프로필 → Lockdown Mode 확인
- **양호 기준**: Normal 또는 Strict Lockdown Mode 활성화
- **취약 기준**: Lockdown Mode 비활성화
- **조치 방법**: vCenter를 통해서만 호스트 관리하도록 Lockdown Mode 활성화

### ESXI-03. 가상 스위치 Promiscuous Mode 비활성화

- **위험도**: 상
- **점검 내용**: vSwitch의 Promiscuous Mode, MAC 주소 변경, Forged Transmits 설정
- **점검 방법**:

  ```bash
  esxcli network vswitch standard policy security get -v <vswitch-name>
  ```

- **양호 기준**: 세 가지 모두 `Reject` 설정
- **취약 기준**: Promiscuous Mode 또는 Forged Transmits `Accept` 설정

### ESXI-04. NTP 서버 설정

- **위험도**: 하
- **점검 내용**: NTP 서버 설정 및 시간 동기화 여부
- **점검 방법**: vSphere Client → 호스트 → 구성 → 시간 구성 확인
- **양호 기준**: NTP 서버 설정 및 동기화 정상
- **취약 기준**: NTP 미설정 또는 동기화 실패

---

## Hyper-V 전용 점검 항목

### HV-01. Hyper-V 관리자 그룹 최소화

- **위험도**: 상
- **점검 내용**: Hyper-V 관리자 그룹(Hyper-V Administrators) 구성원 최소화
- **점검 방법** (PowerShell):

  ```powershell
  Get-LocalGroupMember -Group "Hyper-V Administrators"
  ```

- **양호 기준**: 필요한 관리자만 그룹에 포함
- **취약 기준**: 불필요한 계정이 Hyper-V Administrators 그룹에 포함

### HV-02. Enhanced Session Mode 통제

- **위험도**: 중
- **점검 내용**: Enhanced Session Mode를 통한 VM 접근 통제
- **점검 방법** (PowerShell):

  ```powershell
  Get-VMHost | Select-Object EnableEnhancedSessionMode
  ```

- **양호 기준**: 필요 시에만 Enhanced Session Mode 활성화, 접근 로그 기록
- **취약 기준**: 제어 없이 Enhanced Session Mode 활성화

### HV-03. 가상 스위치 외부 접근 제한

- **위험도**: 중
- **점검 내용**: External 유형 가상 스위치의 사용 제한 및 관리 OS 공유 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-VMSwitch | Select-Object Name, SwitchType, AllowManagementOS
  ```

- **양호 기준**: `AllowManagementOS`가 필요한 경우에만 True, External 스위치 최소화
- **취약 기준**: 불필요한 External 스위치가 관리 OS와 공유
