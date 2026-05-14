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

---

## 1. The Architecture Canvas

Architecture Canvas는 OutSystems에서 **SOA(서비스 지향 아키텍처)** 를 쉽게 설계하기 위한 도구입니다. 재사용 가능한 (마이크로)서비스를 올바르게 추상화하고, 기능적으로 구분되는 모듈들이 각자 독립적인 라이프사이클을 유지하도록 돕습니다.

중대형 OutSystems 환경에서는 20개 이상의 mission-critical 앱, 200개 이상의 상호의존 모듈이 생깁니다. 잘 설계된 아키텍처는 앱과 모듈이 독립적으로 진화할 수 있게 해주며, 의존성과 변경 영향을 최소화합니다. 결과적으로 비용 효율이 높고 유지보수하기 쉬운 구조가 됩니다.

#### 레이어 및 서브 레이어 (Layers and sub layers)

각 레이어와 서브 레이어는 모듈에 담을 기능의 성격을 구분합니다. 애플리케이션은 서로 다른 팀이 개발·유지하며, 새로운 앱은 빠르게 진화하고 이미 많이 사용되는 서비스는 훨씬 천천히 변경됩니다.

> ![Diagram of the OutSystems Architecture Canvas showing different layers including Orchestration Modules, End-user Modules, Core Modules, and Foundation Modules.](https://success.outsystems.com/TK_Resource/105b1fae-c75a-408f-a9b8-c40fb6916fc4)

> ⚠️ **Orchestration 레이어는 O11에서 제거되었습니다.** O10까지는 서로 다른 앱의 화면 간 hyperlink를 strong dependency로 처리했으나, O11에서는 weak reference로 변경되어 Orchestration 레이어가 더 이상 필요하지 않습니다.

#### 서브 레이어 상세 (Sub-layers)

각 항목의 'external' 표기는 해당 서브 레이어의 모듈이 외부에 서비스로 노출될 수 있음을 의미합니다.

> ![Detailed diagram of the OutSystems Architecture Canvas sub layers, including Orchestration, Enduser, Core, and Foundation with their respective external modules.](https://success.outsystems.com/TK_Resource/a0ac451c-e9a4-463f-a500-0f926ff7b431)

| 레이어     | 서브 레이어        | 설명                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| End-user   | Orchestration (*)  | 여러 frontend를 mashup, cross-app 워크플로 (* O11 이전만 해당) |
| End-user   | End-user           | Frontend UI 모듈                                             |
| Core       | Core               | 비즈니스 개념 중심 재사용 서비스 — 엔티티, 비즈니스 규칙, Web blocks 노출 |
| Core       | API                | Core Services를 외부에 API로 노출                            |
| Core       | Core widgets       | 재사용 UI 위젯                                               |
| Core       | Composite logic    | 재사용 비즈니스 로직 조합 또는 데이터 동기화 로직            |
| Core       | Core service       | 재사용 Core 서비스                                           |
| Foundation | Foundation         | 비기능 요구사항 및 통합 모듈 — 모든 비즈니스에서 재사용 가능 |
| Foundation | Style guide        | 재사용 UI 패턴, 테마, 템플릿                                 |
| Foundation | Foundation service | 외부 서비스 연동 wrapping, NFR 지원 (예: Audit logging)      |
| Foundation | Library            | 재사용 라이브러리 및 plug-in                                 |

#### Architecture Canvas 활용 방법

Canvas는 아키텍처 설계의 두 단계에서 사용합니다.

**1. 개념 식별 (Identify concepts)**
기능적·비기능적·통합 요구사항을 구조적으로 수집합니다. 예: 'Customer Management', 'Inventory', 'ERP 연동' 같은 개념들을 Canvas 위에 배치해 어느 레이어에 속하는지 정리합니다.

![Illustration of the Architecture Canvas used for identifying concepts such as User Processes, Business Concepts, and Utilities and Integration.](https://success.outsystems.com/TK_Resource/cd465952-0a3e-4d00-894a-d7d35e4ad7fa)

**2. 모듈 정의 (Define modules)**
식별된 개념들을 권장 패턴에 따라 실제 모듈로 설계합니다.

> ![Graphic representing the iterative process of architecture design, with two steps: 1. Identify Concepts and 2. Define Modules.](https://success.outsystems.com/TK_Resource/4c071788-f0a8-43bf-8bc7-163d4849ef77)

> 이 두 단계는 **반복(iteration)** 이 핵심입니다. 구현 중에 새로운 개념이 나타나면 다시 돌아가서 재설계합니다.

---

## 2. Translating business concepts into application modules

개념을 식별한 후에는 그것을 애플리케이션 모듈로 변환해야 합니다. 이때 다음 두 가지 사이에서 **올바른 균형**을 찾는 것이 핵심입니다.

- 너무 많은 모듈을 다뤄야 하는 상황을 만들지 않을 것
- 각 모듈마다 관리 가능한 복잡도와 독립적인 라이프사이클을 유지할 것

#### Finding the right concept granularity (올바른 개념 단위 찾기)

Architecture Canvas 방식으로 식별된 개념 예시입니다.

> ![Diagram illustrating the Architecture Canvas method with concepts categorized into User processes, Business concepts, and Utilities and integration.](https://success.outsystems.com/TK_Resource/7f0b37d7-942b-4651-aae4-cee23f0b43a2)

이 개념들을 모듈로 구현할 때 몇 개의 모듈이 필요한지는 **개념의 단위(granularity)** 에 달려 있습니다. 아키텍처를 정의할 때 개념을 아주 낮은 수준까지 세분화할 필요는 없지만, 올바른 granularity는 아래 세 가지 기준에 따라 결정됩니다.

**기준 1 — Complexity (복잡도)**
개념이 너무 방대해서 하나의 모듈에 담기 어려울 수 있습니다. 예를 들어 'Customer Management'라는 사용자 프로세스는 (1) 신규 고객 등록, (2) 고객 트렌드 분석, (3) 고객에게 서비스 추가 등 여러 sub-process를 포함할 수 있습니다. 이럴 때는 각 모듈의 복잡도를 관리 가능한 수준으로 유지하기 위해 여러 모듈로 분리합니다.

**기준 2 — Independent life cycles (독립적인 라이프사이클)**
모든 sub-concept을 하나의 모듈에 담아도 복잡하지 않을 것 같더라도, 기능을 독립적으로 관리해야 한다면 모듈을 쪼개야 합니다. 이렇게 하면 (1) 서로 다른 개발자들이 병렬로 개발할 수 있고, (2) 비즈니스 스폰서별로 서로 다른 발전 속도를 유지할 수 있습니다.

**기준 3 — Service isolation (서비스 격리)**
서비스를 올바르게 추상화하기 위한 여러 통합 패턴(integration patterns)이 있으며, 이에 따라 하나의 개념을 여러 기술 컴포넌트로 분리해야 하는 경우가 있습니다. 시나리오에 따라 개념을 어떻게 나눌지가 결정됩니다.

#### Validating your architecture

개념을 모듈로 정리하는 방식이 Architecture Canvas 권고사항에 맞는지 확인하려면 **validate your application architecture** 문서를 통해 검증 방법을 확인하세요.

---

## 3. Validating your application architecture

잘 설계된 아키텍처를 위해 **반드시** 지켜야 할 3가지 규칙이 있습니다. Discovery tool을 통해 모듈 간 의존성을 자동으로 분석하고, 위반 사항과 잘못 배치된 요소(액션, 화면, 엔티티)를 찾아낼 수 있습니다.

> 📌 **[다이어그램 삽입: 규칙 개요 구조도]**

#### Rule #1 — No upward references (상향 참조 금지)

상향 참조란 두 모듈이 직접 또는 간접적으로 순환 의존성을 만드는 것입니다.

예를 들어 **Library E**가 **End-user A**를 소비하면, 클러스터 내 어떤 쌍이든 순환 관계가 됩니다. C와 D를 예로 들면: `C → E → A → D`, 반대로 `D → E → A → C`.

> 📌 **[다이어그램 삽입: Rule #1 다이어그램]**

이 상황에서 **End-user B**는 정당하게 **Core D**를 소비하지만, 전체 클러스터에 종속되어버립니다. 런타임에서 불필요하게 큰 footprint를 갖게 되고, 본인은 인식조차 못한 모듈의 변경에 계속 영향을 받게 됩니다.

**무엇이 문제인가?** 서비스가 제대로 격리되지 않았음을 나타냅니다.

**해결 방법:** 소비되고 있는 요소들을 더 낮은 레이어로 이동시킵니다. A에서 재사용되는 서비스를 **Library**로 이동시키면 됩니다.

#### Rule #2 — No side references among End-users or Orchestrations (횡단 참조 금지)

예시에서 **End-user A**가 **End-user B**의 어떤 요소를 소비합니다. A가 B에 직접 연결된 것뿐 아니라 **D, E, G**까지 간접적으로 상속하게 됩니다.

> 📌 **[다이어그램 삽입: Rule #2 다이어그램]**

**무엇이 문제인가?** End-user 또는 Orchestration 모듈은 재사용 가능한 서비스를 제공해서는 안 됩니다. End-user/Orchestration은 계층 최상단에 위치하기 때문에, 이 모듈을 참조하면 하위 레이어의 간접 의존성을 대량으로 끌어들이게 됩니다.

**해결 방법:** B에서 A가 소비하는 요소들을 찾아 더 낮은 레이어 모듈로 이동시킵니다.

- 비즈니스 관련이라면 → **Core 모듈**로
- 비즈니스와 무관하다면 → **Library**로

#### Rule #3 — No cycles among Cores or Libraries (순환 참조 금지)

Rule #1, #2를 지키면 End-user와 Orchestration은 어떤 순환 참조에도 연루될 수 없습니다. Rule #3은 **Core 또는 Library 간의 순환**을 피하는 것에 관한 규칙입니다.

> 📌 **[다이어그램 삽입: Rule #3 다이어그램]**

순환은 언제나 바람직하지 않습니다. 예상치 못한 영향을 주고 유지보수하기 어려운 코드를 만들기 때문입니다.

**무엇이 문제인가?** 모듈 간 순환은 개념이 올바르게 추상화되지 않았음을 나타냅니다. A와 B 사이의 순환은 다음 중 하나를 의미합니다.

- 두 모듈이 강하게 결합(strongly coupled)되어 있거나
- 한 방향이 바람직하지 않은 것

**해결 방법:** 관계에서 바람직하지 않은 방향의 요소를 다른 모듈로 이동시킵니다.

- 또 다른 재사용 가능한 개념을 나타낸다면 → **새 모듈**로 이동
- 동일한 개념의 일부라면 → **B 자체**로 이동

A와 B가 너무 강하게 결합되어 있다면 **합치는 것(merge)** 을 고려합니다. 합쳤을 때 모듈이 너무 커진다면, 두 기본 개념 위에 세 번째 모듈을 만드는 것이 필요할 수 있습니다. 이 조합 로직이 순환을 유발하는 것입니다.

---

## 4. Service-oriented architectures for OutSystems applications

OutSystems Services로 개념을 추상화할 때, 서비스의 Core가 다음 조건을 만족해야 합니다.

- 내부 및 외부 소비자 모두에게 **공통적**일 것
- 에코시스템 변화에 대응하기 위해 **모든 외부 시스템으로부터 격리**될 것
- **deprecated 예정인 레거시 시스템을 대체**할 준비가 되어 있을 것

#### Implementing your Service-Oriented Architecture

서비스를 구성하는 3개 레벨이 있으며, **Core Services가 외부 시스템으로부터 올바르게 격리**되는 것이 매우 중요합니다.

- The External API
- The Core Services
- The Integration Services

> 📌 **[다이어그램 삽입: SOA 전체 구조도]**

#### External API level

외부 소비자에게 서비스를 노출하는 API들의 집합입니다(REST, SOAP 등).

Core Services 레이어에서 실제 구현된 서비스의 **기술적 wrapper**로, 소비자와 합의된 API 형태로 변환하는 역할만 합니다.

> ⚠️ **이 레벨에는 비즈니스 로직을 추가해서는 안 됩니다.**

여러 버전의 API를 이 레벨에 유지해 소비자 스펙 차이에 대응할 수 있으며, Integration audit을 추가해 외부 소비를 추적할 수 있습니다.

#### Core Services level

모든 비즈니스 규칙과 핵심 엔티티를 포함하는 **서비스의 실제 구현** 레벨입니다. Core Services는 **시스템에 무관(system-agnostic)** 해야 합니다.

- **External APIs**: 외부 소비자 변경이 Core Services에 영향을 주지 않아야 합니다.
- **Integration Services**: 외부 producer 변경·교체 시에도 Core Services는 영향을 받지 않아야 합니다.

레거시 시스템 통합 시 Integration Services를 통해 데이터를 OutSystems entities(로컬 캐시)로 동기화하면 다음이 가능합니다.

- 성능 문제 대응
- Core Services를 외부 부하로부터 보호
- 레거시 시스템 온라인 호출 없이 비즈니스 로직 지원
- 향후 레거시 시스템을 Core Service 소비자에게 영향 없이 deprecated 처리

#### Integration Services

외부 producer를 기술적으로 wrapping하여 데이터 구조를 정규화하고, 통합 복잡성을 추상화하는 레벨입니다.

> ⚠️ **이 레벨에도 비즈니스 로직을 추가해서는 안 됩니다.**

예: Customer 정보가 CRM과 ERP 두 시스템에 분산된 경우, **Customer_IS**가 두 시스템을 통합해 정규화된 데이터를 제공합니다. Core Service는 정보 출처를 알 필요가 없으므로 외부 시스템을 자유롭게 변경할 수 있습니다.

#### Service design patterns

**Integration Patterns for Core Services Abstraction** 문서 참고.

---

## 5. Integration Patterns for Core Services Abstraction

#### 왜 통합 패턴이 필요한가?

세 가지 독립적인 요구사항이 있습니다.

- **업무 독립성(Business Independence)**: 외부 시스템의 변경이 OutSystems 앱에 영향을 주지 않아야 합니다.
- **확장성, 성능, 가용성**: 외부 시스템의 성능·가용성 문제가 앱에 영향을 주지 않아야 합니다.
- **감사 및 모니터링**: 신뢰할 수 있는 통합을 위한 감사 및 모니터링이 가능해야 합니다.

> 📌 **[다이어그램 삽입: 통합 개요 다이어그램]**

#### Integration Services — 외부 시스템을 내부 시스템처럼 wrapping

> 📌 **[다이어그램 삽입: Integration Services 구조표]**

Integration Services는 다음 네 가지 역할을 담당합니다.

- **Abstract the data source** — 데이터 소스를 추상화
- **Synchronize data to OutSystems** — 데이터를 OutSystems로 동기화
- **Act as cache to core service API** — Core Service API의 캐시 역할
- **Error handling** — 오류 처리

#### 데이터 필요에 따른 통합 패턴

패턴 선택은 데이터 요구사항에 따라 결정됩니다.

> 📌 **[다이어그램 삽입: 패턴 선택 플로우차트]**

#### Direct Integration (직접 통합)

외부 시스템을 실시간으로 직접 호출하는 가장 단순한 패턴입니다.

> 📌 **[다이어그램 삽입: Direct Integration 구조]**

**장점:** 항상 최신 데이터, 구현이 단순  
**단점:** 외부 시스템 장애 시 서비스 중단, 성능이 외부 시스템에 종속  
**사용 시점:** 데이터를 퍼블릭하게 읽을 필요가 없고, 데이터 크기가 작을 때

#### Cold Cache (Summary Data)

외부 시스템 데이터를 OutSystems 엔티티에 주기적으로 동기화해두는 패턴입니다.

> 📌 **[다이어그램 삽입: Cold Cache 구조]**

**장점:** 외부 시스템 장애에 독립적, 성능 향상  
**단점:** 데이터가 실시간이 아님 (배치 주기만큼 지연)  
**사용 시점:** 데이터를 퍼블릭하게 읽어야 하고, 실시간성이 중요하지 않을 때

#### Gateway synchronization logic

외부 시스템에서 변경이 발생할 때 OutSystems로 데이터를 push하거나, OutSystems가 주기적으로 변경분(delta)을 pull하는 패턴입니다.

> 📌 **[다이어그램 삽입: Gateway Sync 구조]**

**장점:** 외부 시스템 장애에 독립적, Cold Cache보다 데이터 신선도 향상

#### Distinct real-time sync

IT 요청이 있을 때마다 즉시 동기화하는 패턴입니다. 특정 레코드에 대한 요청 시 실시간으로 외부 시스템과 동기화합니다.

> 📌 **[다이어그램 삽입: Distinct real-time sync 구조]**

#### Ordered real-time sync

메시지 순서를 보장하면서 실시간으로 동기화하는 패턴입니다.

> 📌 **[다이어그램 삽입: Ordered real-time sync 구조]**

**장점:** 메시지 처리 순서 보장, 비동기 처리로 외부 시스템 장애에 독립적  
**단점:** 구현 복잡도 높음  
**사용 시점:** 데이터 처리 순서가 중요하고, 대용량 메시지를 처리해야 할 때

#### 최종 정리

> 📌 **[다이어그램 삽입: 전체 패턴 비교 구조]**

> 📌 **[다이어그램 삽입: 패턴별 비교표]**

---

## 6. Application composition

---

### 6-1. Applying the Architecture Canvas to applications

모듈 아키텍처와 마찬가지로 애플리케이션도 Architecture Canvas 원칙을 따릅니다. 각 애플리케이션은 레이어 중 하나에 배치되며, 애플리케이션의 최상위 모듈의 성격이 해당 애플리케이션의 성격을 결정합니다.

> 📌 **[다이어그램 삽입: 애플리케이션 유형 4종]**

애플리케이션 유형:

- **Orchestration Applications** — Orchestration 레이어 (O10 이하 버전만 해당)
- **End-User Applications** — End-User 레이어
- **Core Applications** — Core 레이어
- **Library Applications** — Foundation 레이어

애플리케이션에도 모듈과 동일한 검증 규칙이 적용됩니다.

1. No upward references
2. No side references among End-User applications or Orchestration applications
3. No cyclic references between two applications

#### Evolving your architecture along with your applications

**First project**

처음에는 보통 하나의 애플리케이션으로 시작합니다.

> 📌 **[다이어그램 삽입: 1st Project 구조]**

**Second project**

두 번째 유사 프로젝트에서 다른 비즈니스 프로세스를 위한 애플리케이션이 추가되고, Core 및 Library 모듈이 몇 개 더 생깁니다.

> 📌 **[다이어그램 삽입: 2nd Project 구조]**

**Third project**

세 번째 발전 단계에서 두 End-user 애플리케이션이 서로 side reference를 가지게 됩니다 — 실제로는 순환(cycle)입니다.

> 📌 **[다이어그램 삽입: 3rd Project 구조 (교차 참조)]**

이는 두 애플리케이션이 강하게 결합되어 있음을 명확히 보여줍니다. 첫 번째 애플리케이션의 새 버전을 Production에 배포하려면 두 번째 애플리케이션도 함께 가져가야 합니다.

**Solving the issues**

공통으로 재사용되는 리소스는 새로운 **Core Application**으로 분리해야 합니다.

> 📌 **[다이어그램 삽입: 해결 구조]**

Core B와 Core C뿐만 아니라, 직접 참조되는 의존성(Library B, Library C)도 이 Core Application으로 이동해야 합니다. 이렇게 하면 두 애플리케이션이 서로 다른 속도로 독립적으로 발전할 수 있습니다.

---

### 6-2. The 4 Rules for Correct Application Composition

애플리케이션을 분리할 필요성은 재사용 요건이 명확해진 후에 항상 고려해야 합니다. 애플리케이션을 미리 작게 쪼개는 것이 항상 옳지는 않습니다.

4가지 규칙의 목적:

- 의존성 최소화
- 배포 단순화
- 라이프사이클 독립성 촉진

**Rule #1 — Correctly layer your modules**
Architecture Canvas 원칙에 따라 모듈을 올바르게 레이어링합니다. 모듈 아키텍처에 위반이 있으면 올바르고 관리 가능한 애플리케이션 아키텍처를 갖추는 것이 불가능합니다.

**Rule #2 — Correctly layer your applications**
Architecture Canvas 원칙을 모듈 대신 애플리케이션에 적용합니다.

**Rule #3 — Don't mix different owners**

서로 다른 팀이 동시에 여러 프로젝트를 개발할 때, 하나의 애플리케이션에 소유자가 두 명이 되면 배포 관리가 복잡해지고 변경 책임이 불명확해집니다.

> 📌 **[다이어그램 삽입: Rule #3 소유자 분리 구조]**

소유권을 집중하는 것이 핵심입니다. 하나의 애플리케이션 소유권을 집중할 수 없다면, 소유권이 명확하게 정의되도록 분리하는 것을 고려해야 합니다.

**Rule #4 — Don't mix different sponsors**

프로젝트가 여러 스폰서에 영향을 미친다면 각 LOB(Line Of Business)를 별도 애플리케이션으로 분리하는 것이 중요합니다. 스폰서마다 예산, 요건, 변경 속도가 다릅니다.

> 📌 **[다이어그램 삽입: Rule #4 LOB 분리 구조]**

LOB별로 애플리케이션을 미리 분리하면 각각 독립적으로 발전할 수 있습니다.

---

### 6-3. Isolating an application Theme

엔터프라이즈 설치 환경에서 애플리케이션들은 보통 동일한 corporate look & feel을 공유합니다. Theme 모듈은 다음 컴포넌트를 포함합니다.

- Theme 자체 (CSS)
- Layout Blocks — 화면 또는 화면 섹션 패턴 정의
- Logo 이미지
- Login flow 및 exception handling
- Menu 및 navigation 지원 로직
- Transversal roles — Manager, Employee 같은 모든 앱에서 재사용 가능한 역할

> ⚠️ Theme 모듈은 global look & feel 요소로만 제한해야 하며, 절대로 assets 및 utilities의 글로벌 저장소로 사용해서는 안 됩니다.

#### Same look and feel, menu and login flow

여러 애플리케이션이 공통 메뉴와 login flow를 포함한 Global Theme을 재사용하는 경우입니다.

> 📌 **[다이어그램 삽입: 공통 메뉴/login flow 구조]**

Block은 소비 모듈의 CSS를 상속하므로 최종 애플리케이션의 look & feel에 맞게 자동으로 적응합니다.

#### Same look and feel, different menus and login flows

애플리케이션들이 서로 독립적이고 global look & feel만 공유하면 된다면, 각 애플리케이션이 자체 theme 모듈을 가져야 합니다.

> 📌 **[다이어그램 삽입: 개별 메뉴/login flow 구조]**

이 경우 각 애플리케이션 theme은 공통 global theme을 상속하되, 자체 menu 및/또는 login flow를 정의합니다.

---

## 7. Designing Scalable Multi-Tenant Applications

OutSystems를 사용하면 단일 소프트웨어 인스턴스로 여러 조직(테넌트)을 서비스할 수 있습니다. 각 테넌트의 데이터는 서로 분리되어 있으며, 단일 애플리케이션으로 관리됩니다.

#### Logical Segregation of Tenants

논리적 테넌트 격리를 구현할 때 두 가지 문제를 고려해야 합니다.

- **Data Segregation**: 테넌트 데이터를 단일 엔티티에 속하도록 격리
- **Cross-Tenant Access**: 파트 사용자가 여러 테넌트 데이터에 접근하는 경우

> 📌 **[다이어그램 삽입: 테넌트 격리 구조]**

**장점:**

- OutSystems에서 구독의 일환으로 완전히 지원됨
- 인프라 운영 비용 절감
- 데이터, 사용자, 세션, 프로세스에 대한 단일 보안 정책
- 테넌트별 쿼리를 작성할 필요가 없어 애플리케이션 개발 속도 향상

**단점:**

- 모든 테넌트 데이터가 동일한 테이블에 있어 데이터 격리 수준이 낮음
- 테넌트당 행 수가 많아질 수 있음 → 성능 최적화를 위해 archiving 전략 적용 필요

#### Ensuring Multi-Tenancy Scalability

확장성을 위해 다음을 권장합니다.

- **Fully multi-tenant** 방식으로 애플리케이션을 개발 — 인프라 변경에 대한 유연성 확보
- **Use page mechanisms** — 테이블에 저장된 정보량 최소화
- **Scale the database vertically**

#### When the Database Reaches the Maximum Vertical Capability

**Scenario 1: Multiple Production Environments**

테넌트별로 별도의 프로덕션 인프라를 사용합니다.

> 📌 **[다이어그램 삽입: Multiple Production Environments 구조]**

이전 프로덕션 데이터를 새 환경으로 마이그레이션하는 3가지 방법:

- IT 도구 활용 (Attunity, Informatica, Information Builders 등)
- System Integration 개발
- Backup restore

**Scenario 2: Manually Partitioning OutSystems Entities**

데이터를 파티셔닝하기 위한 자체 메커니즘을 사용하되, 메타모델은 수동으로 업데이트해야 합니다.

#### Physical Segregation of Tenants

각 테넌트가 자체 데이터베이스를 갖는 물리적 격리 방식입니다.

> 📌 **[다이어그램 삽입: Physical Segregation 구조]**

**장점:** 데이터 격리 수준 높음, 테이블당 행 수가 적어 쿼리 속도 최적화  
**단점:** 테넌트마다 추가 리소스 필요, 개발 속도 저하, 유지보수 복잡

#### Recommendation

**논리적 격리(Logical Segregation)를 기본 옵션으로 권장**합니다. 추가 장점:

- 짧은 개발 주기 — 테이블 생성 및 업데이트가 IDE에서 바로 처리
- 단일 코드 베이스로 애플리케이션 단순화
- OutSystems가 애플리케이션 개발, 배포, 지원 전반에서 더 잘 지원

---

## 8. Microservices Architecture in OutSystems

마이크로서비스 접근 방식에서 아키텍처는 독립적으로 배포 가능한 작은 서비스들로 구성됩니다. 각 서비스는 자체 용량으로 실행되며, 경량 메커니즘으로 통신합니다.

> 📌 **[다이어그램 삽입: 마이크로서비스 기본 구조도]**

마이크로서비스 패러다임에서는:

- 독립적인 배포 가능한 서비스를 중심으로 팀을 구성
- 각 서비스는 독립적으로 확장 가능
- 서로 다른 언어/기술을 사용해 개별 서비스 구현 가능
- 최소한의 중앙 집중식 관리

> 📌 **[다이어그램 삽입: 3계층 분리 구조]**

#### Multiple Microservices Infrastructures

> 📌 **[다이어그램 삽입: Multiple Infrastructures 구조]**

각 OutSystems 인프라는 독립적으로 배포 가능합니다. 마이크로서비스당 하나의 OutSystems 환경을 사용할 필요 없이 여러 마이크로서비스를 동일한 인프라에서 호스팅할 수 있습니다.

#### Central Services for Multiple Substrates

> 📌 **[다이어그램 삽입: Central Services 구조]**

#### Microservices Lifecycle — Managing Versions

- **Add new action**: 새 모듈에 새 액션을 추가하고 모든 소비자에게 배포
- **Change current API version**: 기존 모듈을 변경하고 모든 소비자에게 배포
- **Create new API version**: 새 서비스 메서드를 추가하되 모든 소비자가 평가할 수 있도록 유지

> 📌 **[다이어그램 삽입: Lifecycle 버전 관리 구조]**

#### Infrastructure Scenarios

> 📌 **[다이어그램 삽입: Infrastructure Scenarios 구조]**

---

## 9. From architecture to development

이 아티클은 OutSystems에서 기술적 기반을 마련하고 팀 기술 수립을 위한 실용적인 가이드를 제공합니다. Service-oriented 접근 방식을 기반으로 합니다.

### Architecture blueprint design

#### Requirements

앱 설계 시 필요한 것:

- 팀 역량, 비즈니스 목표, 마감일, 예산 등을 파악하는 요구사항 정의
- 인프라 리뷰
- 기존 아키텍처 레퍼런스 수집

#### Architecture design process

**[Discover → Organize → Assemble]** 3단계 순환 프로세스입니다.

> 📌 **[다이어그램 삽입: Architecture Design Process (원형)]**

> 📌 **[다이어그램 삽입: Architecture Design Process (퍼즐)]**

**Discover (business concepts and integration needs)**

요구사항 수집 단계입니다.

- 기능 요구사항: 어떤 앱/화면/비즈니스 프로세스가 필요한지
- 비기능 요구사항: 사용자 수, 가용성 수준

**Organize (concepts on Architecture Canvas)**

> 📌 **[다이어그램 삽입: Architecture Canvas 레이어별 모듈 배치 표]**

**Assemble (according to recommended patterns)**

> 📌 **[다이어그램 삽입: Blueprint 예시]**

중요한 컴포넌트:

- **Bootstrapper**: 앱 설치 시 Master Data를 로드
- **Backoffice**: 비즈니스 사용자가 앱 구성을 관리
- **Themes**: 비즈니스 앱의 look & feel을 정의
- **Field Composites**: 다른 앱과 공유하는 재사용 가능한 컴포넌트

### Integration patterns — external master entities

**Batch sync 패턴**

> 📌 **[다이어그램 삽입: Batch sync 패턴]**

**장점:** 더 단순한 통합 API, 소스 시스템에 대한 영향 최소화, 동기화가 느려도 무방

**Real-time sync 패턴**

> 📌 **[다이어그램 삽입: Real-time sync 패턴]**

### Application composition

> 📌 **[다이어그램 삽입: Application composition 구조]**

> 📌 **[다이어그램 삽입: 최종 애플리케이션 구성도]**

애플리케이션은 레이어별로 구성됩니다. 각 애플리케이션은 동일한 레이어의 다른 애플리케이션을 참조할 수 있으며, Core 및 Foundation 앱은 다른 앱에 의해 소비될 수 있습니다.

---

### 9-1. Developing from the architecture blueprint

#### Blueprint 검토

> 📌 **[다이어그램 삽입: Blueprint 검토]**

아키텍처 Blueprint를 검토할 때:

- 각 모듈의 책임 범위 확인
- 모듈 간 의존성 방향 확인 (상향 참조 없는지)
- Core Service 소비자(CS Consumer)가 올바른 레이어에 있는지 확인

#### Identify real-life dependencies

> 📌 **[다이어그램 삽입: 실제 의존성 파악]**

실제 의존성을 파악할 때:

- 각 모듈이 실제로 무엇을 소비하는지 확인
- **굵은 선**: 강한 의존성 (변경 시 소비자도 영향받음)
- **점선**: 약한 의존성

#### Creating the modules and applications

> 📌 **[다이어그램 삽입: 모듈 및 애플리케이션 생성]**

- **Root-user application** 생성
- **Core application** 생성
- **Foundation application** 생성 및 모듈 구성

> 📌 **[다이어그램 삽입: 전체 애플리케이션 flow]**

---

*계속 업데이트 중...*
