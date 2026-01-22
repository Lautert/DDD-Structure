## Key Principles from the Guide
- DDD Structure Overview: Each module represents a bounded context (e.g., user, profile) with its own domain, application, and adapter layers. Modules should be loosely coupled, with dependencies flowing inward toward the domain layer (the core business logic).

- Layer Independence: The domain layer must remain completely infrastructure-independent (no JPA, Spring, or HTTP dependencies). The application layer orchestrates domain operations and handles cross-cutting concerns (e.g., transactions, events). The adapter layer deals with external concerns (e.g., persistence, web).

- Dependency Direction: Higher layers (application/adapter) can depend on lower layers (domain), but not vice versa. Inter-module dependencies should prioritize domain abstractions to avoid tight coupling.

- Hexagonal Architecture: Modules communicate via ports (interfaces in the domain layer), allowing the domain to be driven by or drive external adapters without knowing implementation details.

## How Modules Should Interact (Inter-Module Dependencies)
In DDD, when one module (e.g., user) needs to use functionality from another (e.g., profile), it should not directly depend on the other module's application or adapter layers. Instead, it should depend on the domain layer of the other module to maintain architectural integrity. This ensures:

- The domain remains the core and independent.
- Changes in application logic (e.g., REST endpoints) don't ripple across modules.
- Testability is preserved (you can mock domain services easily).

**❌ (Think): If a domain extends other module application/adapter it still a pure domain?**


### Cross-Cutting Concerns (Policies, Events)
- If access control (policies) or events are needed, handle them in the application layer of the calling module (e.g., UserUseCase can apply policies before calling UserDomainService).
- The guide states: "❌ Access control in Domain - Use Policies in Application layer."

## Layer Hierarchy

1. application
2. adapter
3. domain

**IMPORTANT: Higher layers (application/adapter) can depend on lower layers (domain) but not vice versa**

---

## DDD Structure

<pre>
{Entity}/
├── _rules/                                       # 📜 Module documentation and rules (not part of DDD, but helps with governance)
│   └── rules.md
|
├── adapter/
│   ├── in/                                       # ⬇️ INPUT ADAPTERS (Driving Adapters) - Receive external requests
│   │   └── web/                                  # 🌐 REST API Adapter - HTTP/REST implementation for data input
│   │       └── {Entity}Controller.java           #    Spring Controller that translates HTTP requests to Use Case calls
│   │
│   └── out/                                      # ⬆️ OUTPUT ADAPTERS (Driven Adapters) - Implement output interfaces
│       ├── event/                                # 📡 Event Publishing Adapter - Domain event publishing
│       │   │                                     #    Messaging implementation (Kafka, RabbitMQ, etc)
│       │   └── SpringDomainEventPublisher.java   #    Spring implementation of event publisher
│       │
│       ├── persistence/                          # 💾 Persistence Adapter - JPA/Hibernate implementation of repositories
│       │   │                                     #    The "output" to the database is just one of the possible outputs,
│       │   └── Jpa{Entity}RepositoryAdapter.java
│       │                                         
│       └── external                              # 🌐 External Systems Adapter - Integration with external systems
│           │                                     #    Payment systems, external APIs, notifications, Email Advisor.
│
│
├── application/                                  # 🎬 APPLICATION LAYER - Orchestrates application flow
│   │
│   ├── dto/                                      # 📦 Data Transfer Objects - Objects for data transfer between layers
│   │   │                                         #    Decouple external representation from domain objects
│   │   │                                         #    Simple structures, without business logic
│   │   │                                         #    May present very simple validations 
│   │   │                                         #    (e.g., required fields, basic formats, maximum size)
│   │   │
│   │   ├── request/                              # 📥 Request DTOs - API input data
│   │   │   │                                     #    Format and structure validations (not business rules)
│   │   │
│   │   ├── response/                             # 📤 Response DTOs - API output data
│   │   │   │                                     #    Response formatting for the client
│   │   │
│   │   └── shared/                               # 🔄 Shared DTOs - Reusable DTOs between request/response
│   │       │                                     #    Shared common structures
│   │
│   ├── interfaces/                               # 📋 Service Interfaces - Use case contracts (Application Services)
│   │   │                                         #    Defines what the application can do (Use Cases)
│   │   └── I{Entity}Service.java
│   │
│   ├── mapper/                                   # 🔀 Mappers - Conversion between DTOs and Entities
│   │   │                                         #    Responsible for translation between layers (anti-corruption)
│   │   └── {Entity}Mapper.java
│   │
│   └── usecase/                                  # ⚡ Use Cases (Application Services) - Use case implementation
│       │                                         #    Orchestrates domain, transactions and external service calls
│       │                                         #    One Use Case = One action the user can execute
│       │                                         #    Responsible for filtering permissions, authentication and validating **simple business rules**
│       │                                         #    Performs conversions between DTOs and Entities
│       └── CreateOrderUseCase.java
│
├── domain/                                       # 💎 DOMAIN LAYER - Heart of DDD (Core Business Logic)
│   │                                             #    Contains ALL business logic and domain rules
│   │                                             #    INDEPENDENT of frameworks, DB and infrastructure
│   │                                             #    It doesn't care about authentication, only applying the rule of what it represents
│   │                                             #    It receives data, processes and generates output, but doesn't care about the 
│   │                                             #    origin or destination of this data
│   │
│   ├── event/                                    # 📢 Domain Events - Events that represent business facts
│   │   │                                         #    Asynchronous communication between aggregates/bounded contexts
│   │   │                                         #    "Something important happened in the domain"
│   │   └── OrderCreatedEvent.java                #    Example of domain event
│   │
│   ├── exception/                                # ❌ Domain Exceptions - Domain-specific exceptions
│   │   │                                         #    Represent business rule violations
│   │   ├── DomainException.java                  #    Base exception for domain errors
│   │   ├── OrderNotFoundException.java           #    Example of specific exception
│   │   └── InvalidOrderException.java            #    Example of specific exception
│   │
│   ├── factory/                                  # 🏭 Domain Factories - Complex aggregate creation
│   │   │                                         #    Encapsulates complex object creation logic
│   │   │                                         #    Ensures aggregates are created in valid state
│   │   └── {Entity}Factory.java
│   │
│   ├── interfaces/                               # 📜 Domain Service Interfaces - Domain service contracts
│   │   │                                         #    Defines operations that don't naturally belong to an entity
│   │   ├── I{Entity}{SubType}DomainService.java
│   │   └── I{Entity}DomainService.java
│   │
│   ├── model/                                    # 📊 Domain Model - Entities and Value Objects
│   │   │                                         #    Business model representation in code
│   │   │
│   │   ├── CandidateEntity.java                  # 🔑 AGGREGATE ROOT - Main aggregate entity
│   │   │                                         #    Single entry point for modifications in the aggregate
│   │   │                                         #    Ensures aggregate consistency and invariants
│   │   │
│   │   └── valueobject/                          # 💠 Value Objects - Immutable objects without identity 
│   │       │                                     #    (DateRange, Email, Phone, Salary ...)
│   │       │                                     #    Defined only by their attributes
│   │       │                                     #    Validations and behaviors related to attributes
│   │       ├── Email.java                        #    Email with format validation
│   │       ├── PersonalInfo.java                 #    Grouped personal information (Validations "in constructor")
│   │       └── SocialNetworkProfile.java         #    Social network profile
│   │
│   ├── repository/                               # 🗄️ Repository Interfaces - Persistence contracts
│   │   │                                         #    Defines how the domain accesses persisted data
│   │   │                                         #    PORTS (interfaces) - implementation is in Adapters
│   │   │                                         #    Do not create links with JPA/Hibernate here, remember:
│   │   │                                         #    the domain must be agnostic to infrastructure details
│   │   │
│   │   └── port/                                 # 🔌 Repository Ports - Hexagonal Architecture Ports for Repositories
│   │       │                                     #    Define the contract that adapters must implement
│   │       └── {Entity}RepositoryPort.java
│   │
│   ├── service/                                  # ⚙️ Domain Services - Domain operations without natural owner
│   │   │                                         #    Contains business logic that doesn't belong to a specific entity
│   │   │                                         #    Operates on multiple entities or aggregates
│   │   ├── {Entity}DomainService.java
│   │   │
│   │   └── business/                             # 💼 Business Services - Complex business services
│   │       │                                     #    Coordinate multiple domain services
│   │       ├── CandidateMatchingService.java     #    Candidate matching service
│   │       └── ProfileCompletionService.java     #    Profile completion service
│   │
│   ├── policy/                                   # 📏 Domain Policies - Configurable business rules
│   │   │                                         #    Encapsulate rules that may vary by context
│   │   │                                         #    Business strategies (Strategy Pattern applied to domain)
│   │   └── {Entity}Policy.java
│   │
│   └── specification/                            # 🔍 Specifications - Encapsulated business rules
│       │                                         #    Specification Pattern - compositional predicates
│       │                                         #    Allows combining rules with AND, OR, NOT
│       │                                         #    Reusable in queries and validations
│       ├── SalaryInRangeSpecification.java       #    Spec: Is salary in range?
│       └── VisibleToCompaniesSpecification.java  #    Spec: Visible to companies?
│
└── types/
    ├── ECandidateEducationLevel.java            # Education level enum (13 values)
    └── ECandidateLanguageProficiency.java       # Language proficiency enum (7 values)
</pre>

---

## Domain example

```java
// In user module's domain service (e.g., UserDomainService.java)
@Service
public class UserDomainService {

    private final IProfileDomainService profileDomainService;  // Import interface from profile module's domain layer

    @Autowired
    public UserDomainService(ProfileDomainService profileDomainService) {
        this.profileDomainService = profileDomainService;
    }

    public UserEntity registerUser(String email, String profileData) {
        // Domain logic for user registration
        UserEntity user = new UserEntity(email);
        
        // Call profile domain service for profile-related operations
        ProfileEntity profile = profileDomainService.createProfile(profileData);
        
        // Associate and save user
        user.setProfile(profile);
        return repository.save(user);
    }
}
```

```java
// DON'T DO THIS - Violates layer boundaries
@Service
public class UserDomainService {

    private final ProfileUseCase profileUseCase;  // Wrong: Application layer dependency

    // This would force user domain to know about DTOs, mappers, etc.
}
``