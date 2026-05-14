# Designing the Architecture of Your OutSystems Applications

> OutSystems 공식 문서(success.outsystems.com) 한국어 해설

---

## 목차

- [1. The Architecture Canvas](#1-the-architecture-canvas)
- [2. Translating business concepts into application modules](#2-translating-business-concepts-into-application-modules)
- [3. Validating your application architecture](#3-validating-your-application-architecture)
- [4. Service-oriented architectures for OutSystems applications](#4-service-oriented-architectures-for-outsystems-applications)
- [5. Integration Patterns for Core Services Abstraction](#5-integration-patterns-for-core-services-abstraction)
- [6. Application composition](#6-application-composition)
  - [6-1. Applying the Architecture Canvas to applications](#6-1-applying-the-architecture-canvas-to-applications)
  - [6-2. The 4 Rules for Correct Application Composition](#6-2-the-4-rules-for-correct-application-composition)
  - [6-3. Isolating an application Theme](#6-3-isolating-an-application-theme)
- [7. Designing Scalable Multi-Tenant Applications](#7-designing-scalable-multi-tenant-applications)
- [8. Microservices Architecture in OutSystems](#8-microservices-architecture-in-outsystems)

---

## 1. The Architecture Canvas

Architecture Canvas는 OutSystems에서 SOA(Service-Oriented Architecture)를 단순하게 만들기 위한 도구입니다. 재사용 가능한 (마이크로)서비스를 추상화하고, 기능적으로 구분되는 독립적인 모듈들의 올바른 격리를 유지하는 데 도움을 줍니다.

여러 애플리케이션을 개발하고 유지하는 경우, 재사용 공통 모듈을 갖는 것이 유리합니다. 일반적인 중대형 OutSystems 설치 환경은 20개 이상의 mission-critical 앱과 200개 이상의 상호의존 모듈을 지원합니다.

애플리케이션들은 서로 다른 라이프사이클을 가지며 서로 다른 팀이 유지합니다. 새로운 앱은 빠르게 진화하고, 이미 많이 사용되는 서비스는 훨씬 천천히 변경됩니다. 잘 설계된 아키텍처는 애플리케이션과 이를 구성하는 모듈들이 독립적인 라이프사이클을 가지고 독립적으로 배포되도록 하며, 의존성을 최소화하고 전체적인 변경 영향을 줄입니다. 결과적으로 유지보수하기 쉽고 비용 효율이 높은 OutSystems 아키텍처가 됩니다.

### The layers and sub layers

각 레이어와 서브 레이어는 모듈에 담을 기능의 서로 다른 성격을 설정합니다.

![The Architecture Canvas (layers)](https://success.outsystems.com/TK_Resource/105b1fae-c75a-408f-a9b8-c40fb6916fc4)

> ⚠️ **Orchestration 레이어는 O11에서 제거되었습니다.**
> Orchestration 레이어는 서로 다른 두 애플리케이션의 화면 간 hyperlink에 사용되었습니다. 이러한 링크는 각 애플리케이션의 라이프사이클 독립성을 저해하는 strong dependency로 간주됩니다. OutSystems 11에서는 화면 대상이 weak reference로 간주되므로 Orchestration 레이어가 더 이상 필요하지 않으며, 아키텍처 캔버스에서 제거되었습니다.

서브 레이어는 아래에 상세히 표시됩니다.

![The Architecture Canvas (sublayers)](https://success.outsystems.com/TK_Resource/a0ac451c-e9a4-463f-a500-0f926ff7b431)

| 레이어     | 서브 레이어        | 설명                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| End-user   | Orchestration (*)  | 통합 사용자 경험 또는 cross-app 워크플로를 제공하기 위해 여러 frontend를 mashup하는 모듈 (* O11 이전 버전만 해당) |
| End-user   | End-user           | Frontend 사용자 인터페이스 모듈                              |
| Core       | Core               | 비즈니스 개념 중심 재사용 서비스 — 엔티티, 비즈니스 규칙, web-block 노출 |
| Core       | API                | Core Services를 노출하기 위한 API 제공                       |
| Core       | Core widgets       | Core Widgets                                                 |
| Core       | Composite logic    | 재사용 비즈니스 로직: 데이터를 동기화하기 위한 Composition 또는 Logic |
| Core       | Core service       | 재사용 Core 서비스                                           |
| Foundation | Foundation         | 비기능 요구사항 또는 통합 모듈 — 모든 비즈니스 컨텍스트에서 재사용 가능 |
| Foundation | Style Guide        | 재사용 UI 패턴, 테마 및 테마 템플릿                          |
| Foundation | Foundation Service | 외부 서비스를 wrap-up하고 NFR을 지원하는 Integration 서비스 (예: Audit logging) |
| Foundation | Library            | 재사용 라이브러리 및 plug-in                                 |

### Architecture design with the Architecture Canvas

Architecture Canvas는 아키텍처 설계의 두 단계에서 사용됩니다.

**1. 개념 식별 (Identifying concepts — functional, non-functional and integration needs)**

Canvas는 구조적이고 체계적인 방식으로 아키텍처 요구사항을 수집하는 데 도움을 줍니다.

![Architecture Canvas - Identifying concepts](https://success.outsystems.com/TK_Resource/cd465952-0a3e-4d00-894a-d7d35e4ad7fa)

**2. 모듈 정의 (Define modules)**

식별된 개념들을 권장 패턴에 따라 모듈을 구현합니다.

![Architecture Iteration](https://success.outsystems.com/TK_Resource/4c071788-f0a8-43bf-8bc7-163d4849ef77)

아키텍처를 설계하는 것은 일회성 이벤트가 아닙니다. 솔루션이 발전하고 새로운 개념이 등장함에 따라 이 두 단계를 반복하며 아키텍처를 지속적으로 개선해야 합니다.

### Using the Architecture Canvas

Architecture Canvas 사용을 시작하려면 다음 문서를 확인하세요.

- Translating business concepts into application modules
- Validating your application architecture

### More information

OutSystems Forge에 있는 **Electronic Canvas** 도구를 확인하세요. 디지털 Architecture Canvas에서 개념을 배치하고 이동할 수 있도록 하여 아키텍처 설계를 지원하며, 새로운 프로젝트에서 구현해야 하는 모든 아키텍처 요소를 식별하고 구성할 수 있습니다.

애플리케이션 아키텍처 설계 방법에 대해 자세히 알아보려면 **Designing the architecture of your OutSystems applications** 가이드를 확인하세요.

---

## 2. Translating business concepts into application modules

개념을 식별한 후에는 그것을 애플리케이션 모듈로 변환해야 합니다. 이를 올바르게 수행하려면 다음 두 가지 사이에서 올바른 균형을 찾는 것이 핵심입니다.

1. 너무 많은 모듈을 다뤄야 하는 상황을 만들지 않을 것
2. 각 모듈마다 관리 가능한 복잡도와 독립적인 라이프사이클을 가질 것

### Finding the right concept granularity

Architecture Canvas 방식으로 식별한 개념 예시입니다.

![Architecture Canvas method - concept example](https://success.outsystems.com/TK_Resource/7f0b37d7-942b-4651-aae4-cee23f0b43a2)

이 개념들을 구현하기 위해 정의해야 하는 모듈의 수는 개념의 granularity에 따라 달라집니다. 아키텍처를 정의할 때 개념을 아주 낮은 수준에서 캡처할 필요는 없습니다. 그러나 올바른 granularity는 다음 기준에 따라 결정됩니다.

**Complexity (복잡도)**

개념이 너무 방대해서 단일 모듈에 구현하기 어려울 수 있습니다. 예를 들어 사용자 프로세스 **Customer management**는 여러 sub-process를 포함할 수 있습니다: 신규 고객 provisioning, 고객 트렌드 분석, 고객에게 서비스 추가 등.

각 모듈의 복잡도를 관리 가능한 수준으로 유지하기 위해 여러 모듈로 분리합니다.

**Independent life cycles (독립적인 라이프사이클)**

모든 sub-concept이 함께 복잡한 모듈을 만들지 않더라도, 기능을 독립적으로 관리해야 한다면 더 작은 개념으로 모듈을 분리하는 것도 필요합니다.

이를 통해 서로 다른 개발자들 간의 병렬 개발이 가능하거나 서로 다른 비즈니스 스폰서를 위한 서로 다른 발전 속도를 유지할 수 있습니다.

**Service isolation (서비스 격리)**

서비스를 올바르게 추상화하기 위한 여러 integration patterns가 있으며, 이에 따라 하나의 개념을 시나리오에 맞게 여러 기술 컴포넌트로 분리해야 하는 경우가 있습니다.

### Validating your architecture

개념을 구성하는 방식이 Architecture Canvas 권고사항에 맞는지 확인하려면 **validate your application architecture** 문서를 통해 검증 방법을 확인하세요.

### More information

애플리케이션 아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

## 3. Validating your application architecture

잘 설계된 애플리케이션 아키텍처를 달성하기 위해 **반드시** 준수해야 할 3가지 규칙이 있습니다.

구현된 모듈이 이 아키텍처 규칙을 준수하는지는 **Discovery tool**을 사용하여 자동으로 검증할 수 있습니다. 이 도구는 모듈 간 실제 의존성을 분석하고, 위반 사항을 식별하며, 잘못 배치된 요소(액션, 화면, 엔티티)를 정확히 찾아냅니다.

![Architecture validation rules overview](https://success.outsystems.com/TK_Resource/ef4e0190-7927-4ff6-8885-23780534d6aa)

### Rule #1 — No upward references (상향 참조 금지)

상향 참조는 두 모듈이 직접 또는 간접적으로 순환 의존성을 형성하는 경향이 있습니다.

이 예시에서 **Library E**는 **End-user A**를 소비합니다. 식별된 클러스터 내 어떤 쌍이든 순환 관계에 있습니다. C와 D를 예로 들면: `C → E → A → D`, 반대로 `D → E → A → C`.

![Rule #1 - Upward references example](https://success.outsystems.com/TK_Resource/a68f884c-6f71-4b30-a5be-17d4be00a474)

또 다른 예상치 못한 효과는 **End-user B**가 정당하게 **Core D**를 소비하지만 전체 클러스터에 종속된다는 것입니다. 런타임에서 불필요하게 큰 footprint를 갖게 될 뿐만 아니라, 자신이 인식조차 못하는 모듈의 변경에 의해 지속적으로 영향을 받게 됩니다.

**무엇이 문제인가?**
상향 참조 위반은 서비스가 제대로 격리되지 않았음을 명확히 보여줍니다.

**해결 방법:**
소비되고 있는 요소들을 찾아 더 낮은 레이어로 이동시킵니다. 이 경우 **A**에서 재사용 가능한 서비스를 **Library**로 이동시킵니다.

### Rule #2 — No side references among End-users or Orchestrations (횡단 참조 금지)

이 예시에서 **End-user A**가 **End-user B**의 어떤 요소(단순한 포맷팅 함수 같은 것)를 소비합니다. A가 B 모듈에 직접 연결된 것뿐만 아니라 **D, E, G**까지 간접적으로 상속하게 됩니다.

![Rule #2 - Side references example](https://success.outsystems.com/TK_Resource/b01a0a00-e535-49fe-81e1-058d2e190f0e)

**무엇이 문제인가?**
**End-user** 또는 **Orchestration** 모듈은 재사용 가능한 서비스를 제공해서는 안 됩니다. 이는 각 모듈이 올바르게 격리되어 서로 다른 라이프사이클을 가질 수 있게 하기 위함입니다. End-user/Orchestration은 계층 최상단에 위치하기 때문에, 이 모듈을 참조하면 하위 레이어의 간접 의존성을 대량으로 끌어들이게 됩니다.

**해결 방법:**
B에서 A가 소비하는 요소들을 찾아 더 낮은 레이어의 새로운(또는 개념적으로 맞는 기존) 모듈로 이동시킵니다.

- 비즈니스 관련이라면 → **Core 모듈**로
- 비즈니스와 무관하다면 → **Library**로

### Rule #3 — No cycles among Cores or Libraries (순환 참조 금지)

Rule #1, #2를 지키면 End-user와 Orchestration은 어떤 순환 참조에도 연루될 수 없습니다. 세 번째 규칙은 **Core 또는 Library 간의 순환**을 피하는 것에 관한 것입니다.

![Rule #3 - Cycles example](https://success.outsystems.com/TK_Resource/51ab940f-51f5-4013-b357-80cdbf1e51ff)

순환은 언제나 바람직하지 않습니다. 예상치 못한 영향을 주고 유지보수하기 어려운 코드를 만들기 때문입니다.

**무엇이 문제인가?**
모듈 간 순환은 개념이 올바르게 추상화되지 않았음을 나타냅니다. A와 B 사이의 순환은 다음 중 하나를 의미합니다.

- 두 모듈이 강하게 결합(strongly coupled)되어 있거나
- 한 방향이 바람직하지 않은 것

**해결 방법:**
대부분의 경우, 관계에서 바람직하지 않은 방향의 요소를 다른 모듈로 이동시킵니다. 예를 들어 B가 A를 소비한다면, B에서 A를 소비하는 요소들은:

- 또 다른 재사용 가능한 개념을 나타낸다면 → **새 모듈**로 이동
- 동일한 개념의 요소라면 → **B 자체**로 이동

만약 A와 B가 너무 강하게 결합되어 있다면 **합치는 것(merge)** 을 고려합니다. merge 결과 모듈이 너무 커진다면, 두 기본 개념 위에 세 번째 모듈을 만드는 것이 필요할 수 있습니다. 원래 두 모듈은 기본 개념을 담고, 새 모듈은 양쪽 기본 개념을 처리해야 하는 일부 End-user 프로세스를 지원하는 비즈니스 규칙을 담습니다. 이 조합 로직이 순환을 유발하는 것입니다.

### More information

아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

## 4. Service-oriented architectures for OutSystems applications

OutSystems Services로 개념을 추상화할 때, 서비스의 Core가 다음 조건을 만족해야 합니다.

- 내부 및 외부 소비자 모두에게 **공통적**일 것
- 에코시스템 변화에 대응하기 위해 **모든 외부 시스템으로부터 격리**될 것
- **deprecated 예정인 레거시 시스템을 대체**할 준비가 되어 있을 것

이것이 아키텍처를 유연하게 유지하고 변화에 대응할 수 있게 하는 핵심입니다.

### Implementing your Service-Oriented Architecture

아래 예시에서 서비스를 구성하는 3개 레벨을 확인할 수 있습니다.

- The **External API**
- The **Core Services**
- The **Integration Services**

**Core Services**가 외부 시스템으로부터 제대로 격리되는 것이 매우 중요합니다.

![Service-Oriented Architecture diagram](https://success.outsystems.com/TK_Resource/deae802d-4dda-44c8-837b-35dd72e85927)

### External API level

이 레벨은 REST, SOAP 또는 기타 방식을 통해 서비스를 외부 소비자에게 노출하는 API들의 집합입니다.

Core Services 레이어에서 실제로 구현된 서비스의 **기술적 wrapper**에 불과합니다. 이 레벨은 소비자와 합의된 API 형태로 서비스를 변환하는 역할만 합니다.

> ⚠️ **이 레벨에는 비즈니스 로직을 추가해서는 안 됩니다.**

서로 다른 소비자 스펙에 대응하기 위해 이 레벨에 여러 버전의 API를 유지할 수 있습니다. 예를 들어 한 소비자는 Record collection으로 데이터를 보내고, 다른 소비자는 XML로 보낼 수 있습니다. 두 버전 모두 실제 서비스를 호출하기 전에 OutSystems Record List로 변환해야 합니다.

또한 이 레벨에서 Integration audit을 추가해 서비스의 외부 소비를 추적할 수 있습니다.

### Core Services level

이 레벨은 모든 비즈니스 규칙과 핵심 엔티티를 포함하는 **서비스의 실제 구현** 레벨입니다.

**Core Services는 시스템에 무관(system-agnostic)해야 합니다.**

- **External APIs**: 서비스를 소비하는 외부 시스템에 대한 지식은 External API 레벨에만 한정되어야 합니다. 외부 소비자의 추가·변경·제거가 있어도 Core Services에는 영향이 없어야 합니다.
- **Integration Services**: Core Service가 외부 레코드 시스템(외부 producer)을 extend한다면, 그 변경도 Integration Services 레벨에서 추상화되어야 합니다. 외부 producer가 변경되거나 교체되더라도 Core Services는 영향을 받지 않습니다.

레거시 시스템과 통합할 때는 **Integration Services를 통해** 데이터를 OutSystems entities(로컬 캐시)로 동기화하는 것이 일반적입니다. 이 방식은 다음을 가능하게 합니다.

- 성능 문제 대응
- Core Services를 앱 외부 부하로부터 보호
- 레거시 시스템에 온라인 호출 없이 비즈니스 로직 지원
- 향후 레거시 시스템을 Core Service 소비자에게 영향 없이 deprecated 처리

비즈니스 감사(audit)는 외부 또는 내부 호출 여부와 관계없이 모든 비즈니스 트랜잭션을 추적할 수 있습니다.

### Integration Services

이 레벨은 외부 producer를 기술적으로 wrapping하여 데이터 구조를 정규화하고, 외부 시스템의 통합 복잡성(오류 복구, 외부 시스템 인증 등)을 추상화합니다.

> ⚠️ **이 레벨에도 비즈니스 로직을 추가해서는 안 됩니다.**

Integration Service는 특정 개념이 여러 시스템에 분산되어 있다는 사실도 추상화할 수 있습니다. 예를 들어 Customer 정보가 CRM과 ERP 두 시스템에 모두 있는 경우, Library 모듈 **Customer_IS**가 두 시스템을 모두 통합하여 정규화된 Customer 데이터 구조로 mash-up된 모든 API를 제공합니다. 이 Integration Service를 소비하는 Core Service는 정보가 어디서 오는지 알 필요가 없습니다. 덕분에 Core Services에 영향 없이 외부 시스템을 자유롭게 변경할 수 있습니다.

### Service design patterns

아래 문서에서 구현 방법에 대한 추가 정보를 확인하세요.

- **Integration Patterns for Core Services Abstraction**

### More information

아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

## 5. Integration Patterns for Core Services Abstraction

### Why do you need integration patterns? (왜 통합 패턴이 필요한가?)

세 가지 독립적인 요구사항이 있습니다.

- **업무 독립성(Business Independence)**: 외부 시스템의 변경이 OutSystems 앱에 영향을 주지 않아야 합니다.
- **확장성, 성능, 가용성(Scalability, performance, and availability)**: 외부 시스템의 성능·가용성 문제가 앱에 영향을 주지 않아야 합니다.
- **감사, 모니터링, 통합 신뢰성(Auditing, monitoring, and integration reliability)**: 신뢰할 수 있는 통합을 위한 감사 및 모니터링이 가능해야 합니다.

![Integration pattern overview](https://success.outsystems.com/TK_Resource/05051ba3-cb1f-49b2-aef8-a37ba2590eb0)

단순히 Integration Services만으로는 충분하지 않습니다. Integration Services는 외부 시스템의 기술적 복잡성을 추상화하지만, 외부 시스템의 성능·가용성 문제는 여전히 Core Services에 영향을 줄 수 있습니다.

Integration Services와 Core Service 사이에 아무런 격리 없이 통합하면 다음과 같은 문제가 생깁니다.

- **Scoring ML**: 성능 문제 — 외부 시스템 응답 속도에 종속
- **Backoffice ML**: 외부 시스템 장애 시 앱 전체에 영향
- **Flasc**: 실시간 호출로 인한 과부하
- **Field App**: 외부 시스템 변경 시 Core Service까지 수정 필요

### Integration Services — wrapping calls to an external system

![Integration Services wrapping concerns](https://success.outsystems.com/TK_Resource/59716818-9fcc-4993-932c-7f9907e12809)

Integration Services는 외부 시스템 호출을 wrapping할 때 다음 주요 사항을 담당합니다.

| 역할                     | 설명                                      |
| ------------------------ | ----------------------------------------- |
| **Abstract Data Source** | 내부 데이터 소스 (OutSystems 로컬 엔티티) |
| **Synchronize data**     | 외부 DB 동기화                            |
| **Act as cache**         | Full cache with delta sync                |
| **Error handling**       | Real-time sync, Ordered real-time sync    |

각 항목의 책임:

- **Abstract the data source**: 데이터 소스를 추상화하여 Core Service가 데이터 출처를 알 필요 없게 함
- **Synchronize data to OutSystems**: 외부 데이터를 OutSystems 엔티티로 동기화
- **Act as cache to Core Service API**: Core Service가 로컬 캐시를 통해 빠르게 데이터 접근
- **Error handling**: 외부 시스템 오류 처리 및 재시도 로직

### Integration patterns according to data needs (데이터 필요에 따른 통합 패턴)

패턴 선택은 두 가지 기준으로 결정됩니다.

- **Reading Public 여부**: 데이터를 퍼블릭하게 읽어야 하는지 → Yes이면 캐시 필요
- **Small Files 여부**: 데이터 크기가 작은지 → Yes이면 Direct Integration 가능

![Integration pattern decision tree](https://success.outsystems.com/TK_Resource/90d69c79-2871-40cf-85b4-75f37d454042)

> 📌 **[다이어그램 삽입: 통합 패턴 전체 플로우차트 (Reading Public → Small Files → 각 패턴 분기)]**

---

### Direct Integration (직접 통합)

외부 시스템을 실시간으로 직접 호출하는 가장 단순한 패턴입니다. Integration Service가 외부 시스템을 직접 호출하고, Core Service는 Integration Service를 통해 데이터를 가져옵니다.

![Direct Integration pattern](https://success.outsystems.com/TK_Resource/459b4387-37d7-487b-a58b-6d0abcc1d242)

**Input API:**

- Simple integration API
- Less impact on source system
- Not affected by sync

**장점:**

- 항상 최신 데이터
- 구현이 단순

**단점:**

- 외부 시스템 장애 시 서비스 중단
- 성능이 외부 시스템에 종속
- Slow sync may be OK

**사용 시점:** 데이터를 퍼블릭하게 읽을 필요가 없고, 데이터 크기가 작을 때

---

### Cold Cache (Summary Data)

외부 시스템 데이터를 OutSystems 엔티티에 주기적으로 동기화해두는 패턴입니다. Batch 방식으로 정기적으로 외부 데이터를 로컬에 캐시합니다.

![Cold Cache pattern](https://success.outsystems.com/TK_Resource/0efbac8d-c7b6-4c62-9934-9f9b715c2e65)

**언제 Cold Cache를 사용하는가?**

아직 통합되지 않은 레거시 시스템이 있을 때 적합합니다.

- **Cold Cache to HW Sync**: 하드웨어 동기화를 위한 Cold Cache 사용
- **Load from the server to sync**: 서버에서 데이터를 로드하여 동기화

**장점:**

- 외부 시스템 장애에 독립적
- 성능 향상 (로컬 캐시 조회)

**단점:**

- 데이터가 실시간이 아님 (배치 주기만큼 지연)

**사용 시점:** 데이터를 퍼블릭하게 읽어야 하고, 실시간성이 중요하지 않을 때

---

### Gateway synchronization logic (게이트웨이 동기화 로직)

외부 시스템에서 변경이 발생할 때 OutSystems로 데이터를 push하거나, OutSystems가 주기적으로 변경분(delta)을 pull하는 패턴입니다. 동기화 로직을 Core Service에서 분리하여 별도 모듈로 격리합니다.

![Gateway synchronization logic pattern](https://success.outsystems.com/TK_Resource/fce06b8e-b137-470b-9816-810270bbbf27)

**장점:**

- 외부 시스템 장애에 독립적
- Cold Cache보다 데이터 신선도 향상
- Core Service와 동기화 로직의 명확한 분리

**주요 모듈 구성:**

- **Customer_CS**: Core Service — 비즈니스 로직 담당
- **Customer_Sync**: 동기화 로직 전담 모듈 (Gateway에서 분리)
- **Customer_IS**: Integration Service — 외부 시스템 wrapping

**Input API / Output API:** 동기화 전후 데이터 흐름 명확히 정의

---

### Distinct real-time sync (개별 실시간 동기화)

IT 요청이 있을 때마다 즉시 동기화하는 패턴입니다. 전체 데이터가 아닌 필요한 특정 레코드만 실시간으로 외부 시스템과 동기화합니다.

**언제 사용하는가?**

IT needs a return message — 즉각적인 응답이 필요한 경우에 사용합니다.

![Distinct real-time sync pattern](https://success.outsystems.com/TK_Resource/b694bf68-ebc6-4ec0-a8a0-9692865d5171)

**특징:**

- 특정 레코드에 대한 요청 시 실시간으로 외부 시스템과 동기화
- 전체 동기화가 아닌 필요한 레코드만 처리
- Sync in a queue request (N queues) — 큐 기반 요청 처리

**장점:**

- 항상 최신 특정 레코드 데이터
- 필요한 데이터만 동기화하여 효율적

**단점:**

- 외부 시스템 응답 시간에 의존

---

### Ordered real-time sync (순서 보장 실시간 동기화)

메시지 순서를 보장하면서 실시간으로 동기화하는 패턴입니다. 메시지 큐 기반으로 처리 순서를 보장합니다.

**언제 사용하는가?**

다음 두 조건 모두 해당할 때 사용합니다.

- If you can't tolerate changes in the order of all messages arriving in the systems (메시지 순서 변경을 허용할 수 없을 때)
- Create message with async request (비동기 요청으로 메시지 생성)

![Ordered real-time sync pattern](https://success.outsystems.com/TK_Resource/3084ac2f-e225-4f8f-940c-5e16bbe877e6)

**특징:**

- 메시지 처리 순서 완전 보장
- 비동기 처리로 외부 시스템 장애에 독립적
- 큐(Queue)를 통한 메시지 순서 관리

**장점:**

- 데이터 처리 순서 보장
- 시스템 부하 분산 가능

**단점:**

- 구현 복잡도 높음
- 즉각적인 응답이 필요한 경우 부적합

**사용 시점:** 데이터 처리 순서가 중요하고, 대용량 메시지를 처리해야 할 때

---

### Final Summary (최종 정리)

> 📌 **[다이어그램 삽입: 전체 패턴 통합 구조도 — Customer_CS / Customer_IS / CRM / ERP 포함 모든 패턴 비교]**

> 📌 **[다이어그램 삽입: 패턴별 비교표 — Internal data source / Database sync / Full cache with delta sync / Real-time / Ordered real-time 행별 특성 정리]**

---

## 6. Application composition

---

### 6-1. Applying the Architecture Canvas to applications

모듈 아키텍처와 마찬가지로 애플리케이션도 Architecture Canvas 원칙을 따릅니다. 각 애플리케이션은 레이어 중 하나에 배치되며, 애플리케이션의 최상위 모듈 유형이 해당 애플리케이션의 성격을 결정합니다.

![Application types by layer](https://success.outsystems.com/TK_Resource/e4b0eb51-d09b-415e-adc0-40392eb2bace)

애플리케이션 유형:

- **Orchestration Applications** — Orchestration 레이어. O10 이하 버전에서만 고려됩니다.
- **End-User Applications** — End-User 레이어
- **Core Applications** — Core 레이어
- **Library Applications** — Foundation 레이어

애플리케이션은 동일한 검증 규칙을 따르며, 애플리케이션 간 관계도 준수합니다.

1. No upward references
2. No side references among End-User applications or Orchestration applications
3. No cyclic references between two applications

동일한 이유가 모듈과 애플리케이션 모두에 적용됩니다. 예를 들어 **End-User** 애플리케이션은 재사용 가능한 서비스를 제공해서는 안 됩니다. 이를 통해 라이프사이클 독립성을 보장합니다. End-User 애플리케이션은 **Core** 및 **Library** 모듈을 포함할 수 있지만, 다른 모듈에 의해서만 소비되어야 합니다.

### Evolving your architecture along with your applications

#### First project

일반적으로 처음에는 여러 애플리케이션 정의에 대해 생각하지 않고 프로젝트당 하나의 애플리케이션으로 시작합니다.

첫 번째 프로젝트에서는 아키텍처 설계 단계에서 구상한 모든 모듈을 포함하는 하나의 애플리케이션이 생성되며, QA 환경에 배포될 준비가 됩니다.

![First project structure](https://success.outsystems.com/TK_Resource/e943848f-cb5d-4805-aff2-60d99e314db5)

#### Second project

두 번째 유사한 프로젝트에서 다른 비즈니스 프로세스를 위한 또 다른 애플리케이션이 추가되고, Core 및 Library 모듈이 몇 개 더 추가됩니다.

![Second project structure](https://success.outsystems.com/TK_Resource/c2d8f561-8cb3-40cc-8129-b92defefbe0f)

#### Third project

세 번째 발전 프로젝트에서 **End-user #1**이 Core C를 사용하기 시작하고, **End-user #2**가 Core B를 사용하기 시작합니다. 규칙 위반은 없지만 두 End-user 애플리케이션이 서로 side reference를 가지게 됩니다 — 실제로는 순환(cycle)입니다.

![Third project - cyclic dependencies](https://success.outsystems.com/TK_Resource/bdcb97b6-f8c7-4c89-b4c9-17852a5a0bab)

이는 두 애플리케이션이 강하게 결합되어 있음을 명확히 보여줍니다. 첫 번째 애플리케이션의 새 버전을 Production에 배포하려면 두 번째 애플리케이션도 함께 가져가야 하며, 그 반대도 마찬가지입니다.

#### Solving the issues

공통으로 재사용되는 리소스는 새로운 **Core Application**으로 분리해야 합니다.

![Solving the issues - Core Application isolation](https://success.outsystems.com/TK_Resource/28e67ac7-0e57-4a30-85f9-920c82da9b53)

Core B와 Core C뿐만 아니라, End-user 애플리케이션에서 직접 참조되는 의존성(Library B, Library C)도 이 Core Application으로 이동해야 합니다. Upward reference를 피하기 위해서입니다.

이렇게 하면 두 애플리케이션이 서로 다른 속도로 독립적으로 발전할 수 있습니다. 또한 Core Application은 서로 다른 비즈니스를 위한 여러 End-user 애플리케이션에서 재사용될 수 있습니다.

아키텍처 검증도 Discovery tool로 수행할 수 있습니다.

### Correctly composing applications

애플리케이션을 올바르게 구성할 때 중요한 포인트를 다루는 4가지 규칙이 있습니다: **The 4 Rules for Correct Application Composition**.

또한 고려해야 할 중요한 측면은 애플리케이션 테마를 격리하여 애플리케이션 간 통일된 look & feel을 공유하는 것입니다: **Isolating an application Theme**.

### More information

애플리케이션 아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

### 6-2. The 4 Rules for Correct Application Composition

애플리케이션을 더 작은 것으로 분리할 필요성은, 재사용 요건이 알려진 후에 항상 해결해야 하는 것입니다. 그러나 처음부터 애플리케이션을 작게 쪼개는 것이 항상 명확하지는 않습니다.

이 4가지 규칙은 다음을 위해 애플리케이션 구성을 안내합니다.

- 의존성 최소화
- 배포 단순화
- 라이프사이클 독립성 촉진

첫 두 규칙은 Architecture Canvas 방법론을 올바르게 사용한 결과입니다. 나머지 두 규칙은 다른 전략적 고려사항을 반영합니다.

#### Rule #1 — Correctly layer your modules

Architecture Canvas 원칙에 따릅니다. 모듈 아키텍처에 위반이 있으면 올바르고 관리 가능한 애플리케이션 아키텍처를 갖추는 것이 불가능합니다.

#### Rule #2 — Correctly layer your applications

모듈 대신 애플리케이션에 적용할 때 Architecture Canvas 원칙을 따릅니다.

#### Rule #3 — Don't mix different owners

서로 다른 팀이 동시에 여러 프로젝트를 개발할 때, 양쪽 프로젝트에 영향을 미치는 공통 서비스에 대한 요건이 생기는 경우가 많습니다. 병목을 피하기 위해 두 팀 모두 공통 리소스를 변경할 수 있어야 합니다.

하나의 애플리케이션에 소유자가 두 명 이상이 되면 복잡한 배포 관리가 발생합니다. 무엇이 변경되었는지에 대한 책임이 불명확해집니다.

![Rule #3 - Different owners split](https://success.outsystems.com/TK_Resource/b11c4ee1-cade-4316-9edf-d61df48853ed)

소유권을 집중하는 것이 핵심입니다. 하나의 애플리케이션 소유권을 집중할 수 없다면, 소유권이 명확하게 정의되도록 분리하는 것을 고려해야 합니다.

#### Rule #4 — Don't mix different sponsors

프로젝트가 여러 스폰서에 영향을 미친다면 각 LOB(Line Of Business)를 별도의 애플리케이션으로 분리하는 것이 중요합니다. 서로 다른 스폰서는 서로 다른 예산, 요건, 변경 속도를 가집니다.

다음 예시는 Auto, Life, Property 등 서로 다른 LOB의 모든 종류의 보험 시뮬레이터에 대한 웹 액세스를 부여하는 공통 Portal을 보여줍니다.

![Rule #4 - Different sponsors split](https://success.outsystems.com/TK_Resource/bf08d7c8-2848-4159-98da-28beaaf6935b)

LOB별로 애플리케이션을 미리 분리하면 각각 독립적으로 발전할 수 있습니다. 모든 공통 서비스의 격리는 예상되는 횡단적 영향으로 인해 신중하게 계획해야 하는 것과 각 LOB 내에서 유연하게 변경 가능한 것 사이의 경계를 명확하게 설정합니다.

### More information

애플리케이션 아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

### 6-3. Isolating an application Theme

엔터프라이즈 설치 환경에서 애플리케이션들은 보통 동일한 corporate look & feel을 공유합니다. 공유 컴포넌트이므로 엔터프라이즈 아키텍처에서 글로벌 Theme 모듈을 올바르게 배치하는 것이 핵심입니다.

Theme 모듈은 다음 컴포넌트를 포함합니다.

- Theme 자체 (CSS)
- Layout Blocks — 화면 또는 화면 섹션 패턴 정의
- Logo 이미지
- Login flow 및 exception handling
- Menu 및 navigation 지원 로직
- Transversal roles — Manager 또는 Employee처럼 모든 앱에서 재사용 가능한 역할 (메뉴 항목 표시에 영향을 미칠 수 있음)

> ⚠️ **Theme 모듈은 global look & feel 요소로만 제한해야 하며, 절대로 assets 및 utilities의 글로벌 저장소로 사용해서는 안 됩니다.**

#### Same look and feel, menu and login flow

다음 예시는 공통 메뉴와 login flow를 공급하여 여러 애플리케이션에서 재사용되는 Global Theme을 보여줍니다. 이 시나리오는 모든 애플리케이션이 공통 Portal에 속하는 경우에 적합합니다.

![Same look and feel - shared global theme](https://success.outsystems.com/TK_Resource/e2be76be-d281-4aaf-aa30-16050ec01338)

이는 OutSystems UI가 제공하는 built-in 테마 중 하나를 기반으로 합니다. Common Services가 UI 컴포넌트(Blocks)를 제공한다면, 다른 look & feel을 사용하는 애플리케이션에서도 재사용할 수 있도록 built-in base theme을 기반으로 해야 합니다. Block은 소비 모듈의 CSS를 상속하므로 최종 애플리케이션의 look & feel에 맞게 자동으로 적응합니다. 애플리케이션 내부의 서비스는 해당 애플리케이션에 특화되어 있으므로 Global Theme을 안전하게 사용할 수 있습니다.

#### Same look and feel, different menus and login flows

애플리케이션들이 서로 독립적이고 global look & feel만 공유하면 된다면, 각 애플리케이션이 자체 theme 모듈을 가져야 합니다.

![Same look and feel - individual app themes](https://success.outsystems.com/TK_Resource/f9d1683d-6981-4035-a539-ad22e308ad5c)

이 경우 각 애플리케이션 theme은 공통 global theme을 상속하되, 자체 menu 및/또는 login flow를 정의합니다. look & feel의 전체적인 변경은 Global Theme 모듈에서 수행됩니다. 특정 menu, login 메커니즘(통합, 연합, 다른 user provider 등)은 각 애플리케이션 theme에서 전문화됩니다.

### More information

애플리케이션 아키텍처 설계 방법에 대한 전체 가이드는 **Designing the architecture of your OutSystems applications**에서 확인하세요.

---

## 7. Designing Scalable Multi-Tenant Applications

OutSystems를 사용하면 단일 소프트웨어 인스턴스로 여러 조직(테넌트)을 서비스할 수 있습니다. 이는 여러 공통 애플리케이션을 사용하는 단일 소프트웨어 인스턴스를 유지하면서 각 운영의 데이터를 분리하여 유지하는 것과는 매우 다릅니다.

### Logical Segregation of Tenants

논리적 테넌트 격리를 구현할 때 두 가지 문제를 고려해야 합니다.

- **Data Segregation**: 테넌트에 속하는 데이터를 단일 엔티티에 격리하는 것이 필요합니다.
- **Cross-Tenant Access**: 파트 사용자가 여러 테넌트에 속한 데이터에 접근해야 하는 경우입니다.

OutSystems는 논리적 격리의 멀티테넌트 방식을 따릅니다. 화면 및 비즈니스 로직 앱은 공유되지만 데이터와 최종 사용자는 격리됩니다. 사용자는 동일한 URL을 사용하여 접근하며, OutSystems는 인증 시 자동으로 테넌트를 추론합니다.

![Logical Segregation - tenant isolation](https://success.outsystems.com/TK_Resource/15a16951-86a9-4336-b690-3d73c86c3f48)

데이터베이스 레벨에서 멀티테넌트 테이블을 정의할 때 테넌트 ID가 있는 열이 해당 테이블에 추가됩니다. 이 열은 OutSystems에 의해 관리되며, 데이터베이스 접근 시 올바른 테넌트의 데이터만 자동으로 필터링됩니다.

**장점:**

- OutSystems에서 구독의 일환으로 완전히 지원됨
- 인프라 운영 비용 절감
- 데이터, 사용자, 세션, 프로세스에 대한 단일 보안 정책
- 테넌트별 특정 쿼리를 작성할 필요가 없어 애플리케이션 개발 속도 향상

**단점:**

- 모든 테넌트의 데이터가 같은 테이블에 있어 데이터 격리 수준이 낮음
- 테넌트당 행 수가 높아질 수 있음 → archiving 전략 적용 필요
- 행 수가 많을수록 테이블당 인덱스 수가 증가

### Ensuring Multi-Tenancy Scalability

확장성을 위해 다음을 권장합니다.

- **Fully multi-tenant** 방식으로 애플리케이션을 개발 — 인프라 변경에 대한 유연성 확보
- **Use page mechanisms** — 테이블에 저장된 정보량 최소화
- **Scale the database vertically**

### When the Database Reaches the Maximum Vertical Capability

#### Scenario 1: Multiple Production Environments

테넌트별로 별도의 프로덕션 인프라를 사용합니다. 개발, Acceptance, PreProduction 환경은 공유합니다.

![Multiple Production Environments](https://success.outsystems.com/TK_Resource/c55ab1bf-f7c5-4f82-8b11-5d49752a0716)

이전 프로덕션 데이터를 새 환경으로 마이그레이션하는 3가지 방법:

- **IT 도구 활용** — Attunity, Informatica, Information Builders 등
- **System Integration 개발** — 외부 DB에서 데이터를 가져와 새 환경에 import하는 OutSystems 앱과 웹 서비스 사용
- **Backup restore** — 이전 프로덕션 DB 전체 백업 후 새 DB에 restore. 그 후 마이그레이션되지 않은 테넌트 데이터 삭제

Common Master Data가 있는 경우, 여러 프로덕션 인프라가 있더라도 애플리케이션이 공통 카탈로그에 연결될 수 있도록 해야 합니다.

#### Scenario 2: Manually Partitioning OutSystems Entities

OutSystems는 데이터를 파티셔닝하기 위한 자체 메커니즘이 없지만, 메타모델에 수동으로 업데이트하면 됩니다.

### Security Concerns

OutSystems에서 개발자가 다른 테넌트 데이터를 볼 수 있는 유일한 방법은 런타임 시 TenantId를 명시적으로 사용하는 경우입니다.

이를 방지하기 위해 자동화된 검증 프로세스를 구축해야 합니다.

- `TenantSwitch`가 사용된 모든 모듈을 식별하는 보고서
- 모든 멀티테넌트 테이블이 단일 테넌트로 사용되는 보고서
- 프로덕션 배포 전 항상 보고서 실행

### Physical Segregation of Tenants

각 테넌트가 자체 데이터베이스를 갖는 물리적 격리 방식입니다. 테넌트마다 다른 데이터에 접근하려면 Database Access Layer를 구현해야 합니다.

![Physical Segregation architecture](https://success.outsystems.com/TK_Resource/c6d9c5e2-1ef3-429e-8f9a-88a4fd8ca67f)

**장점:**

- 데이터 격리 수준 높음
- 테이블당 행 수가 적어 쿼리 속도 최적화

**단점:**

- 테넌트마다 추가 리소스 필요
- 데이터 모델이 OutSystems IDE 외부에서 관리되어야 하므로 개발 속도 저하
- 테넌트 데이터베이스 동기화가 필요해 유지보수 복잡

### Recommendation

**논리적 격리(Logical Segregation)를 기본 옵션으로 권장**합니다.

추가 장점:

- 짧은 개발 주기 — 테이블 생성 및 업데이트가 IDE에서 바로 처리
- 단일 코드 베이스로 애플리케이션 단순화
- OutSystems가 애플리케이션 개발, 배포, 지원 전반에서 더 잘 지원

### Documentation

- OutSystems Multi-tenant
- How to Build a Multi-tenant Application in OutSystems
- Multi-tenant base database factory pattern

---

## 8. Microservices Architecture in OutSystems

마이크로서비스 접근 방식에서 아키텍처는 독립적으로 배포 가능한 작은 서비스들로 구성됩니다. 각 서비스는 자체 용량으로 실행되며, 경량 메커니즘으로 통신합니다.

마이크로서비스 패러다임에서는:

- 독립적인 배포 가능한 서비스를 중심으로 팀을 구성
- 각 서비스는 독립적으로 확장 가능
- 서로 다른 언어/기술을 사용해 개별 서비스 구현 가능
- 최소한의 중앙 집중식 관리
- 서비스별로 업그레이드 가능

![Microservices vs Monolithic architecture](https://success.outsystems.com/TK_Resource/ee7449e9-711b-4665-8abc-d611668f5bcd)

마이크로서비스 접근 방식을 사용하면 아키텍처는 세 가지 인프라로 구성됩니다.

- **External Services** — 외부 소비자 및 시스템
- **Microservices Infrastructure** — 마이크로서비스 운영 환경
- **Application Infrastructure** — 애플리케이션 운영 환경

![Microservices extended services](https://success.outsystems.com/TK_Resource/26643e7c-b752-4971-ab46-124ea3da7f36)

### Multiple Microservices Infrastructures

![Multiple Microservices Infrastructures](https://success.outsystems.com/TK_Resource/3ca38069-f2d1-4995-9603-508306bded2c)

각 OutSystems 인프라는 독립적으로 배포 가능합니다. 마이크로서비스당 하나의 OutSystems 환경을 사용할 필요 없이 여러 마이크로서비스를 동일한 인프라에서 호스팅할 수 있습니다. 결과로 발생하는 독립적인 배포 의존성에 주의하세요.

### Central Services for Multiple Substrates

![Central Services for Multiple Substrates](https://success.outsystems.com/TK_Resource/fb6588ad-5e6d-453d-b241-0ae164617918)

### Microservices Lifecycle — Managing Versions

마이크로서비스를 제공할 때 항상 염두에 두어야 하는 것은 서비스를 버전 관리하는 방법입니다.

- **Add new action**: 새 모듈에 새 액션을 추가하고 모든 소비자에게 배포
- **Change current API version**: 기존 모듈을 변경하고 모든 소비자에게 배포
- **Create new API version**: 새 서비스 메서드를 추가하되 모든 소비자가 평가할 수 있도록 유지

![Microservices Lifecycle - Managing Versions](https://success.outsystems.com/TK_Resource/83a1b6f6-936e-451b-911f-d5b9d6e06029)

### Infrastructure Scenarios

![Infrastructure Scenarios](https://success.outsystems.com/TK_Resource/15b43ae4-e0ad-4aec-af5c-4540a903f036)
