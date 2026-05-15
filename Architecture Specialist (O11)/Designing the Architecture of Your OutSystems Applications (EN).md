# Designing the Architecture of Your OutSystems Applications

> Based on OutSystems official documentation (success.outsystems.com)

---

## Table of Contents

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

The Architecture Canvas is an OutSystems architecture tool to make the design of Service-Oriented Architectures (SOA) simple. It promotes the correct abstraction of reusable (micro)services and the correct isolation of distinct functional modules, in cases where you are developing and maintaining multiple applications that reuse common modules. A typical medium to large OutSystems installation will support 20+ mission critical applications and 200+ interdependent modules.

These applications/modules have different change life cycles and are maintained and sponsored by different teams. New applications tend to evolve fast while highly reused services will change much slower. The most important benefit you get out of a well designed architecture is that the applications and the modules that compose them will preserve independent lifecycles and decrease to a minimum dependencies and overall change impact. The result is a cost-effective OutSystems architecture design, easier to maintain and evolve.

### The layers and sub layers

Each layer and sub layer sets a different nature of the functionality to be captured in a module.

![The Architecture Canvas (layers)](https://success.outsystems.com/TK_Resource/105b1fae-c75a-408f-a9b8-c40fb6916fc4)

> ⚠️ **The Orchestration layer was removed in O11.** The Orchestration layer was used for hyperlinks between screens of two different applications. Such links are considered strong dependencies, which compromise each application's lifecycle independence. In OutSystems 11, screen destinations are considered weak references, so an orchestration layer is no longer required, and as such has been removed from the architecture canvas.

Sub layers are shown in detail below.

![The Architecture Canvas (sublayers)](https://success.outsystems.com/TK_Resource/a0ac451c-e9a4-463f-a500-0f926ff7b431)

| Layer | Sub-layer | Description |
|-------|-----------|-------------|
| End-user | Orchestration (*) | Modules that mashup several frontends in order to provide a unified user experience or a cross application workflow (* Applicable to versions prior to OutSystems 11) |
| End-user | End-user | Frontend User Interface modules |
| Core | Core | Reusable services around business concepts, exporting entities, business rules and web-blocks |
| Core | API | Provides APIs to expose your Core Services |
| Core | Core widgets | Core Widgets |
| Core | Composite logic | Reusable Business Logic: Composition or Logic to Synchronize Data |
| Core | Core service | Reusable Core Service |
| Foundation | Foundation | Non-functional requirements or integration modules, reusable in any business context |
| Foundation | Style Guide | Reusable UI Patterns, themes and theme templates |
| Foundation | Foundation Service | Integration services to wrap-up external services and services to support NFRs (e.g. Audit logging) |
| Foundation | Library | Reusable libraries and plug-ins |

### Architecture design with the Architecture Canvas

The Architecture Canvas is used in two different stages of the architecture design.

**1. Identifying concepts (functional, non-functional and integration needs)**

The canvas helps collecting architecture requirements in a structured and systematic way.

![Architecture Canvas - Identifying concepts](https://success.outsystems.com/TK_Resource/cd465952-0a3e-4d00-894a-d7d35e4ad7fa)

**2. Define modules**

Design the modules that implement the identified concepts, following recommended patterns.

![Architecture Iteration](https://success.outsystems.com/TK_Resource/4c071788-f0a8-43bf-8bc7-163d4849ef77)

Designing an architecture is not a one-time event. It is a continuous process. The architecture must be iterated, cycling these two stages, as a solution evolves and new concepts emerge from your business.

### Using the Architecture Canvas

To start using the Architecture Canvas check the following articles:
- Translating business concepts into application modules
- Validating your application architecture

### More information

Check out the Electronic Canvas tool, available in the OutSystems Forge. It assists you in the design of a new architecture by allowing you to place and move around your concepts in a digital Architecture Canvas. It enables you to identify and organize all the architectural elements that need to be implemented in a new project.

---

## 2. Translating business concepts into application modules

Once you identify your concepts, you must translate them into application modules. To do so correctly, it is key you find the correct balance between:

1. Not having to handle too many modules; and
2. Having a manageable complexity and life cycle independence for each module.

### Finding the right concept granularity

Consider the following example of concepts identified with the Architecture Canvas method.

![Architecture Canvas method - concept example](https://success.outsystems.com/TK_Resource/7f0b37d7-942b-4651-aae4-cee23f0b43a2)

The number of modules that need to be defined to implement these concepts depends on the concept granularity. To define an architecture, concepts don't need to be captured at very low level (like in this example). However, the correct granularity is influenced by the following criteria:

**Complexity**

A concept may be too vast to be implemented in a single module. For example, user process **Customer management** may include several sub-processes: provisioning a new customer, analyzing customer trends, adding a service to a customer, among others.

Split it into several modules to keep the complexity of each module manageable.

**Independent life cycles**

Even if you predict that all the sub-concepts together don't result in a complex module, splitting a module in smaller concepts is also required if you need to manage functionality independently.

This allows you to parallelize development among different developers or keep different evolution paces for different business sponsors.

**Service isolation**

There are several integration patterns to correctly abstract services, requiring you to split a concept in several technical components according to the scenario.

### Validating your architecture

To make sure the way you organized your concepts comply with Architecture Canvas recommendations, check the following article to learn how you can validate your application architecture.

### More information

Check the complete guide on how to design your application architecture in Designing the architecture of your OutSystems applications.

---

## 3. Validating your application architecture

There are 3 rules you **must always** comply with in order to achieve a well-designed application architecture.

The compliance of the implemented modules with these architecture rules can be automatically verified using the Discovery tool. It analyses the actual dependencies among modules, identifying violations and pinpointing the elements (actions, screens, entities) that are assembled in the wrong place.

![Architecture validation rules overview](https://success.outsystems.com/TK_Resource/ef4e0190-7927-4ff6-8885-23780534d6aa)

### Rule #1 — No upward references

An upward reference tends to create a cluster where any 2 modules, directly or indirectly, have a circular dependency.

In this example, since **Library E** consumes **End-user A**, any pair of elements inside the identified cluster is in a circular relation. Take **C** and **D** for example: `C → E → A → D` and reverse `D → E → A → C`.

![Rule #1 - Upward references example](https://success.outsystems.com/TK_Resource/a68f884c-6f71-4b30-a5be-17d4be00a474)

Another unexpected effect is that **End-user B** is legitimately consuming **Core D**, and becomes dependent of the entire cluster. Not only its runtime will get an unnecessarily large footprint, but it will also be constantly impacted (polluted) with changes made in modules that it should not even be aware of.

**What is wrong?** An upward violation clearly identifies that services are not properly isolated.

**How to fix it?** Find the elements that are being consumed and move them to a lower layer. In this case, move the reusable service in **A** to a **Library**, eventually to **E** itself, if it fits in the same concept.

### Rule #2 — No side references among End-users or Orchestrations

In this example, **End-user A** is consuming some element of **End-user B** (maybe something as simple as a formatting function). Not only it got coupled to module **B**, but it unnecessarily inherited **D**, **E** and **G**.

![Rule #2 - Side references example](https://success.outsystems.com/TK_Resource/b01a0a00-e535-49fe-81e1-058d2e190f0e)

**What is wrong?** **End-user** or **Orchestration** modules should not provide reusable services. This ensures that they are correctly isolated, allowing them to have different lifecycles — different versioning paces due to different sponsors or development teams. This isolation is critical since End-users and Orchestrations are at the top of the hierarchy. A reference to such modules tends to bring along a huge set of indirect dependencies from lower layers.

**How to fix it?** Find out which elements of **B** being consumed by **A** and move them to a new (or an existing, if there is a conceptual fit) lower layer module:
- To a **Core** module if it is business related.
- To a **Library** if it is business-agnostic.

### Rule #3 — No cycles among Cores or Libraries

Respecting rules #1 and #2, End-users and Orchestrations cannot be involved in any cyclic references. The third rule is about avoiding cycles among **Cores** or **Libraries**, since those are allowed to have side references.

![Rule #3 - Cycles example](https://success.outsystems.com/TK_Resource/51ab940f-51f5-4013-b357-80cdbf1e51ff)

A cycle is always undesirable, since it brings unexpected impacts and hard-to-manage code.

**What is wrong?** A cycle between modules indicates that the concepts are not correctly abstracted. A cycle between **A** and **B** either indicates that:
- They are strongly coupled; or
- One of the directions is undesirable. For instance, **A** should be conceptually consuming **B** because the concept in **A** extends **B**.

**How to fix it?** Most of the times, if there is a clearly an undesirable direction in the relation concepts must be moved to another module. For example, if it is **B** that should not be consuming **A**, then the elements of **A** consumed by **B** should be:
- Moved to a new module, if they represent another reusable concept; or
- Moved to **B** itself, if they were misplaced elements of the same concept.

If **A** and **B** are too strongly coupled, they should be merged together. If that merge results in a too large module, it may be necessary to create a third module on top of the two. The original two modules represent base concepts and the new module normally supply business rules, to support some end-user process, that need to handle both base concepts. This composition logic is the one causing the cycle.

### More information

To learn more about how to design your application architecture check the Designing the architecture of your OutSystems applications guide.

---

## 4. Service-oriented architectures for OutSystems applications

When abstracting your concepts into OutSystems Services, you must make sure that the core of your Services is:
- Common to both internal and external consumers.
- Isolated from all external systems - to cope with ecosystem changes.
- Prepared to replace old legacy systems that are planned to be deprecated.

This is key to allow you to keep your architecture flexible and able to cope with changes.

### Implementing your Service-Oriented Architecture

In the example depicted below, you can see 3 levels composing the services:
- The **External API**
- The **Core Services**
- The **Integration Services**

Note that the example reflects how it is vital that **Core Services** are to be properly isolated from external systems.

![Service-Oriented Architecture diagram](https://success.outsystems.com/TK_Resource/deae802d-4dda-44c8-837b-35dd72e85927)

### External API level

This level is a composition of APIs that exposes the services to external consumers - using REST, SOAP or other.

It is merely a technical wrapper for the real services implemented at the Core Services layer. This level translates those services to the APIs agreed with external systems.

> ⚠️ **No business logic should be added at this level.**

Several versions of the API may be kept at this level to comply with different consumer specifications. You can add Integration audits at this level to keep track of external consumption of the service.

### Core Services level

This level implements the services, with all the business rules and core entities.

**Core Services** should be system-agnostic:
- **External APIs**: Knowledge of external systems consuming the services is limited to the External API level. Any change, addition or removal of an external consumer should only impact the External API.
- **Integration Services**: If the Core Service extends an external system of records (an external producer), this should also be abstracted by the Integration Services level. If an external producer changes or is replaced, as long as that change is abstracted by the IS in the Core Services, the Core Services are not impacted.

When integrating with legacy systems it is common to synchronize data (through Integration Services) into OutSystems entities (a local cache) in the Core Services. The approach enables:
- Coping with performance issues.
- Protecting Core Services from extra applicational load.
- Supporting the business logic without online calls to the legacy systems.
- Deprecating old legacy systems in the future, without impacting any consumer of the Core Service.

A business audit can keep track of all the business transactions, regardless if it is an external or internal call.

### Integration Services

This level is a technical wrapper of external producers, normalizing data structures and abstracting integration complexity (recovering from errors, authentication on external systems, among others).

> ⚠️ **No business logic should be added at this level.**

An Integration Service can also abstract the fact that a certain concept is distributed across different systems. For example, complementary Customer information is kept in both the CRM and the ERP systems. Library module **Customer_IS** integrates with both systems, providing all the APIs required to retrieve mashed-up Customer information in a normalized Customer data structure. A Core Service consuming this Integration Service, has no knowledge of where the information is coming from. This gives you the flexibility to change the external systems without any impact on the Core Services.

### Service design patterns

Check some OutSystems architecture patterns to get further details on how you can implement the following cases:
- Integration Patterns for Core Services Abstraction

---

## 5. Integration Patterns for Core Services Abstraction

### Why do you need integration patterns?

If you need to integrate with an external system of records to retrieve master data or perform transactions, it is a good practice to follow patterns that abstract the concept you want to import into OutSystems to promote:

- **System independence**: changing or replacing the external system only affects the implementation of the concept abstraction and not the consumers of the concept.
- **Extensibility, normalization, and optimization**: opportunity to extend the concept with extra information and business rules, normalize representations and cache information to improve performance.

![Integration pattern overview](https://success.outsystems.com/TK_Resource/05051ba3-cb1f-49b2-aef8-a37ba2590eb0)

**1- Core Service**
- Extend the original service
- System agnostic (resilient to external changes)

**2- Integration Service**
- Abstracts the original API(s), matching to internal structures and concepts (e.g. composing a customer concept with complementary information from different systems)
- Hide the integration type (SOAP, REST, XIF, HTTP request, file upload, ...)
- Cope with technical requirements (access credentials, authorization, error handling, integration auditing, ...)

The idea is to have an Integration Service per independent concept (e.g. Customers) and not a single integration point per external system:
- One concept may be supported in different systems and the Integration Service abstracts that complexity.
- One system may support more than one concept (e.g. an ERP with Customers and Products) but each concept has its own life cycle (a consumer of Customers may not be interested in Products and should not be impacted by changes on the latter).

### Integration Services — wrapping calls to an external system

The Integration Service wraps-up any call to the external system, hiding the integration complexity and abstracting a service easy to be reused. The following canvas guides you on the major concerns that should be tackled when wrapping a call to an external system, regardless of the specific integration protocol being used (REST, SOAP, DLL Wrapper, etc.).

**Integration Wrapper Canvas:**

![Integration Services wrapping concerns](https://success.outsystems.com/TK_Resource/59716818-9fcc-4993-932c-7f9907e12809)

- **Validation / Security logic**: Check consistency of inputs. If required, check if user in session is eligible to perform this call. Raise a user exception to propagate the error.
- **Authentication logic**: Prepare any authentication credentials required to call the external API.
- **Map inputs**: Map the normalized inputs into the format required by the external API.
- **Call and audit the external API**: Call the external API and audit the call if integration trail is required.
- **Normalize error handling**: Treat Success and Error message outputs, raising exceptions instead of returning those outputs — the expected OutSystems behavior.
- **Normalize outputs**: Map the returned structures into the expected normalized outputs.
- **Exception handling**: Normally not necessary, as errors should be trapped by the caller. Can be used to audit integration exceptions or normalize exception messages that can end up in the user interface.

### Integration patterns according to data needs

This section presents several patterns, according to different needs (business concepts and non-functional requirements). The key point is that no matter what kind of integration is required, consumers of the core service (in our example, **Customer_CS**) always get the same type of abstraction, and are unaware of the external system of records.

When fetching data from an external system of records we can classify two types of fields, according to the nature of data to understand your integration needs:

- **Summary fields**: Used for listing or searching entries. Typically these fields don't change over time, like a **Customer name** or change very rarely like the **Customer city**.
- **Detail fields**: Only required to display details for a single entry. These fields are more subject to frequent changes and to be considered sensitive to be replicated, as a **Customer balance**.

Before explaining the integration patterns, use the following decision tree to help you select the patterns, according to your data needs. For your application, pick all the patterns collected in your selected path.

![Integration pattern decision tree](https://success.outsystems.com/TK_Resource/90d69c79-2871-40cf-85b4-75f37d454042)

> 💡 The bolded answers show the most common and recommended path — it benefits from supplying entities for mashup, search and scaffolding in OutSystems, which accelerate the implementation without the remaining complexity that is rarely needed.

---

### Direct integration

This pattern simply represents a direct integration with the external system, using the Integration Service for service abstraction.

![Direct Integration pattern](https://success.outsystems.com/TK_Resource/459b4387-37d7-487b-a58b-6d0abcc1d242)

In this scenario, Customer data is coming from an ERP. **Customer_IS** interfaces with the ERP, abstracting a normalized API, both for retrieving information and performing transactions. This creates the flexibility to change the external systems without impact on core services, as long as the **Integration Service API** is maintained.

Also, note that this scenario does not include entities in **Customer_CS** to keep a local replica of data. Not keeping replicated data outside the ERP might be due to a business constraint, or because (almost) real-time demand is incompatible with having a delayed data replication (information changes too often).

However, this pattern has several **limitations**:
- Extra load on the ERP each time data is retrieved.
- More perceived latency caused by extra online communication with the ERP.
- Impossibility to maximize the power of OutSystems Aggregates and Advanced Queries to retrieve Customer data or to combine it with other information.
- A constant need to extend **Customer_IS** and a strong dependency on the ERP team to provide new APIs each time a different data retrieval is required.

| Advantages | Disadvantages |
|------------|---------------|
| Data is always up-to-date | More APIs (one per retrieval use case) |
| | More latency |
| | More hits on external system |

---

### Cold Cache Summary Data

When you feel the need to improve the performance of your integrations with external systems, a good starting point is to cache data that doesn't change frequently. This data is called "cold data", that gives the name to this caching pattern, **Cold Cache**.

Cold Cache is used when it is too costly to synchronize the entire database, and detail is only required for single entries (not lists). While summary data must be present to search for any entry, synchronizing the entire entry is unnecessary when only 10% of them will be actually visited in detail.

When this pattern is implemented:
- Only summary data that it's frequently listed, joined or searched is cached (full detail for a single entry can be fetched directly on the external system).
- Full and frequent synchronization is avoided (summary information doesn't change often).

#### Cold Cache with Batch Sync

When implementing a Cold Cache, the simplest design approach is to create the caching data model and set up a batch synchronization process to manage the cached data.

Adding local entities to **Customer_CS** will overcome the limitations of the Direct Integration pattern and actually create a full-blown Core Service.

![Cold Cache with Batch Sync pattern](https://success.outsystems.com/TK_Resource/0efbac8d-c7b6-4c62-9934-9f9b715c2e65)

The Integration Service becomes simple and stable. Instead of providing a myriad of actions for different data retrieval needs, it only has to supply a method to fetch all customer relevant data, updated since the last sync.

**Customer_CS** has a timer to regularly synchronize information through the Integration Service. This synchronization should be unidirectional, to avoid complex merges of information — from the ERP (the master of data) to the Core Service. On the opposite direction, when an update is made in **Customer_CS**, you must be careful to make sure that the update is successful and synchronously committed in the ERP first (a write-through policy).

| Advantages | Disadvantages |
|------------|---------------|
| Simpler API | Data may be outdated |
| Enable data mashup in OutSystems | |
| Less impact on the source system | |
| Core Service customers not affected by the synchronization | |

---

### Isolate synchronization logic

When the synchronization process is too complex and constantly tuned, it is recommended to extract it from the Core Service, further isolating **Customer_CS** from the external system.

![Isolate synchronization logic pattern](https://success.outsystems.com/TK_Resource/fce06b8e-b137-470b-9816-810270bbbf27)

Another reason to do it is when the synchronization requires to orchestrate several integrations — for e.g., If Customers are stored in one system and Customer Contracts in another, then the synchronization needs to make sure that customers are synced before contracts since contracts refer to customers.

Consumers of **Customer_CS** don't need to be impacted by the synchronization code. Additionally, if in the future the ERP is deprecated and replaced by functionality built in OutSystems, stripping out the synchronization code has no impact.

In this example, **Customer_Sync** is the one regularly fetching updated information through the Integration Service, to sync into **Customer_CS**. **Customer_CS** still consumes **Customer_IS** to perform online transactions.

---

### Real-Time Sync

Normally, a cold cache with summary data that is required for search, listing or mashup, is very stable not requiring real-time sync. However, some situations require summary information to include fields that need to be up to date in real time for the application to work consistently (e.g.: You need to retrieve the information of your fleet from an external system, and it is key to include the current position of each vehicle in summary listings — which is a frequently changing field).

![Real-Time Sync pattern](https://success.outsystems.com/TK_Resource/b694bf68-ebc6-4ec0-a8a0-9692865d5171)

Real-time sync requires the external system of records to callback OutSystems, notifying some change in real time (in this example a customer update or insert). The **API** and the **IS** modules completely isolate the Core Service, making it agnostic to the external system and to the synchronization process.

| Advantages | Disadvantages |
|------------|---------------|
| Simpler API | More inter-system dependency |
| Enable data mashup in OutSystems | Not appropriate for a high load of changes, by not providing a queue |
| Less impact on the source system | |
| Data is always current | |

---

### Queued real-time sync

While the default real-time sync is relatively simple to implement, it's not appropriate for a high volume of changes. When this is the case, the solution is to implement a queue, so the synchronization is processed in parallel.

![Queued real-time sync pattern](https://success.outsystems.com/TK_Resource/3084ac2f-e225-4f8f-940c-5e16bbe877e6)

① Quick insert of update request in a queue.
② BPT reacts to inbound message, updating **Customer_CS**.

---

### Ordered real-time sync

If the external system is prepared to fire multiple update requests concurrently (e.g., with multi-threading), there is no certainty that those requests will be received and processed in the correct order. Here is a variant to cope with that problem.

![Ordered real-time sync pattern](https://success.outsystems.com/TK_Resource/3084ac2f-e225-4f8f-940c-5e16bbe877e6)

① Send message with sequence control.
② Wait for all related messages being received, before enqueuing an entry identifying that cluster.
Light BPT reacts to inbound cluster, processing all related messages in correct order.

---

### Lazy Load Details

The patterns above address the common integration patterns when dealing with summary **data fields**. What about the detail fields? If it's frequent to reuse details after fetching them from the external system, it's recommended that you set up a lazy load pattern, as fetching details can be too costly to be done on demand.

**How to implement a Lazy Load Details pattern?**

1. Try to fetch data from the local cache. If not found or outdated, get entry detail from the external system and cache it (read-through caching).
   - Cached details can have an expiration date, to force refreshing it from the source after a certain period.
   - Alternatively, before reusing cached data, ask the external system of records for the last update timestamp of that information to compare with the local timestamp and decide if the entry is outdated or not.
2. If Summary data is also being cached, have a separate detail entity to cache details.
3. To save space, have a timer doing regular cleaning of outdated detail data.

> 💡 **It's common to combine different patterns: cache summary data regularly and lazy load details when needed.**

---

### Transparency Service

When there are multiple sources for the same type of information, usually with different formats and APIs, the synchronization process becomes more complex.

![Diagram showing a transparency service that synchronizes customer data from multiple systems into a single Core Service.](https://success.outsystems.com/TK_Resource/cf96ffe0-e0bd-4763-844a-b1f3a4c967a5)

**Customer_Sync** orchestrates the synchronization with all Systems of records, updating a single Customer data replica in **Customer_CS**. This creates a **transparency service**.

In this example, **Customer_CS** is not able to update Customers. This pattern is the most common transparency service, where information comes from different sources but does not flow in the opposite way. Examples are electricity or toll readings coming from different sources/formats. They can be corrected or fixed locally, but never sent back to the source.

However, if sending changes back is a requirement, an extra module must be added. **Customer_IS** abstracts which interface (or driver) will actually send the transaction to the correct system.

![Diagram of a transparency service with an Integration Service routing requests to the correct system driver.](https://success.outsystems.com/TK_Resource/4efb992c-625a-4610-b7d2-2f3468fbefd3)

1. The integration service abstracts the existence of different systems, routing requests to the correct driver.
2. Each system is interfaced with a different driver. All drivers provide an API with the same signatures.
3. Different customers may be in different systems.

---

## 6. Application composition

---

### 6-1. Applying the Architecture Canvas to applications

The same way individual modules are classified in an Architecture Canvas, each application is also placed in one of the layers. The following representation displays the nature of each module type, as well as the application adopting the nature of its top-most module.

Applications should be placed to the same layer type as the topmost module type in it. The following representation displays the nature of each application, as well as the modules that compose them.

![Application types by layer](https://success.outsystems.com/TK_Resource/e4b0eb51-d09b-415e-adc0-40392eb2bace)

Application types:
- **Orchestration Applications** — Orchestration layer (applicable to OutSystems 10 or lower versions only)
- **End-User Applications** — End-User layer
- **Core Applications** — Core layer
- **Library Applications** — Foundation layer

Applications are subject to the same validation rules, respecting the relations between them:
1. No upward references
2. No side references among End-User applications or Orchestration applications
3. No cyclic references between two applications

The same rationale applies for modules and for applications. For example, an **End-User** application should not provide reusable services to ensure they are correctly isolated, allowing them to have independent lifecycles. An End-user application can contain modular **Core** and **Library** modules, as long as they are consumed only by other modules or the same application.

### Evolving your architecture along with your applications

To better understand the dependencies among applications, consider the following typical scenarios.

#### First project

Commonly, people don't start thinking about defining several applications, with one application per project.

In the first project, an application is created to hold all the modules that were conceived at the architecture design stage. This application results from the blending of components that will eventually be versioned and deployed to a Quality Assurance environment.

![First project structure](https://success.outsystems.com/TK_Resource/e943848f-cb5d-4805-aff2-60d99e314db5)

#### Second project

On a second similar project, another application for a different business process is created, adding a few more Core and Library modules.

![Second project structure](https://success.outsystems.com/TK_Resource/c2d8f561-8cb3-40cc-8129-b92defefbe0f)

#### Third project

Soon, in a third evolution project, **End-user #1** starts reusing **Core C** and **End-user #2** starts reusing **Core B**. Although there is no violation in terms of module dependencies, the two **End-user** applications have side references to each other — a cycle in fact.

![Third project - cyclic dependencies](https://success.outsystems.com/TK_Resource/bdcb97b6-f8c7-4c89-b4c9-17852a5a0bab)

This clearly implies that the two applications are strongly coupled. Deploying a new version of the first to Production may require to take the second along, and vice-versa.

#### Solving the issues

The commonly reused resources must be isolated in a new application, as displayed in the following diagram — the **Core Application**.

![Solving the issues - Core Application isolation](https://success.outsystems.com/TK_Resource/28e67ac7-0e57-4a30-85f9-920c82da9b53)

Not only **Core B** and **Core C**, that are directly referenced, must be moved to this application, but also all their dependencies (**Library B** and **Library C**) to avoid upward references to the **End-user** applications.

With this configuration, the two applications can evolve independently at different paces. In addition, the **Core** application isolates the critical resources that must be carefully managed to leverage impacts, and what can be flexibly changed inside each application.

Validation of the application architecture can also be done with the Discovery tool.

### Correctly composing applications

There are four rules that will help you make sure that you address the critical points when composing your applications. Another common aspect you need to take into account is isolating an application Theme to share the same look & feel among your applications.

---

### 6-2. The 4 Rules for Correct Application Composition

The need to split applications, after the reuse needs are known, is something you always need to address. However, splitting one application into smaller ones, upfront, is not always obvious.

These 4 rules guide application composition in order to:
- Minimize dependencies
- Simplify deployments
- Promote life cycle independence

The first two rules are the result of correctly using the Architecture Canvas methodology. The remaining two take into account other strategic considerations, driving the decision for creating smaller applications upfront.

#### Rule #1 — Correctly layer your modules

Follow the Architecture Canvas principles. If the module architecture contains violations, it may be impossible to get to a correct and manageable application architecture.

#### Rule #2 — Correctly layer your applications

Follow the Architecture Canvas principles when applied to applications instead of modules.

#### Rule #3 — Don't mix different owners

With different teams developing different projects simultaneously, it's common to get requirements from both projects that affect common services. To avoid bottlenecks, one option is to allow both teams to change the common resources.

Having more than one owner for an application results in complex deployment management, as accountability for what has been changed becomes unclear.

![Rule #3 - Different owners split](https://success.outsystems.com/TK_Resource/b11c4ee1-cade-4316-9edf-d61df48853ed)

Promoting ownership is key. If it's not possible to concentrate the ownership of one application, consider splitting it in such a way that ownership is clearly defined.

#### Rule #4 — Don't mix different sponsors

If a project affects several sponsors, it's important to isolate each LOB (Line Of Business) in a different application. Different sponsors have different budgets, requirements and change paces.

The following example shows a common Portal to grant web access to all sorts of insurance simulators, from different LOBs: Auto, Life, and Property.

![Rule #4 - Different sponsors split](https://success.outsystems.com/TK_Resource/bf08d7c8-2848-4159-98da-28beaaf6935b)

Splitting the application upfront, by LOB, enables the independent evolution of each one. The isolation of all the common services clearly sets the border between what must be carefully planned due to the expected transversal impact, and what can be flexibly changed inside each LOB.

---

### 6-3. Isolating an application Theme

In an enterprise installation, applications usually share the same corporate look & feel. Being a shared component, it is key to correctly position a global Theme module in the enterprise architecture.

A Theme module includes the following components:
- The theme itself (the CSS)
- Layout Blocks, to define screen or screen section patterns
- Logo images
- Login flow and exception handling
- Menu and navigation support logic
- Transversal roles (normally job roles like Manager or Employee) that can be reused in any application. This roles may influence the menu entries to be displayed.

> ⚠️ **The Theme module should be restricted to global look & feel elements and never be used as a global repository of assets and utilities.**

#### Same look and feel, menu and login flow

The following example shows a Global theme that is reused by several applications, supplying a common menu and login flow. This scenario makes sense if all the applications belong to a common Portal.

![Same look and feel - shared global theme](https://success.outsystems.com/TK_Resource/e2be76be-d281-4aaf-aa30-16050ec01338)

It is based on one of the built-in themes supplied in OutSystems UI. If the Common services supply UI components (Blocks), they should be based on the built-in base theme, since they are reusable by other applications that might be using a different look & feel. A Block will assume the CSS of the consuming module, thus adapting to the look & feel of the final application. Services inside each application are specific for that application, hence they can safely use the Global theme.

#### Same look and feel, different menus and login flows

If the applications are independent from each other and all they need is to share the global look & feel, then each application must have its own theme module.

![Same look and feel - individual app themes](https://success.outsystems.com/TK_Resource/f9d1683d-6981-4035-a539-ad22e308ad5c)

In this case, each application theme inherits the common global theme, but defines its own menu and/or login flow. Any overall change to the look & feel is performed in the Global theme module. Specific menus and login mechanisms (integrated, federated, different user provider, among others) are specialized in each application theme.

---

## 7. Designing Scalable Multi-Tenant Applications

OutSystems enables you to use a single software instance to service multiple client organizations — or tenants — for on-premise deployments and for software-as-a-service (SaaS) scenarios.

### Logical Segregation of Tenants

When implementing logical segregation of tenants, there are two issues to consider:

- **Data Segregation**: The need for a user in a system to only see data that belongs to a single entity. This is very different from data that belongs to multiple tenants.
- **Cross-Tenant Access**: The need for a user in a part to access data that belongs to multiple tenants.

OutSystems follows a multi-tenant approach of logical segregation. Each database server provides each a separate set of computing resources for all tenants. Screens and business logic apps are shared, but data and end users are kept apart.

![Logical Segregation - tenant isolation](https://success.outsystems.com/TK_Resource/15a16951-86a9-4336-b690-3d73c86c3f48)

At the database level, when defining one table as multi-tenant, a column with the Tenant ID is added to that table. This column is managed by OutSystems in a way that's transparent to the developer, automatically reducing development costs.

**Advantages:**
- Is fully supported and maintained by OutSystems as part of an installation subscription.
- Decreases infrastructure operation costs.
- Enforces a single security policy for data, users, sessions, and processes.
- Simplifies and speeds up application development because developers don't have to write tenant-specific queries.

**Disadvantages:**
- Lower degree of data isolation: data from all tenants is in the same table, which has a column that identifies what tenant that row belongs to.
- A higher number of rows per table. To overcome this potential performance bottleneck, you should use an archiving strategy.
- The more rows, the higher the number of indexes per table.

### Ensuring Multi-Tenancy Scalability

The OutSystems platform is the one that queries different tenants in the same entities while maintaining data ownership. It does not raise concerns about scalability when compared to separate database catalogs for tenant isolation. You can start increasing Database scalability by taking the following steps appropriately:

- Develop the application using a **fully multi-tenant** method, since this will give you the flexibility for later changes in the infrastructure.
- Use **page mechanisms** to minimize the amount of information stored in online tables.
- **Scale the database vertically**.

### When the Database Reaches the Maximum Vertical Capability

When opting the single database approach is no longer viable option, there are two ways to resolve it.

#### Scenario 1: Multiple Production Environments

Separate specific tenants into their own production infrastructure. To achieve this, you should:

![Multiple Production Environments](https://success.outsystems.com/TK_Resource/c55ab1bf-f7c5-4f82-8b11-5d49752a0716)

- After the environment is properly allocated, use the Lifetime to publish the required applications to the new production environment.
- Migrate data from the previous production to the new one. There are three ways to do this:
  - IT tools (Attunity, Informatica, Information Builders) — to extract the tenant information of all tables to the OutSystems infrastructure, using the database connection credentials.
  - System Implementation: Develop an OutSystems application that is able to extract and import the tenant data into the new environment.
  - Backup restore: Request a full backup restore of the production database into a new database.
- If there is a common Master Data, to complete the separation of tenants into their own production infrastructure, the application needs to connect to a common catalog.

#### Scenario 2: Manually Partitioning OutSystems Entities

OutSystems has no mechanism for partitioning data from different tenants, but changes to the meta-model can be applied manually.

### Security Concerns

It's understandable that there are concerns over one tenant seeing data that belongs to another. A common misconception is that only physical separation can provide an appropriate level of security. In fact, logical separation can provide an appropriate level of security as long as the data is only physically accessible from shared infrastructure.

With OutSystems, the only way that a tenant can see data from another tenant is when the developer accesses data explicitly during development.

To avoid these two scenarios, there is no other way than preventing human errors by building automated validation processes. Since the OutSystems meta-model contains information on entities and entities as referenced in BPT, you can use meta-model for the following additional validations:
- Report that identifies all modules where TenantSwitch is used.
- Report that signals all multi-tenant tables that are used as single-tenant tables.
- Have a scheduled job to run the reports and notify administrators.
- Always run the report before a production deployment.

### Physical Segregation of Tenants

An alternative option to logical segregation is to follow a physical segregation approach, where each tenant has its own database.

![Physical Segregation architecture](https://success.outsystems.com/TK_Resource/c6d9c5e2-1ef3-429e-8f9a-88a4fd8ca67f)

To have different databases per tenant, you need to implement a Database Access Layer to access the correct database per tenant. Once the user is authenticated, he needs to be mapped to the correct database connection so the data is fetched from the corresponding database.

**Advantages:**
- Highly degree of data isolation.
- Lower number of rows per table, pursuing speed is near optimal.

**Disadvantages:**
- More complex and time consuming because the data model of each tenant database needs to be kept synchronous.
- Slower development speed because each tenant requires additional resources.
- Maintenance is more complex and time consuming because the data model of each tenant database needs to be kept synchronous.

### Recommendation

Logical segregation has several advantages over physical segregation. The biggest disadvantage is the potential risk of exposing tenant data, by enforcing guidelines, that risk is mitigated. Since the implementation cost of automated processes in quite lower than the cost of having physical organization of apps, we recommend using the multi-tenant option available in OutSystems with logical segregation.

Other advantages include:
- Shorter development cycles with table creation and updates directly in the IDE.
- Simpler application maintenance by keeping a single code base.
- More straightforward to use this in support during application development, and staging.

### Documentation

- OutSystems Multi-tenant
- How to Build a Multi-tenant Application in OutSystems
- Multi-tenant base database factory pattern

---

## 8. Microservices Architecture in OutSystems

In a Service-Oriented Architecture (SOA), core services are abstracted for correct reuse. But that does not constitute a microservices-style implementation, since all services reside in the same environment (installation), along with OutSystems applications.

In a microservices approach, each service has its own infrastructure and is decoupled. Communication with and between services is only achieved through a loosely coupled lightweight mechanism — like a REST API.

Although the SOA approach already enforces a modular approach, microservices enhance the following properties:
- Continuous deployment of services with fewer impacts to consumers.
- Smaller footprints as the consuming applications don't include the service code.
- Less monolithic servers that can scale independently, according to each service demand.

The major disadvantage of microservices in OutSystems is that you can no longer benefit from the platform's RAD (Rapid Application Development) capabilities as they can only consume a REST API:
- No entities to accelerate data fetching through aggregates.
- Because of the previous limitation, all data retrieval needs must be supplied through the microservice API.
- No reusable Blocks supplied by the services.

![Microservices vs Monolithic architecture](https://success.outsystems.com/TK_Resource/ee7449e9-711b-4665-8abc-d611668f5bcd)

To adopt a microservices approach, services need to be placed in a different infrastructure and applications will consume those services through the REST API. To leverage OutSystems RAD, an application-side extension to microservices can be implemented.

Instead of consuming the microservice directly, Extended Services augment the core services by caching data in local entities, and providing reusable blocks, or composite business logic. None of these are new, as they match the recommendations for any external API.

![Extended Services augmenting Core Services](https://success.outsystems.com/TK_Resource/26643e7c-b752-4971-ab46-124ea3da7f36)

### Multiple Microservice Infrastructures

Usually in a microservices approach, each microservice resides in its own infrastructure. This allows each infrastructure to scale independently and not be impacted by the load of other microservices.

![Multiple Microservice Infrastructures](https://success.outsystems.com/TK_Resource/3ca38069-f2d1-4995-9603-508306bded2c)

The split should be done between independent microservices. If two microservices are coupled with strong dependencies — for instance, with lots of database references — they should not be split into different infrastructures, to take advantage of the platform's full capabilities, and avoid the limitations of communicating only through a REST API.

You can start by having all your microservices in the same infrastructure, and isolate them as they demand their own infrastructure and growth policy. To ease this operation, when using a common infrastructure, isolate each microservice into a different DB Catalog.

### Central Services for Multiple Subsidiaries

Another fit for microservices is the support for central services: extending central back-end systems that need to be consumed by different subsidiaries. Those subsidiaries implement custom applications in their own installation.

![Central Services for Multiple Subsidiaries](https://success.outsystems.com/TK_Resource/fb6588ad-5e6d-453d-b241-0ae164617918)

### Microservices Lifecycle — Managing Versions

Multiple versions of a microservice can be kept by:
- Exposing multiple versions of the service definition in the REST API.
- Having several versions of an action in the core module implementing the actual logic.

![Microservices Lifecycle - Managing Versions](https://success.outsystems.com/TK_Resource/83a1b6f6-936e-451b-911f-d5b9d6e06029)

#### When to Create a New Version

The isolation of microservices allows their continuous deployment without impacting the consumers. There is no need to publicly announce a new version, as long as the API signatures don't change.

When the API needs to be changed, we need to decide whether we can update the current version or need to create a new version of the microservice API.

**Change the current API version**, if changes only include:

1. **New method for a new action**: Consumers that don't need the new method are not impacted, as they don't have to refresh the definition of the REST service.

   ![Diagram showing the addition of a new method to an existing microservices API without impacting consumers.](https://success.outsystems.com/TK_Resource/c8ff4180-6bc6-4168-ab96-299127ec44c1)

2. **Signature change that needs to be forced to every consumer**: In this case, breaking the current version forces the consumers to refresh the REST service.

   ![Diagram illustrating a forced signature change in a microservices API and its impact on consumers.](https://success.outsystems.com/TK_Resource/3034e65b-2976-4698-afb7-c88aacca0db2)

**Create new API version**, if a method signature changed and it can't be forced to every consumer, so the previous version needs to be kept to preserve backward compatibility. There are two situations:

1. **Non-disruptive change of method signature**: A signature change is non-disruptive if it only includes new optional attributes.

   ![Diagram showing a non-disruptive change to a microservices API method signature with optional attributes.](https://success.outsystems.com/TK_Resource/53503623-45e1-48a3-989d-a56be532ef5a)

2. **Disruptive change of method signature**: A signature change is disruptive if it includes new mandatory attributes or changes existing attribute names or types.

   ![Diagram depicting a disruptive change to a microservices API method signature, requiring versioning.](https://success.outsystems.com/TK_Resource/b2df3628-8662-4c72-9675-6b167f7ea612)

   In this case, the previous versions should be deprecated since they can't be made compatible with the new implementation. Hence, consumers should be forced to adopt the new version.

### Infrastructure Scenarios

#### Scenario A — Single Environment for Semi-isolated Microservices

![Scenario A - Single Environment](https://success.outsystems.com/TK_Resource/15b43ae4-e0ad-4aec-af5c-4540a903f036)

To isolate different microservices, simply add more microservices zones — Zone B example.

#### Scenario B — Separate Environment for Fully Isolated Microservices

![Diagram of separate environments for fully isolated microservices, each with its own infrastructure.](https://success.outsystems.com/TK_Resource/90384efe-a511-4ef0-8b52-c52085798ff4)

To isolate different microservices, simply add more microservices infrastructures — Infrastructure B example.

#### Comparing Scenarios

![Comparative diagram of single and multiple infrastructure scenarios for microservices.](https://success.outsystems.com/TK_Resource/a4c23004-3e5c-468a-89d7-522d0bc66f4c)

---

## 9. From architecture to development

This set of articles provide guidance on how to translate an architecture blueprint into a real OutSystems application.

This article focuses on how to design the architectural blueprint of the application. The second article guides you on how to start developing the application based on the blueprint. It also includes the code sample where you can evaluate the coding and architecture best practices put into action.

### Architecture blueprint design

Imagine you have a customer, the **Soccer Fields Fictitious**, that wants to build a web application to allow their clients to reserve soccer fields online.

> **Disclaimer:** this is a hypothetical use case, from a fictitious client.

#### Requirements

The application must:
- Display a list of soccer fields, together with each field characteristics (size, availability, price/hour, ...)
- Allow players to select a soccer field and book it.

Note that an external system provides the list of soccer fields, meaning it's not in the scope of this application to handle the actual field management or payment.

#### Architecture design process

The architecture design process follows a 3 step approach.

![Diagram of the architecture design iteration process with three segments: Disclose, Organize, and Assemble.](https://success.outsystems.com/TK_Resource/edf37634-5e44-4b8c-8dbe-d0a22c9de9cb)

| Icon | Step | Description |
|------|------|-------------|
| ![Icon representing the Disclose step in the architecture design process.](https://success.outsystems.com/TK_Resource/2a4cea33-5b36-47cd-9ce8-c5a63d75177e) | **Disclose** business concepts and integration needs | Identify business concepts and integration requirements |
| ![Icon representing the Organize step in the architecture design process.](https://success.outsystems.com/TK_Resource/d833b165-4250-445f-8a77-89421b686b4a) | **Organize** concepts on the architecture Layer canvas | Place concepts on the architecture layer canvas |
| ![Icon representing the Assemble step in the architecture design process.](https://success.outsystems.com/TK_Resource/21a923e3-229a-42fb-a838-1180e10e04c5) | **Assemble** Matching recommended patterns | Assemble following recommended patterns |

It's important to understand that you should iterate these three steps multiple times, until you feel comfortable with the design, and eventually repeat the process during the implementation, as you discover new concepts and/or details. This article exemplifies a simple application and makes a single iteration only.

---

### Disclose (business concepts and integration needs)

First of all, let's understand the functional and non-functional requirements. In this step, you need to find out all the user stories as you can, understand the integration needs, and the UX expectations.

**Functional Requirements**

Search for a Field using the following filters:
- Name of the field
- City
- By field size (how many players can play together)

Allow the booking of fields by players planning to use them.

**Non Functional Requirements**

- Be compatible with the leading browsers on the market.
- Retrieve the list of soccer fields from an external system, by consuming a REST API.
- The system needs to work online and the UI must adapt to mobile devices.

---

### Organize (concepts on architecture canvas)

Let's have a quick recap of the Architecture Canvas principles:

| Layer | Description |
|-------|-------------|
| ![Icon for the End User Layer, indicating user interfaces and processes.](https://success.outsystems.com/TK_Resource/36c9078b-c54d-493a-8518-b736e37fa9e8) | User interfaces and processes, reusing Core and Library to implement the user stories. |
| ![Icon for the Core Layer, indicating core business services.](https://success.outsystems.com/TK_Resource/07bb59f3-50a1-4f49-9da6-77c4effd8611) | Services around business concepts, exporting reusable entities, business rules, and business widgets. |
| ![Icon for the Foundation Layer, indicating non-business services.](https://success.outsystems.com/TK_Resource/6b782105-7723-47f3-ab5c-35a0e998212b) | Business-agnostic services to extend the framework with highly reusable assets, UI Patterns, connectors to external systems, and integration of native code. |

Based on these principles, you end up with the following concepts of the application on the following layer canvas:

![Architecture Layer Canvas showing the distribution of application components across different layers.](https://success.outsystems.com/TK_Resource/35ed7e75-34dc-42ae-b62d-06dbdbcb0d96)

The main concepts identified were:
- **SoccerFields**: The main application that holds the interaction with the users.
- **Bookings**: The functional requirement, that allows players to book fields.
- **Players**: The players that use the application and use/book the fields.
- **Fields**: The main field concept, used for searching and to do the bookings by the players.
- **Fields Integrations**: An external system stores the Soccer field, so you need a non-functional requirement module to connect to the external system through an API.
- **Soccer Fields Theme**: The look and feel of the application needs to be mobile responsive.

---

### Assemble (matching recommended patterns)

And now, Assemble! Let's remember the principles involved here:
- Join concepts if they're conceptually related.
- Don't join concepts if they're too complex or have different life cycles.
- Isolate the reusable logic from all the integration logic that consumers don't care about.

Based on these principles and looking into the architecture patterns and best practices, this article focuses on the following concepts:

**Online (Field_IS module)**

To access an external API, use the **Field_IS** module (IS means integration service). The responsibility of this model is to know the external API and normalize the data for internal access so the Field_CS and Field_Sync modules don't need to know any particularity of the API. In terms of abstraction, this is very important. Using this pattern makes your application more resistant to the API changes, and it's not forced to absorb what you don't want from the API in your data model.

**Batch Sync (Field_Sync module)**

Use this second pattern to facilitate filters and to deliver a better user experience (by enhancing performance). Keep on your side a summary of the Field entity so that you can do the main searches in the local entity without accessing the external API. To achieve this, the **Field_Sync** module is going to synchronize field data once a day.

![Diagram illustrating integration patterns for external master entities with batch sync.](https://success.outsystems.com/TK_Resource/f271e453-2876-4125-87df-50a6c2c4facb)

**Business Logic Module (Booking_BL module)**

Another recommendation is to isolate Business Logic (Actions) or Core Widgets (Webblocks), to manage complexity, composition or to have its own lifecycle. In this case, it wasn't necessary to create a _CW module. Maybe you can have it later, but for now, the **Booking_BL** module addresses all business logic required.

By applying these architectural patterns and using a common naming convention to the module names you end up with the following architecture blueprint for the modules:

![Blueprint diagram showing the architecture of the Soccer Fields application with various modules.](https://success.outsystems.com/TK_Resource/cbd2216f-5e78-4adf-972f-14132212d2c3)

Looking at the main functionalities and responsibilities of each module, you have the following:

| Module | Layer | Responsibility |
|--------|-------|----------------|
| **SoccerFields** | End-user | End-user application with all screens for finding and booking soccer fields. |
| **Booking_BL** | Core | Contains all logic related to managing a booking, combining the logic of field and player concepts. |
| **Player_CS** | Core | Reusable core service with the player entity and related entities. |
| **Field_CS** | Core | Reusable core service that hosts a local replica of external data, namely field entity and related entities - the cold cache. |
| **Field_Sync** | Core | Logic to synchronize data from an external system into Field_CS. This module is responsible for isolating all logic to keep a cold cache only with a summary of the Field entity. |
| **Field_IS** | Foundation | Technical wrapper to consume and normalize the Field API. This module is responsible for abstracting and knowing all particularities of the external API. |
| **SF_Th** | Foundation | Theme module (non-functional) for the main look and feel of the Soccer Field application. |

---

### Application composition

You need to think about architecture not only when creating your modules, but also when creating your applications and how to organize them.

An OutSystems application is a set of modules defined in Service Studio that constitute a minimal deployment unit for LifeTime (tool responsible for application lifecycle management in OutSystems).

Consider the figure below:

![Illustration of the modules of an OutSystems application organized by layer.](https://success.outsystems.com/TK_Resource/f8ea43af-173e-4d71-b791-8f89a035f682)

Applications need to respect layers as modules need, but what defines an application layer is the topmost layer of the modules inside it. In other words, if you have an application composed of modules from the Foundation layer only, then this application is a Foundation Application. If you have another application with modules of the foundation layer but also with one or more modules of the core layer, then this application is a core application.

Here's the application composition for the Soccer Fields App:

![Diagram showing the composition of the Soccer Fields application with its modules and layers.](https://success.outsystems.com/TK_Resource/d5f4941c-55b4-4246-9c87-8d9e2ab78c9f)

As you can see, there are two applications:
- **Soccer Fields App**: Contain the SoccerFields user module, the front-end application users interact with, used for style customizations too. Inside, there's also the SF_Th module, in case the application has any custom style applied.
- **Field Core Services**: Contain all the field related modules.

To learn how to put into action the final architecture blueprint, see the Developing from the architecture blueprint article.

---

### 9-1. Developing from the architecture blueprint

This article focus on putting into action the final architecture blueprint.

The previous article explained the use case of a fictitious client. The Soccer Fields Fictitious client wants to build a web application to allow their clients to reserve soccer fields online.

> Before proceeding, download from the OutSystems Forge the Soccer Fields App resources, the Soccer Field Sample API, and the Field Core Services App, and install them on your personal environment to better follow the instructions provided.

#### Requirements

The application must:
- Display a list of soccer fields, together with each field characteristics (size, availability, price/hour, …)
- Allow players to select a soccer field and book it.
- An external system provides the list of soccer fields, meaning it's not in the scope of this application to handle the actual field management or payment.

#### Architecture design

In the previous article, you ended up with a final architecture blueprint. The image below shows it and sets the starting point of the development phase.

![Diagram of the final architecture blueprint for the Soccer Fields application.](https://success.outsystems.com/TK_Resource/5d1b1b1b-1b1b-1b1b-1b1b-1b1b1b1b1b1b)

The following sections explain the process of translating a blueprint into an actual OutSystems application.

#### Understanding the blueprint

This is the first step. To understand what you are looking at:
- You need to be able to identify which elements are modules and which elements are applications.
- You need to understand the nature and goal of each element, so you can place your code in the right place.

The following table shows some conventions:

| Convention | Description |
|------------|-------------|
| ![Icon representing modules in the architecture blueprint with rounded rectangle shapes.](https://success.outsystems.com/TK_Resource/9730e3a0-2252-4fae-8c3d-db93ab8a66b5) | Rectangles with round corners represent the modules. The rectangle color allows mapping each module to the level in the Architecture Canvas. |
| ![Icon representing applications in the architecture blueprint with rectangles containing modules.](https://success.outsystems.com/TK_Resource/d34c4ef6-3fc6-4958-a2b8-c5bb6f5a6024) | Rectangles with modules inside represent Applications. The rectangle color is lighter than the module color, and it also allows you to map the application to the architecture layers. |

In this example, there are two applications:
- **Soccer Fields App**: this is the main application that contains the code to interact with the users and book the fields.
- **Field Core Services**: this is a supporting application, that allows you to connect and obtain field information coming from an external source.

#### Identify module dependencies

A good way of making sure the architecture is solid and you understand it, is to identify relationships and hierarchies between modules. Understanding which module depends on which, it's a good first step to avoid breaking the Architecture Canvas validation rules, validated with the Discovery tool.

This is most important at the core level, where incorrect dependencies lead to circular reference, due to the lack of proper concept isolation. This results in code refactoring, that tend to have high risk and costs.

![Diagram illustrating the dependencies between modules in the architecture blueprint.](https://success.outsystems.com/TK_Resource/09c95460-bf3a-4d9e-9892-38979e92d6f3)

Here are some insights:
- **Booking_BL** works by the composition of two core concepts, fields, and players. These two core concepts are independent, meaning that when a player books a field, the Booking_BL module keeps that association.
- **Field_CS** contains a local replica of the existing fields, for performance reasons. The Field_Sync module synchronizes this local replica, and reads the information from Field_IS and writes it into Field_CS.
- Writing operations into the external system uses the connection between Field_CS and Field_IS: when the user books the fields, it informs the external system.
- Note that **Field_IS** is the only module that connects to the external REST API, normalizing the retrieved information into an internal format. This level of abstraction is crucial for your system to evolve independently of the external systems.

#### Creating the modules and applications

This section goes through the development of modules and applications. The first step is to open Service Studio and get your hands dirty!

**The end-user application**

Following your architecture design let's create the first application and modules:

1. In Service Studio, choose **New Application**, select **Web App**, and pick the Liverpool theme. Name the application "Soccer Fields" and click on **CREATE APP**.

2. Create the first module — a theme module named **SF_Th**. Select the Responsive module type, then click on the **CREATE MODULE** button.

   ![Screenshot of the module creation interface in Service Studio with the module name 'SF_Th' and module type options.](https://success.outsystems.com/TK_Resource/9ddf5b13-d19f-42b6-9332-170c64895b0b)

3. Delete the **Emails** and **MainFlow** UI flows, then Publish.

   ![Screenshot highlighting the 'Emails' and 'MainFlow' UI flows to be deleted in Service Studio.](https://success.outsystems.com/TK_Resource/1128b71d-128e-4583-add7-4612cbff3a6d)

4. Create a new Responsive module named **SoccerFields** and publish it.

   ![Screenshot showing the process of creating a new front-end module named 'SoccerFields' in Service Studio.](https://success.outsystems.com/TK_Resource/adc2dbd9-c25f-48eb-9bc2-886f1d239208)

5. Set the **SoccerFields** module as the Home module.

   ![Screenshot depicting how to set the 'SoccerFields' module as the Home module in Service Studio.](https://success.outsystems.com/TK_Resource/6ac50af4-b79a-456a-930c-fef299b20935)

6. Create **Booking_BL** module (Blank type).

   ![Screenshot of the Service Studio interface for creating a new module named 'Booking_BL'.](https://success.outsystems.com/TK_Resource/8c63a8fe-067c-4d5d-bb30-04ac175b486c)

7. Create **Player_CS** module (Blank type).

   ![Screenshot showing the creation of a new module named 'Player_CS' in Service Studio.](https://success.outsystems.com/TK_Resource/e9e91f66-21c1-4c27-9e81-d027192f146b)

**The core application**

Repeat the first step done for the "Soccer Fields" application, but this time name the application **"Field Core Services"**.

Create the first module **Field_IS** with the **Blank** module type (not Responsive). The "Field Core Services" is a core services application and it shouldn't host any end-user screens.

![Screenshot of the Service Studio interface for creating a new module named 'Field_IS'.](https://success.outsystems.com/TK_Resource/ba53e6e8-e465-4281-97b7-da354933a2a9)

After clicking on CREATE MODULE, publish your application. Then create the remaining modules: **Field_CS** and **Field_Sync** (both Blank type).

**The foundation application**

For this sample, it's not necessary to have a foundation application due to the simplicity of this example and reusability needs.

#### Implementation

**Architecture patterns**

For this use case, there is an important integration pattern, to retrieve the soccer field information.

![Diagram showing the architectural patterns for retrieving soccer field information.](https://success.outsystems.com/TK_Resource/fbb8cf11-9baf-45b7-90ce-cf17a6ddc395)

Focus on 3 architectural patterns:
1. **Integration with an External System**
2. **Local Replica of data**
3. **Synchronization of data**

**① Integrating with an External Service**

*Consume the external API*

Open the **Field_IS** module, click on the Logic tab, and under the Integrations folder, right-click on the REST node and select **Consume REST API...**. Click on **ADD ALL METHODS** and fill the URL of the swagger file.

![Screenshot of the REST API methods in Service Studio after consuming an external API.](https://success.outsystems.com/TK_Resource/239e75fb-062d-47a3-a32d-452d281dcb9c)

*Send the API Key*

Implement the **OnBeforeRequest** event to send a valid key in the `API_KEY` request header (valid key: **"outsystems_is_awesome"**).

![Screenshot showing the process of adding a new 'OnBeforeRequest' event to send the API key.](https://success.outsystems.com/TK_Resource/201afd21-019f-467f-a151-fde47fbf6815)

![Screenshot of the logic flow for implementing the 'OnBeforeRequest' event in Service Studio.](https://success.outsystems.com/TK_Resource/3f53820a-c4bd-43c3-b3ec-3158bb356534)

*Retrieve a list of Fields with summary data*

Create a structure named **Field_IS_Summary** — the output of an action providing the list of Fields from the external API to upper layers.

![Screenshot showing the 'Field_IS_Summary' structure with attributes like Name, Dimension, and City.](https://success.outsystems.com/TK_Resource/48b6a827-1b15-4cfb-8a0e-35eb4cfe5385)

Create the server action **Field_IS_GetSummaryFields** (Public = Yes).

![Screenshot of the logic flow for creating the 'Field_IS_GetSummaryFields' server action in Service Studio.](https://success.outsystems.com/TK_Resource/7b3e7e96-0b99-4f9b-89c7-a794c4ec65fc)

**② Local Replica of data**

On the **Field_CS**, create a new entity **Field**. Set Public and Expose Read-only properties to Yes.

![Screenshot of the 'Field' entity properties in Service Studio with attributes like Name, Dimension, and City.](https://success.outsystems.com/TK_Resource/c4723be2-e8d3-45e3-9831-312f29b218cf)

Create **Field_Create** and **Field_CreateOrUpdate** CRUD actions.

![Screenshot showing the creation of CRUD actions for the 'Field' entity in Service Studio.](https://success.outsystems.com/TK_Resource/c5b7452e-c8f3-4e81-a268-efb259bc58c6)

**③ Synchronization of data**

The **Field_Sync** module makes the asynchronous synchronization from the external system into the local cached data.

![Screenshot of the logic flow for the synchronization algorithm of field data in Service Studio.](https://success.outsystems.com/TK_Resource/90e1c89d-7057-4f47-b92d-33c934f4d4de)

Synchronization algorithm steps:
1. Set a logic timeout (10 minutes). GetSyncFieldCurrPages gets possible stored progress to continue processing.
2. This if logic guarantees the start and recursion of the sync flow if there's still data to sync.
3. The actual call to the IS module action occurs here, triggering the call to the external system.
4. On this loop, cycle through all the results and insert/update them on the Field Entity.
5. Check if reaching the logic timeout and if there are more records to process.
   - If not timed out and records remain: continue cycling.
   - If timed out: store progress and wake the timer again before ending current execution.

---

## 10. Manage technical debt

AI Mentor Studio (https://aimentorstudio.outsystems.com/) is the OutSystems technical debt monitoring tool. It enables IT leaders to visualize complex cross-portfolio architectures and identify problem areas while also helping developers follow best practices and avoid common pitfalls.

As organizations strive to expedite time-to-market and empower non-professional developers (citizen developers) to create business apps themselves, controlling technical debt naturally becomes a top concern.

> ℹ️ This content covers AI Mentor Studio for technical debt management in O11. For AI powered app generation in ODC, see Build apps with AI.

With AI Mentor Studio, technical debt can be effectively managed at every stage of the development lifecycle so that when departmental applications evolve to become enterprise-wide solutions, nothing needs to be rewritten.

For architects and development team leaders, it provides an integrated, bird's eye view of their organization's technical debt to identify problem areas and prioritize accordingly. Developers can view detailed reports on what best practices are being violated, their impact, and how to fix them.

To integrate AI Mentor Studio's data with third-party tools, use the AI Mentor Studio API.

### What AI Mentor Studio analyzes

AI Mentor Studio analyzes the code produced by developers and provides insights regarding code quality that may impact team agility.

#### Code Analysis

AI Mentor Studio runs a set of predefined rules throughout the produced low-code, with the goal of uncovering code patterns in the following categories:
- **Performance**
- **Architecture**
- **Maintainability**
- **Security**

Check here for the list of patterns currently being analyzed.

#### Architecture auto-classification

AI Mentor Studio is self-sufficient when it comes to analyzing the architecture of the apps in your infrastructure. With the help of an AI engine, AI Mentor Studio analyzes a module's code and its relationships to identify where it fits in the overall architecture.

AI auto-classification allows you to onboard factories into the AI Mentor Studio and classify each module so that it fits into the right architecture layer. Having the right architecture as important as it enables all other code analysis.

By default, all new infrastructures added to AI Mentor Studio use AI auto-classification. You also have the option of turning AI auto-classification off, however, this means that for architecture classification, you must have Discovery installed.

### Access to AI Mentor Studio features

The availability or scope of some features of AI Mentor Studio depend on the OutSystems Edition associated with your infrastructure:

- **Teams**: The ability to filter the app portfolio and technical debt findings per LifeTime team is available for Standard and Enterprise Editions.
- **Date Time Machine**: The ability to check all the history of technical debt analysis is available for Standard and Enterprise Editions. Basic Editions can access one month of technical debt analysis history.

Infrastructures associated with a Free Edition can't use AI Mentor Studio.

### How to access AI Mentor Studio

Before you can start using AI Mentor Studio in your infrastructure and your IT user must be associated with AI Mentor Studio. Learn about the prerequisites and how to set up AI Mentor Studio.

To access AI Mentor Studio, go to https://aimentorstudio.outsystems.com/ and login with your OutSystems account.

![img](https://success.outsystems.com/img/Documentation_UI.Gettingstarted__CANuzgqwWsu4b6Di99qi9w.png?CANuzgqwWsu4b6Di99qi9w)

---

## 10-1. How does AI Mentor Studio work

> ℹ️ Architecture Dashboard is now AI Mentor Studio.

AI Mentor Studio includes the following components:

- **AI Mentor Studio SaaS**: A "Software as a Service" that processes and shows all data collected by the AI Mentor Studio LifeTime Plugin.
- **AI Mentor Studio LifeTime Plugin**: A LifeTime plugin that's published in a platform installation (on-premises or cloud) with environment probes to collect data and communicate with the AI Mentor Studio SaaS.

### Communication

The communication between AI Mentor Studio's components differs depending on your authentication method.

**If you authenticate with your OutSystems account**

Communications between the AI Mentor Studio plugin and the AI Mentor Studio SaaS are always initiated by the plugin. This reduces connectivity requirements on your side since all that needs to be ensured is connectivity from the Plugin in the LifeTime environment to the AI Mentor Studio SaaS endpoint.

![Diagram illustrating the communication flow initiated by the AI Mentor Studio plugin when using OutSystems account authentication.](https://success.outsystems.com/TK_Resource/90f9dd40-ca73-4271-a131-36bf1a187ab1)

**If you authenticate with your IT User account**

AI Mentor Studio needs to be able to connect with your infrastructure to ensure the login is correct. When a user authenticates or accepts the privacy policy, the AI Mentor Studio SaaS needs to communicate with the AI Mentor Studio plugin. Thus, when using IT User authentication, the communication between the AI Mentor Studio SaaS and the AI Mentor Studio plugin is **bidirectional**.

![Diagram showing bidirectional communication between AI Mentor Studio SaaS and the AI Mentor Studio plugin when using IT User account authentication.](https://success.outsystems.com/TK_Resource/3a90cbaa-7cea-40d2-b77e-0b6932a4e071)

With either authentication method, the plugin can use a forward proxy to connect to the AI Mentor Studio SaaS endpoint.

### Data collected in plugin and sent to SaaS

AI Mentor Studio collects the following data from your infrastructure:

- Platform metamodel data, including infrastructure activation code, environments information (name and Platform Server version), teams, list of apps and modules (including name and identifier), and platform configurations.
- Modules and dependency information for code analysis.
- Upon acceptance of the agreement, during AI Mentor Studio set up: users information (name, username, email address, user creation date, last login date) and LifeTime permissions.
- Optionally: Discovery snapshot data (architectural references, applications, and modules) for architecture analysis.

### Data in transit

- The Plugin and SaaS share data in binary format through a well-known HTTPS endpoint.
- IP or DNS addresses aren't transmitted.
- No ports besides the defaults need to be open for the correct use of AI Mentor Studio Probes.
- No firewall issues should arise, although you need to be able to access the endpoint detailed in How to set up AI Mentor Studio.

### Data at rest in SaaS

Data of each installation is kept in the OutSystems cloud and isolated from all other installations using the platform's multi-tenant mechanisms. This ensures data from one installation isn't accessible by users of other installations.

### More information about security and compliance

Read more about security and compliance in the following FAQ sections:
- Security, legal and compliance - registration in AI Mentor Studio
- Security, legal and compliance - personal information

### Permissions

The permissions that IT users have while using AI Mentor Studio with an infrastructure, depend on the role and permissions set in LifeTime for the code-analysis environment of that infrastructure.

Roles and their permissions can be assigned as a default role, as a role for a team, or as role for an app.
- The permissions of a role assigned for a team override the permissions for the team's apps assigned by the default role.
- The permissions of a role assigned for an app override the permissions for the specific app assigned by the default role and of roles assigned for a team.

#### Main features permissions

**View teams**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | No | Assigned team | No |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | | | |
| Change and Deploy Applications | Team apps¹ | Assigned team | No |
| Full Control | | | |

¹ Team apps: Except in cases where the permission level of a specific app is set lower than that assigned to a team.

**View apps and modules**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | All apps¹ | Team apps² | Assigned apps |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | | | |
| Change and Deploy Applications | | | |
| Full Control | | | |

¹ All apps: Except in cases where the permission level of a specific app (or teams assigned to that app) is set lower than that assigned to a role or team.
² Team apps: Except in cases where the permission level of a specific app is set lower than that assigned to a team.

**Open findings report**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | No | | |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | All apps¹ | Team apps² | Assigned apps |
| Change and Deploy Applications | | | |
| Full Control | | | |

**Export findings report**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | No | | |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | All apps¹ | Team apps² | Assigned apps |
| Change and Deploy Applications | | | |
| Full Control | | | |

**Resolve findings**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | No | | |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | No | | |
| Change and Deploy Applications | All apps¹ | Team apps² | Assigned apps |
| Full Control | | | |

**Overview dashboard**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| No Access | No | | |
| List Application | No | | |
| Monitor and Add Dependencies | | | |
| Open and Debug Applications | All apps¹ | Team apps² | Assigned apps |
| Change and Deploy Applications | | | |
| Full Control | | | |

#### Maintenance and operations permissions

**Ignore modules**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| Open and Debug Applications or lower | No | | |
| Change and Deploy Applications | All modules¹ | Team modules² | Assigned app's modules |

**Enable AI auto-classification**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| Open and Debug Applications or lower | No | | |
| Change and Deploy Applications | Yes | No | |
| Full Control | | | |

**Override module AI auto-classification**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| Open and Debug Applications or lower | No | | |
| Change and Deploy Applications | All modules¹ | No | |
| Full Control | | | |

**Update probes**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| Open and Debug Applications or lower | No | | |
| Change and Deploy Applications | Yes | No | |
| Full Control | | | |

**Manage AI Mentor Studio API**

| LifeTime permissions | Default role | Assigned to a team | Assigned to an app |
|----------------------|-------------|-------------------|-------------------|
| Open and Debug Applications or lower | No | | |
| Change and Deploy Applications | Yes | No | |
| Full Control | | | |
