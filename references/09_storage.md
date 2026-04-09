# 2장. 분산 스토리지 / 빅데이터 보안 가이드

대상: Ceph, Hadoop

---

## 목차

- [Ceph](#ceph)
- [Hadoop](#hadoop)

---

## Ceph

### CP-01. Ceph Dashboard 접근 제한

- **위험도**: 상
- **점검 내용**: Ceph 관리 대시보드 인증 설정 및 외부 노출 여부
- **점검 방법**:

  ```bash
  ceph mgr services
  ceph dashboard get-jwt-token-ttl
  ```

- **양호 기준**: HTTPS 사용, 인증 활성화, 허가된 네트워크에서만 접근
- **취약 기준**: HTTP 사용하거나 인증 없이 외부 노출
- **조치 방법**:

  ```bash
  ceph config set mgr mgr/dashboard/ssl true
  ceph dashboard create-self-signed-cert
  ```

### CP-02. Ceph 인증 (CephX) 활성화

- **위험도**: 상
- **점검 내용**: Ceph 클러스터 내부 인증(CephX) 활성화 여부
- **점검 방법**:

  ```bash
  ceph auth list
  grep -E "auth_cluster_required|auth_service_required|auth_client_required" /etc/ceph/ceph.conf
  ```

- **양호 기준**: `auth_cluster_required = cephx`, `auth_service_required = cephx`, `auth_client_required = cephx`
- **취약 기준**: CephX 인증 비활성화 (`none` 설정)

### CP-03. 사용자 권한 최소화

- **위험도**: 중
- **점검 내용**: Ceph 사용자별 최소 권한 부여 여부
- **점검 방법**:

  ```bash
  ceph auth list
  ceph auth get client.<username>
  ```

- **양호 기준**: 애플리케이션별 전용 계정, 접근 가능한 pool 제한
- **취약 기준**: 모든 pool에 읽기/쓰기 권한 부여

### CP-04. 네트워크 설정 분리

- **위험도**: 중
- **점검 내용**: 공용 네트워크와 클러스터 네트워크 분리 여부
- **점검 방법**:

  ```bash
  grep -E "public_network|cluster_network" /etc/ceph/ceph.conf
  ```

- **양호 기준**: `public_network`(클라이언트 접근)와 `cluster_network`(내부 레플리케이션) 분리
- **취약 기준**: 단일 네트워크 사용 (클라이언트 트래픽과 레플리케이션 트래픽 혼용)

### CP-05. 데이터 암호화 설정

- **위험도**: 중
- **점검 내용**: 저장 데이터 암호화(at-rest encryption) 설정 여부
- **점검 방법**:

  ```bash
  ceph osd dump | grep encryption
  ```

- **양호 기준**: OSD 레벨 암호화 적용
- **취약 기준**: 저장 데이터 암호화 미적용

### CP-06. 감사 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep "log_to_file\|log_file" /etc/ceph/ceph.conf
  ceph config get mon debug_mon
  ```

- **양호 기준**: 감사 로그 활성화, 로그 보관 정책 수립
- **취약 기준**: 로그 미설정

---

## Hadoop

### HD-01. Kerberos 인증 활성화

- **위험도**: 상
- **점검 내용**: Hadoop 클러스터 Kerberos 인증 설정 여부
- **점검 방법**:

  ```bash
  grep -E "hadoop.security.authentication" /etc/hadoop/conf/core-site.xml
  ```

- **양호 기준**: `hadoop.security.authentication=kerberos`
- **취약 기준**: `hadoop.security.authentication=simple` (인증 없이 접근 가능)
- **조치 방법**: Kerberos KDC 구성 및 각 데몬 keytab 설정

### HD-02. HDFS 권한 설정

- **위험도**: 상
- **점검 내용**: HDFS 파일 시스템 권한 설정 및 슈퍼유저 최소화
- **점검 방법**:

  ```bash
  hdfs dfs -ls -R / | grep -E "^d|rwx"
  grep "dfs.permissions" /etc/hadoop/conf/hdfs-site.xml
  ```

- **양호 기준**: `dfs.permissions=true`, 민감 디렉토리 접근 권한 최소화
- **취약 기준**: `dfs.permissions=false` (권한 검사 비활성화)

### HD-03. YARN 리소스 관리자 접근 제한

- **위험도**: 상
- **점검 내용**: YARN ResourceManager Web UI 접근 통제
- **점검 방법**:

  ```bash
  grep -E "yarn.resourcemanager.webapp.address" /etc/hadoop/conf/yarn-site.xml
  ss -tlnp | grep 8088
  ```

- **양호 기준**: 내부 IP만 바인딩, 인증 설정
- **취약 기준**: 0.0.0.0:8088 바인딩으로 외부 노출

### HD-04. 데이터 전송 암호화

- **위험도**: 중
- **점검 내용**: Hadoop RPC 및 데이터 전송 TLS/암호화 설정
- **점검 방법**:

  ```bash
  grep -E "dfs.encrypt.data.transfer|hadoop.rpc.protection" /etc/hadoop/conf/hdfs-site.xml /etc/hadoop/conf/core-site.xml
  ```

- **양호 기준**: `dfs.encrypt.data.transfer=true`, `hadoop.rpc.protection=privacy` 또는 `integrity`
- **취약 기준**: 평문 데이터 전송

### HD-05. NameNode 웹 UI 인증

- **위험도**: 상
- **점검 내용**: HDFS NameNode Web UI(50070/9870) 접근 통제
- **점검 방법**:

  ```bash
  curl http://localhost:9870/  # 인증 없이 접근 가능하면 취약
  ss -tlnp | grep -E "50070|9870"
  ```

- **양호 기준**: 내부 네트워크에서만 접근 가능 또는 인증 설정
- **취약 기준**: 인터넷에 NameNode UI 노출

### HD-06. 감사 로그 설정

- **위험도**: 중
- **점검 방법**:

  ```bash
  grep -E "audit" /etc/hadoop/conf/hdfs-site.xml
  grep "dfs.namenode.audit.loggers" /etc/hadoop/conf/hdfs-site.xml
  ```

- **양호 기준**: HDFS 감사 로그 활성화, 파일 접근·생성·삭제 등 기록
- **취약 기준**: 감사 로그 비활성화

### HD-07. ZooKeeper 접근 제한

- **위험도**: 상
- **점검 내용**: ZooKeeper 접근 제어 목록(ACL) 설정 여부
- **점검 방법**:

  ```bash
  echo "getAcl /hadoop" | zkCli.sh -server <zk-host>:2181
  grep -E "zookeeper.sasl|zookeeper.auth" /etc/hadoop/conf/core-site.xml
  ```

- **양호 기준**: ZNode ACL 설정, SASL 인증 적용
- **취약 기준**: 모든 사용자에게 full 권한(`world:anyone:cdrwa`)

### HD-08. 취약한 사용자 확인

- **위험도**: 상
- **점검 내용**: Hadoop 슈퍼유저(hdfs, yarn) 권한 최소화
- **점검 방법**:

  ```bash
  id hdfs
  id yarn
  grep -E "dfs.cluster.administrators|yarn.admin.acl" /etc/hadoop/conf/*.xml
  ```

- **양호 기준**: Hadoop 서비스 전용 계정 사용, 관리자 ACL 최소화
- **취약 기준**: root 또는 과도한 권한을 가진 계정으로 Hadoop 서비스 실행
