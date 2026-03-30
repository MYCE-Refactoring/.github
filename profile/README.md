# 🎟  MYCE : 박람회 예매/운영 서비스 
<img width="1447" height="805" alt="image" src="https://github.com/user-attachments/assets/2a02e673-0b5e-4a11-91f7-367e47f49161" />

**배포url** : https://www.myce.live

**테스트 계정**  
- 일반 사용자 및 박람회 관리자 계정 : **leoleo / qwer1234**  
- 플랫폼관리자 계정 : **myceadmin01 / qwe123**  

<br/>

## 🧑‍💻 프로젝트 소개

MICE는 Meeting, IncentiveTravel, Convention, Exhibition/Event의 앞글자를 딴 용어입니다.  
MYCE는 위 MICE의 개념을 기반으로, 박람회를 쉽고 스마트하게 운영할 수 있는 **박람회 생애주기 관리 플랫폼**입니다.  
온라인 박람회 **개최부터 예약, 결제, 정산까지 원스톱으로 제공하는 종합 서비스**를 제공하며 다음과 같은 복합적인 의미를 담아내고자 했습니다.
```
- Meet Your Clients & Exhibitions : 고객과 박람회를 한곳에서 함께 관리하세요.
- Manage Your Conferences & Expos : 당신의 박람회를 체계적으로 관리하세요.
- Make Your Conventions Easy: 박람회 운영을 쉽고 스마트하게 만들어보세요.
```

## Myce Reafactoring - MSA + Multi-Module 구조 설계
<img width="1080" height="608" alt="Image" src="https://github.com/user-attachments/assets/88cab5d0-c6c3-44d6-8e41-7de98ae8884c" />

## IaC(Infrastructure as Code)
물리적 환경을 수동으로 사용하는 대신 코드를 사용하여 인프라를 프로비저닝하고 관리하는 프로세스

> 프로비저닝?
> 어떤 서비스를 제공하기까지 준비한 과정

### Terraform
하시코프에서 만든 IaC Tool로, 사람이 읽을 수 있는 구성 파일을 작성하여 클라우드 인프라 구성 요소를 프로비저닝, 업데이트 및 삭제 할 수 있도록 합니다.  
myce-infra에서는 AWS 기반 프로비저닝을 실행합니다.
  
**사전 작업(bootstrap)**
- keypair 생성 및 파일 다운로드
- tfstate 기록 및 Config 저장용 S3 생성

**인프라 프로비저닝(main)** 
- 네트워크 구성
  - VPC 및 Public/Private Subnet 정의
  - Internet Gateway 구축
  - Route Table 생성, Subnet 연결 및 IGW 라우팅 규칙 정의
- 보안 구성
  - 서비스별 Security Group 정책 정의
  - SSH / HTTP / DB 인바운드 규칙 설정
- 컴퓨팅 리소스 구성
  - 서비스별 EC2 인스턴스 구축
  - 보안 구성 연결
  - 서비스 : public / private / bastion / nat
- 데이터베이스 구성
  - MySQL RDS 프로비저닝
  - 사용자 정보 및 데이터베이스 설정
- 생성 EC2 IP 파일 업로드
  - 사전작업 시 생성한 S3 사용

### Ansible
기존의 동일 작업 처리 시 쉘 스크립트를 실행시켜 직접 배포 작업을 수행했으나, 사용자 트래픽이 늘어나고 관리해야할 서버가 많아지면서 동시에 자동화할 필요가 생겼습니다. 이런 문제 해결을 위해 Ansible 배포 도구 구축을 진행했습니다.

**인프라 프로비저닝(env)**
- Nat Instance 구성
  - IP Forwarding 설정
  - nftables 기반 NAT 규칙 설정
- Nat Instance에 NginX 설치 및 설정파일 추가
- Docker 및 Docker Compose 설치

**배포 작업(deploy)**
- 백엔드 프로세스 배포
  - .env 설정 및 Docker Image 다운로드
  - Docker Compose 실행
->  Backend Actions 수행 후 트리거 발생 시 수행

<br/>

## ✨ 주요 기능
<img width="1408" height="642" alt="image" src="https://github.com/user-attachments/assets/4dfd731a-e2e7-43b5-926c-2ad04b68f3d4" />

### 🖋️ 분류별 상세 기능

| 구분 | 기능 |
|------|------|
| &nbsp; &nbsp;&nbsp; 사용자 관리 &nbsp;&nbsp;&nbsp;| &nbsp; <ul><li><b>[회원가입/로그인](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/auth)</b>: 일반 회원가입, 소셜 로그인 (OAuth2)</li><li><b>회원등급 시스템</b>: Bronze, Silver, Gold, Platinum, Diamond 등급별 혜택 &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</li><li><b>[마이페이지](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/member)</b>: 개인정보 관리, 예약 내역, 결제 내역</li><li><b>다국어 지원</b>: 한국어, 영어 등 다국어 인터페이스</li></ul> &nbsp; |
| &nbsp; &nbsp;&nbsp;[박람회 관리](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/expo) &nbsp;| &nbsp; <ul><li><b>박람회 등록</b>: 상세 정보, 이미지, 위치, 일정 설정</li><li><b>카테고리 관리</b>: 다양한 박람회 카테고리 분류 </li><li><b>부스 관리</b>: 박람회 내 개별 부스 정보 관리 </li><li><b>이벤트 관리</b>: 박람회 내 특별 이벤트 스케줄링 </li><li><b>승인 시스템</b>: 플랫폼 관리자의 박람회 승인 프로세스</li></ul> &nbsp;|
| &nbsp; &nbsp;&nbsp;예약 & 티켓&nbsp; | &nbsp; <ul></li><li><b>티켓 시스템</b>: 다양한 티켓 타입 및 가격 설정 </li><li><b>온라인 예약</b>: 실시간 예약 및 재고 관리</li><li><b>[QR 코드](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/qrcode)</b>: 예약 확인 및 입장용 QR 코드 생성 </li><li><b>비회원 예약</b>: 게스트 사용자 예약 지원</li></ul>&nbsp; |
| &nbsp; &nbsp;&nbsp;[결제 & 정산](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/payment) | &nbsp; <ul></li><li><b>통합 결제</b>: 카드결제, 계좌이체, 가상계좌 등 </li><li><b>마일리지 시스템</b>: 등급별 마일리지 적립 및 사용 </li><li><b>환불 처리</b>: 자동화된 환불 프로세스 </li><li><b>정산 관리</b>: 박람회 주최자 정산 시스템</li></ul> &nbsp;|
| &nbsp;&nbsp;&nbsp;&nbsp; [광고 관리](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/advertisement) &nbsp;|&nbsp; <ul></li><li><b>배너 광고</b>: 메인페이지 광고 위치별 관리 </li><li><b>광고 신청</b>: 광고주 신청 및 승인 시스템</li><li><b>요금 설정</b>: 위치별 광고 요금 관리</li></ul> &nbsp;|
| &nbsp; &nbsp;&nbsp;소통 & 지원&nbsp; | &nbsp; <ul></li><li><b>[실시간 채팅](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/chat)</b>: WebSocket 기반 고객 지원 채팅 </li><li><b>[AI 챗봇](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/ai)</b>: Spring AI + AWS Bedrock 연동</li><li><b>[알림 시스템](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/notification)</b>: 실시간 푸시 알림</li><li><b>[이메일 발송](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/notification)</b>: 템플릿 기반 이메일 시스템</li></ul> &nbsp;|
| &nbsp; &nbsp;&nbsp;관리자 기능&nbsp; | &nbsp; <ul></li><li><b>[대시보드](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/dashboard)</b>: 통계 및 현황 모니터링 </li><li><b>박람회/광고 생애 관리</b>: 신청 승인 및 결제/정산 처리</li><li><b>사용자 관리</b>: 회원 정보 및 권한 관리</li><li><b>[시스템 설정](https://github.com/LIKE-LION-MYCE/myce-server/tree/develop/src/main/java/com/myce/system)</b>: 요금, 템플릿 등 시스템 설정</li></ul> &nbsp;|


### 🌟 주요 특징
- 실시간 통신: WebSocket(STOMP)과 SSE를 활용한 실시간 채팅 및 알림 전달
- 결제 : 토스페이먼츠 OpenAPI를 활용한 결제 시스템 구축
- 보안: Spring Security + JWT를 통한 인증/인가 처리
- 확장성: Docker 컨테이너화 및 AWS 클라우드 인프라 설정
- 모니터링: Prometheus & Grafana를 통한 실시간 시스템 모니터링
- API 문서화: Swagger를 통한 자동화된 API 문서화 설정

<br/>
