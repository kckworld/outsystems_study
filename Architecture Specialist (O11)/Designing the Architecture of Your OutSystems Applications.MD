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
- [9. From architecture to development](#9-from-architecture-to-development)
  - [9-1. Developing from the architecture blueprint](#9-1-developing-from-the-architecture-blueprint)
- [10. Manage technical debt](#10-manage-technical-debt)
  - [10-1. How does AI Mentor Studio work](#10-1-how-does-ai-mentor-studio-work)

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

### Why do you need integration patterns?

외부 레코드 시스템과 통합하여 마스터 데이터를 조회하거나 트랜잭션을 수행해야 하는 경우, OutSystems로 가져오려는 개념을 추상화하는 패턴을 따르는 것이 좋습니다. 이를 통해 다음을 촉진할 수 있습니다.

- **System independence (시스템 독립성)**: 외부 시스템을 변경하거나 교체할 때 개념 추상화의 구현에만 영향을 미치며, 개념의 소비자에게는 영향을 주지 않습니다.
- **Extensibility, normalization, and optimization (확장성, 정규화, 최적화)**: 추가 정보와 비즈니스 규칙으로 개념을 확장하고, 표현을 정규화하며, 성능 향상을 위해 정보를 캐시하는 기회를 제공합니다.

![Integration pattern overview](https://success.outsystems.com/TK_Resource/05051ba3-cb1f-49b2-aef8-a37ba2590eb0)

**1- Core Service**

- 원래 서비스를 확장
- 시스템 무관(system agnostic) — 외부 변경에 탄력적

**2- Integration Service**

- 원래 API를 추상화하여 내부 구조와 개념에 매핑 (예: 여러 시스템의 보완 정보로 Customer 개념 구성)
- 통합 유형 숨김 (SOAP, REST, XIF, HTTP request, file upload 등)
- 기술적 요구사항 처리 (접근 자격증명, 권한 부여, 오류 처리, 통합 감사 등)

Integration Service는 외부 시스템별 단일 통합 포인트가 아닌 **독립적인 개념별**로 가져야 합니다 (예: Customers용 하나).

- 하나의 개념이 여러 시스템에서 지원될 수 있으며, Integration Service가 그 복잡성을 추상화합니다.
- 하나의 시스템이 여러 개념을 지원할 수 있지만(예: Customers와 Products가 있는 ERP), 각 개념은 자체 라이프사이클을 가집니다. Customers 소비자는 Products에 관심이 없어야 하며, Products 변경에 영향을 받지 않아야 합니다.

### Integration Services — wrapping calls to an external system

Integration Service는 외부 시스템에 대한 모든 호출을 wrap-up하여 통합 복잡성을 숨기고 재사용하기 쉬운 서비스를 추상화합니다. 아래 캔버스는 사용되는 통합 프로토콜(REST, SOAP, DLL Wrapper 등)에 관계없이 외부 시스템 호출을 wrapping할 때 처리해야 할 주요 사항들을 안내합니다.

**Integration Wrapper Canvas:**

![Integration Services wrapping concerns](https://success.outsystems.com/TK_Resource/59716818-9fcc-4993-932c-7f9907e12809)

- **Validation / Security logic**: 입력의 일관성을 확인합니다. 필요한 경우, 세션의 사용자가 이 호출을 수행할 자격이 있는지 확인합니다. 오류를 전파하기 위해 user exception을 발생시킵니다.
- **Authentication logic**: 외부 API 호출에 필요한 인증 자격증명을 준비합니다.
- **Map inputs**: 정규화된 입력을 외부 API에서 요구하는 형식으로 매핑합니다.
- **Call and audit the external API**: 외부 API를 호출하고, integration trail이 필요한 경우 호출을 감사합니다.
- **Normalize error handling**: Success 및 Error 메시지 출력을 처리하여, 해당 출력을 반환하는 대신 exception을 발생시킵니다 — OutSystems에서 기대하는 동작입니다.
- **Normalize outputs**: 반환된 구조를 예상되는 정규화된 출력으로 매핑합니다.
- **Exception handling**: 일반적으로 필요하지 않습니다. 오류는 호출자가 처리해야 합니다. 통합 exception을 감사하거나 사용자 인터페이스에 표시될 수 있는 exception 메시지를 정규화하는 데 사용할 수 있습니다.

### Integration patterns according to data needs

이 섹션은 다양한 요구사항(비즈니스 개념 및 비기능 요구사항)에 따른 여러 패턴을 제시합니다. 핵심은 어떤 종류의 통합이 필요하든 관계없이 Core Service의 소비자(예: Customer_CS)는 항상 동일한 유형의 추상화를 얻으며, 외부 레코드 시스템을 알지 못한다는 것입니다.

외부 레코드 시스템에서 데이터를 가져올 때 통합 요구사항을 이해하기 위해 데이터의 성격에 따라 두 가지 유형의 필드로 분류할 수 있습니다.

- **Summary fields**: 항목을 나열하거나 검색하는 데 사용됩니다. 일반적으로 이러한 필드는 시간이 지나도 변하지 않습니다 (예: **Customer name**) 또는 매우 드물게 변경됩니다 (예: **Customer city**).
- **Detail fields**: 단일 항목의 세부 정보를 표시하는 데만 필요합니다. 이러한 필드는 더 빈번하게 변경되며 복제하기에 민감합니다 (예: **Customer balance**).

패턴을 설명하기 전에, 아래 의사결정 트리를 사용하여 데이터 요구사항에 따라 적합한 패턴을 선택하세요. 선택한 경로에서 수집된 모든 패턴을 적용하세요.

![Integration pattern decision tree](https://success.outsystems.com/TK_Resource/90d69c79-2871-40cf-85b4-75f37d454042)

> 💡 **굵게 표시된 답변이 가장 일반적이고 권장되는 경로입니다.** OutSystems에서 mashup, 검색, scaffolding을 위한 엔티티 제공의 이점을 활용하여 거의 필요하지 않은 나머지 복잡성 없이 구현을 가속화합니다.

---

### Direct integration (직접 통합)

이 패턴은 Integration Service를 사용하여 서비스를 추상화하면서 외부 시스템과 직접 통합하는 것을 나타냅니다.

![Direct Integration pattern](https://success.outsystems.com/TK_Resource/459b4387-37d7-487b-a58b-6d0abcc1d242)

이 시나리오에서 Customer 데이터는 ERP에서 가져옵니다. **Customer_IS**는 ERP와 인터페이스하여 정보 조회와 트랜잭션 수행 모두를 위한 정규화된 API를 추상화합니다. **Integration Service API**가 유지되는 한 외부 시스템을 변경해도 Core Service에 영향을 주지 않는 유연성을 제공합니다.

또한 이 시나리오에서는 **Customer_CS**에 로컬 데이터 복제본을 유지하기 위한 엔티티를 포함하지 않습니다. ERP 외부에 복제 데이터를 유지하지 않는 이유는 비즈니스 제약이거나, 거의 실시간 수요가 지연된 데이터 복제와 양립할 수 없기 때문입니다(정보가 너무 자주 변경됨).

그러나 이 패턴에는 몇 가지 **한계**가 있습니다.

- 데이터를 조회할 때마다 ERP에 추가 부하 발생
- ERP와의 추가 온라인 통신으로 인한 지연 증가
- OutSystems Aggregates와 Advanced Queries를 최대한 활용하거나 다른 정보와 결합할 수 없음
- 다른 데이터 조회가 필요할 때마다 **Customer_IS**를 확장하고 ERP 팀에 새 API를 제공받아야 하는 지속적인 필요성. 이 의존성을 피하기 위해 개발자들이 50개 필드를 반환하는 메서드를 4개만 필요한 경우에도 재사용하는 경향이 있음

| 장점               | 단점                                    |
| ------------------ | --------------------------------------- |
| 데이터가 항상 최신 | 조회 사용 사례별 API 필요 (더 많은 API) |
|                    | 더 높은 지연                            |
|                    | 외부 시스템에 더 많은 부하              |

---

### Cold Cache Summary Data (콜드 캐시 요약 데이터)

외부 시스템과의 통합 성능을 개선하고 싶을 때, 좋은 출발점은 자주 변경되지 않는 데이터를 캐시하는 것입니다. 이 데이터를 "cold data"라고 하며, 이 캐싱 패턴의 이름이 **Cold Cache**입니다.

Cold Cache는 전체 데이터베이스를 동기화하기에 비용이 너무 많이 들고, 세부 정보는 단일 항목에만 필요한 경우(목록 아님)에 사용됩니다. 요약 데이터는 어떤 항목이든 검색할 수 있도록 존재해야 하지만, 실제로 세부 정보를 확인하는 항목이 10%에 불과할 때 전체 항목을 동기화하는 것은 불필요합니다.

이 패턴이 구현될 때:

- 자주 나열, 조인, 검색되는 요약 데이터만 캐시됩니다 (단일 항목의 전체 세부 정보는 외부 시스템에서 직접 가져올 수 있음)
- 전체적이고 빈번한 동기화를 피합니다 (요약 정보는 자주 변경되지 않음)

#### Cold Cache with Batch Sync

Cold Cache를 구현할 때, 가장 간단한 설계 방법은 캐싱 데이터 모델을 만들고 캐시된 데이터를 관리하기 위한 배치 동기화 프로세스를 설정하는 것입니다.

**Customer_CS**에 로컬 엔티티를 추가하면 Direct Integration 패턴의 한계를 극복하고 실제 완전한 Core Service를 생성합니다.

![Cold Cache with Batch Sync pattern](https://success.outsystems.com/TK_Resource/0efbac8d-c7b6-4c62-9934-9f9b715c2e65)

Integration Service가 단순하고 안정적으로 됩니다. 다양한 데이터 조회 요구를 위한 수많은 액션을 제공하는 대신, 마지막 동기화 이후 업데이트된 모든 고객 관련 데이터를 가져오는 메서드 하나만 제공하면 됩니다.

**Customer_CS**는 타이머를 사용하여 Integration Service를 통해 정기적으로 정보를 동기화합니다. 이 동기화는 ERP(데이터의 마스터)에서 Core Service로의 단방향이어야 하며, 복잡한 정보 병합을 피해야 합니다. 반대 방향으로 **Customer_CS**에서 업데이트가 이루어질 때는 먼저 ERP에 성공적으로 동기적으로 커밋되어야 합니다(write-through 정책).

| 장점                                       | 단점                         |
| ------------------------------------------ | ---------------------------- |
| 더 단순한 API                              | 데이터가 최신이 아닐 수 있음 |
| OutSystems에서 데이터 mashup 가능          |                              |
| 소스 시스템에 대한 영향 최소화             |                              |
| Core Service 고객이 동기화에 영향받지 않음 |                              |

---

### Isolate synchronization logic (동기화 로직 격리)

동기화 프로세스가 너무 복잡하고 지속적으로 조정이 필요한 경우, Core Service에서 이를 추출하여 **Customer_CS**를 외부 시스템으로부터 더욱 격리하는 것이 권장됩니다.

![Isolate synchronization logic pattern](https://success.outsystems.com/TK_Resource/fce06b8e-b137-470b-9816-810270bbbf27)

이를 수행하는 또 다른 이유는 동기화가 여러 통합을 오케스트레이션해야 할 때입니다. 예를 들어 Customers가 한 시스템에 저장되고 Customer Contracts가 다른 시스템에 저장된 경우, 계약이 고객을 참조하므로 동기화 시 고객이 계약보다 먼저 동기화되어야 합니다.

**Customer_CS**의 소비자는 동기화 코드에 영향을 받을 필요가 없습니다. 또한 향후 ERP가 deprecated되고 OutSystems에 내장된 기능으로 교체되더라도 동기화 코드를 제거하는 것이 영향을 미치지 않습니다.

이 예시에서 **Customer_Sync**는 Integration Service를 통해 정기적으로 업데이트된 정보를 가져와 **Customer_CS**에 동기화합니다. **Customer_CS**는 여전히 온라인 트랜잭션 수행을 위해 **Customer_IS**를 소비합니다.

---

### Real-Time Sync (실시간 동기화)

일반적으로 검색, 나열 또는 mashup에 필요한 요약 데이터의 Cold Cache는 매우 안정적이어서 실시간 동기화가 필요하지 않습니다. 그러나 일부 상황에서는 요약 정보에 애플리케이션이 일관되게 작동하기 위해 실시간으로 최신 상태를 유지해야 하는 필드가 포함되어야 합니다 (예: 외부 시스템에서 차량 정보를 조회해야 하고, 자주 변경되는 필드인 각 차량의 현재 위치를 요약 목록에 포함하는 것이 중요한 경우).

![Real-Time Sync pattern](https://success.outsystems.com/TK_Resource/b694bf68-ebc6-4ec0-a8a0-9692865d5171)

실시간 동기화는 외부 레코드 시스템이 OutSystems에 실시간으로 어떤 변경(이 예시에서는 고객 업데이트 또는 삽입)을 알리기 위해 콜백해야 합니다. **API**와 **IS** 모듈이 Core Service를 완전히 격리하여 외부 시스템과 동기화 프로세스에 대해 무관하게 만듭니다.

| 장점                              | 단점                                       |
| --------------------------------- | ------------------------------------------ |
| 더 단순한 API                     | 더 많은 시스템 간 의존성                   |
| OutSystems에서 데이터 mashup 가능 | 큐를 제공하지 않아 변경이 많은 경우 부적합 |
| 소스 시스템에 대한 영향 최소화    |                                            |
| 데이터가 항상 최신                |                                            |

---

### Queued real-time sync (큐 기반 실시간 동기화)

기본 실시간 동기화는 구현하기 비교적 간단하지만, 변경이 많은 경우에는 적합하지 않습니다. 이 경우 해결책은 큐를 구현하여 동기화가 병렬로 처리되도록 하는 것입니다.

![Queued real-time sync pattern](https://success.outsystems.com/TK_Resource/3084ac2f-e225-4f8f-940c-5e16bbe877e6)

① 업데이트 요청을 큐에 신속하게 삽입합니다.
② BPT가 인바운드 메시지에 반응하여 **Customer_CS**를 업데이트합니다.

---

### Ordered real-time sync (순서 보장 실시간 동기화)

외부 시스템이 여러 업데이트 요청을 동시에(예: 멀티스레딩으로) 전송하도록 준비된 경우, 해당 요청들이 올바른 순서로 수신되고 처리된다는 보장이 없습니다. 이 문제를 해결하기 위한 변형입니다.

![Ordered real-time sync pattern](https://success.outsystems.com/TK_Resource/3084ac2f-e225-4f8f-940c-5e16bbe877e6)

① ERP가 시퀀스 제어와 함께 메시지를 전송합니다 (Send message with sequence control).
② 해당 클러스터를 식별하는 항목을 큐에 넣기 전에 관련된 모든 메시지가 수신되기를 기다립니다 (Wait for all related messages being received, before enqueuing an entry identifying that cluster).
Light BPT가 올바른 순서로 모든 관련 메시지를 처리하며 인바운드 클러스터에 반응합니다 (Light BPT reacts to inbound cluster, processing all related messages in correct order).

---

### Lazy Load Details (지연 로딩 세부 정보)

위의 패턴들은 **요약 데이터 필드**를 다룰 때의 일반적인 통합 패턴을 다룹니다. 그렇다면 **세부 필드**는 어떻게 할까요? 외부 시스템에서 세부 정보를 가져온 후 재사용하는 빈도가 높다면, Lazy Load 패턴을 설정하는 것이 권장됩니다. 세부 정보를 온디맨드로 가져오는 것은 비용이 너무 많이 들 수 있기 때문입니다.

**Lazy Load Details 패턴 구현 방법:**

1. 로컬 캐시에서 데이터를 가져옵니다. 없거나 오래된 경우, 외부 시스템에서 항목 세부 정보를 가져와 캐시합니다(read-through caching).
   - 캐시된 세부 정보는 만료 날짜를 가질 수 있어, 일정 기간 후 소스에서 강제로 새로고침할 수 있습니다.
   - 또는 캐시된 데이터를 재사용하기 전에, 외부 레코드 시스템에 해당 정보의 마지막 업데이트 타임스탬프를 요청하여 로컬 타임스탬프와 비교하고 항목이 최신인지 아닌지를 결정합니다.
2. 요약 데이터도 캐시되는 경우, 세부 정보를 캐시하기 위한 별도의 세부 엔티티를 가집니다.
3. 공간을 절약하기 위해 오래된 세부 데이터를 정기적으로 정리하는 타이머를 가집니다.

> 💡 **다양한 패턴을 결합하는 것이 일반적입니다: 요약 데이터는 정기적으로 캐시하고, 세부 정보는 필요할 때 lazy load합니다.**

---

### Transparency Service (투명성 서비스)

동일한 유형의 정보에 대해 여러 소스가 있는 경우, 일반적으로 서로 다른 형식과 API를 사용하여 동기화 프로세스가 더 복잡해집니다.

![Diagram showing a transparency service that synchronizes customer data from multiple systems into a single Core Service.](https://success.outsystems.com/TK_Resource/cf96ffe0-e0bd-4763-844a-b1f3a4c967a5)

**Customer_Sync**는 모든 레코드 시스템과의 동기화를 오케스트레이션하여 **Customer_CS**에 단일 Customer 데이터 복제본을 업데이트합니다. 이것이 **투명성 서비스(transparency service)** 를 생성합니다.

이 예시에서 **Customer_CS**는 고객을 업데이트할 수 없습니다. 이 패턴은 정보가 다른 소스에서 오지만 반대 방향으로는 흐르지 않는 가장 일반적인 투명성 서비스입니다. 예시로는 서로 다른 소스/형식에서 오는 전기 또는 통행료 읽기가 있습니다. 로컬에서 수정하거나 고칠 수 있지만, 소스로 다시 전송되지는 않습니다.

그러나 변경 사항을 다시 전송하는 것이 요구사항인 경우, 추가 모듈이 필요합니다. **Customer_IS**는 어떤 인터페이스(또는 드라이버)가 실제로 트랜잭션을 올바른 시스템으로 전송할지 추상화합니다.

![Diagram of a transparency service with an Integration Service routing requests to the correct system driver.](https://success.outsystems.com/TK_Resource/4efb992c-625a-4610-b7d2-2f3468fbefd3)

1. Integration Service는 서로 다른 시스템의 존재를 추상화하여 올바른 드라이버로 요청을 라우팅합니다.
2. 각 시스템은 서로 다른 드라이버로 인터페이스됩니다. 모든 드라이버는 동일한 시그니처를 가진 API를 제공합니다 (ERP1_Drv, ERP2_Drv, SaaS_Drv → ERP1, ERP2, SaaS).
3. 서로 다른 고객이 서로 다른 시스템에 있을 수 있습니다.

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

SOA(Service-Oriented Architecture)에서는 올바른 재사용을 위해 Core Service를 추상화합니다. 그러나 모든 서비스가 OutSystems 애플리케이션과 함께 동일한 환경(설치)에 상주하므로 마이크로서비스 스타일 구현을 구성하지는 않습니다.

마이크로서비스 접근 방식에서는 각 서비스가 자체 인프라를 가지며 분리됩니다. 서비스와의 통신 및 서비스 간 통신은 REST API와 같은 느슨하게 결합된 경량 메커니즘을 통해서만 이루어집니다.

SOA 접근 방식이 이미 모듈식 접근 방식을 적용하고 있지만, 마이크로서비스는 다음 속성을 향상시킵니다.

- **소비자에 대한 영향이 적은 서비스의 지속적 배포**
- **더 작은 footprint** — 소비 애플리케이션이 서비스 코드를 포함하지 않음
- **각 서비스 수요에 따라 독립적으로 확장할 수 있는 덜 모놀리식한 서버**

OutSystems에서 마이크로서비스의 주요 **단점**은 REST API만 소비할 수 있기 때문에 플랫폼의 RAD(Rapid Application Development) 기능을 더 이상 활용할 수 없다는 것입니다.

- Aggregates를 통한 데이터 조회를 가속화하는 엔티티 없음
- 위의 제한으로 인해 모든 데이터 조회 요구사항은 마이크로서비스 API를 통해 제공되어야 함
- 서비스가 제공하는 재사용 가능한 Blocks 없음

![Microservices vs Monolithic architecture](https://success.outsystems.com/TK_Resource/ee7449e9-711b-4665-8abc-d611668f5bcd)

마이크로서비스 접근 방식을 채택하려면 서비스를 다른 인프라에 배치해야 하며, 애플리케이션은 REST API를 통해 해당 서비스를 소비합니다. OutSystems RAD를 활용하기 위해 마이크로서비스에 대한 애플리케이션 측 확장을 구현할 수 있습니다.

마이크로서비스를 직접 소비하는 대신, **Extended Services**가 로컬 엔티티에 데이터를 캐싱하고 재사용 가능한 Blocks 또는 복합 비즈니스 로직을 제공함으로써 Core Service를 보강합니다. 이것들은 새로운 것이 아니며, 모든 외부 API에 대한 권장 사항과 일치합니다.

![Extended Services augmenting Core Services](https://success.outsystems.com/TK_Resource/26643e7c-b752-4971-ab46-124ea3da7f36)

### Multiple Microservice Infrastructures

일반적으로 마이크로서비스 접근 방식에서는 각 마이크로서비스가 자체 인프라에 상주합니다. 이를 통해 각 인프라가 독립적으로 확장되고 다른 마이크로서비스의 부하에 영향을 받지 않습니다.

![Multiple Microservice Infrastructures](https://success.outsystems.com/TK_Resource/3ca38069-f2d1-4995-9603-508306bded2c)

분리는 독립적인 마이크로서비스 간에 이루어져야 합니다. 두 마이크로서비스가 강한 의존성으로 결합된 경우(예: 많은 데이터베이스 참조) — 플랫폼의 전체 기능을 활용하고 REST API를 통한 통신만의 한계를 피하기 위해 서로 다른 인프라로 분리해서는 안 됩니다.

모든 마이크로서비스를 동일한 인프라에서 시작하고, 자체 인프라와 성장 정책이 필요할 때 격리할 수 있습니다. 이 작업을 용이하게 하려면 공통 인프라를 사용할 때 각 마이크로서비스를 다른 DB Catalog로 격리하세요.

### Central Services for Multiple Subsidiaries

마이크로서비스에 적합한 또 다른 사용 사례는 중앙 서비스 지원입니다: 서로 다른 자회사가 소비해야 하는 중앙 백엔드 시스템을 확장하는 것입니다. 해당 자회사는 자체 설치에서 커스텀 애플리케이션을 구현합니다.

![Central Services for Multiple Subsidiaries](https://success.outsystems.com/TK_Resource/fb6588ad-5e6d-453d-b241-0ae164617918)

### Microservices Lifecycle — Managing Versions

마이크로서비스의 여러 버전은 다음을 통해 유지할 수 있습니다.

- REST API에서 서비스 정의의 여러 버전 노출
- 실제 로직을 구현하는 Core 모듈에서 액션의 여러 버전 유지

![Microservices Lifecycle - Managing Versions](https://success.outsystems.com/TK_Resource/83a1b6f6-936e-451b-911f-d5b9d6e06029)

#### When to Create a New Version

마이크로서비스의 격리를 통해 API 시그니처가 변경되지 않는 한 새 버전을 공개적으로 발표할 필요 없이 지속적으로 배포할 수 있습니다.

API를 변경해야 할 때, 현재 버전을 업데이트할 수 있는지 또는 마이크로서비스 API의 새 버전을 만들어야 하는지 결정해야 합니다.

**현재 API 버전 변경 (Change the current API version)**

다음만 포함되는 경우 현재 버전을 변경합니다.

1. **새 액션을 위한 새 메서드 추가**: 새 메서드가 필요하지 않은 소비자는 REST 서비스 정의를 새로 고칠 필요가 없으므로 영향을 받지 않습니다.

   > 📌 **[다이어그램 삽입: 새 메서드 추가 — 기존 소비자 영향 없이 새 메서드가 Core 모듈에 추가되는 구조]**

2. **모든 소비자에게 강제해야 하는 시그니처 변경**: 현재 버전을 변경하면 소비자가 REST 서비스를 새로 고치도록 강제합니다.

   > 📌 **[다이어그램 삽입: 시그니처 변경 — 소비자가 REST 서비스를 강제로 새로 고치는 구조]**

**새 API 버전 생성 (Create new API version)**

메서드 시그니처가 변경되었지만 모든 소비자에게 강제할 수 없어 이전 버전을 하위 호환성 유지를 위해 보존해야 하는 경우 새 버전을 생성합니다. 두 가지 상황이 있습니다.

1. **비파괴적 메서드 시그니처 변경 (Non-disruptive change)**: 새로운 선택적 속성만 포함된 경우 비파괴적입니다.

   > 📌 **[다이어그램 삽입: 비파괴적 변경 — 선택적 속성 추가로 이전 버전 유지하는 구조]**

2. **파괴적 메서드 시그니처 변경 (Disruptive change)**: 새로운 필수 속성이 포함되거나 기존 속성 이름 또는 타입이 변경된 경우 파괴적입니다.

   > 📌 **[다이어그램 삽입: 파괴적 변경 — 필수 속성 추가 또는 속성 변경으로 새 버전 강제 적용하는 구조]**

   이 경우 이전 버전은 새 구현과 호환되지 않으므로 deprecated 처리해야 합니다. 따라서 소비자는 새 버전을 채택하도록 강제됩니다.

### Infrastructure Scenarios

#### Scenario A — Single Environment for Semi-isolated Microservices

![Scenario A - Single Environment](https://success.outsystems.com/TK_Resource/15b43ae4-e0ad-4aec-af5c-4540a903f036)

서로 다른 마이크로서비스를 격리하려면 마이크로서비스 Zone을 추가하기만 하면 됩니다 (Zone B 예시).

#### Scenario B — Separate Environment for Fully Isolated Microservices

> 📌 **[다이어그램 삽입: Scenario B — 완전히 격리된 마이크로서비스를 위한 별도 환경 구조]**

서로 다른 마이크로서비스를 격리하려면 마이크로서비스 인프라를 추가하기만 하면 됩니다 (Infrastructure B 예시).

#### Comparing Scenarios

> 📌 **[다이어그램 삽입: Comparing Scenarios — 단일 인프라 vs 다중 인프라 비교 구조]**

---

## 9. From architecture to development

이 시리즈 문서는 아키텍처 Blueprint를 실제 OutSystems 애플리케이션으로 변환하는 방법에 대한 가이드를 제공합니다.

이 문서(9번)는 애플리케이션의 아키텍처 Blueprint를 설계하는 방법에 집중합니다. 다음 문서(10번)는 Blueprint를 기반으로 애플리케이션 개발을 시작하는 방법을 안내합니다. 코딩 및 아키텍처 모범 사례를 실제로 확인할 수 있는 코드 샘플도 포함되어 있습니다.

### Architecture blueprint design

가상의 고객 **Soccer Fields Fictitious**가 고객들이 온라인으로 축구장을 예약할 수 있는 웹 애플리케이션을 구축하려 한다고 가정합니다.

> ※ 이것은 가상의 클라이언트를 사용한 가상의 사용 사례입니다.

#### Requirements

애플리케이션은 다음을 충족해야 합니다.

- 각 필드 특성(크기, 가용성, 시간당 가격 등)과 함께 축구장 목록을 표시
- 플레이어가 축구장을 선택하고 예약할 수 있도록 허용

외부 시스템이 축구장 목록을 제공하므로, 실제 필드 관리나 결제 처리는 이 애플리케이션의 범위에 포함되지 않습니다.

#### Architecture design process

아키텍처 설계 프로세스는 3단계 접근 방식을 따릅니다.

![Diagram of the architecture design iteration process with three segments: Disclose, Organize, and Assemble.](https://success.outsystems.com/TK_Resource/edf37634-5e44-4b8c-8dbe-d0a22c9de9cb)

| 아이콘                                                       | 단계                                                   | 설명                               |
| ------------------------------------------------------------ | ------------------------------------------------------ | ---------------------------------- |
| ![Icon representing the Disclose step in the architecture design process.](https://success.outsystems.com/TK_Resource/2a4cea33-5b36-47cd-9ce8-c5a63d75177e) | **Disclose** business concepts and integration needs   | 비즈니스 개념과 통합 요구사항 발굴 |
| ![Icon representing the Organize step in the architecture design process.](https://success.outsystems.com/TK_Resource/d833b165-4250-445f-8a77-89421b686b4a) | **Organize** concepts on the architecture Layer canvas | 아키텍처 레이어 Canvas에 개념 배치 |
| ![Icon representing the Assemble step in the architecture design process.](https://success.outsystems.com/TK_Resource/21a923e3-229a-42fb-a838-1180e10e04c5) | **Assemble** Matching recommended patterns             | 권장 패턴에 맞게 조합              |

세 단계를 여러 번 반복하는 것이 중요합니다. 설계에 만족할 때까지 반복하고, 구현 중에 새로운 개념이나 세부 사항을 발견할 때도 반복합니다. 이 문서는 간단한 애플리케이션을 예시로 하여 단일 반복만 수행합니다.

---

### Disclose (business concepts and integration needs)

먼저 기능적 및 비기능적 요구사항을 이해합니다. 이 단계에서는 가능한 한 많은 사용자 스토리를 파악하고, 통합 요구사항과 UX 기대치를 이해해야 합니다.

**Functional Requirements (기능 요구사항)**

다음 필터를 사용하여 필드를 검색합니다.

- 필드 이름
- 도시
- 필드 크기(함께 플레이할 수 있는 플레이어 수)

플레이어가 사용 예정인 필드를 예약할 수 있도록 허용합니다.

**Non Functional Requirements (비기능 요구사항)**

- 시장의 주요 브라우저와 호환
- REST API를 사용하여 외부 시스템에서 축구장 목록 조회
- 시스템이 온라인으로 작동하며 UI가 모바일 장치에 적응해야 함

---

### Organize (concepts on architecture canvas)

Architecture Canvas 원칙을 간단히 정리하면 다음과 같습니다.

| 레이어                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![Icon for the End User Layer, indicating user interfaces and processes.](https://success.outsystems.com/TK_Resource/36c9078b-c54d-493a-8518-b736e37fa9e8) | 사용자 인터페이스와 프로세스 — Core와 Library를 재사용하여 사용자 스토리를 구현 |
| ![Icon for the Core Layer, indicating core business services.](https://success.outsystems.com/TK_Resource/07bb59f3-50a1-4f49-9da6-77c4effd8611) | 비즈니스 개념 중심 서비스 — 재사용 가능한 엔티티, 비즈니스 규칙, 비즈니스 위젯 노출 |
| ![Icon for the Foundation Layer, indicating non-business services.](https://success.outsystems.com/TK_Resource/6b782105-7723-47f3-ab5c-35a0e998212b) | 비즈니스 무관 서비스 — 고도로 재사용 가능한 assets, UI 패턴, 외부 시스템 커넥터, 네이티브 코드 통합으로 프레임워크 확장 |

이 원칙을 기반으로 다음 레이어 Canvas에 애플리케이션의 개념을 배치합니다.

> 📌 **![Architecture Layer Canvas showing the distribution of application components across different layers.](https://success.outsystems.com/TK_Resource/35ed7e75-34dc-42ae-b62d-06dbdbcb0d96)**

식별된 주요 개념들:

- **SoccerFields**: 사용자와의 인터랙션을 담당하는 메인 애플리케이션
- **Bookings**: 플레이어가 필드를 예약할 수 있게 하는 기능 요구사항
- **Players**: 애플리케이션을 사용하고 필드를 예약하는 플레이어
- **Fields**: 검색 및 플레이어의 예약에 사용되는 메인 필드 개념
- **Fields Integrations**: 외부 시스템에 축구장이 저장되므로 API를 통해 외부 시스템에 연결하는 비기능 요구사항 모듈
- **Soccer Fields Theme**: 애플리케이션의 look & feel이 모바일 반응형이어야 함

---

### Assemble (matching recommended patterns)

Assemble 단계에서 적용하는 원칙들:

- 개념적으로 관련된 개념은 합칩니다.
- 너무 복잡하거나 서로 다른 라이프사이클을 가진 경우 개념을 합치지 않습니다.
- 소비자가 신경 쓰지 않아도 되는 모든 통합 로직에서 재사용 가능한 로직을 격리합니다.

이 원칙들과 아키텍처 패턴 및 모범 사례를 바탕으로 다음 개념에 집중합니다.

**Online (Field_IS 모듈)**

외부 API에 접근하기 위해 **Field_IS** 모듈(IS = Integration Service)을 사용합니다. 이 모듈의 책임은 외부 API를 파악하고 내부 접근을 위해 데이터를 정규화하는 것입니다. 이를 통해 Field_CS와 Field_Sync 모듈은 API의 특수성을 알 필요가 없습니다. 추상화 측면에서 이것은 매우 중요합니다. 이 패턴을 사용하면 애플리케이션이 API 변경에 더 탄력적이며, 데이터 모델에서 원하지 않는 내용을 API에서 흡수할 필요가 없습니다.

**Batch Sync (Field_Sync 모듈)**

필터를 용이하게 하고 더 나은 사용자 경험(성능 향상)을 제공하기 위해 두 번째 패턴을 사용합니다. Field 엔티티의 요약본을 로컬에 유지하여 외부 API에 접근하지 않고 로컬 엔티티에서 주요 검색을 수행할 수 있습니다. 이를 위해 **Field_Sync** 모듈이 하루에 한 번 필드 데이터를 동기화합니다.

> ![Diagram illustrating integration patterns for external master entities with batch sync.](https://success.outsystems.com/TK_Resource/f271e453-2876-4125-87df-50a6c2c4facb)

**Business Logic Module (Booking_BL 모듈)**

복잡성 관리, 조합 또는 자체 라이프사이클을 갖기 위해 비즈니스 로직(Actions) 또는 Core Widgets(Webblocks)를 격리하는 것을 권장합니다. 이 경우 _CW 모듈을 만들 필요가 없었습니다. 나중에 생길 수도 있지만, 지금은 **Booking_BL** 모듈이 필요한 모든 비즈니스 로직을 처리합니다.

이러한 아키텍처 패턴을 적용하고 모듈 이름에 공통 명명 규칙을 사용하면 다음과 같은 모듈 아키텍처 Blueprint가 완성됩니다.

> ![Blueprint diagram showing the architecture of the Soccer Fields application with various modules.](https://success.outsystems.com/TK_Resource/cbd2216f-5e78-4adf-972f-14132212d2c3)

각 모듈의 주요 기능 및 책임:

| 모듈             | 레이어     | 책임                                                         |
| ---------------- | ---------- | ------------------------------------------------------------ |
| **SoccerFields** | End-user   | 축구장 검색 및 예약을 위한 모든 화면을 포함하는 End-user 애플리케이션 |
| **Booking_BL**   | Core       | 예약 관리와 관련된 모든 로직 포함, 필드와 플레이어 개념의 로직을 결합 |
| **Player_CS**    | Core       | 플레이어 엔티티 및 관련 엔티티를 포함하는 재사용 가능한 Core Service |
| **Field_CS**     | Core       | 외부 데이터의 로컬 복제본(Cold Cache)을 호스팅하는 재사용 가능한 Core Service — 필드 엔티티 및 관련 엔티티 포함 |
| **Field_Sync**   | Core       | 외부 시스템에서 Field_CS로 데이터를 동기화하는 로직. Field 엔티티의 요약본만으로 Cold Cache를 유지하는 모든 로직을 격리하는 책임 |
| **Field_IS**     | Foundation | Field API를 소비하고 정규화하는 기술적 wrapper. 외부 API의 모든 특수성을 추상화하고 파악하는 책임 |
| **SF_Th**        | Foundation | Soccer Field 애플리케이션의 메인 look & feel을 위한 Theme 모듈 (비기능) |

---

### Application composition

모듈을 만들 때뿐만 아니라 애플리케이션을 만들고 구성하는 방법에 대해서도 아키텍처를 생각해야 합니다.

OutSystems 애플리케이션은 Service Studio에서 정의된 모듈 세트로, LifeTime(OutSystems의 애플리케이션 라이프사이클 관리 도구)의 최소 배포 단위를 구성합니다.

> ![Illustration of the modules of an OutSystems application organized by layer.](https://success.outsystems.com/TK_Resource/f8ea43af-173e-4d71-b791-8f89a035f682)

애플리케이션도 모듈과 마찬가지로 레이어를 준수해야 합니다. 애플리케이션의 레이어를 결정하는 것은 그 안에 있는 모듈들 중 **최상위 레이어**입니다. 즉, Foundation 레이어의 모듈만으로 구성된 애플리케이션은 Foundation Application입니다. Foundation 레이어 모듈도 있지만 하나 이상의 Core 레이어 모듈이 있는 경우 그 애플리케이션은 Core Application입니다.

Soccer Fields App의 애플리케이션 구성은 다음과 같습니다.

> ![Diagram showing the composition of the Soccer Fields application with its modules and layers.](https://success.outsystems.com/TK_Resource/d5f4941c-55b4-4246-9c87-8d9e2ab78c9f)

두 개의 애플리케이션이 있습니다.

- **Soccer Fields App**: SoccerFields 사용자 모듈(사용자가 상호작용하는 프론트엔드 애플리케이션, 스타일 커스터마이징에도 사용)과 SF_Th 모듈(애플리케이션에 커스텀 스타일이 적용된 경우)을 포함합니다.
- **Field Core Services**: 필드 관련 모든 모듈을 포함합니다.

최종 아키텍처 Blueprint를 실제로 구현하는 방법은 **Developing from the architecture blueprint** 문서(10번)를 참고하세요.

---

### 9-1. Developing from the architecture blueprint

이 문서는 최종 아키텍처 Blueprint를 실제로 구현하는 방법에 집중합니다.

이전 문서(9번)에서 가상 클라이언트 Soccer Fields Fictitious의 사용 사례를 설명했습니다. 이 클라이언트는 고객들이 온라인으로 축구장을 예약할 수 있는 웹 애플리케이션을 구축하려 합니다.

> 진행하기 전에 OutSystems Forge에서 Soccer Fields App resources, Soccer Field Sample API, Field Core Services App을 다운로드하여 개인 환경에 설치하세요.

#### Requirements

애플리케이션은 다음을 충족해야 합니다.

- 각 필드 특성(크기, 가용성, 시간당 가격 등)과 함께 축구장 목록을 표시
- 플레이어가 축구장을 선택하고 예약할 수 있도록 허용
- 외부 시스템이 축구장 목록을 제공 (실제 필드 관리나 결제 처리는 이 애플리케이션의 범위 외)

#### Architecture design

이전 문서에서 최종 아키텍처 Blueprint가 완성되었습니다. 아래 이미지는 개발 단계의 시작점을 보여줍니다.

> ![Diagram of the final architecture blueprint for the Soccer Fields application.](https://success.outsystems.com/TK_Resource/d8b783bc-8041-41bf-987e-b0ded139beca)

---

#### Understanding the blueprint (Blueprint 이해)

첫 번째 단계는 Blueprint를 이해하는 것입니다.

- 어떤 요소가 모듈이고 어떤 요소가 애플리케이션인지 식별할 수 있어야 합니다.
- 각 요소의 성격과 목표를 이해하여 올바른 위치에 코드를 배치할 수 있어야 합니다.

다음은 Blueprint에서 사용하는 표기 규칙입니다.

| 표기                                                         | 의미                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![Icon representing modules in the architecture blueprint with rounded rectangle shapes.](https://success.outsystems.com/TK_Resource/9730e3a0-2252-4fae-8c3d-db93ab8a66b5) | 모듈을 나타냅니다. 색상으로 Architecture Canvas의 레이어를 구분합니다. |
| ![Icon representing applications in the architecture blueprint with rectangles containing modules.](https://success.outsystems.com/TK_Resource/d34c4ef6-3fc6-4958-a2b8-c5bb6f5a6024) | 애플리케이션을 나타냅니다. 모듈 색상보다 밝은 색이며 아키텍처 레이어를 매핑할 수 있습니다. |

> 📌 **[다이어그램 삽입: Blueprint 표기 규칙 — 모듈(둥근 모서리) vs 애플리케이션(모듈 포함 직사각형) 아이콘 설명]**

아키텍처 설계는 각 요소의 성격을 문서화해야 하며, 아키텍트가 이러한 세부 정보를 추가할 책임이 있습니다.

이 예시에서 두 개의 애플리케이션이 있습니다.

- **Soccer Fields App**: 사용자와 상호작용하고 필드를 예약하는 코드를 포함하는 메인 애플리케이션
- **Field Core Services**: 외부 소스에서 오는 필드 정보를 연결하고 획득하는 지원 애플리케이션

---

#### Identify module dependencies (모듈 의존성 파악)

아키텍처가 견고한지 확인하고 이를 이해하는 좋은 방법은 모듈 간의 관계와 계층 구조를 파악하는 것입니다. 어떤 모듈이 어디에 의존하는지 이해하는 것은 Discovery tool로 검증하는 Architecture Canvas 검증 규칙을 위반하지 않기 위한 첫 번째 단계입니다.

이것은 특히 Core 레벨에서 중요합니다. 잘못된 의존성은 적절한 개념 격리 부재로 인해 순환 참조를 유발합니다. 이는 코드 리팩토링으로 이어지며 높은 위험과 비용을 수반합니다.

> ![Diagram illustrating the dependencies between modules in the architecture blueprint.](https://success.outsystems.com/TK_Resource/09c95460-bf3a-4d9e-9892-38979e92d6f3)

주요 인사이트:

- **Booking_BL**: 필드와 플레이어라는 두 Core 개념의 조합으로 작동합니다. 이 두 Core 개념은 독립적이며, 플레이어가 필드를 예약할 때 Booking_BL 모듈이 해당 연관을 유지합니다.
- **Field_CS**: 성능상의 이유로 기존 필드의 로컬 복제본을 포함합니다. Field_Sync 모듈이 이 로컬 복제본을 동기화하며, Field_IS에서 정보를 읽어 Field_CS에 씁니다.
- **Field_IS**: 외부 REST API에 연결하는 유일한 모듈로, 조회한 정보를 내부 형식으로 정규화합니다. 이 추상화 수준은 시스템이 외부 시스템과 독립적으로 발전하는 데 매우 중요합니다.

아키텍처 Blueprint에 이 정보가 포함되어야 합니다. 없다면 이 작업을 수행하고 아키텍트와 결과를 검증하는 것을 권장합니다.

---

#### Creating the modules and applications (모듈 및 애플리케이션 생성)

이 섹션은 모듈과 애플리케이션 개발 과정을 안내합니다. 첫 번째 단계는 Service Studio를 열어 시작하는 것입니다.

**The end-user application (End-user 애플리케이션)**

아키텍처 설계에 따라 첫 번째 애플리케이션과 모듈을 생성합니다.

1. Service Studio에서 **New Application**을 선택하고 **Web App**을 선택한 후 Liverpool 테마를 선택합니다. 애플리케이션 이름을 "Soccer Fields"로 지정하고 **CREATE APP**을 클릭합니다.

2. End-user 애플리케이션에서 처음 생성할 모듈은 **Theme 모듈**입니다. Theme 모듈을 통해 모범 사례에 따라 애플리케이션의 스타일을 격리하고 커스터마이징할 수 있습니다. 모듈 이름을 **SF_Th**로 변경하고 Responsive 모듈 유형을 선택한 후 **CREATE MODULE** 버튼을 클릭합니다.

   > ![Screenshot of the module creation interface in Service Studio with the module name 'SF_Th' and module type options.](https://success.outsystems.com/TK_Resource/9ddf5b13-d19f-42b6-9332-170c64895b0b)

3. 애플리케이션을 게시하기 전에 Emails와 MainFlow UI 흐름을 삭제합니다. 그 후 **1-Click Publish** 버튼을 클릭합니다.

   > ![Screenshot highlighting the 'Emails' and 'MainFlow' UI flows to be deleted in Service Studio.](https://success.outsystems.com/TK_Resource/1128b71d-128e-4583-add7-4612cbff3a6d)

4. 이 애플리케이션의 프론트엔드 모듈 **SoccerFields**를 생성합니다. 새 Responsive 모듈을 만들고 게시합니다.

   > ![Screenshot showing the process of creating a new front-end module named 'SoccerFields' in Service Studio.](https://success.outsystems.com/TK_Resource/adc2dbd9-c25f-48eb-9bc2-886f1d239208)

5. **SoccerFields** 모듈을 Home 모듈로 설정합니다.

   > ![Screenshot depicting how to set the 'SoccerFields' module as the Home module in Service Studio.](https://success.outsystems.com/TK_Resource/6ac50af4-b79a-456a-930c-fef299b20935)

6. 이 애플리케이션에 **Booking_BL**과 **Player_CS** 두 개의 새 모듈을 추가합니다. **Blank 모듈 유형**을 선택합니다.

   > ![Screenshot of the Service Studio interface for creating a new module named 'Booking_BL'.](https://success.outsystems.com/TK_Resource/8c63a8fe-067c-4d5d-bb30-04ac175b486c)

   > ![Screenshot showing the creation of a new module named 'Player_CS' in Service Studio.](https://success.outsystems.com/TK_Resource/e9e91f66-21c1-4c27-9e81-d027192f146b)

**The core application (Core 애플리케이션)**

"Soccer Fields" 애플리케이션과 동일한 첫 번째 단계를 반복하되, 이번에는 애플리케이션 이름을 **"Field Core Services"** 로 지정합니다.

첫 번째 Core 모듈(**Field_IS**)은 Responsive가 아닌 **Blank 모듈 유형**을 사용합니다. "Field Core Services"는 Core 서비스 애플리케이션이므로 모범 사례에 따라 End-user 화면을 호스팅해서는 안 됩니다.

> ![Screenshot of the Service Studio interface for creating a new module named 'Field_IS'.](https://success.outsystems.com/TK_Resource/ba53e6e8-e465-4281-97b7-da354933a2a9)

**CREATE MODULE**을 클릭한 후 **1-Click Publish** 버튼으로 애플리케이션을 게시합니다. 이후 나머지 모듈인 **Field_CS**와 **Field_Sync**를 동일한 방식으로 생성합니다.

**The foundation application (Foundation 애플리케이션)**

이 샘플에서는 예시의 단순성과 재사용성 요구사항으로 인해 Foundation 애플리케이션이 필요하지 않습니다.

---

#### Implementation (구현)

애플리케이션과 모듈이 생성되었으니 이제 프로그래밍을 시작합니다.

필드 API를 시뮬레이션하기 위해 REST로 통신할 OutSystems 사전 생성 단순 가짜 API 모듈을 사용합니다. API는 매우 단순하며 리소스는 필드 하나뿐입니다. 모든 요청에는 `API_KEY` 요청 헤더에 유효한 키를 전송해야 합니다. 샘플이므로 유효한 API_KEY는 **"outsystems_is_awesome"** 입니다.

**Architecture patterns**

이 사용 사례에는 축구장 정보를 조회하기 위한 중요한 통합 패턴이 있습니다.

> ![Diagram showing the architectural patterns for retrieving soccer field information.](https://success.outsystems.com/TK_Resource/fbb8cf11-9baf-45b7-90ce-cf17a6ddc395)

세 가지 아키텍처 패턴에 집중합니다.

1. **Integration with an External System** — 외부 시스템과의 통합
2. **Local Replica of data** — 데이터 로컬 복제
3. **Synchronization of data** — 데이터 동기화

---

**Translating the pattern into code (패턴을 코드로 변환)**

**① Integrating with an External Service (외부 서비스 통합)**

외부 API와의 통합부터 시작합니다. **Field_IS** 모듈이 적합한 위치입니다. 이 모듈은 외부 API를 소비하고 정규화하는 기술적 wrapper이며, 외부 API의 모든 특수성을 추상화하고 격리하는 책임을 집니다.

*Consume the external API (외부 API 소비)*

Field_IS 모듈을 열고 Logic 탭에서 Integrations 폴더 아래 REST 노드를 우클릭하여 **Consume REST API...**를 선택합니다. 팀이 swagger spec을 제공했다고 가정하고, **ADD ALL METHODS**를 클릭하고 swagger 파일의 URL을 입력합니다.

> ![Screenshot of the REST API methods in Service Studio after consuming an external API.](https://success.outsystems.com/TK_Resource/239e75fb-062d-47a3-a32d-452d281dcb9c)

*Send the API Key (API 키 전송)*

외부 API와의 통합을 완료하려면 `API_KEY` 요청 헤더에 유효한 키를 전송해야 합니다. **OnBeforeRequest** 이벤트를 구현합니다. 엔드포인트를 선택하고 On Before Request 속성 옆 드롭다운을 클릭하여 **New OnBeforeRequest**를 선택합니다.

> ![Screenshot showing the process of adding a new 'OnBeforeRequest' event to send the API key.](https://success.outsystems.com/TK_Resource/201afd21-019f-467f-a151-fde47fbf6815)

> ![Screenshot of the logic flow for implementing the 'OnBeforeRequest' event in Service Studio.](https://success.outsystems.com/TK_Resource/3f53820a-c4bd-43c3-b3ec-3158bb356534)

*Retrieve a list of Fields with summary data (요약 데이터로 필드 목록 조회)*

먼저 외부 API에서 가져온 필드 목록을 상위 레이어에 제공하는 액션의 출력이 될 **Field_IS_Summary** 구조체를 생성합니다.

소비자를 외부 데이터 유형으로부터 추상화하고 외부 데이터 유형의 향후 변경 영향을 방지하기 위해, 외부 API가 자동 생성한 구조체를 재사용하지 않습니다.

> ![Screenshot showing the 'Field_IS_Summary' structure with attributes like Name, Dimension, and City.](https://success.outsystems.com/TK_Resource/48b6a827-1b15-4cfb-8a0e-35eb4cfe5385)

**Field_IS_GetSummaryFields** Server Action을 생성하고 Public 속성을 "Yes"로 설정합니다.

> ![Screenshot of the logic flow for creating the 'Field_IS_GetSummaryFields' server action in Service Studio.](https://success.outsystems.com/TK_Resource/7b3e7e96-0b99-4f9b-89c7-a794c4ec65fc)

---

**② Local Replica of data (데이터 로컬 복제)**

성능을 높이고 더 나은 사용자 경험을 제공하기 위해 애플리케이션 측에 데이터 로컬 복제본을 만듭니다.

*Public Local Entity Read only*

**Field_CS**에 새 **Field** 엔티티를 생성합니다. Field_IS summary 구조체가 반환할 모든 필드를 추가합니다. Public 속성과 **Expose Read-only** 속성을 모두 "Yes"로 설정합니다. 외부 모듈이 데이터를 직접 변경하지 못하도록 방지합니다.

> ![Screenshot of the 'Field' entity properties in Service Studio with attributes like Name, Dimension, and City.](https://success.outsystems.com/TK_Resource/c4723be2-e8d3-45e3-9831-312f29b218cf)

*Creation of CRUD actions (CRUD 액션 생성)*

엔티티를 Read-only로 노출하므로 해당 엔티티에 데이터 생성을 중앙화하는 액션이 필요합니다. 이 접근 방식의 또 다른 장점은 로깅 등의 일반 작업을 중앙 액션에 집중할 수 있어 중복 코드를 방지한다는 것입니다.

Server Actions에 새 폴더를 만들고 **Field_Create** 액션을 추가합니다. Field를 입력 파라미터로, Field 식별자 id를 출력 파라미터로 설정합니다. 액션 본문에 CreateOrUpdateField 내부 액션을 드래그하고 마지막에 생성된 id를 반환하는 할당을 추가합니다.

> ![Screenshot showing the creation of CRUD actions for the 'Field' entity in Service Studio.](https://success.outsystems.com/TK_Resource/c5b7452e-c8f3-4e81-a268-efb259bc58c6)

동일한 방법으로 **Field_CreateOrUpdate** 액션을 생성합니다.

---

**③ Synchronization of data (데이터 동기화)**

**Field_Sync** 모듈은 외부 시스템에서 로컬 캐시된 데이터로의 비동기 동기화 로직을 담당합니다. 이 모듈은 Field_IS와 Field_CS를 참조하면서 타이머와 로직을 보유합니다.

특정 시간에 동기화 타이머가 트리거됩니다. 동기화 알고리즘은 Field_IS_GetSummaryFields를 사용하여 외부 시스템에서 필드 데이터의 서브셋을 가져옵니다.

> ![Screenshot of the logic flow for the synchronization algorithm of field data in Service Studio.](https://success.outsystems.com/TK_Resource/90e1c89d-7057-4f47-b92d-33c934f4d4de)

동기화 알고리즘의 주요 단계:

1. **로직 타임아웃 설정 (10분)**: GetSyncFieldCurrPages aggregate가 이전 타이머의 가능한 진행 상황을 가져와 처리를 계속합니다. 타이머 설계 모범 사례를 따르기 위한 것입니다 (절대 타임아웃 없음, 작업 반복 없음, 완전성 보장, 데이터 무결성 보장).
2. **시작 및 재귀 보장**: 로컬 캐시에 동기화할 데이터가 아직 있는 경우 동기화 흐름의 시작과 재귀를 보장하는 if 로직.
3. **IS 모듈 액션 호출**: 외부 시스템 호출을 트리거하고 동기화할 레코드 수를 반환합니다.
4. **결과 순환**: 모든 결과를 순환하며 Field 엔티티에 삽입/업데이트합니다.
5. **타임아웃 확인**: 로직 타임아웃(10분)에 도달했는지, 처리할 레코드가 더 있는지 확인합니다.
   - 10분 타임아웃에 도달하지 않고 처리할 레코드가 있으면 알고리즘을 한 번 더 순환합니다.
   - 10분 로직 타임아웃에 도달했으면 지금까지의 진행 상황을 저장하고 현재 실행을 종료하기 전에 타이머를 다시 깨웁니다 (타이머 모범 사례 보장).

---

---

## 10. Manage technical debt

AI Mentor Studio(https://aimentorstudio.outsystems.com/)는 OutSystems 기술 부채 모니터링 도구입니다. IT 리더가 크로스 포트폴리오 아키텍처를 시각화하고 문제 영역을 식별하는 동시에, 개발자가 모범 사례를 따르고 일반적인 함정을 피하도록 돕습니다.

조직이 시장 출시 시간을 단축하고 비즈니스 앱을 만드는 비전문 개발자(시민 개발자)의 역량을 강화하려 함에 따라, 기술 부채 관리가 자연스럽게 최우선 과제가 됩니다.

> ℹ️ 이 콘텐츠는 O11의 기술 부채 관리를 위한 AI Mentor Studio를 다룹니다. ODC의 AI 기반 앱 생성은 Build apps with AI를 참고하세요.

AI Mentor Studio를 사용하면 기술 부채를 개발 라이프사이클의 모든 단계에서 효과적으로 관리할 수 있습니다. 부서 애플리케이션이 기업 전체 솔루션으로 발전함에 따라 아무것도 다시 작성할 필요가 없습니다.

아키텍트와 개발팀 리더에게는 통합되고 조직의 기술 부채에 대한 조감도를 제공하여 문제 영역을 파악하고 우선순위를 정할 수 있습니다. 개발자는 어떤 모범 사례가 위반되고 있는지, 그 영향 및 수정 방법에 대한 상세 보고서를 볼 수 있습니다.

타사 도구와 AI Mentor Studio 데이터를 통합하려면 AI Mentor Studio API를 사용하세요.

### What AI Mentor Studio analyzes

AI Mentor Studio는 개발자가 생성한 코드를 분석하고 팀 민첩성에 영향을 줄 수 있는 코드 품질에 대한 인사이트를 제공합니다.

#### Code Analysis

AI Mentor Studio는 생성된 low-code 전반에 걸쳐 사전 정의된 규칙 세트를 실행하여 다음 카테고리에서 코드 패턴을 발견하는 것을 목표로 합니다.

- **Performance** — 성능
- **Architecture** — 아키텍처
- **Maintainability** — 유지보수성
- **Security** — 보안

현재 분석 중인 패턴 목록은 공식 문서에서 확인할 수 있습니다.

#### Architecture auto-classification

AI Mentor Studio는 인프라의 앱 아키텍처 분석에 있어 자급자족합니다. AI 엔진의 도움으로 AI Mentor Studio는 모듈의 코드와 그 관계를 분석하여 전체 아키텍처에서 어디에 맞는지 파악합니다.

AI auto-classification을 통해 AI Mentor Studio에 팩토리를 온보딩하고 올바른 아키텍처 레이어에 맞도록 각 모듈을 분류할 수 있습니다. 이를 통해 다른 모든 코드 분석이 가능해집니다.

기본적으로 AI Mentor Studio에 추가된 모든 새 인프라는 AI auto-classification을 사용합니다. 아키텍처 분류를 위해서는 Discovery가 설치되어 있어야 합니다.

### Access to AI Mentor Studio features

일부 AI Mentor Studio 기능의 가용성 또는 범위는 인프라와 연관된 OutSystems Edition에 따라 다릅니다.

- **Teams**: LifeTime 팀별로 앱 포트폴리오와 기술 부채 결과를 필터링하는 기능은 Standard 및 Enterprise Edition에서 사용 가능합니다.
- **Date Time Machine**: 기술 부채 분석의 전체 내역을 확인하는 기능은 Standard 및 Enterprise Edition에서 사용 가능합니다. Basic Edition은 1개월의 기술 부채 분석 내역에 접근할 수 있습니다.

Free Edition과 연관된 인프라는 AI Mentor Studio를 사용할 수 없습니다.

### How to access AI Mentor Studio

AI Mentor Studio를 인프라에서 사용하기 전에 IT 사용자가 AI Mentor Studio와 연관되어야 합니다.

AI Mentor Studio에 접근하려면 https://aimentorstudio.outsystems.com/ 으로 이동하여 OutSystems 계정으로 로그인하세요.

> ![img](https://success.outsystems.com/img/Documentation_UI.Gettingstarted__CANuzgqwWsu4b6Di99qi9w.png?CANuzgqwWsu4b6Di99qi9w)

---

## 10-1. How does AI Mentor Studio work

> ℹ️ Architecture Dashboard is now AI Mentor Studio.

AI Mentor Studio는 다음 컴포넌트로 구성됩니다.

- **AI Mentor Studio SaaS**: AI Mentor Studio LifeTime Plugin이 수집한 모든 데이터를 처리하고 표시하는 SaaS(Software as a Service)
- **AI Mentor Studio LifeTime Plugin**: 데이터를 수집하고 AI Mentor Studio SaaS와 통신하는 환경 probe와 함께 플랫폼 설치(온프레미스 또는 클라우드)에 게시되는 LifeTime 플러그인

### Communication (통신)

AI Mentor Studio 컴포넌트 간의 통신은 인증 방법에 따라 다릅니다.

**OutSystems 계정으로 인증하는 경우**

AI Mentor Studio 플러그인과 AI Mentor Studio SaaS 간의 통신은 항상 플러그인에 의해 시작됩니다. LifeTime 환경의 플러그인에서 AI Mentor Studio SaaS 엔드포인트로의 연결만 보장하면 되므로 연결 요구사항이 줄어듭니다.

> ![Diagram illustrating the communication flow initiated by the AI Mentor Studio plugin when using OutSystems account authentication.](https://success.outsystems.com/TK_Resource/90f9dd40-ca73-4271-a131-36bf1a187ab1)

**IT User 계정으로 인증하는 경우**

AI Mentor Studio가 로그인이 올바른지 확인하기 위해 인프라에 연결할 수 있어야 합니다. 사용자가 인증하거나 개인정보 보호 정책에 동의할 때 AI Mentor Studio SaaS가 AI Mentor Studio 플러그인과 통신해야 합니다. LifeTime probe에서 AI Mentor Studio SaaS 엔드포인트로 데이터를 전송하기 위해 플러그인이 SaaS와 통신해야 합니다. 따라서 IT User 인증을 사용할 때는 AI Mentor Studio SaaS와 플러그인 간의 통신이 **양방향**입니다.

> ![Diagram showing bidirectional communication between AI Mentor Studio SaaS and the AI Mentor Studio plugin when using IT User account authentication.](https://success.outsystems.com/TK_Resource/3a90cbaa-7cea-40d2-b77e-0b6932a4e071)

두 인증 방법 모두 플러그인이 forward proxy를 사용하여 AI Mentor Studio SaaS 엔드포인트에 연결할 수 있습니다.

### Data collected in plugin and sent to SaaS (수집 데이터)

AI Mentor Studio는 인프라에서 다음 데이터를 수집합니다.

- **Platform metamodel data**: 인프라 활성화 코드, 환경 정보(이름 및 Platform Server 버전), 팀, 앱 및 모듈 목록(이름 및 식별자 포함), 플랫폼 구성
- **모듈 및 의존성 정보**: 코드 분석용
- **동의 후 수집**: 사용자 정보(이름, 사용자명, 이메일 주소, 사용자 생성일, 마지막 로그인일) 및 LifeTime 권한
- **선택적**: 아키텍처 분석을 위한 Discovery 스냅샷 데이터(아키텍처 참조, 애플리케이션, 모듈)

### Data in transit (전송 중 데이터)

- 플러그인과 SaaS는 잘 알려진 HTTPS 엔드포인트를 통해 바이너리 형식으로 데이터를 공유합니다.
- IP 또는 DNS 주소는 전송되지 않습니다.
- AI Mentor Studio Probe의 올바른 사용을 위해 기본 포트 외에 추가로 열어야 하는 포트가 없습니다.
- 방화벽 문제가 발생하지 않아야 하지만, AI Mentor Studio 설정 방법에 상세히 설명된 엔드포인트에 접근할 수 있어야 합니다.

### Data at rest in SaaS (SaaS 저장 데이터)

각 설치 데이터는 OutSystems 클라우드에 보관되며 플랫폼의 멀티테넌트 메커니즘을 사용하여 다른 모든 설치와 격리됩니다. 이를 통해 한 설치의 데이터가 다른 설치의 사용자에게 접근되지 않습니다.

보안 및 규정 준수에 대한 자세한 내용은 다음 FAQ 섹션을 참고하세요.

- Security, legal and compliance - registration in AI Mentor Studio
- Security, legal and compliance - personal information

### Permissions (권한)

IT 사용자가 인프라와 함께 AI Mentor Studio를 사용할 때 갖는 권한은 해당 인프라의 코드 분석 환경에 대해 LifeTime에 설정된 역할과 권한에 따라 달라집니다.

IT 사용자가 코드 분석 환경에 대해 갖는 권한 세트는 할당된 역할의 권한과 역할이 할당되는 방식에 따라 결정됩니다.

역할과 해당 권한은 **기본 역할**, **팀에 대한 역할**, **앱에 대한 역할**로 할당될 수 있습니다.

- 팀에 할당된 역할의 권한은 기본 역할이 할당한 팀의 앱에 대한 권한보다 우선합니다.
- 앱에 할당된 역할의 권한은 기본 역할 및 팀에 할당된 역할이 특정 앱에 할당한 권한보다 우선합니다.

#### Main features permissions (주요 기능 권한)

**View teams (팀 보기)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | No           | Assigned team      | No                 |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    |              |                    |                    |
| Change and Deploy Applications | Team apps¹   | Assigned team      | No                 |
| Full Control                   |              |                    |                    |

¹ Team apps: 특정 앱의 권한 수준이 팀에 할당된 것보다 낮게 설정된 경우 제외

**View apps and modules (앱 및 모듈 보기)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | All apps¹    | Team apps²         | Assigned apps      |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    |              |                    |                    |
| Change and Deploy Applications |              |                    |                    |
| Full Control                   |              |                    |                    |

¹ All apps: 특정 앱(또는 해당 앱에 할당된 팀)의 권한 수준이 역할 또는 팀에 할당된 것보다 낮게 설정된 경우 제외
² Team apps: 특정 앱의 권한 수준이 팀에 할당된 것보다 낮게 설정된 경우 제외

**Open findings report (결과 보고서 열기)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | No           |                    |                    |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    | All apps¹    | Team apps²         | Assigned apps      |
| Change and Deploy Applications |              |                    |                    |
| Full Control                   |              |                    |                    |

**Export findings report (결과 보고서 내보내기)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | No           |                    |                    |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    | All apps¹    | Team apps²         | Assigned apps      |
| Change and Deploy Applications |              |                    |                    |
| Full Control                   |              |                    |                    |

**Resolve findings (결과 해결)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | No           |                    |                    |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    | No           |                    |                    |
| Change and Deploy Applications | All apps¹    | Team apps²         | Assigned apps      |
| Full Control                   |              |                    |                    |

**Overview dashboard (개요 대시보드)**

| LifeTime 권한                  | Default role | Assigned to a team | Assigned to an app |
| ------------------------------ | ------------ | ------------------ | ------------------ |
| No Access                      | No           |                    |                    |
| List Application               | No           |                    |                    |
| Monitor and Add Dependencies   |              |                    |                    |
| Open and Debug Applications    | All apps¹    | Team apps²         | Assigned apps      |
| Change and Deploy Applications |              |                    |                    |
| Full Control                   |              |                    |                    |

#### Maintenance and operations permissions (유지보수 및 운영 권한)

**Ignore modules (모듈 무시)**

| LifeTime 권한                        | Default role | Assigned to a team | Assigned to an app     |
| ------------------------------------ | ------------ | ------------------ | ---------------------- |
| Open and Debug Applications or lower | No           |                    |                        |
| Change and Deploy Applications       | All modules¹ | Team modules²      | Assigned app's modules |

¹ All apps: 특정 앱(또는 해당 앱에 할당된 팀)의 권한 수준이 역할 또는 팀에 할당된 것보다 낮게 설정된 경우 제외
² Team apps: 특정 앱의 권한 수준이 팀에 할당된 것보다 낮게 설정된 경우 제외

**Enable AI auto-classification (AI 자동 분류 활성화)**

| LifeTime 권한                        | Default role | Assigned to a team | Assigned to an app |
| ------------------------------------ | ------------ | ------------------ | ------------------ |
| Open and Debug Applications or lower | No           |                    |                    |
| Change and Deploy Applications       | Yes          | No                 |                    |
| Full Control                         |              |                    |                    |

**Override module AI auto-classification (모듈 AI 자동 분류 재정의)**

| LifeTime 권한                        | Default role | Assigned to a team | Assigned to an app |
| ------------------------------------ | ------------ | ------------------ | ------------------ |
| Open and Debug Applications or lower | No           |                    |                    |
| Change and Deploy Applications       | All modules¹ | No                 |                    |
| Full Control                         |              |                    |                    |

¹ All modules: 특정 모듈(또는 해당 모듈에 할당된 팀)의 권한 수준이 역할 또는 팀에 할당된 것보다 낮게 설정된 경우 제외

**Update probes (Probe 업데이트)**

| LifeTime 권한                        | Default role | Assigned to a team | Assigned to an app |
| ------------------------------------ | ------------ | ------------------ | ------------------ |
| Open and Debug Applications or lower | No           |                    |                    |
| Change and Deploy Applications       | Yes          | No                 |                    |
| Full Control                         |              |                    |                    |

**Manage AI Mentor Studio API**

| LifeTime 권한                        | Default role | Assigned to a team | Assigned to an app |
| ------------------------------------ | ------------ | ------------------ | ------------------ |
| Open and Debug Applications or lower | No           |                    |                    |
| Change and Deploy Applications       | Yes          | No                 |                    |
| Full Control                         |              |                    |                    |
