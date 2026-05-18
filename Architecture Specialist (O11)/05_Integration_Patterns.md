# OutSystems Architecture Specialist (O11)
## Integration Patterns for Core Services Abstraction
1단원 벼락치기 정리노트

> 🌐 **스타일 버전 (색상/다이어그램 완전 재현):**
> [htmlpreview로 보기](https://htmlpreview.github.io/?https://github.com/kckworld/outsystems_study/blob/main/Architecture%20Specialist%20(O11)/05_Integration_Patterns.html)

---

> **원장님 핵심 결론**
> 이 단원은 외부 시스템(ERP, CRM 등)을 OutSystems Core Service에서 직접 알지 못하게 숨기고, Integration Service가 외부 API/프로토콜/인증/오류 처리를 감싸도록 만드는 설계 패턴입니다. 시험에서는 "누가 외부 시스템을 알아야 하는가?", "Integration Service를 시스템별로 만들지 개념별로 만들지", "캐시/동기화/직접연계 중 무엇을 선택할지"를 묻습니다.

---

## 1. 원문 문장과 한글 해석

| English sentence | Korean meaning |
|---|---|
| Why do you need integration patterns? | 왜 Integration Pattern이 필요한가? |
| System independence | 외부 시스템이 바뀌어도 내부 Core 개념과 Consumer에 주는 영향을 줄이는 독립성 |
| Extensibility, normalization, and optimization | 추가 정보/비즈니스 규칙 확장, 표현 표준화, 캐시를 통한 성능 개선 |
| Integration Service per independent concept, not a single integration point per external system | 외부 시스템별 단일 연계점이 아니라, 독립적인 업무 개념별 Integration Service를 둔다 |
| Consumers of the core service always get the same type of abstraction | Core Service 소비자는 외부 시스템 종류와 무관하게 동일한 추상화 계층만 본다 |

---

## 2. 왜 Integration Pattern이 필요한가?

자료의 시작 질문은 "Why do you need integration patterns?"입니다. 외부 system of record에서 마스터 데이터를 조회하거나 트랜잭션을 수행해야 할 때, 처음부터 패턴을 정해야 나중에 시스템 변경, 성능 문제, 중복 구현을 줄일 수 있습니다.

| 필요성 | 설명 | 시험 키워드 |
|---|---|---|
| System independence | ERP/CRM 같은 외부 시스템이 바뀌어도 Core Service와 화면 앱이 크게 흔들리지 않게 한다. | resilient to external changes |
| Extensibility | 외부 데이터에 내부 비즈니스 규칙이나 보완 정보를 더할 수 있다. | extend the concept |
| Normalization | 외부 시스템마다 다른 구조를 내부 표준 구조로 변환한다. | normalize representations |
| Optimization | 필요한 경우 캐시/동기화를 사용해 성능을 개선한다. | cache information |

> ⚠️ 시험 함정: "외부 시스템이 ERP 하나니까 End-user 앱에서 ERP API를 직접 호출하면 된다"는 식의 답은 위험합니다. 이 단원은 외부 연계를 Integration Service로 감싸고, Core Service 소비자는 외부 시스템을 몰라야 한다는 방향입니다.

---

## 3. 그림 1 - Core Service와 Integration Service

```mermaid
graph TD
  Consumer["Consumer\n(End-user / Other Core)"]
  CS["① Customer_CS\nCore Service\n(System agnostic)"]
  IS["② Customer_IS\nIntegration Service\n(API 추상화 / 기술 요구사항 처리)"]
  ERP[("ERP")]
  CRM[("CRM")]

  Consumer --> CS
  CS --> IS
  IS --> ERP
  IS --> CRM
```

*그림 1. Customer_CS(Core Service) → Customer_IS(Integration Service) → ERP/CRM 구조*

그림은 Customer라는 독립적인 업무 개념을 기준으로 Customer_CS와 Customer_IS를 분리합니다. Customer_CS는 Core Service이고, Customer_IS는 Integration Service입니다.

| 구성요소 | 역할 | 시험 판단 |
|---|---|---|
| Customer_CS | Core Service. Customer라는 업무 개념을 내부에 안정적으로 제공한다. | Consumer가 사용하는 추상화 계층 |
| Customer_IS | Integration Service. ERP/CRM API를 감싸고 내부 구조에 맞게 변환한다. | 외부 시스템 복잡성을 숨김 |
| ERP / CRM | 실제 System of Record 또는 외부 시스템 | Core/End-user가 직접 알아야 할 대상이 아님 |

| 번호 | 영어 원문 요지 | 한글 설명 |
|---|---|---|
| 1 - Core Service | Extend the original service / System agnostic | Core Service는 원래 서비스를 확장하고, 특정 외부 시스템에 종속되지 않아야 합니다. |
| 2 - Integration Service | Abstracts the original API(s) | Integration Service는 원 API를 감싸고, 내부 구조와 개념에 맞게 매핑합니다. |
| 2 - Integration Service | Hide the integration type | REST, SOAP, HTTP request, file upload 등 세부 연계 방식을 숨깁니다. |
| 2 - Integration Service | Cope with technical requirements | 인증, 권한, 에러 처리, 감사 로그 같은 기술 요구사항을 처리합니다. |

---

## 4. 핵심 원칙 - 시스템별이 아니라 개념별 Integration Service

자료의 가장 중요한 문장은 "Integration Service per independent concept, not a single integration point per external system"입니다.

> *영어 시험 문장: The idea is to have an Integration Service per independent concept and not a single integration point per external system.*

한글 해석: 외부 시스템마다 하나의 연계 모듈을 만드는 것이 아니라, Customer, Product, Order 같은 독립적인 업무 개념별로 Integration Service를 만든다는 뜻입니다.

| 나쁜 설계 | 좋은 설계 |
|---|---|
| ERP_IS 하나에 Customer, Product, Order를 모두 넣음 | Customer_IS, Product_IS, Order_IS처럼 개념별 분리 |
| Consumer가 ERP/CRM 차이를 알아야 함 | Consumer는 Customer_CS만 보고 외부 시스템을 모름 |
| Products 변경이 Customers 소비자에게 영향 가능 | 각 개념이 독립 lifecycle을 가짐 |

---

## 5. 그림 2 - Integration Wrapper Canvas

```mermaid
flowchart TD
  A["Validation / Security logic\n입력값 일관성 확인, 사용자 권한 확인"]
  B["Authentication logic\n외부 API 호출에 필요한 인증 정보 준비"]
  C["Map inputs\n내부 입력을 외부 API 형식으로 변환"]
  D["Call and audit the external API\n외부 API 호출 및 감사 기록"]
  E["Normalize error handling\n외부 오류 메시지를 내부 방식으로 표준화"]
  F["Normalize outputs\n외부 응답을 내부 표준 구조로 변환"]

  A --> B --> C --> D --> E --> F
```

*그림 2. 외부 시스템 호출을 감쌀 때 고려해야 할 Wrapper Canvas*

Integration Service는 외부 시스템 호출을 감싸는 Wrapper 역할을 합니다. 단순히 API를 호출하는 것만이 아니라, 입력 검증부터 인증, 입력 매핑, API 호출 감사, 오류 정규화, 출력 정규화까지 담당합니다.

| Wrapper Canvas 항목 | 한글 설명 | 시험 포인트 |
|---|---|---|
| Validation / Security logic | 입력값 일관성 확인, 사용자 권한 확인 | 권한/검증은 호출 전 체크 |
| Authentication logic | 외부 API 호출에 필요한 인증 정보 준비 | 토큰, credential 처리 |
| Map inputs | 내부 입력을 외부 API 형식으로 변환 | 내부 모델과 외부 모델 분리 |
| Call and audit the external API | 외부 API 호출 및 필요 시 감사 기록 | audit trail |
| Normalize error handling | 외부 API 성공/오류 메시지를 내부 방식으로 표준화 | 오류 메시지 그대로 반환은 위험 |
| Normalize outputs | 외부 응답을 내부 표준 구조로 변환 | Consumer가 외부 구조를 몰라야 함 |
| Exception handling | 보통 caller가 예외를 처리하지만, 통합 예외 메시지 표준화에 사용 가능 | 무조건 여기서 다 처리한다는 의미는 아님 |

---

## 6. Summary fields와 Detail fields

Integration Pattern 선택은 데이터 성격에 따라 달라집니다. 자료에서는 외부 시스템에서 가져오는 필드를 Summary fields와 Detail fields로 나눕니다.

| 구분 | 영어 설명 | 한글 설명 | 예시 |
|---|---|---|---|
| Summary fields | Used for listing or searching entries | 목록/검색에 필요한 필드. 보통 자주 바뀌지 않음 | Customer name, Customer city |
| Detail fields | Only required to display details for a single entry | 단건 상세 화면에서만 필요한 필드. 자주 바뀌거나 민감할 수 있음 | Customer balance |

> ⚠️ 시험 함정: 목록/검색에 필요한 Summary fields와 단건 상세에 필요한 Detail fields를 구분해야 합니다. 상세 필드가 민감하거나 자주 바뀌면 무조건 복제하지 않고 Lazy Load Details 같은 패턴을 고려합니다.

---

## 7. 그림 3 - 데이터 필요에 따른 Integration Pattern 결정 트리

```mermaid
flowchart TD
  Start([시작]) --> Q1{"OutSystems 안에\n데이터 복제해도 되는가?"}
  Q1 -->|Yes| Cold["Cold Cache Summary"]
  Q1 -->|No| Direct["Direct Integration"]

  Cold --> Q2{"Summary fields가\n자주 변하는가?"}
  Q2 -->|No| Batch["Batch Sync"]
  Q2 -->|Yes| Q3{"변경량이\n많은가?"}
  Q3 -->|No| Realtime["Real-time Sync"]
  Q3 -->|Yes| Q4{"순서가\n중요한가?"}
  Q4 -->|No| Queued["Queued Real-time Sync"]
  Q4 -->|Yes| Ordered["Ordered Real-time Sync"]

  Realtime --> Q5{"Detail fields가 반복 사용되고\n가져오는 비용이 큰가?"}
  Q5 -->|Yes| Lazy["Lazy Load Details"]
  Q5 -->|No| Q6{"같은 개념에\n소스가 여러 개인가?"}

  Direct --> Q6
  Q6 -->|Yes| Trans["Transparency Service"]
  Q6 -->|No| End([끝])
  Lazy --> End
  Trans --> End
```

*그림 3. 데이터 필요에 따라 Direct Integration, Cache, Sync, Lazy Load, Transparency Service를 선택하는 결정 트리*

결정 트리는 먼저 "OutSystems 안에 일부 데이터를 복제해도 되는가?"를 묻습니다. 복제할 필요가 없거나 복제하면 안 되면 Direct Integration으로 갑니다. 복제해도 되고 검색/매시업이 필요하면 Cold Cache Summary부터 시작해 Summary 데이터 변경 빈도, 변경량, 순서 중요도를 보고 Batch Sync, Real-time Sync, Queued Real-time Sync, Ordered Real-time Sync를 선택합니다.

| 패턴 | 언제 선택? | 핵심 의미 |
|---|---|---|
| Direct Integration | 복제하지 않고 외부 시스템을 직접 조회/처리해야 할 때 | Integration Service를 통한 직접 연계 |
| Cold Cache Summary | 검색/목록/매시업을 위해 일부 Summary 데이터를 OutSystems에 보관할 때 | 조회 성능과 scaffold/search 지원 |
| Batch Sync | Summary 데이터가 자주 변하지 않을 때 | 일정 주기 동기화 |
| Real-time Sync | 변경량이 낮고 빠른 반영이 필요할 때 | 즉시 동기화 |
| Queued Real-time Sync | 변경량이 높아 비동기 큐 처리가 필요할 때 | 큐로 부하 완화 |
| Ordered Real-time Sync | 업데이트 순서가 중요할 때 | 순서 보장 동기화 |
| Lazy Load Details | 상세 필드가 반복 사용되고 가져오는 비용이 클 때 | 상세 데이터는 필요 시 로딩 |
| Transparency Service | 같은 개념의 데이터 소스가 여러 개일 때 | Consumer에게 소스 차이를 숨김 |

---

## 8. 그림 4 - Direct Integration 시작 부분

```mermaid
graph TD
  CS["Customer_CS\nCore Service"]
  IS["Customer_IS\nIntegration Service"]
  ERP[("ERP")]

  CS --> IS --> ERP
```

*그림 4. Direct Integration 패턴: 외부 시스템을 직접 연계하되, Integration Service를 통해 추상화*

Direct Integration은 외부 시스템과 직접 통신하는 패턴이지만, 중요한 점은 End-user나 Core Consumer가 ERP를 직접 호출한다는 뜻이 아닙니다. 자료 문장처럼 "using the Integration Service for service abstraction"이 핵심입니다.

> *영어 시험 문장: This pattern simply represents a direct integration with the external system, using the Integration Service for service abstraction.*

한글 해석: 이 패턴은 외부 시스템과 직접 통합하는 것을 의미하지만, 서비스 추상화를 위해 Integration Service를 사용한다는 뜻입니다.

---

## 9. 시험 함정 포인트

- Integration Service는 외부 시스템별 하나가 아니라 독립적인 업무 개념별로 설계하는 것이 기본 방향입니다.
- Core Service 소비자는 ERP/CRM 같은 외부 system of record를 몰라야 합니다.
- REST/SOAP/File Upload 같은 기술 방식은 Integration Service 안에 숨깁니다.
- Foundation/Core 구분 문제에서는 Integration Service가 기술 래퍼처럼 보이지만, Customer_IS처럼 특정 업무 개념에 묶이면 Core Services Abstraction 맥락에서 Core 개념 주변에 배치될 수 있습니다.
- Summary fields는 목록/검색용이고, Detail fields는 단건 상세용입니다. 이 구분이 동기화 패턴 선택 기준입니다.
- Exception handling은 보통 caller가 담당하지만, 통합 예외 메시지 표준화나 audit 목적으로 Integration Wrapper에서 고려할 수 있습니다.

---

## 10. 실전 문제풀이

**Q1.** Which statement best describes the purpose of an Integration Service?
> Q1. Integration Service의 목적을 가장 잘 설명하는 것은?
- A. To let end-user applications directly call ERP APIs
- B. To abstract external APIs and map them to internal structures and concepts
- C. To replace all Core Services
- D. To store all external data permanently in OutSystems

<details>
<summary>정답 보기</summary>

✅ **정답: B** — Integration Service는 외부 API를 추상화하고 내부 구조/개념에 맞게 매핑합니다.

</details>

**Q2.** The recommended approach is to create Integration Services based on:
> Q2. Integration Service를 만드는 권장 방식의 기준은?
- A. Each external system only / B. Each screen flow / C. Each independent business concept / D. Each database table name

<details>
<summary>정답 보기</summary>

✅ **정답: C** — 자료의 핵심 문장: Integration Service per independent concept.

</details>

**Q3.** Summary fields are mainly used for:
> Q3. Summary fields는 주로 무엇에 사용되는가?
- A. Listing or searching entries / B. Displaying sensitive details of a single record / C. Handling API credentials / D. Normalizing exception messages

<details>
<summary>정답 보기</summary>

✅ **정답: A** — Summary fields는 목록/검색용입니다.

</details>

**Q4.** Which concern belongs in the Integration Wrapper Canvas?
> Q4. Integration Wrapper Canvas에 속하는 항목은?
- A. Map inputs / B. Design the app logo / C. Create end-user menus / D. Define screen navigation

<details>
<summary>정답 보기</summary>

✅ **정답: A** — Wrapper Canvas에는 validation, authentication, map inputs, call/audit API, normalize errors/outputs 등이 들어갑니다.

</details>

**Q5.** Direct Integration means:
> Q5. Direct Integration이 의미하는 것은?
- A. The screen directly calls the external API
- B. The Core Service must know ERP-specific structures
- C. The Integration Service abstracts a direct call to the external system
- D. All data must be cached in OutSystems

<details>
<summary>정답 보기</summary>

✅ **정답: C** — 직접 연계라도 Integration Service를 통한 service abstraction이 핵심입니다.

</details>

---

## 11. 30초 복습 키워드

| 키워드 | 바로 떠올릴 내용 |
|---|---|
| Core Service | 업무 개념을 안정적으로 제공, 외부 시스템에 독립적 |
| Integration Service | 외부 API/프로토콜/인증/오류/출력 정규화를 감싸는 계층 |
| per independent concept | Customer_IS, Product_IS처럼 개념별 분리 |
| Summary fields | 목록/검색용, 캐시/동기화 후보 |
| Detail fields | 단건 상세용, 민감/빈번 변경 가능, Lazy Load 고려 |
| Direct Integration | 복제하지 않지만 Integration Service로 추상화 |
| Transparency Service | 동일 개념에 여러 데이터 소스가 있을 때 소스 차이 숨김 |
