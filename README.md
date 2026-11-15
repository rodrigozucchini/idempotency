0. Cómo encarar el problema
0.1 Framework de 4 pasos: entender el problema, diseño de alto nivel, deep dive, cierre
0.2 Requisitos funcionales vs no funcionales
0.3 Capacity estimation y back-of-envelope: QPS, storage, bandwidth, potencias de 2, tabla de latencias
0.4 SLA / SLO / SLI, los “nueves” de disponibilidad
0.5 Trade-offs y cómo justificarlos
0.6 Ciclo de vida del diseño (SDLC), HLD vs LLD, diagramas de HLD
1. Estilos de arquitectura
1.1 Monolito, microservicios, monolito modular
1.2 Event-driven, serverless, pub/sub
1.3 Stateless vs stateful
1.4 Request-driven vs event-driven, message-driven vs event-driven
1.5 Arquitectura hexagonal / clean architecture, DDD (bounded contexts)
1.6 Containerización y orquestación (Docker, Kubernetes)
2. Redes y comunicación
2.1 IP, OSI, TCP vs UDP, handshake
2.2 DNS, caching DNS, TTL
2.3 HTTP/1.1, HTTP/2, HTTP/3 (QUIC), keep-alive
2.4 REST, GraphQL, gRPC, SOAP, diseño de APIs (versionado, paginación, idempotencia)
2.5 WebSockets, long/short polling, SSE, webhooks
2.6 Proxies: forward vs reverse
2.7 API Gateway, BFF
2.8 CDN, edge caching, CDN vs edge server
2.9 TLS/SSL, mTLS
3. Escalabilidad y balanceo de carga
3.1 Escalado vertical vs horizontal
3.2 Cuellos de botella de escalabilidad
3.3 Load balancers L4 vs L7, algoritmos (round robin, least connections, IP hash, weighted)
3.4 Stateful vs stateless load balancing, load balancing vs failover
3.5 Consistent hashing
3.6 Auto scaling, health checks, sticky sessions
3.7 Concurrencia vs paralelismo, thread pools, async I/O
3.8 Escalar de 0 a millones de usuarios
4. Bases de datos y almacenamiento
4.1 SQL vs NoSQL, cuándo elegir cada uno
4.2 Tipos: relacional, key-value, documento, columnar, grafo, time-series, vectoriales
4.3 Normalización y desnormalización
4.4 Índices (B-Tree, LSM-tree, hash), query planning, optimización de queries
4.5 Transacciones, ACID vs BASE, niveles de aislamiento, MVCC, deadlocks
4.6 Replicación (master-slave, multi-master, sync/async, leaderless)
4.7 Sharding y particionamiento, resharding, hot keys
4.8 Block vs object vs file storage
4.9 OLTP vs OLAP, data warehouse, data lake
4.10 Búsqueda: índice invertido, Elasticsearch
4.11 Redis y estructuras in-memory
4.12 Serialización (JSON, Protobuf, Avro) y evolución de esquemas
4.13 Connection pooling, read replicas, migraciones sin downtime
5. Caching
5.1 Estrategias: cache-aside, read-through, write-through, write-back, write-around
5.2 Políticas de eviction (LRU, LFU, FIFO, TTL)
5.3 Cache distribuido y su diseño
5.4 Cold vs warm cache
5.5 Invalidación de cache
5.6 Problemas: stampede, penetration, avalanche, hot keys
5.7 Caché en cliente, CDN, app y DB
6. Mensajería y procesamiento asíncrono
6.1 Message queues, pub/sub
6.2 Kafka, RabbitMQ, SQS: diferencias y cuándo usar cada uno
6.3 Semánticas de entrega (at-most-once, at-least-once, exactly-once)
6.4 Idempotencia, deduplicación
6.5 Dead letter queues, retries, backpressure
6.6 Orden de mensajes, particiones, consumer groups
6.7 Event sourcing, event streaming, CQRS
6.8 Manejo de errores y recuperación de estado en EDA
6.9 Transactional outbox, CDC
6.10 Batch vs stream processing (MapReduce, Spark, Flink)
6.11 Job schedulers y cron distribuido
7. Sistemas distribuidos
7.1 Introducción y problemas de diseño (fallas parciales, latencia, concurrencia)
7.2 Disponibilidad, consistencia, confiabilidad, mantenibilidad
7.3 Teorema CAP, PACELC
7.4 Modelos de consistencia: strong, eventual, causal, read-your-writes
7.5 Consenso: Paxos, Raft, Zab
7.6 Leader election, quorum, split brain
7.7 Relojes: NTP, relojes lógicos, vector clocks, Lamport
7.8 Transacciones distribuidas: 2PC, 3PC, Saga
7.9 Locks distribuidos (Redlock, fencing tokens, ZooKeeper/etcd)
7.10 Gossip protocol, failure detection, heartbeat
7.11 Anti-entropy: Merkle trees, hinted handoff, read repair
7.12 CRDTs
7.13 Service discovery y coordinación (ZooKeeper, etcd, Consul)
7.14 Comunicación segura en sistemas distribuidos
8. Resiliencia y tolerancia a fallos
8.1 Fault tolerance, redundancia, failover
8.2 Timeouts, retries con exponential backoff + jitter
8.3 Circuit breaker, bulkhead, load shedding, graceful degradation
8.4 Rate limiting y algoritmos (token bucket, leaky bucket, fixed/sliding window)
8.5 Backup y disaster recovery (RPO/RTO), multi-AZ, multi-región, active-active vs active-passive
8.6 Chaos engineering
9. Seguridad
9.1 Medidas de seguridad generales, SSDLC
9.2 Autenticación vs autorización
9.3 Sesiones, JWT, OAuth 2.0, OIDC, SSO, API keys
9.4 RBAC / ABAC
9.5 Cifrado en tránsito y en reposo, gestión de secretos, hashing de passwords
9.6 OWASP Top 10 (SQLi, XSS, CSRF), DDoS, WAF
9.7 Privacidad y cumplimiento (GDPR, PCI-DSS para pagos)
10. Observabilidad, testing y operaciones
10.1 Logging, métricas, alertas, dashboards
10.2 Distributed tracing
10.3 Unit, integration, load y stress testing
10.4 CI/CD
10.5 Estrategias de deploy: rolling, blue-green, canary, feature flags
10.6 Infrastructure as Code, GitOps
10.7 Service mesh, sidecar
10.8 Costo vs performance, estimación de costos, FinOps
10.9 Optimización de performance, latencia vs throughput
11. Estructuras de datos y algoritmos específicos
11.1 Bloom filter, HyperLogLog, Count-Min Sketch
11.2 Trie (autocomplete)
11.3 Geohash, quadtree, S2 / H3
11.4 Generadores de IDs únicos (Snowflake, UUID, ULID)
11.5 Base62 / hashing para URL shortener
11.6 Consistent hashing ring con virtual nodes
11.7 LSM-tree, SSTable, skip list
11.8 Índice invertido, ranking
11.9 DSA para system design
12. Low Level Design (LLD)
12.1 OOP, modularidad e interfaces
12.2 SOLID, DRY, KISS, YAGNI
12.3 Patrones creacionales, estructurales y de comportamiento (los 23 de GoF)
12.4 UML: clases, secuencia, actividad, componentes
12.5 OOAD, concurrencia a nivel de código (locks, thread safety)
12.6 Problemas clásicos de LLD: parking lot, ascensor, biblioteca, cajero, tic-tac-toe, rate limiter, LRU cache
13. Casos de diseño (para practicar)

Vol. 1

13.1 Rate limiter
13.2 Key-value store
13.3 Generador de IDs únicos
13.4 URL shortener
13.5 Web crawler
13.6 Notification system
13.7 News feed
13.8 Chat system
13.9 Search autocomplete
13.10 YouTube
13.11 Google Drive

Vol. 2

13.12 Proximity service (Yelp)
13.13 Nearby friends
13.14 Google Maps
13.15 Distributed message queue
13.16 Metrics monitoring
13.17 Ad click aggregation
13.18 Hotel reservation
13.19 Distributed email
13.20 S3-like object storage
13.21 Real-time leaderboard
13.22 Payment system
13.23 Digital wallet
13.24 Stock exchange

De GeeksforGeeks

13.25 Dropbox
13.26 Twitter
13.27 Netflix
13.28 Uber
13.29 BookMyShow
13.30 Facebook Messenger

Extras comunes

13.31 WhatsApp
13.32 Instagram
13.33 TikTok
13.34 Pastebin
13.35 Ticketmaster
13.36 Airbnb
13.37 E-commerce (carrito + checkout + inventario)
13.38 Sistema de métodos de pago
13.39 Distributed lock service
13.40 Task scheduler
13.41 Sistema de feature flags
13.42 API gateway
13.43 Logging centralizado
13.44 Web analytics
13.45 Sistema de votos/likes
13.46 Sistema de reservas con concurrencia
14. Temas modernos que casi nadie lista
14.1 Arquitectura para IA/LLM: RAG, vector DBs, embeddings, inference serving, caché de prompts, colas para jobs de IA, rate limiting por tokens
14.2 Multi-tenancy
14.3 Event-driven con CDC y data pipelines
14.4 A/B testing y experimentación
14.5 Edge computing y serverless a escala
14.6 Cloud nativo (sidecar, strangler fig, ambassador, saga, bulkhead)
14.7 Idempotencia y exactly-once en pagos
15. Pagos y sistemas financieros

Fundamentos del dominio

15.1 Actores: cliente, comerciante, adquirente, emisor, red de tarjetas, procesador, gateway, PSP
15.2 Flujo de una tarjeta: autorización, captura, clearing, settlement
15.3 Autorización vs captura (pre-autorizaciones)
15.4 Reembolsos, contracargos y disputas
15.5 Métodos de pago: tarjeta, transferencia, débito directo, billeteras, QR, efectivo, BNPL, cripto
15.6 Pagos locales (Argentina): Mercado Pago, transferencias 3.0 / CVU-CBU, QR interoperable, Debin, cuotas
15.7 Multimoneda, conversión de divisas y tipos de cambio
15.8 Pagos recurrentes y suscripciones (billing, reintentos, dunning)
15.9 Pagos a terceros: marketplaces, split payments, payouts

Diseño de un sistema de pagos

15.10 Payment service, executor y orchestrator
15.11 Integración con PSPs (Stripe, Adyen, Mercado Pago), hosted page vs integración directa
15.12 Idempotencia con idempotency keys (evitar doble cobro)
15.13 Máquina de estados del pago
15.14 Webhooks: confirmación, firmas, reintentos y orden de eventos
15.15 Reconciliación contra el PSP y el banco
15.16 Timeouts y estados desconocidos (¿se cobró o no?)
15.17 Reintentos seguros y exactly-once efectivo
15.18 Consistencia entre servicios: Saga, outbox, 2PC y sus límites
15.19 Smart routing y fallback entre PSPs

Ledger y contabilidad

15.20 Ledger y contabilidad de partida doble
15.21 Datos inmutables y append-only, event sourcing aplicado a dinero
15.22 Montos como enteros en centavos, nunca float; redondeo
15.23 Saldos, holds / reservas, disponible vs contable
15.24 Auditoría y trazabilidad

Billetera digital

15.25 Transferencias entre cuentas con consistencia fuerte
15.26 Transacciones distribuidas y sharding de cuentas
15.27 Prevención de doble gasto, bloqueos y concurrencia
15.28 Límites, KYC y AML

Seguridad y cumplimiento

15.29 PCI-DSS y reducción de scope
15.30 Tokenización, cifrado, HSM, gestión de claves
15.31 3D Secure / SCA
15.32 Detección de fraude: reglas, scoring, ML, velocity checks
15.33 KYC, AML, regulaciones locales (BCRA, UIF)
15.34 Protección y retención de datos financieros

Operación y confiabilidad

15.35 Alta disponibilidad y degradación controlada
15.36 Rate limiting y protección contra card testing
15.37 Monitoreo: tasa de éxito, latencia, contracargos, alertas por PSP
15.38 Disaster recovery, auditorías y sandbox

Casos de diseño de pagos

15.39 Payment system (checkout con PSP)
15.40 Digital wallet
15.41 Suscripciones y billing
15.42 Marketplace con split payments y payouts
15.43 Reembolsos y contracargos
15.44 Motor de detección de fraude
15.45 Reconciliación diaria
15.46 Stock exchange / matching engine
15.47 E-commerce con carrito, stock y pago atómico
