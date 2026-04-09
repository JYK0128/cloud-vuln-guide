# Cloud Vulnerability Inspection Guide (2024) - AI Agent Skill

![KISA](https://img.shields.io/badge/Guide-KISA_2024-blue)
![License](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey)
![Version](https://img.shields.io/badge/Version-1.0.0-green)

본 저장소는 과학기술정보통신부와 한국인터넷진흥원(KISA)에서 발행한 **"클라우드 취약점 점검 가이드 (2024)"**를 AI 에이전트가 이해하고 실행할 수 있도록 최적화한 범용 스킬 패키지입니다.

## 🚀 개요

클라우드 인프라(하이퍼바이저, 서버, 데이터베이스, 컨테이너 등)의 보안 취약점을 체계적으로 점검하고, 발견된 문제에 대한 구체적인 조치 방안을 안내합니다.

단순한 텍스트 가이드를 넘어, 에이전트가 실제 환경에서 실행할 수 있는 **명령어(CLI)**와 **양호/취약 판정 기준**을 데이터베이스화하여 제공하며, Antigravity를 포함한 다양한 에이전트 환경에서 즉시 활용 가능한 표준 구조를 따릅니다.

## 📂 폴더 구조

```text
cloud-vuln-guide/
├── SKILL.md             # 에이전트 지시서 및 워크플로우 (Core Logic)
├── README.md            # 본 문서
├── NOTICE.md            # 저작권 및 출처 공지
├── references/          # 기술 도메인별 점검 데이터 (Knowledge Base)
│   ├── 02_hypervisor.md
│   ├── 03_server.md
│   ├── 05_database.md
│   └── ... (총 13개 카테고리)
└── resources/           # 원본 자료 (PDF)
```

## 🛠️ 설치 방법

### 방법 1: CLI 명령어로 설치 (권장)

스킬 관리 기능을 지원하는 에이전트 환경에서 아래 명령어로 설치할 수 있습니다.

```bash
npx skills install https://github.com/JYK0128/cloud-vuln-guide
```

### 방법 2: 수동 설치 (Git Clone)

에이전트의 스킬 경로로 직접 이동하여 설치합니다.

```bash
cd path/to/your/agent/skills/
git clone https://github.com/JYK0128/cloud-vuln-guide.git
```

## ️📖 사용 방법

에이전트에게 다음과 같은 자연어로 명령하면 스킬이 활성화됩니다.

* "우리 프로젝트의 Docker 환경 보안 점검해줘."
* "Linux 서버 보안 가이드라인 보여줘."
* "Redis DB 설정이 안전한지 확인하고 싶어."
* "하이퍼바이저 접근 통제 설정 방법 알려줘."

에이전트는 자동으로 `references/`의 해당 파일을 분석하여 **위험도, 점검 방법, 조치 방안**을 단계별로 안내합니다.

## 🛡️ 점검 카테고리

1. **하이퍼바이저**: KVM, Xen, ESXi, Hyper-V
2. **서버**: Linux, Windows Server
3. **PC**: Windows, macOS, Linux
4. **데이터베이스**: MySQL, MSSQL, Redis, ES, MongoDB, PostgreSQL, Oracle 등
5. **웹서버/WAS**: Apache, Nginx, IIS, Tomcat
6. **컨테이너/가상화**: Docker, Kubernetes, OpenStack
7. **미들웨어**: PHP, RabbitMQ, Node.js
8. **스토리지**: Ceph, Hadoop, SAN, NAS, Object Storage
9. **네트워크 및 정보보호**: 라우터, 스위치, 방화벽, IPS, WAF, VPN

## ⚖️ 저작권 및 면책 조항

* 본 스킬의 지식 베이스는 KISA 공공저작물을 활용하였습니다.
* 제공되는 가이드는 참고용이며, 실제 시스템 적용 전 반드시 테스트 환경에서 검증하시기 바랍니다. 모든 작업의 최종 책임은 사용자에게 있습니다.

---
Created by **JYK0128**
