# 2장. 컨테이너 / 가상화 인프라 보안 가이드

대상: Docker, Kubernetes(Master/Worker Node), OpenStack

---

## 목차

- [Docker](#docker)
- [Kubernetes](#kubernetes)
- [OpenStack](#openstack)

---

## Docker

### DK-01. Docker 데몬 소켓 접근 통제

- **위험도**: 상
- **점검 내용**: Docker 소켓(/var/run/docker.sock) 접근 권한 및 TCP 소켓 노출 여부
- **점검 방법**:

  ```bash
  ls -la /var/run/docker.sock
  ps -ef | grep dockerd | grep -E "tcp:|host:"
  cat /etc/docker/daemon.json | grep -E "hosts|tls"
  ```

- **양호 기준**: Docker 소켓 root 소유, TCP 소켓 미사용 또는 TLS 인증 적용
- **취약 기준**: `tcp://0.0.0.0:2375` 등 인증 없는 TCP 소켓 노출
- **조치 방법**: `/etc/docker/daemon.json`에서 TLS 설정 또는 TCP 소켓 제거

### DK-02. 컨테이너 루트 권한 실행 제한

- **위험도**: 상
- **점검 내용**: 컨테이너가 root 사용자로 실행 여부
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep -E "User|Privileged"
  docker ps -q | xargs docker inspect --format='{{.Id}}: User={{.Config.User}} Privileged={{.HostConfig.Privileged}}'
  ```

- **양호 기준**: 비특권 사용자로 실행, Privileged=false
- **취약 기준**: root 사용자 실행 또는 Privileged=true
- **조치 방법**: Dockerfile에 `USER <non-root-user>` 추가, `--privileged` 플래그 미사용

### DK-03. 컨테이너 파일 시스템 읽기 전용

- **위험도**: 중
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep ReadonlyRootfs
  ```

- **양호 기준**: `ReadonlyRootfs: true`
- **취약 기준**: 쓰기 가능한 루트 파일 시스템
- **조치 방법**: `docker run --read-only` 옵션 사용

### DK-04. 네트워크 모드 설정

- **위험도**: 중
- **점검 내용**: host 네트워크 모드 사용 여부
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep '"NetworkMode"'
  ```

- **양호 기준**: NetworkMode가 host 미사용 (bridge, custom network 사용)
- **취약 기준**: `NetworkMode: host` (호스트 네트워크 스택 공유)

### DK-05. 호스트 볼륨 마운트 최소화

- **위험도**: 상
- **점검 내용**: 민감한 호스트 경로 마운트 여부
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep -A 20 '"Binds"'
  ```

- **양호 기준**: 불필요한 호스트 경로 마운트 없음
- **취약 기준**: `/etc`, `/var/run/docker.sock`, `/proc` 등 민감 경로 마운트

### DK-06. 이미지 취약점 스캔

- **위험도**: 중
- **점검 방법**:

  ```bash
  # Trivy 사용
  trivy image <image-name>
  # Docker Scout
  docker scout cves <image-name>
  ```

- **양호 기준**: 알려진 CVE(중·상 등급) 없는 최신 이미지 사용
- **취약 기준**: 장기간 업데이트 되지 않은 기반 이미지 사용

### DK-07. Docker 콘텐츠 신뢰(DCT) 활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  echo $DOCKER_CONTENT_TRUST
  ```

- **양호 기준**: `DOCKER_CONTENT_TRUST=1`
- **취약 기준**: DCT 비활성화 (서명되지 않은 이미지 pull 가능)

### DK-08. 리소스 제한 설정

- **위험도**: 중
- **점검 내용**: CPU/메모리 리소스 제한 설정 (DoS 방지)
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep -E "Memory|CpuQuota|CpuPeriod"
  ```

- **양호 기준**: Memory, CPU 제한 설정
- **취약 기준**: 리소스 무제한 (Memory=0, CpuQuota=0)

### DK-09. 보안 프로파일 적용 (Seccomp/AppArmor)

- **위험도**: 중
- **점검 방법**:

  ```bash
  docker inspect <container-id> | grep -E "SeccompProfile|AppArmorProfile"
  ```

- **양호 기준**: Seccomp 프로파일 적용, AppArmor 프로파일 적용
- **취약 기준**: 보안 프로파일 없음 (`unconfined`)

---

## Kubernetes

### 공통 점검 항목 (Master + Worker)

#### K8S-01. 최신 버전 유지

- **위험도**: 상
- **점검 방법**:

  ```bash
  kubectl version --short
  ```

- **양호 기준**: 지원되는 마이너 버전(N-2 이내) 사용
- **취약 기준**: EOL(End-of-Life) 버전 사용

#### K8S-02. RBAC 활성화 및 최소 권한

- **위험도**: 상
- **점검 방법**:

  ```bash
  kubectl get clusterrolebindings -o wide | grep cluster-admin
  kubectl auth can-i --list --as system:anonymous
  ```

- **양호 기준**: RBAC 활성화, anonymous 사용자 권한 없음, cluster-admin 최소화
- **취약 기준**: 익명 사용자에게 API 접근 허용

#### K8S-03. 네트워크 정책 적용

- **위험도**: 중
- **점검 방법**:

  ```bash
  kubectl get networkpolicies --all-namespaces
  ```

- **양호 기준**: 네임스페이스별 NetworkPolicy 적용 (기본 거부 정책 포함)
- **취약 기준**: NetworkPolicy 없음 (모든 파드 간 통신 허용)

### Master Node 전용 점검 항목

#### K8S-M01. kube-apiserver 보안 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  ps -ef | grep kube-apiserver
  cat /etc/kubernetes/manifests/kube-apiserver.yaml
  ```

- **점검 항목**:
  - `--anonymous-auth=false` (익명 인증 비활성화)
  - `--audit-log-path` 설정 (감사 로그)
  - `--authorization-mode=RBAC` (RBAC 인증)
  - `--tls-min-version=VersionTLS12` (TLS 최소 버전)
  - `--disable-admission-plugins`에 `AlwaysAdmit` 미포함

#### K8S-M02. etcd 보안 설정

- **위험도**: 상
- **점검 내용**: etcd에 저장된 Kubernetes 시크릿 암호화 및 접근 통제
- **점검 방법**:

  ```bash
  cat /etc/kubernetes/manifests/etcd.yaml | grep -E "trusted-ca-file|cert-file|key-file"
  ps -ef | grep etcd | grep -E "listen-client-urls"
  ```

- **양호 기준**: TLS 인증 설정, localhost 또는 내부 IP에만 바인딩
- **취약 기준**: etcd가 외부에 인증 없이 노출

#### K8S-M03. Kubernetes 대시보드 접근 제한

- **위험도**: 상
- **점검 방법**:

  ```bash
  kubectl get svc -n kubernetes-dashboard
  kubectl get serviceaccount -n kubernetes-dashboard
  ```

- **양호 기준**: 대시보드 미설치 또는 RBAC 최소 권한 적용, 외부 미노출
- **취약 기준**: 대시보드가 클러스터 전체 권한으로 외부 노출

### Worker Node 전용 점검 항목

#### K8S-W01. kubelet 인증 설정

- **위험도**: 상
- **점검 방법**:

  ```bash
  cat /var/lib/kubelet/config.yaml | grep -E "authentication|authorization"
  ps -ef | grep kubelet | grep -E "anonymous-auth|authorization-mode"
  ```

- **양호 기준**:
  - `authentication.anonymous.enabled: false`
  - `authorization.mode: Webhook`
- **취약 기준**: 익명 접근 허용, 인증 없이 kubelet API 접근 가능

#### K8S-W02. Pod Security Standards 적용

- **위험도**: 중
- **점검 방법**:

  ```bash
  kubectl get ns -o yaml | grep -E "pod-security"
  ```

- **양호 기준**: `baseline` 또는 `restricted` Pod Security Standards 적용
- **취약 기준**: Pod Security Standards 미적용 (privileged 파드 임의 실행 가능)

---

## OpenStack

### OS-01. Keystone 인증 강화

- **위험도**: 상
- **점검 내용**: Keystone 토큰 만료 시간, 패스워드 정책 설정
- **점검 방법**:

  ```bash
  cat /etc/keystone/keystone.conf | grep -E "expiration|password_hash"
  openstack token issue
  ```

- **양호 기준**: 토큰 만료 시간 적절 설정(1시간 이내), 패스워드 복잡도 정책
- **취약 기준**: 토큰 만료 시간 미설정 또는 과도하게 긴 만료 시간

### OS-02. 보안 그룹 최소화

- **위험도**: 상
- **점검 내용**: 인스턴스 보안 그룹에서 불필요한 포트 개방 여부
- **점검 방법**:

  ```bash
  openstack security group rule list --long
  openstack security group list
  ```

- **양호 기준**: 업무상 필요한 포트만 허용, 0.0.0.0/0 인바운드 최소화
- **취약 기준**: 모든 포트 전체 허용 (0.0.0.0/0, all traffic)

### OS-03. API 엔드포인트 암호화

- **위험도**: 상
- **점검 내용**: 모든 OpenStack 서비스 API 엔드포인트 HTTPS 사용 여부
- **점검 방법**:

  ```bash
  openstack endpoint list --long
  ```

- **양호 기준**: 모든 엔드포인트 HTTPS 사용
- **취약 기준**: HTTP 엔드포인트 존재

### OS-04. 감사 로그 활성화

- **위험도**: 중
- **점검 방법**:

  ```bash
  cat /etc/nova/nova.conf | grep audit
  cat /etc/neutron/neutron.conf | grep notification
  ```

- **양호 기준**: 각 서비스별 감사 로그 활성화
- **취약 기준**: 감사 로그 미활성화

### OS-05. Metadata API 접근 제한

- **위험도**: 중
- **점검 내용**: 인스턴스에서 169.254.169.254 메타데이터 API 접근 통제
- **점검 방법**:

  ```bash
  # 인스턴스 내부에서
  curl http://169.254.169.254/latest/meta-data/
  ```

- **조치 방법**: Link-local 주소에 대한 접근 제한 또는 IMDSv2 강제 (AWS-like 환경)
