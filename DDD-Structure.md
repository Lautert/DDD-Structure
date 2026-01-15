{Entity}/
├── _rules/                                       # 📜 Documentação e regras do módulo (não faz parte do DDD, mas auxilia na governança)
│   └── rules.md
|
├── adapter/
│   ├── in/                                       # ⬇️ INPUT ADAPTERS (Driving Adapters) - Recebem requisições externas
│   │   └── web/                                  # 🌐 REST API Adapter - Implementação HTTP/REST para entrada de dados
│   │       └── {Entity}Controller.java           #    Controller Spring que traduz HTTP requests para chamadas de Use Cases
│   │
│   └── out/                                      # ⬆️ OUTPUT ADAPTERS (Driven Adapters) - Implementam interfaces de saída
│       ├── event/                                # 📡 Event Publishing Adapter - Publicação de eventos de domínio
│       │   │                                     #    Implementação de mensageiria (Kafka, RabbitMQ, etc)
│       │   └── SpringDomainEventPublisher.java   #    Implementação Spring do publicador de eventos
│       │
│       ├── persistence/                          # 💾 Persistence Adapter - Implementação JPA/Hibernate dos repositórios
│       │   │                                     #    A "saída" para o banco de dados é apenas uma das possíveis saídas,
│       │   └── Jpa{Entity}RepositoryAdapter.java
│       │                                         
│       └── external                              # 🌐 External Systems Adapter - Integração com sistemas externos
│           │                                     #    Sistemas de pagamento, APIs externas, notificações, Email Advisor.
│
│
├── application/                                  # 🎬 APPLICATION LAYER - Orquestra o fluxo da aplicação
│   │
│   ├── dto/                                      # 📦 Data Transfer Objects - Objetos para transferência de dados entre camadas
│   │   │                                         #    Desacoplam a representação externa dos objetos de domínio
│   │   │                                         #    Estruturas simples, sem lógica de negócio
│   │   │                                         #    Pode apresentar validações muito simples 
│   │   │                                         #    (ex: campos obrigatórios, formatos básicos, tamanho máximo)
│   │   │
│   │   ├── request/                              # 📥 Request DTOs - Dados de entrada da API
│   │   │   │                                     #    Validações de formato e estrutura (não regras de negócio)
│   │   │
│   │   ├── response/                             # 📤 Response DTOs - Dados de saída da API
│   │   │   │                                     #    Formatação de resposta para o cliente
│   │   │
│   │   └── shared/                               # 🔄 Shared DTOs - DTOs reutilizáveis entre request/response
│   │       │                                     #    Estruturas comuns compartilhadas
│   │
│   ├── interfaces/                               # 📋 Service Interfaces - Contratos dos casos de uso (Application Services)
│   │   │                                         #    Define o que a aplicação pode fazer (Use Cases)
│   │   └── I{Entity}Service.java
│   │
│   ├── mapper/                                   # 🔀 Mappers - Conversão entre DTOs e Entidades
│   │   │                                         #    Responsável pela tradução entre camadas (anti-corruption)
│   │   └── {Entity}Mapper.java
│   │
│   └── usecase/                                  # ⚡ Use Cases (Application Services) - Implementação dos casos de uso
│       │                                         #    Orquestra o domínio, transações e chamadas a serviços externos
│       │                                         #    Um Use Case = Uma ação que o usuário pode executar
│       │                                         #    Responsável por filtrar permissões, autenticação e validar **regras de negócio simples**
│       │                                         #    Realiza conversões entre DTOs e Entidades
│       └── CreateOrderUseCase.java
│
├── domain/                                       # 💎 DOMAIN LAYER - Coração do DDD (Core Business Logic)
│   │                                             #    Contém TODA a lógica de negócio e regras do domínio
│   │                                             #    INDEPENDENTE de frameworks, BD e infraestrutura
│   │                                             #    Ele não se importa com autenticação, apenas aplicar a regra de negócio
│   │                                             #    Ele recebe os dados processa gera um output, mas não se importa com a 
│   │                                             #    origem ou destino desses dados
│   │
│   ├── event/                                    # 📢 Domain Events - Eventos que representam fatos de negócio
│   │   │                                         #    Comunicação assíncrona entre agregados/bounded contexts
│   │   │                                         #    "Algo importante aconteceu no domínio"
│   │   └── OrderCreatedEvent.java                #    Exemplo de evento de domínio
│   │
│   ├── exception/                                # ❌ Domain Exceptions - Exceções específicas do domínio
│   │   │                                         #    Representam violações de regras de negócio
│   │   ├── DomainException.java                  #    Exceção base para erros de domínio
│   │   ├── OrderNotFoundException.java           #    Exemplo de exceção específica
│   │   └── InvalidOrderException.java            #    Exemplo de exceção específica
│   │
│   ├── factory/                                  # 🏭 Domain Factories - Criação complexa de agregados
│   │   │                                         #    Encapsula lógica de criação de objetos complexos
│   │   │                                         #    Garante que agregados sejam criados em estado válido
│   │   └── {Entity}Factory.java
│   │
│   ├── interfaces/                               # 📜 Domain Service Interfaces - Contratos de serviços de domínio
│   │   │                                         #    Define operações que não pertencem naturalmente a uma entidade
│   │   ├── I{Entity}{SubType}DomainService.java
│   │   └── I{Entity}DomainService.java
│   │
│   ├── model/                                    # 📊 Domain Model - Entidades e Value Objects
│   │   │                                         #    Representação do modelo de negócio em código
│   │   │
│   │   ├── CandidateEntity.java                  # 🔑 AGGREGATE ROOT - Entidade principal do agregado
│   │   │                                         #    Ponto de entrada único para modificações no agregado
│   │   │                                         #    Garante consistência e invariantes do agregado
│   │   │
│   │   └── valueobject/                          # 💠 Value Objects - Objetos imutáveis sem identidade 
│   │       │                                     #    (DateRange, Email, Phone, Salary ...)
│   │       │                                     #    Definidos apenas por seus atributos
│   │       │                                     #    Validações e comportamentos relacionados aos atributos
│   │       ├── Email.java                        #    Email com validação de formato
│   │       ├── PersonalInfo.java                 #    Informações pessoais agrupadas
│   │       └── SocialNetworkProfile.java         #    Perfil de rede social
│   │
│   ├── repository/                               # 🗄️ Repository Interfaces - Contratos de persistência
│   │   │                                         #    Define como o domínio acessa dados persistidos
│   │   │                                         #    PORTS (interfaces) - implementação está nos Adapters
│   │   │                                         #    Não se deve criar vínculos com JPA/Hibernate aqui, lembre-se:
│   │   │                                         #    o domínio deve ser agnóstico a detalhes de infraestrutura
│   │   │
│   │   └── port/                                 # 🔌 Repository Ports - Hexagonal Architecture Ports for Repositories
│   │       │                                     #    Definem o contrato que os adapters devem implementar
│   │       └── {Entity}RepositoryPort.java
│   │
│   ├── service/                                  # ⚙️ Domain Services - Operações de domínio sem dono natural
│   │   │                                         #    Contém lógica de negócio que não pertence a uma entidade específica
│   │   │                                         #    Opera sobre múltiplas entidades ou agregados
│   │   ├── {Entity}DomainService.java
│   │   │
│   │   └── business/                             # 💼 Business Services - Serviços de negócio complexos
│   │       │                                     #    Coordenam múltiplos serviços de domínio
│   │       ├── CandidateMatchingService.java     #    Serviço de matching de candidatos
│   │       └── ProfileCompletionService.java     #    Serviço de completude de perfil
│   │
│   ├── policy/                                   # 📏 Domain Policies - Regras de negócio configuráveis
│   │   │                                         #    Encapsulam regras que podem variar por contexto
│   │   │                                         #    Estratégias de negócio (Strategy Pattern aplicado ao domínio)
│   │   └── {Entity}Policy.java
│   │
│   └── specification/                            # 🔍 Specifications - Regras de negócio encapsuladas
│       │                                         #    Specification Pattern - predicados composicionais
│       │                                         #    Permite combinar regras com AND, OR, NOT
│       │                                         #    Reutilizáveis em queries e validações
│       ├── SalaryInRangeSpecification.java       #    Spec: Salário está na faixa?
│       └── VisibleToCompaniesSpecification.java  #    Spec: Visível para empresas?
│
└── types/
    ├── ECandidateEducationLevel.java            # Education level enum (13 values)
    └── ECandidateLanguageProficiency.java       # Language proficiency enum (7 values)
