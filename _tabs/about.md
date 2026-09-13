---
title: About Me
icon: fas fa-user
order: 5
toc: false
---

# 👋 It's Eunji

## 🧑‍💻 Profile

**Software Engineer · Cloud / Infra Engineer**

안녕하세요, 클라우드 및 인프라 영역에서 **시스템의 안정성과 생산성을 개선하는 일**에 관심이 많은 소프트웨어 엔지니어 **정은지**입니다.

운영 관점에서의 문제를 기술적으로 해결하는 과정을 좋아하며, **공유와 협업을 기반으로 성장하는 오픈소스 생태계**에 지속적으로 기여하고자 합니다.


**Interests**
- Cloud / Infrastructure / Platform Engineering
- Kubernetes & Distributed Systems


## 🛠 Skills

### Languages
- C / C++
- Java
- Python
- JavaScript / TypeScript
- Go

### Frameworks & Platforms
- **Frontend**: React, React Native  
- **Backend**: Spring Boot (Java)  
- **Cloud / Infra**: AWS, GCP, Kubernetes  

### Tools
- Git, GitHub
- CI/CD Tooling (Git Actions, Jenkins)
- Linux / Container Tooling


## 💼 Careers

### **NHN Cloud**
**Site Reliability Engineer**  
2026.04 ~ Present

클라우드 서비스 3종을 **23개 Kubernetes 클러스터(운영 11개)** 위에서 운영하며,
git → Jenkins → Argo CD → Harbor 파이프라인으로 운영 환경 격주 정기 배포와
개발 환경 상시 배포를 담당한다.

**⚙️ 배포 형상 표준화 · 마이그레이션 자동화**
담당자마다 제각각이던 배포 형상을 단일 네이밍 컨벤션으로 통일했다. helm/Jenkinsfile 을
자동 변환하는 스크립트를 만들고 변환 전후를 `helm diff` 로 비교해 리소스 동일성을
기계적으로 검증, **210개 애플리케이션** 전환에 들던 약 105시간을 약 4.5시간으로 줄이고
스크립트와 가이드를 팀에 공유했다.

**🔧 CI 서버 무중단 메이저 업그레이드**
45개 서비스가 의존하는 Jenkins 를 **LTS 9개 구간을 건너뛰어** 올렸다. 되돌릴 수 없는
데이터 마이그레이션 지점을 미리 식별해 롤백 전략을 '백업 복원' 으로 한정하고, 플러그인
의존성은 설치 실패 로그에서 요구 버전을 파싱하는 루프로 수렴시켰다. **야간 3시간 작업창
내 무중단 완료, 플러그인 로드 실패 0건.**

**🛡️ DNS 장애 오진 정정 · 재발 방지**
팀의 초기 진단(검색 도메인 문제)에 대해 SERVFAIL/NXDOMAIN 동작과 비정상적으로 짧은
응답 지연을 근거로 이견을 제기, 실제 원인(fallback 오설정)을 규명했다. 임시 라우팅으로
서비스 영향을 먼저 끊고, **SERVFAIL 알림 규칙**을 추가해 동일 유형을 조기 탐지하도록 했다.

**🤖 Network ACL 자동화 · 가시성 전사 적용**
트래픽을 감지해 Network ACL 을 자동 구성하는 파이프라인과 가시성 도구를
**10개 클러스터에 Argo CD app-of-apps 형상으로 전사 적용**했다.

**Site Reliability Engineer (Internship)**  
2025.01 ~ 2026.03

클라우드 서비스 인프라 운영과 CI/CD 실무에 참여하며 정규 SRE 로 전환.


### **Dongguk University [CSDC Labs](https://sites.google.com/dgu.ac.kr/csdc/)**
**학부 연구생**  
2024.05 ~ Present

- 지능IoT학과 홈페이지 풀스택 개발 및 유지보수
- 연구자를 위한 GPU 리소스 할당 최적화 관련 연구

## 📽️ Projects

정리 중...

## 🌱 Open Source

### Member
- **Kubernetes** – SIG release enhancements shadow (v1.33)
- **argo-cd** – contributing member

### Contributor
- [argoproj/argo-cd](https://github.com/argoproj/argo-cd/issues?q=involves%3Abianbbc87)
- [argoproj-labs/argo-agent](https://github.com/argoproj-labs/argocd-agent/issues?q=involves%3Abianbbc87)


## 🎤 Talks & Speaking

### 2025
- **Goorm Univ Onboarding Seminar**  
  *협업의 시작과 역할 설정* – [Instargram](https://www.instagram.com/p/DKUW_AESCjX/?img_index=3)
- **OSSCA Argoproj 성과발표회**  
  *OSSCA Argoproj 성과발표회* – [Youtube](https://youtu.be/u0srWy-hOS0)
- **AWS Cloud Clubs Seminar (at Dongguk)**
  *Amazon Q CLI를 이용한 가성비 인프라 설계하기* – [Article](https://www.linkedin.com/feed/update/urn:li:activity:7357079551035781120/)

### 2023
- GDSC Seminar (at Dongguk)
  *생각하는 NPC (Generative agent 논문 요약)* – [Youtube](https://www.youtube.com/watch?v=Yx5o27Jt_DM)

## 👥 Community

### Organizer
- GDG Campus Korea Organizer (2025 ~ )
- ArgoCD KR Organizer (2024 ~ )
- AWS Cloud Clubs at Dongguk Captain (2025 ~ )
- UMC (University Make Us) 8기 교육국장 (2025.02 ~ 2025.08)
- 오픈소스 기여 모임 10기 Mentor (2025 ~ )

### Member
- GDSC(Google Developer Student Club) at Dongguk Web/App Member (2023.08 ~ 2024.08)
- OSSCA (Opensource Contribution Academy) ArgoCD Mentee (2024.08 ~ 2024.12)
- OSSCA (Opensource Contribution Academy) Argoproj Lead Mentee (2025.08 ~ 2025.12)
- UMC (University Make Us) 5기 Web Member (2023.06 ~ 2023.01)
- UMC (University Make Us) 6기 Springboot Member (2024.02 ~ 2024.08)
- UMC (University Make Us) 7기 PM Member / 교내 부회장 (2024.06 ~ 2025.01)
- Goorm Univ 1기 Frontend(Web) Member (2023.08 ~ 2023.12)
- Goorm Univ 3기 Backend Member (2025 ~ )


## 🏆 Honors & Awards

### 2025
- Excellence Award – Capstone Design Final Presentation
    - Issued by Dongguk University · Dec 2025
- Excellence Award – SMART Tournament
    - Issued by Dongguk University · Dec 2025
- Encouragement Award – ArgoProj Track
    - Issued by Open Source Software Contribution Academy · Dec 2025
- Grand Prize – IoT In-Jeju Challenge
    - Issued by Ministry of Science and ICT (MSIT), Republic of Korea · Aug 2025
- Encouragement Award – SMART Tournament
    - Issued by Dongguk University COSS · Jun 2025

### 2024
- Excellence Award – 3rd Goormthon
    - Issued by Goormthon Univ · Dec 2024
- Director’s Award (NIPA) – Argo CD Project
    - Issued by Open Source Software Contribution Academy · Nov 2024
    - (Awarded by the National IT Industry Promotion Agency, Korea)
- Grand Prize – “Camputhon” Hackathon
    - Issued by Dongguk University · Oct 2024
- Track Surprise Award – Apple Developer Academy
    - Junction Asia · Aug 2024
- Director’s Award (KLID) – National Disaster & Safety Public Data Startup Competition
    - Issued by Ministry of the Interior and Safety (Korea) · Jul 2024
    - (Awarded by Korea Local Information Development Institute)
- Grand Prize – UMC Seoul Hackathon
    - Issued by University MakeUs Challenge (UMC) · Jan 2024

### 2023
- Grand Prize – 1st Goormthon Hackathon
    - Issued by Goormthon Univ · Oct 2023
- Encouragement Award – Adventure Design Contest
    - Issued by Dongguk University · Oct 2023


## 🎓 Education

**Dongguk University**  
B.Eng. in Computer Science & Engineering (2021–2026)  
*Transferred from Sculpture (College of Fine Arts) in 2023*

## 🔗 Links

- **Email**: bianbbc87@gmail.com  
- **LinkedIn**: https://www.linkedin.com/in/eunji-jung-173288296/  
- **GitHub**: https://github.com/bianbbc87

---