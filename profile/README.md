## :truck: Sparta Logistics (스파르타 물류)
MSA 기반 B2B 국내 물류 관리 및 AI 배송 최적화 시스템

Gu-Pang은 전국 17개 허브를 거점으로 기업 간(B2B) 물품 보관 및 배송을 관리하는 통합 물류 솔루션입니다. Hub-to-Hub Relay 모델과 AI 기반 배송 시한 예측을 통해 물류 효율을 극대화합니다.

## :busts_in_silhouette: 팀원 역할 분담

| [김도현](https://github.com/kimdh32022) | [김관형](https://github.com/kwanhyoungkim) | [박상은](https://github.com/ddangme) | [김시온](https://github.com/sionkim0126) | [김강현](https://github.com/mkcellokh) |
|:------------------------------------:|:---------------------------------------:|:-------------------------------------:|:----------------------------------:|:-----------------------------------:|
|  공통(Common), <br/> 주문(Order)      |회사(Company), <br/>   상품(Product)        |공통(Common),<br/>           허브(Hub),infra             |       배달(Delivery), AI        |             유저(User)              |

## :hammer_and_pick: ERD
<img width="1268" height="1807" alt="Untitled (1)" src="https://github.com/user-attachments/assets/3749112f-a19d-42cd-b18e-56ee30a49e05" />

## :building_construction: System Architecture (Microservices Architecture)
본 프로젝트는 서비스 간의 <b>느슨한 결합(Loose Coupling)</b>과 <b>높은 응집도(High Cohesion)</b>를 지향하는 MSA(Microservices Architecture) 방식으로 설계되었습니다.

#### Key Architectural Features for MSA
##### Centralized Global Governance (Shared Library):

각 마이크로서비스가 독립적으로 운영되면서도 시스템 전체의 일관성을 유지하기 위해 common 라이브러리를 구축했습니다.

이를 통해 전역 예외 처리, 공통 응답 규격, JPA Audit 등을 Auto-Configuration 방식으로 각 서비스에 자동 주입하여 코드 중복을 제거하고 운영 효율을 극대화합니다.

##### Dynamic Service Discovery & Routing:

Netflix Eureka를 도입하여 분산된 마이크로서비스 인스턴스의 위치를 동적으로 관리합니다.

Spring Cloud Gateway가 모든 서비스의 단일 진입점 역할을 하며, Keycloak 기반의 중앙 집중형 JWT 인증 및 권한 필터링을 수행하여 보안 경계를 강화합니다.

##### Logic Isolation:

서비스 간 데이터 참조는 직접적인 DB 접근 대신 FeignClient를 통한 인터페이스 기반 통신(REST API)으로 수행하여 도메인 간 독립성을 보장합니다.

## :hammer_and_wrench: Tech Stack & Development Environment
### Backend & Core
Framework: Spring Boot 3.x

Build Tool: Gradle

Service Discovery: Spring Cloud Eureka

API Gateway: Spring Cloud Gateway

Communication: Spring Cloud OpenFeign (Declarative REST Client)

### Data & Storage
Main Database: PostgreSQL

~~Caching: Redis (허브 간 최단 경로 및 고정 정보 캐싱)~~

Entity Mapping: p_ 접두사 테이블 규칙 준수 및 UUID 식별자 사용

### Security & Documentation
Authentication: Keycloak 기반 JWT 인증 및 권한 관리

~~API Documentation: Springdoc-openapi (Swagger UI) - 프론트엔드 협업용 자동화 문서 제공~~

Version Control: Git (GitHub 전용 레포지토리 운영)

## :package: 서비스 구성
User Service: 사용자 관리
Order Service: 주문 처리, 상품
Delivery Service: 배송 관리
Company Service: 상품 관리
Hub Service: 허브 관리
```
:open_file_folder: Project Structure
Plaintext
.
├── common/           # [Library] 공통 설정, BaseEntity, GlobalException (Shared .jar)
├── gateway-server/            # [Entry] API 라우팅 및 JWT 인증 필터
├── eureka-server/             # [Discovery] 서비스 등록 및 상태 관리
├── user-service/              # [Domain] 사용자 관리 및 승인 프로세스 (PENDING/APPROVE)
├── hub-service/               # [Domain] 17개 허브 정보 및 경로(Route) 관리 (Redis Caching)
├── company-service/           # [Domain] 업체 및 상품 마스터 관리
├── order-service/             # [Domain] 주문 생성 및 재고 관리
└── delivery-service/          # [Domain] 릴레이 배송 로직, 담당자 배정, AI 시한 예측
```
## :rocket: Key Features
#### 허브 및 스마트 경로 관리 (Hub & Route Management)
Hub-to-Hub Relay 모델: 전국 17개 거점 허브를 연결하며, 장거리 배송 시 '게이트웨이 허브'를 거치는 릴레이 경로를 생성합니다.

최적 경로 알고리즘: 다익스트라(Dijkstra) 알고리즘을 구현하여 실시간 최단 시간/거리 경로를 도출합니다.

Performance Caching: 빈번하게 조회되는 허브 정보와 이동 경로를 Redis에 캐싱하여 마이크로서비스 간 불필요한 DB 조회를 최소화합니다.

#### 주문 및 재고 동기화 (Order & Stock Management)
Transactional Order Flow: 주문 생성 시 상품의 재고를 즉시 확인 및 차감하며, 주문 취소 시 재고를 복원하여 데이터 일관성을 보장합니다.

Inventory Validation: 허브 내 재고 부족 시 주문이 실패하도록 제약 조건을 관리하며, 업체별 소속 허브를 기준으로 물량을 조절합니다.

#### 지능형 배송 및 담당자 배정 (Delivery & Agent Assignment)
Round-Robin 배정: 각 허브에 소속된 배송 담당자를 배송 순번(Sequence)에 따라 자동 배정합니다.

구분된 배송 책임: '허브 간 배송(Hub-to-Hub)'과 '최종 업체 배송(Hub-to-Company)' 담당자를 엄격히 분리하여 배송 효율을 관리합니다.

배송 경로 추적: 주문 시점부터 최종 완료까지 구간별 상태(Route Record)를 실시간으로 기록하고 모니터링합니다.

#### AI 기반 예측 및 알림 (AI Analytics & Notification)
AI 발송 시한 예측: Spring AI를 활용하여 배송 경로, 주문 수량, 담당자 근무 시간(09~18시)을 분석하고, 납기 준수를 위한 최종 발송 시한을 도출합니다.

Slack 연동: 주문 발생 즉시 발송 허브 담당자에게 주문 상세 정보와 AI가 계산한 발송 시한을 Slack 메시지로 자동 전송합니다.

#### 사용자 및 권한 승인 (User & Approval Process)
승인 기반 회원가입: 이용자가 가입 신청 시 승인대기 상태로 저장되며, 마스터/허브 관리자의 승인(APPROVE) 후 서비스를 이용할 수 있습니다.





:gear: Installation & Setup (Docker)
