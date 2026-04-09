# 2장. 서버 보안 가이드

대상: Linux (CentOS, Ubuntu, RHEL 등), Windows Server

---

## 목차

- [Linux 서버 점검 항목](#linux-서버-점검-항목)
- [Windows 서버 점검 항목](#windows-서버-점검-항목)

---

## Linux 서버 점검 항목

### L-01. root 계정 원격 접속 제한

- **위험도**: 상
- **점검 내용**: SSH를 통한 root 계정 직접 로그인 허용 여부
- **점검 방법**:

  ```bash
  grep "^PermitRootLogin" /etc/ssh/sshd_config
  ```

- **양호 기준**: `PermitRootLogin no`
- **취약 기준**: `PermitRootLogin yes` 또는 설정 없음(기본값이 yes인 구버전)
- **조치 방법**:

  ```bash
  # /etc/ssh/sshd_config 편집
  PermitRootLogin no
  systemctl restart sshd
  ```

### L-02. 패스워드 복잡도 설정

- **위험도**: 상
- **점검 내용**: 패스워드 복잡도 정책(최소 길이, 문자 조합) 설정 여부
- **점검 방법**:

  ```bash
  cat /etc/security/pwquality.conf
  # 또는
  cat /etc/pam.d/system-auth | grep pam_pwquality
  ```

- **양호 기준**: 최소 8자 이상, 대소문자/숫자/특수문자 조합
- **취약 기준**: 복잡도 정책 미적용
- **조치 방법**:

  ```bash
  # /etc/security/pwquality.conf 편집
  minlen = 8
  dcredit = -1
  ucredit = -1
  lcredit = -1
  ocredit = -1
  ```

### L-03. 계정 잠금 정책

- **위험도**: 상
- **점검 내용**: 로그인 실패 시 계정 잠금 정책 설정 여부
- **점검 방법**:

  ```bash
  cat /etc/pam.d/system-auth | grep pam_tally2
  # 또는
  cat /etc/pam.d/common-auth | grep pam_faillock
  ```

- **양호 기준**: 5회 이상 실패 시 계정 잠금(최소 30분)
- **취약 기준**: 계정 잠금 정책 미설정
- **조치 방법**: PAM 설정에 `pam_faillock` 또는 `pam_tally2` 모듈 적용

### L-04. 패스워드 만료 정책

- **위험도**: 중
- **점검 내용**: 패스워드 최대 사용 기간 설정 여부
- **점검 방법**:

  ```bash
  cat /etc/login.defs | grep -E "PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE"
  chage -l <username>
  ```

- **양호 기준**: PASS_MAX_DAYS 90 이하
- **취약 기준**: PASS_MAX_DAYS 미설정 또는 99999 등 장기 설정
- **조치 방법**:

  ```bash
  # /etc/login.defs 편집
  PASS_MAX_DAYS   90
  PASS_MIN_DAYS   1
  PASS_WARN_AGE   7
  ```

### L-05. 불필요한 계정 제거

- **위험도**: 중
- **점검 내용**: 사용하지 않는 시스템 계정 및 일반 계정 존재 여부
- **점검 방법**:

  ```bash
  cat /etc/passwd | grep -v "nologin\|false" | awk -F: '$3 >= 1000'
  last | head -20
  ```

- **양호 기준**: 업무상 필요한 계정만 존재
- **취약 기준**: 퇴직자, 테스트 계정 등 불필요한 계정 존재
- **조치 방법**: `userdel -r <username>`으로 불필요한 계정 삭제

### L-06. sudo 권한 관리

- **위험도**: 상
- **점검 내용**: sudo 권한 부여 현황 및 불필요한 권한 부여 여부
- **점검 방법**:

  ```bash
  cat /etc/sudoers
  cat /etc/sudoers.d/*
  ```

- **양호 기준**: 필요한 명령어만 권한 부여, `NOPASSWD` 미사용(불가피한 경우 제외)
- **취약 기준**: `ALL=(ALL) ALL` 또는 `NOPASSWD: ALL` 설정
- **조치 방법**: 최소 권한 원칙에 따라 sudo 설정 수정

### L-07. SSH 설정 강화

- **위험도**: 상
- **점검 내용**: SSH 프로토콜 버전, 빈 패스워드, 유휴 세션 타임아웃 등
- **점검 방법**:

  ```bash
  cat /etc/ssh/sshd_config | grep -E "Protocol|PermitEmptyPasswords|ClientAliveInterval|MaxAuthTries"
  ```

- **양호 기준**:
  - `Protocol 2`
  - `PermitEmptyPasswords no`
  - `ClientAliveInterval 300` (5분)
  - `MaxAuthTries 5` 이하
- **취약 기준**: SSH Protocol 1 사용, 빈 패스워드 허용
- **조치 방법**: `/etc/ssh/sshd_config` 수정 후 `systemctl restart sshd`

### L-08. 파일 및 디렉토리 권한 설정

- **위험도**: 상
- **점검 내용**: 주요 시스템 파일 및 디렉토리 권한 적절성
- **점검 방법**:

  ```bash
  ls -l /etc/passwd /etc/shadow /etc/group /etc/sudoers
  find / -perm -4000 -o -perm -2000 2>/dev/null | head -30
  ```

- **양호 기준**:
  - `/etc/passwd`: 644
  - `/etc/shadow`: 000 또는 400
  - SUID/SGID 파일 최소화
- **취약 기준**: shadow 파일이 모든 사용자에게 읽기 가능

### L-09. 세션 타임아웃 설정

- **위험도**: 중
- **점검 내용**: 콘솔/터미널 세션 자동 종료 설정
- **점검 방법**:

  ```bash
  echo $TMOUT
  cat /etc/profile | grep TMOUT
  cat /etc/profile.d/*.sh | grep TMOUT
  ```

- **양호 기준**: TMOUT=300 이하 (5분 이내 자동 종료)
- **취약 기준**: TMOUT 미설정 또는 0

### L-10. 불필요한 서비스 비활성화

- **위험도**: 중
- **점검 내용**: 불필요한 데몬/서비스 실행 여부
- **점검 방법**:

  ```bash
  systemctl list-units --type=service --state=active
  ss -tlnp
  ```

- **양호 기준**: 업무상 필요한 서비스만 실행
- **취약 기준**: Telnet, FTP, rsh, rlogin 등 취약 서비스 실행 중
- **조치 방법**: `systemctl disable --now <service-name>`

### L-11. 로그 관리

- **위험도**: 중
- **점검 내용**: 시스템 로그 설정 및 보관 정책
- **점검 방법**:

  ```bash
  cat /etc/rsyslog.conf
  cat /etc/logrotate.conf
  ls -la /var/log/
  ```

- **양호 기준**: 주요 로그(auth, syslog, secure) 기록, 3개월 이상 보관
- **취약 기준**: 로그 미설정 또는 단기 보관

### L-12. 커널 보안 파라미터

- **위험도**: 중
- **점검 내용**: sysctl 보안 관련 파라미터 설정 여부
- **점검 방법**:

  ```bash
  sysctl -a | grep -E "ip_forward|send_redirects|accept_redirects|syncookies"
  ```

- **양호 기준**:
  - `net.ipv4.ip_forward=0` (라우터가 아닌 경우)
  - `net.ipv4.conf.all.send_redirects=0`
  - `net.ipv4.tcp_syncookies=1`
- **조치 방법**: `/etc/sysctl.conf` 수정 후 `sysctl -p`

### L-13. 업데이트 및 패치 관리

- **위험도**: 상
- **점검 내용**: 보안 패치 적용 현황 확인
- **점검 방법**:

  ```bash
  # CentOS/RHEL
  yum check-update --security
  # Ubuntu
  apt list --upgradable 2>/dev/null | grep -i security
  ```

- **양호 기준**: 최신 보안 패치 적용
- **취약 기준**: 알려진 CVE가 있는 패키지 미패치

---

## Windows 서버 점검 항목

### W-01. Administrator 계정 관리

- **위험도**: 상
- **점검 내용**: 기본 Administrator 계정 비활성화 또는 이름 변경 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-LocalUser | Select-Object Name, Enabled, LastLogon
  ```

- **양호 기준**: 기본 Administrator 계정 비활성화 또는 이름 변경
- **취약 기준**: 기본 Administrator 계정 활성화 상태로 사용
- **조치 방법**: 계정 이름 변경 또는 비활성화 후 별도 관리자 계정 사용

### W-02. 패스워드 정책

- **위험도**: 상
- **점검 내용**: 로컬 보안 정책의 패스워드 정책 설정
- **점검 방법** (PowerShell):

  ```powershell
  net accounts
  # 또는 secpol.msc → 계정 정책 → 암호 정책
  ```

- **양호 기준**: 최소 길이 8자 이상, 복잡성 조건 사용, 최대 사용 기간 90일
- **취약 기준**: 패스워드 정책 미적용

### W-03. 계정 잠금 정책

- **위험도**: 상
- **점검 내용**: 로그인 실패 시 계정 잠금 설정
- **점검 방법** (PowerShell):

  ```powershell
  net accounts
  ```

- **양호 기준**: 5회 이상 실패 시 잠금, 잠금 기간 30분 이상
- **취약 기준**: 계정 잠금 정책 미설정 (0 = 잠금 없음)

### W-04. 공유 폴더 제거

- **위험도**: 중
- **점검 내용**: 불필요한 공유 폴더 존재 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-SmbShare
  net share
  ```

- **양호 기준**: 업무상 필요한 공유만 존재, IPC$, ADMIN$ 등 숨겨진 공유 통제
- **취약 기준**: 불필요한 공유 폴더 존재 및 Everyone 읽기/쓰기 권한

### W-05. 원격 데스크톱(RDP) 보안 설정

- **위험도**: 상
- **점검 내용**: RDP 서비스 활성화 여부 및 접근 통제
- **점검 방법** (PowerShell):

  ```powershell
  Get-ItemProperty "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections
  Get-NetFirewallRule -DisplayName "*Remote Desktop*" | Select-Object DisplayName, Enabled
  ```

- **양호 기준**: RDP 불필요 시 비활성화, 필요 시 특정 IP 대역만 허용, NLA 활성화
- **취약 기준**: RDP 활성화 + 모든 IP 허용

### W-06. 이벤트 로그 설정

- **위험도**: 중
- **점검 내용**: Windows 이벤트 로그 크기 및 보존 정책
- **점검 방법** (PowerShell):

  ```powershell
  Get-EventLog -List | Select-Object Log, MaximumKilobytes, OverflowAction
  ```

- **양호 기준**: 보안 로그 최소 65MB 이상, 덮어쓰기 방지
- **취약 기준**: 로그 크기 부족으로 인한 자동 삭제

### W-07. Windows Update 설정

- **위험도**: 상
- **점검 내용**: 자동 업데이트 설정 및 최신 패치 적용 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
  ```

- **양호 기준**: 자동 업데이트 활성화 또는 정기 패치 프로세스 운영
- **취약 기준**: 오랫동안 패치 미적용 (3개월 이상)

### W-08. 불필요한 서비스 비활성화

- **위험도**: 중
- **점검 내용**: 불필요한 Windows 서비스 실행 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-Service | Where-Object Status -eq 'Running' | Select-Object Name, DisplayName, StartType
  ```

- **양호 기준**: Telnet, FTP, TFTP 등 취약 서비스 비활성화
- **취약 기준**: 사용하지 않는 취약 서비스 실행 중

### W-09. 방화벽 활성화

- **위험도**: 상
- **점검 내용**: Windows 방화벽 활성화 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-NetFirewallProfile | Select-Object Name, Enabled
  ```

- **양호 기준**: 도메인/개인/공용 모든 프로필에서 방화벽 활성화
- **취약 기준**: 방화벽 비활성화

### W-10. 바이러스 백신 및 EDR

- **위험도**: 상
- **점검 내용**: 바이러스 백신 설치 및 최신 업데이트 여부
- **점검 방법** (PowerShell):

  ```powershell
  Get-MpComputerStatus | Select-Object AMEngineVersion, AntispywareSignatureLastUpdated, RealTimeProtectionEnabled
  ```

- **양호 기준**: 백신 설치, 실시간 보호 활성화, 최신 시그니처 업데이트
- **취약 기준**: 백신 미설치 또는 비활성화
