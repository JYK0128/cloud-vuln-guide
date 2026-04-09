---
name: cloud-vuln-guide
version: 1.0.0
author: JYK0128
license: CC BY 4.0 (KISA Checklist Reference)
description: 클라우드 취약점 점검 가이드(2024) 기반의 보안 점검을 수행하는 스킬. 하이퍼바이저, 서버(Linux/Windows), PC, 데이터베이스(MySQL, MSSQL, Redis, Elasticsearch, MongoDB, PostgreSQL, Cubrid, CouchDB, SQLite, Tibero, InfluxDB, Oracle), 웹서버(Apache, Nginx, IIS, Tomcat), 컨테이너(Docker, Kubernetes, OpenStack), 미들웨어(PHP, RabbitMQ, Node.js), 스토리지(Ceph, Hadoop, BOSH), 네트워크 장비, 정보보호시스템 등에 대해 취약점 점검 항목을 안내하거나 점검 결과를 분석할 때 반드시 사용. 사용자가 "취약점 점검", "보안 점검", "클라우드 보안", "하드닝", "설정 검토" 등을 언급할 때 항상 이 스킬을 활용.
---

# 클라우드 취약점 점검 가이드 (2024)

과학기술정보통신부·한국인터넷진흥원(KISA) 발행 **클라우드 취약점 점검 가이드(2024)** 기반 스킬.  
점검 대상 시스템에 맞는 참조 파일을 읽고, 점검 항목별로 구체적인 가이드를 제공한다.

---

## 📋 문서 구조 개요

### 1장. 개요

- **1.1** 개요
- **1.2** 목적 및 활용
- **1.3** 유의사항

자세한 내용 → `references/01_overview.md`

---

### 2장. 보안 가이드 (점검 대상별 분류)

점검 대상에 따라 아래 카테고리 중 해당하는 참조 파일을 읽어라.

| 카테고리 | 대상 시스템 | 참조 파일 |
| --- | --- | --- |
| 🖥️ 하이퍼바이저 | KVM, XenServer, ESXi, Hyper-V | `references/02_hypervisor.md` |
| 🐧 서버 | Linux, Windows | `references/03_server.md` |
| 💻 PC | Windows, MAC, Linux | `references/04_pc.md` |
| 🗄️ 데이터베이스 | MySQL, MS-SQL, Redis, Elasticsearch, MongoDB, PostgreSQL, Cubrid, CouchDB, SQLite, Tibero, InfluxDB, Oracle | `references/05_database.md` |
| 🌐 웹서버 / WAS | Apache, Nginx, IIS, Tomcat | `references/06_webserver.md` |
| 📦 컨테이너 / 가상화 인프라 | Docker, Kubernetes(Master/Worker), OpenStack | `references/07_container.md` |
| ⚙️ 미들웨어 / 런타임 | PHP, RabbitMQ, Node.js | `references/08_middleware.md` |
| 💾 분산 스토리지 / 빅데이터 | Ceph, Hadoop | `references/09_storage.md` |
| ☁️ PaaS 플랫폼 | BOSH(Director), BOSH(UAA) | `references/10_bosh.md` |
| 🔌 네트워크 장비 | Network Device | `references/11_network.md` |
| 🛡️ 정보보호시스템 | 방화벽, IPS, WAF 등 | `references/12_security_system.md` |
| 💿 스토리지 | SAN, NAS 등 | `references/13_storage_device.md` |

---

## 🔄 점검 워크플로우

### Step 1 – 점검 대상 파악

사용자로부터 점검 대상 시스템 종류를 확인하라.  
예: "Docker 컨테이너 환경을 점검하고 싶어요" → 컨테이너 카테고리

### Step 2 – 해당 참조 파일 읽기

위 표에서 해당 참조 파일을 `view_file`로 열어라.  
파일이 크면 목차(TOC)를 먼저 확인하고 필요한 섹션만 읽어라.

### Step 3 – 점검 항목 안내

각 점검 항목에 대해 다음 형식으로 안내하라:

```text
## [점검 항목 번호] 항목명
- **위험도**: 상/중/하
- **점검 내용**: 무엇을 확인하는지
- **점검 방법**: 실행할 명령어 또는 확인 절차
- **양호 기준**: 어떤 상태가 안전한지
- **취약 기준**: 어떤 상태가 위험한지
- **조치 방법**: 취약할 경우 조치 방법
```

### Step 4 – 점검 결과 분석 (선택)

사용자가 점검 결과(로그, 설정 파일, 명령어 출력)를 제공하면 취약 여부를 판단하고 조치 권고를 제공하라.

---

## 📌 주요 원칙

1. **실제 점검 항목 기반**: 가이드 문서의 실제 점검 항목 번호와 내용을 기준으로 답변하라.
2. **버전·환경 고려**: 동일한 소프트웨어라도 버전에 따라 설정 경로와 명령어가 다를 수 있으므로 사용자 환경을 반드시 확인하라.
3. **위험도 우선 정렬**: 위험도 '상(High)' 항목부터 먼저 안내하는 것을 기본으로 한다.
4. **조치 전 검증 권고**: 모든 보안 조치 전에 서비스 영향도를 검토하도록 안내하라.
5. **한국어 기본**: 점검 결과와 가이드는 한국어로 제공하되, 명령어·기술 용어는 원문 유지.

---

## 📚 원본 자료 (Source Documents)

상세한 점검 항목의 원문이나 그림/표 등을 확인해야 할 경우 아래 원본 문서를 참조하라.

- **파일명**: `클라우드 취약점 점검 가이드(2024).pdf`
- **경로**: `resources/클라우드 취약점 점검 가이드(2024).pdf`
