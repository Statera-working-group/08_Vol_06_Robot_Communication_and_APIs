**Volume 06 Robot Communication and APIs**


# 05. Message Brokers

##  

## 05.01 Message Broker Concepts and Major Product Comparison

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

A message broker is middleware that receives messages from producers, manages their delivery, and forwards them to one or more consumers. Instead of requiring every robot, service, database, and application to communicate directly, the broker creates an intermediary communication layer. This reduces point-to-point dependencies and allows distributed robotic systems to evolve without tightly coupling every component.

The fundamental broker model separates message production from message consumption. A producer publishes commands, telemetry, events, alarms, or application data without needing detailed knowledge of the receiving application. Consumers independently subscribe to or retrieve the information they require. This temporal and structural decoupling is especially useful when robots, edge computers, fleet servers, and cloud services operate at different processing rates.

Message brokers commonly implement queue-based messaging, publish-subscribe messaging, or persistent event streaming. A queue normally distributes messages among competing consumers, while publish-subscribe systems deliver information to multiple interested subscribers. Event-streaming platforms retain ordered records so consumers can process or replay them independently. These patterns may coexist within a modern robotic communication architecture.

Reliability depends on how a broker handles acknowledgement, persistence, retry, ordering, duplication, and failure recovery. Some applications prioritize extremely low latency and accept limited durability, while others require messages to survive process or server failures. Robot telemetry may tolerate occasional loss, whereas mission transitions, safety-related events, maintenance records, and operational transactions often require stronger delivery guarantees and durable storage.

Apache Kafka is designed primarily as a distributed event-streaming platform rather than a traditional transient message queue. Messages are stored as ordered records within partitioned topics, and consumers maintain offsets describing their processing position. This architecture supports high throughput, horizontal scalability, replay, and long-lived event histories, making Kafka attractive for fleet telemetry, analytics pipelines, digital twins, and large-scale robot data platforms.

RabbitMQ follows a messaging-oriented architecture centered on queues, exchanges, bindings, acknowledgements, and routing rules. Producers can publish messages to exchanges that route them toward appropriate queues using direct, topic, fanout, or header-based mechanisms. Its flexible routing and mature delivery semantics make RabbitMQ suitable for enterprise workflows, job distribution, command processing, backend integration, and applications requiring explicit queue behavior.

NATS emphasizes lightweight operation, simple communication semantics, and very low messaging overhead. Core NATS provides high-performance publish-subscribe and request-reply communication, while JetStream adds persistence, message replay, acknowledgements, consumer state, and stream management. This combination is useful for edge systems and distributed robotics where fast service communication is required but selected data must also be stored reliably.

MQTT brokers such as Eclipse Mosquitto, EMQX, and HiveMQ focus on lightweight publish-subscribe communication using hierarchical topics. MQTT provides QoS levels, retained messages, persistent sessions, and mechanisms appropriate for intermittently connected devices. In robotics, MQTT is particularly effective for telemetry, device state, fleet monitoring, IoT integration, and communication across constrained or unreliable wireless networks.

Redis Streams provides another approach by extending an in-memory data platform with persistent stream structures, consumer groups, message identifiers, and acknowledgement mechanisms. It can be effective when applications already depend heavily on Redis and require fast local event processing. However, architectural decisions should distinguish Redis Streams from dedicated large-scale streaming systems because their operational models, persistence strategies, and scaling characteristics differ.

The products therefore address overlapping but different design priorities. Kafka emphasizes durable distributed event logs and large-scale streaming, RabbitMQ emphasizes queues and sophisticated message routing, NATS emphasizes lightweight high-speed messaging with optional persistence through JetStream, and MQTT brokers emphasize efficient device-oriented publish-subscribe communication. Selecting among them should begin with communication semantics rather than product popularity.

Throughput and latency must also be interpreted according to workload characteristics. Kafka can achieve very high aggregate throughput by batching records and distributing partitions across brokers, but its architecture is not intended to replace every low-latency control channel. NATS can provide efficient messaging for small, frequent messages, while RabbitMQ offers rich routing at additional processing cost. MQTT minimizes protocol overhead for device and telemetry communication.

Scalability involves more than increasing message rate. Engineers must consider partitioning, clustering, replication, consumer scaling, connection counts, message size, retention periods, network bandwidth, and failure domains. A broker supporting millions of small telemetry messages may behave differently when carrying large sensor payloads. Cameras, LiDAR point clouds, and large AI artifacts are therefore often stored externally while brokers transport metadata, references, and events.

Robotic systems also require careful separation between messaging and deterministic control. A message broker is well suited to mission events, telemetry, fleet coordination, diagnostics, cloud synchronization, and asynchronous workflows, but it should not automatically become the transport for hard real-time motor or safety loops. Such loops usually require deterministic buses, real-time middleware, shared-memory mechanisms, or other communication paths with bounded timing behavior.

Security becomes critical because a broker can connect many robots and services through a common infrastructure. Production systems should therefore consider TLS encryption, mutual authentication, credentials, access-control lists, topic or queue authorization, network segmentation, audit logging, and credential rotation. Broker permissions should follow least-privilege principles so that telemetry publishers cannot automatically obtain authority to publish commands or modify operational streams.

High availability introduces additional architectural decisions. Broker clusters may replicate data and distribute clients across multiple nodes, but replication alone does not guarantee continuous service. Applications must define reconnection behavior, retry policies, duplicate handling, idempotent processing, backpressure, and degraded-operation strategies. Edge robots should also define what happens when the central broker or network becomes temporarily unavailable.

Backpressure is particularly important in physical systems because data generation can exceed downstream processing capacity. A fleet of robots may continuously produce localization, battery, perception, diagnostic, and mission events even when analytics services are overloaded. Brokers can buffer or retain data, but retention is finite. Systems therefore need explicit policies for prioritization, sampling, expiration, aggregation, and controlled dropping of noncritical information.

A practical robot architecture may use several messaging technologies simultaneously. MQTT can connect robots and IoT devices to fleet infrastructure, NATS can support lightweight internal service communication, RabbitMQ can coordinate enterprise workflows, and Kafka can collect durable operational events for analytics and historical processing. This is not necessarily unnecessary duplication when each broker serves a clearly defined communication plane and operational requirement.

Broker selection should consequently evaluate delivery semantics, persistence, ordering, replay, routing flexibility, latency, throughput, connection scale, operational complexity, ecosystem integration, security, and failure recovery together. The correct question is not which broker is universally fastest or most powerful, but which messaging model best matches each robot communication path. A disciplined selection process creates a scalable event backbone without placing inappropriate requirements on a single technology.

메시지 브로커(Message Broker)는 생산자(Producer)로부터 메시지를 수신하고 전달 과정을 관리하여 하나 이상의 소비자(Consumer)에게 전달하는 미들웨어(Middleware)이다. 모든 로봇, 서비스, 데이터베이스(Database), 애플리케이션(Application)이 직접 통신하도록 구성하는 대신, 브로커(Broker)는 중간 통신 계층(Communication Layer)을 형성한다. 이를 통해 지점 간 의존성(Point-to-Point Dependency)을 줄이고 분산 로봇 시스템(Distributed Robotic System)의 구성 요소를 강하게 결합하지 않고도 독립적으로 발전시킬 수 있다.

기본적인 브로커 모델(Broker Model)은 메시지 생산(Message Production)과 메시지 소비(Message Consumption)를 분리한다. 생산자는 수신 애플리케이션의 세부 구조를 알지 못해도 명령(Command), 텔레메트리(Telemetry), 이벤트(Event), 경보(Alarm), 애플리케이션 데이터(Application Data)를 발행할 수 있다. 소비자는 필요한 정보를 독립적으로 구독하거나 가져온다. 이러한 시간적·구조적 디커플링(Temporal and Structural Decoupling)은 로봇, 엣지 컴퓨터(Edge Computer), 플릿 서버(Fleet Server), 클라우드 서비스(Cloud Service)가 서로 다른 처리 속도로 동작하는 환경에서 특히 유용하다.

메시지 브로커는 일반적으로 큐 기반 메시징(Queue-Based Messaging), 발행-구독 메시징(Publish-Subscribe Messaging), 영속적 이벤트 스트리밍(Persistent Event Streaming)을 구현한다. 큐(Queue)는 일반적으로 경쟁하는 소비자들에게 메시지를 분배하며, 발행-구독 시스템(Publish-Subscribe System)은 관심 있는 여러 구독자에게 정보를 전달한다. 이벤트 스트리밍 플랫폼(Event-Streaming Platform)은 정렬된 레코드(Ordered Record)를 보존하여 소비자가 이를 독립적으로 처리하거나 재생(Replay)할 수 있도록 한다. 현대적인 로봇 통신 아키텍처에서는 이러한 패턴들이 함께 사용될 수 있다.

신뢰성(Reliability)은 브로커가 승인(Acknowledgement), 영속성(Persistence), 재시도(Retry), 순서 보장(Ordering), 중복(Duplication), 장애 복구(Failure Recovery)를 어떻게 처리하는지에 따라 달라진다. 일부 애플리케이션은 매우 낮은 지연시간(Latency)을 우선하고 제한적인 내구성(Durability)을 허용하지만, 다른 애플리케이션은 프로세스나 서버 장애가 발생하더라도 메시지가 유지되어야 한다. 로봇 텔레메트리는 간헐적인 손실을 허용할 수 있지만, 임무 상태 전환(Mission Transition), 안전 관련 이벤트(Safety-Related Event), 유지보수 기록(Maintenance Record), 운영 트랜잭션(Operational Transaction)은 더 강력한 전달 보장을 요구하는 경우가 많다.

아파치 카프카(Apache Kafka)는 전통적인 일시적 메시지 큐(Transient Message Queue)보다는 분산 이벤트 스트리밍 플랫폼(Distributed Event-Streaming Platform)을 중심으로 설계되었다. 메시지는 파티션된 토픽(Partitioned Topic) 내부에 순서가 있는 레코드로 저장되며, 소비자는 자신의 처리 위치를 나타내는 오프셋(Offset)을 관리한다. 이러한 아키텍처는 높은 처리량(Throughput), 수평 확장성(Horizontal Scalability), 재생(Replay), 장기간 이벤트 이력(Long-Lived Event History)을 지원하므로 플릿 텔레메트리, 분석 파이프라인(Analytics Pipeline), 디지털 트윈(Digital Twin), 대규모 로봇 데이터 플랫폼에 적합하다.

래빗엠큐(RabbitMQ)는 큐(Queue), 익스체인지(Exchange), 바인딩(Binding), 승인(Acknowledgement), 라우팅 규칙(Routing Rule)을 중심으로 하는 메시징 지향 아키텍처(Messaging-Oriented Architecture)를 따른다. 생산자는 메시지를 익스체인지에 발행하고, 익스체인지는 다이렉트(Direct), 토픽(Topic), 팬아웃(Fanout), 헤더 기반(Header-Based) 방식으로 적절한 큐에 메시지를 전달할 수 있다. 유연한 라우팅과 성숙한 전달 의미론(Delivery Semantics)을 제공하므로 기업 업무 흐름(Enterprise Workflow), 작업 분배(Job Distribution), 명령 처리(Command Processing), 백엔드 통합(Backend Integration), 명시적인 큐 동작이 필요한 애플리케이션에 적합하다.

내츠(NATS)는 경량 운영(Lightweight Operation), 단순한 통신 의미론(Simple Communication Semantics), 매우 낮은 메시징 오버헤드(Messaging Overhead)를 중시한다. 코어 내츠(Core NATS)는 고성능 발행-구독(Publish-Subscribe) 및 요청-응답(Request-Reply) 통신을 제공하고, 제트스트림(JetStream)은 영속성, 메시지 재생, 승인, 소비자 상태(Consumer State), 스트림 관리(Stream Management)를 추가한다. 이러한 조합은 빠른 서비스 통신과 선택적인 데이터의 안정적인 저장을 동시에 요구하는 엣지 시스템(Edge System)과 분산 로보틱스(Distributed Robotics)에 유용하다.

이클립스 모스키토(Eclipse Mosquitto), EMQX, 하이브엠큐(HiveMQ)와 같은 MQTT 브로커는 계층적 토픽(Hierarchical Topic)을 사용하는 경량 발행-구독 통신에 중점을 둔다. MQTT는 서비스 품질(QoS), 보존 메시지(Retained Message), 영속 세션(Persistent Session), 간헐적으로 연결되는 장치에 적합한 여러 메커니즘을 제공한다. 로보틱스에서는 특히 텔레메트리, 장치 상태(Device State), 플릿 모니터링(Fleet Monitoring), 사물인터넷 통합(IoT Integration), 제한적이거나 불안정한 무선 네트워크를 통한 통신에 효과적이다.

레디스 스트림(Redis Streams)은 인메모리 데이터 플랫폼(In-Memory Data Platform)에 영속적인 스트림 구조(Persistent Stream Structure), 소비자 그룹(Consumer Group), 메시지 식별자(Message Identifier), 승인 메커니즘을 추가하는 또 다른 접근법이다. 애플리케이션이 이미 레디스(Redis)에 크게 의존하면서 빠른 로컬 이벤트 처리가 필요한 경우 효과적이다. 그러나 운영 모델(Operational Model), 영속성 전략(Persistence Strategy), 확장 특성(Scaling Characteristics)이 다르므로 레디스 스트림과 전용 대규모 스트리밍 시스템(Dedicated Large-Scale Streaming System)을 아키텍처 관점에서 구분해야 한다.

따라서 각 제품은 서로 겹치는 기능을 제공하면서도 서로 다른 설계 우선순위를 가진다. 카프카(Kafka)는 내구성 있는 분산 이벤트 로그(Durable Distributed Event Log)와 대규모 스트리밍을 강조하고, 래빗엠큐(RabbitMQ)는 큐와 정교한 메시지 라우팅을 강조한다. 내츠(NATS)는 제트스트림을 통한 선택적 영속성과 함께 경량·고속 메시징을 중시하며, MQTT 브로커는 효율적인 장치 지향 발행-구독(Device-Oriented Publish-Subscribe) 통신을 중시한다. 따라서 제품의 인지도보다 통신 의미론(Communication Semantics)을 먼저 고려해야 한다.

처리량(Throughput)과 지연시간(Latency) 역시 워크로드 특성(Workload Characteristics)에 따라 해석해야 한다. 카프카는 레코드를 배치(Batch) 처리하고 여러 브로커에 파티션을 분산하여 매우 높은 전체 처리량을 달성할 수 있지만, 모든 저지연 제어 채널(Low-Latency Control Channel)을 대체하도록 설계된 것은 아니다. 내츠는 작고 빈번한 메시지에 효율적인 통신을 제공할 수 있으며, 래빗엠큐는 추가적인 처리 비용과 함께 풍부한 라우팅 기능을 제공한다. MQTT는 장치 및 텔레메트리 통신에서 프로토콜 오버헤드(Protocol Overhead)를 최소화한다.

확장성(Scalability)은 단순히 메시지 처리 속도를 증가시키는 것 이상의 문제이다. 엔지니어는 파티셔닝(Partitioning), 클러스터링(Clustering), 복제(Replication), 소비자 확장(Consumer Scaling), 연결 수(Connection Count), 메시지 크기(Message Size), 보존 기간(Retention Period), 네트워크 대역폭(Network Bandwidth), 장애 영역(Failure Domain)을 함께 고려해야 한다. 수백만 개의 작은 텔레메트리 메시지를 처리하는 브로커도 대용량 센서 페이로드(Sensor Payload)를 처리할 때는 다른 특성을 보일 수 있다. 따라서 카메라 영상, 라이다 포인트 클라우드(LiDAR Point Cloud), 대규모 AI 산출물은 외부 저장소에 보관하고 브로커에서는 메타데이터(Metadata), 참조 정보(Reference), 이벤트를 전달하는 방식이 일반적으로 적합하다.

로봇 시스템에서는 메시징(Messaging)과 결정론적 제어(Deterministic Control)를 신중하게 분리해야 한다. 메시지 브로커는 임무 이벤트(Mission Event), 텔레메트리, 플릿 조정(Fleet Coordination), 진단(Diagnostics), 클라우드 동기화(Cloud Synchronization), 비동기 워크플로(Asynchronous Workflow)에 적합하지만, 하드 실시간 모터 제어(Hard Real-Time Motor Control)나 안전 루프(Safety Loop)의 전송 수단으로 자동 선택해서는 안 된다. 이러한 루프에는 일반적으로 결정론적 버스(Deterministic Bus), 실시간 미들웨어(Real-Time Middleware), 공유 메모리(Shared Memory) 또는 제한된 타이밍 동작을 보장할 수 있는 별도의 통신 경로가 필요하다.

보안(Security)은 하나의 브로커가 공통 인프라를 통해 다수의 로봇과 서비스를 연결할 수 있기 때문에 매우 중요하다. 따라서 실제 운영 시스템에서는 TLS 암호화(TLS Encryption), 상호 인증(Mutual Authentication), 자격 증명(Credential), 접근 제어 목록(ACL), 토픽 또는 큐 권한 부여(Authorization), 네트워크 분할(Network Segmentation), 감사 로깅(Audit Logging), 자격 증명 순환(Credential Rotation)을 고려해야 한다. 브로커 권한은 최소 권한 원칙(Least-Privilege Principle)을 따라야 하며, 텔레메트리를 발행할 권한이 있다는 이유만으로 명령을 발행하거나 운영 스트림을 변경할 권한까지 자동으로 부여되어서는 안 된다.

고가용성(High Availability)은 추가적인 아키텍처 결정을 요구한다. 브로커 클러스터(Broker Cluster)는 데이터를 복제하고 여러 노드에 클라이언트를 분산할 수 있지만, 복제만으로 서비스 연속성(Service Continuity)이 보장되는 것은 아니다. 애플리케이션은 재연결(Reconnection), 재시도 정책(Retry Policy), 중복 처리(Duplicate Handling), 멱등 처리(Idempotent Processing), 백프레셔(Backpressure), 성능 저하 상태의 운영 전략(Degraded-Operation Strategy)을 정의해야 한다. 엣지 로봇 역시 중앙 브로커 또는 네트워크를 일시적으로 사용할 수 없을 때의 동작을 정의해야 한다.

백프레셔(Backpressure)는 데이터 생성 속도가 후속 처리 능력을 초과할 수 있는 물리 시스템(Physical System)에서 특히 중요하다. 로봇 플릿은 분석 서비스가 과부하 상태에서도 위치추정(Localization), 배터리, 인지(Perception), 진단, 임무 이벤트를 지속적으로 생성할 수 있다. 브로커는 데이터를 버퍼링(Buffering)하거나 보존할 수 있지만 저장 능력은 유한하다. 따라서 시스템에는 우선순위화(Prioritization), 샘플링(Sampling), 만료(Expiration), 집계(Aggregation), 중요하지 않은 정보의 제어된 폐기(Controlled Dropping)에 대한 명시적인 정책이 필요하다.

실용적인 로봇 아키텍처에서는 여러 메시징 기술을 동시에 사용할 수 있다. MQTT는 로봇 및 사물인터넷 장치를 플릿 인프라(Fleet Infrastructure)에 연결하고, 내츠는 경량 내부 서비스 통신을 지원하며, 래빗엠큐는 기업 업무 흐름을 조정하고, 카프카는 분석과 이력 처리를 위한 내구성 있는 운영 이벤트를 수집할 수 있다. 각 브로커가 명확하게 정의된 통신 평면(Communication Plane)과 운영 요구사항을 담당한다면 이러한 구성은 반드시 불필요한 중복이라고 볼 수 없다.

따라서 브로커 선택(Broker Selection)은 전달 의미론(Delivery Semantics), 영속성, 순서 보장, 재생, 라우팅 유연성(Routing Flexibility), 지연시간, 처리량, 연결 확장성(Connection Scale), 운영 복잡성(Operational Complexity), 생태계 통합(Ecosystem Integration), 보안, 장애 복구를 종합적으로 평가해야 한다. 핵심 질문은 어떤 브로커가 보편적으로 가장 빠르거나 강력한지가 아니라, 각각의 로봇 통신 경로에 어떤 메시징 모델이 가장 적합한가이다. 체계적인 선택 과정을 통해 단일 기술에 부적절한 요구사항을 집중시키지 않으면서 확장 가능한 이벤트 백본(Event Backbone)을 구축할 수 있다.

##  

## 05.02 Apache Kafka Architecture: Partitions, Offsets, Consumer Groups

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

Apache Kafka is a distributed event-streaming platform designed to move, store, and process large volumes of ordered event data across scalable clusters. Unlike conventional message queues that often remove messages after successful consumption, Kafka maintains events in durable append-only logs. This model allows multiple applications to consume the same data independently and enables historical events to be replayed when processing must be repeated.

Kafka organizes event records into named logical streams called topics. A topic may represent robot telemetry, mission events, diagnostics, localization updates, fleet commands, or application transactions. Producers publish records to a topic without needing to know which consumers will process them. Consumers independently subscribe to topics, creating loose coupling between data-generating robot systems and downstream analytics, monitoring, storage, or control applications.

A Kafka topic is divided into one or more partitions, which are fundamental units of storage and parallelism. Each partition is an ordered, immutable sequence of records that continuously grows as producers append new events. Partitioning allows a topic to be distributed across multiple Kafka brokers, enabling storage capacity and processing workload to scale horizontally rather than being limited to a single server.

Every record written to a partition receives a sequential numerical position called an offset. The offset identifies a record\'s location within that specific partition and increases as new records are appended. Offsets are therefore local to partitions rather than globally unique across an entire topic. A consumer uses these offsets to track how far it has progressed through each partition and determine which records should be processed next.

Offsets provide Kafka with an important separation between message storage and message consumption. Reading a record does not normally delete it from the partition. Instead, Kafka retains records according to configured retention policies, while consumers maintain their own logical processing positions. Consequently, several consumers can read the same event history at different speeds without interfering with one another or changing the underlying stored log.

Kafka can retain records for a configured time period or according to storage limits, independent of whether consumers have already processed them. This behavior enables event replay, recovery, auditing, debugging, and reconstruction of downstream application state. A robot analytics service can process current telemetry while another application later starts from an earlier offset to rebuild historical statistics or reproduce the sequence surrounding a system failure.

Partitioning is also the basis of Kafka\'s parallel processing model. Different partitions of the same topic can be consumed simultaneously, allowing processing capacity to increase as additional consumers are deployed. For a large robot fleet, records can be distributed across partitions according to robot identifiers, site identifiers, mission identifiers, or other keys so that processing workloads can be divided while preserving meaningful ordering relationships.

When a producer supplies a message key, Kafka can use that key to determine the target partition. Records associated with the same key can therefore be directed consistently to the same partition under a stable partitioning scheme. For robotics, using a robot ID as the key can preserve the order of events generated by an individual robot while still distributing different robots across multiple partitions for scalable fleet-wide processing.

Ordering guarantees in Kafka are fundamentally partition-scoped. Kafka preserves the sequence of records within an individual partition, but it does not provide one universal ordering across all partitions of a topic. System designers must therefore determine which events require relative ordering and select partition keys accordingly. Excessively broad ordering requirements can reduce parallelism, while inappropriate partitioning may separate events whose sequence has operational significance.

Kafka consumers read records from partitions and normally operate as members of consumer groups. A consumer group represents a logical application whose members cooperate to process a topic. Kafka assigns partitions among the consumers in that group so that a partition is processed by only one group member at a time. This model distributes processing workload while preventing multiple members of the same application from unnecessarily processing identical partition data.

Consumer groups enable horizontal scaling of downstream services. If a topic has multiple partitions, additional consumer instances can be added to the same group and partitions can be distributed among them. However, the useful degree of parallelism is bounded by the number of partitions. If a topic has eight partitions and a group contains more than eight active consumers, some consumers cannot receive a partition assignment for that topic.

Different consumer groups can independently consume the same Kafka topic. A fleet-monitoring service, predictive-maintenance pipeline, digital-twin service, and long-term analytics platform may each use a separate consumer group while reading identical robot events. Each group maintains its own progress, allowing applications to process the same event stream independently without requiring producers to duplicate and transmit the data separately for every downstream system.

When consumers join, leave, fail, or change their subscriptions, Kafka may redistribute partition assignments among the remaining group members. This process is known as consumer-group rebalancing. Rebalancing supports dynamic scaling and failure recovery, but it can temporarily interrupt processing and therefore becomes an important operational consideration for latency-sensitive applications. Stable membership and appropriate consumer configuration help reduce unnecessary redistribution.

Consumer offset management determines when Kafka considers records successfully processed. Consumers can periodically commit their current offsets automatically, or applications can control offset commits explicitly. Committing too early can cause records to be skipped after a processing failure, while committing after processing can cause some records to be processed again if failure occurs before the updated offset is committed. Applications must therefore align offset strategy with required processing semantics.

These behaviors lead to common delivery models such as at-most-once and at-least-once processing, while stronger processing guarantees require coordinated design across producers, consumers, and downstream state. In robotic systems, duplicate processing can be particularly important when events trigger commands, mission transitions, database updates, or external actions. Consumers should therefore use idempotent operations or deduplication mechanisms whenever repeated processing could produce undesirable physical or operational effects.

Kafka brokers form a cluster that stores topic partitions and serves producer and consumer requests. Partitions can be replicated across brokers to improve fault tolerance. One replica acts as the current leader for a partition, while other replicas maintain copies according to Kafka\'s replication mechanisms. If the broker hosting a leader becomes unavailable, an eligible replica can take over, allowing the cluster to continue serving the partition after leadership changes.

Partition count, replication configuration, record size, retention, producer batching, acknowledgement settings, and consumer processing speed jointly determine Kafka system behavior. Increasing partitions can improve parallelism but also increases metadata, replication, and operational overhead. Similarly, long retention periods improve replay capability but require greater storage capacity. Kafka architecture therefore requires capacity planning based on actual event rates, failure requirements, and processing patterns.

For robot fleets, Kafka is especially effective as a durable event backbone between operational systems and data-intensive services. Robots or edge gateways can publish normalized telemetry and events, while independent consumer groups feed monitoring systems, data lakes, digital twins, AI training pipelines, and maintenance analytics. Large binary sensor payloads are often better stored in object storage, with Kafka carrying timestamps, metadata, event descriptors, and references to the stored objects.

Kafka should not be interpreted as a replacement for deterministic robot-control communication. Its partitions, offsets, consumer groups, retention, and replay capabilities are optimized for scalable event movement and processing rather than hard real-time actuator loops. A robust robotics architecture can therefore use ROS 2, real-time middleware, or field-level networks for immediate control while Kafka provides the durable asynchronous event layer connecting fleets, edge infrastructure, on-premise systems, and cloud analytics.

아파치 카프카(Apache Kafka)는 확장 가능한 클러스터(Cluster) 전체에서 대규모의 순서화된 이벤트 데이터(Ordered Event Data)를 이동, 저장, 처리하도록 설계된 분산 이벤트 스트리밍 플랫폼(Distributed Event-Streaming Platform)이다. 성공적으로 소비된 메시지를 제거하는 경우가 많은 기존 메시지 큐(Message Queue)와 달리, 카프카는 이벤트를 내구성 있는 추가 전용 로그(Durable Append-Only Log)에 유지한다. 이러한 모델을 통해 여러 애플리케이션이 동일한 데이터를 독립적으로 소비할 수 있으며, 처리를 다시 수행해야 할 경우 과거 이벤트를 재생(Replay)할 수 있다.

카프카는 이벤트 레코드(Event Record)를 토픽(Topic)이라고 하는 이름이 지정된 논리적 스트림(Logical Stream)으로 구성한다. 토픽은 로봇 텔레메트리(Robot Telemetry), 임무 이벤트(Mission Event), 진단(Diagnostics), 위치추정 업데이트(Localization Update), 플릿 명령(Fleet Command), 애플리케이션 트랜잭션(Application Transaction) 등을 나타낼 수 있다. 생산자(Producer)는 어떤 소비자(Consumer)가 데이터를 처리할지 알 필요 없이 토픽에 레코드를 발행한다. 소비자는 독립적으로 토픽을 구독하므로 데이터를 생성하는 로봇 시스템과 후속 분석, 모니터링, 저장 또는 제어 애플리케이션 사이의 느슨한 결합(Loose Coupling)이 가능하다.

카프카 토픽(Kafka Topic)은 하나 이상의 파티션(Partition)으로 나뉘며, 파티션은 저장과 병렬 처리(Parallelism)의 기본 단위이다. 각 파티션은 생산자가 새로운 이벤트를 추가함에 따라 지속적으로 증가하는 순서화되고 변경 불가능한 레코드 시퀀스(Ordered Immutable Sequence of Records)이다. 파티셔닝(Partitioning)을 사용하면 토픽을 여러 카프카 브로커(Kafka Broker)에 분산할 수 있으므로 저장 용량과 처리 워크로드를 단일 서버로 제한하지 않고 수평적으로 확장할 수 있다.

파티션에 기록되는 모든 레코드는 오프셋(Offset)이라는 순차적인 숫자 위치를 부여받는다. 오프셋은 해당 특정 파티션 내부에서 레코드의 위치를 식별하며 새로운 레코드가 추가될수록 증가한다. 따라서 오프셋은 전체 토픽에서 전역적으로 고유한 값이 아니라 각 파티션에 종속된 값이다. 소비자는 이러한 오프셋을 사용하여 각 파티션에서 어느 위치까지 처리했는지를 추적하고 다음에 처리해야 하는 레코드를 결정한다.

오프셋은 카프카에서 메시지 저장(Message Storage)과 메시지 소비(Message Consumption)를 분리하는 중요한 기능을 제공한다. 일반적으로 레코드를 읽는다고 해서 해당 레코드가 파티션에서 삭제되는 것은 아니다. 카프카는 설정된 보존 정책(Retention Policy)에 따라 레코드를 유지하고, 소비자는 자신의 논리적인 처리 위치(Logical Processing Position)를 별도로 관리한다. 따라서 여러 소비자가 서로 간섭하거나 저장된 기본 로그를 변경하지 않으면서 동일한 이벤트 이력을 서로 다른 속도로 읽을 수 있다.

카프카는 소비자가 이미 레코드를 처리했는지 여부와 관계없이 설정된 기간 또는 저장 용량 제한에 따라 레코드를 보존할 수 있다. 이러한 동작은 이벤트 재생(Event Replay), 복구(Recovery), 감사(Auditing), 디버깅(Debugging), 후속 애플리케이션 상태의 재구성(State Reconstruction)을 가능하게 한다. 로봇 분석 서비스는 현재 텔레메트리를 처리하는 동시에, 다른 애플리케이션은 나중에 이전 오프셋부터 처리를 시작하여 과거 통계를 재구축하거나 시스템 장애 전후의 이벤트 순서를 재현할 수 있다.

파티셔닝은 카프카 병렬 처리 모델(Parallel Processing Model)의 기반이기도 하다. 동일한 토픽의 서로 다른 파티션을 동시에 소비할 수 있으므로 추가 소비자를 배치함에 따라 처리 능력을 증가시킬 수 있다. 대규모 로봇 플릿(Robot Fleet)의 경우 로봇 식별자(Robot Identifier), 사이트 식별자(Site Identifier), 임무 식별자(Mission Identifier) 또는 다른 키(Key)를 기준으로 레코드를 파티션에 분배함으로써 의미 있는 순서 관계를 유지하면서 처리 워크로드를 분산할 수 있다.

생산자가 메시지 키(Message Key)를 제공하면 카프카는 해당 키를 사용하여 대상 파티션(Target Partition)을 결정할 수 있다. 따라서 안정적인 파티셔닝 방식(Stable Partitioning Scheme)에서는 동일한 키와 연결된 레코드를 일관되게 같은 파티션으로 전달할 수 있다. 로보틱스에서는 로봇 ID(Robot ID)를 키로 사용하여 개별 로봇이 생성한 이벤트의 순서를 보존하는 동시에 서로 다른 로봇의 데이터를 여러 파티션에 분산함으로써 플릿 전체의 확장 가능한 처리를 구현할 수 있다.

카프카의 순서 보장(Ordering Guarantee)은 기본적으로 파티션 범위(Partition-Scoped)에서 제공된다. 카프카는 개별 파티션 내부의 레코드 순서를 보존하지만 토픽의 모든 파티션을 포괄하는 하나의 전역적인 순서를 제공하지는 않는다. 따라서 시스템 설계자는 어떤 이벤트 사이에 상대적인 순서가 필요한지 판단하고 이에 따라 파티션 키(Partition Key)를 선택해야 한다. 지나치게 광범위한 순서 보장 요구는 병렬성을 감소시킬 수 있으며, 부적절한 파티셔닝은 운영적으로 중요한 순서를 가진 이벤트들을 서로 분리할 수 있다.

카프카 소비자(Kafka Consumer)는 파티션에서 레코드를 읽으며 일반적으로 소비자 그룹(Consumer Group)의 구성원으로 동작한다. 소비자 그룹은 여러 구성원이 협력하여 하나의 토픽을 처리하는 논리적 애플리케이션(Logical Application)을 나타낸다. 카프카는 그룹의 소비자들에게 파티션을 할당하여 하나의 파티션이 특정 시점에는 하나의 그룹 구성원에 의해 처리되도록 한다. 이러한 모델은 동일 애플리케이션의 여러 구성원이 동일한 파티션 데이터를 불필요하게 중복 처리하는 것을 방지하면서 처리 워크로드를 분산한다.

소비자 그룹은 후속 서비스(Downstream Service)의 수평 확장(Horizontal Scaling)을 가능하게 한다. 토픽이 여러 파티션을 가지고 있다면 동일한 그룹에 소비자 인스턴스(Consumer Instance)를 추가하고 파티션을 이들 사이에 분배할 수 있다. 그러나 실질적으로 활용 가능한 병렬 처리 수준은 파티션 수에 의해 제한된다. 예를 들어 토픽에 8개의 파티션이 있고 하나의 그룹에 8개보다 많은 활성 소비자가 존재한다면 일부 소비자는 해당 토픽에 대한 파티션을 할당받지 못한다.

서로 다른 소비자 그룹은 동일한 카프카 토픽을 독립적으로 소비할 수 있다. 플릿 모니터링 서비스(Fleet-Monitoring Service), 예측 유지보수 파이프라인(Predictive-Maintenance Pipeline), 디지털 트윈 서비스(Digital-Twin Service), 장기 분석 플랫폼(Long-Term Analytics Platform)은 각각 별도의 소비자 그룹을 사용하면서 동일한 로봇 이벤트를 읽을 수 있다. 각 그룹은 자체적인 처리 진행 상태를 유지하므로 생산자가 각 후속 시스템을 위해 데이터를 별도로 복제하고 전송하지 않아도 동일한 이벤트 스트림을 독립적으로 처리할 수 있다.

소비자가 그룹에 참여하거나 그룹에서 이탈하고, 장애가 발생하거나 구독 정보가 변경되면 카프카는 남아 있는 그룹 구성원들에게 파티션 할당을 다시 분배할 수 있다. 이 과정을 소비자 그룹 리밸런싱(Consumer-Group Rebalancing)이라고 한다. 리밸런싱은 동적 확장(Dynamic Scaling)과 장애 복구(Failure Recovery)를 지원하지만 일시적으로 처리를 중단시킬 수 있으므로 지연시간에 민감한 애플리케이션에서는 중요한 운영 고려사항이 된다. 안정적인 멤버십(Stable Membership)과 적절한 소비자 설정을 통해 불필요한 재분배를 줄일 수 있다.

소비자 오프셋 관리(Consumer Offset Management)는 카프카가 언제 레코드를 성공적으로 처리된 것으로 간주할지를 결정한다. 소비자는 현재 오프셋을 주기적으로 자동 커밋(Automatic Commit)하거나 애플리케이션이 오프셋 커밋을 명시적으로 제어할 수 있다. 너무 일찍 커밋하면 처리 장애 이후 일부 레코드를 건너뛸 수 있고, 처리 이후 커밋하는 방식에서는 갱신된 오프셋이 커밋되기 전에 장애가 발생할 경우 일부 레코드가 다시 처리될 수 있다. 따라서 애플리케이션은 요구되는 처리 의미론(Processing Semantics)에 맞추어 오프셋 전략을 구성해야 한다.

이러한 동작은 최대 한 번 처리(At-Most-Once Processing), 최소 한 번 처리(At-Least-Once Processing)와 같은 일반적인 전달 모델(Delivery Model)로 이어지며, 더욱 강력한 처리 보장을 위해서는 생산자, 소비자, 후속 상태(Downstream State)를 함께 고려한 설계가 필요하다. 로봇 시스템에서는 이벤트가 명령, 임무 상태 전환, 데이터베이스 갱신 또는 외부 동작을 발생시키는 경우 중복 처리가 특히 중요하다. 따라서 반복 처리가 바람직하지 않은 물리적 또는 운영적 결과를 발생시킬 수 있다면 소비자는 멱등 연산(Idempotent Operation)이나 중복 제거 메커니즘(Deduplication Mechanism)을 사용해야 한다.

카프카 브로커(Kafka Broker)는 토픽 파티션을 저장하고 생산자 및 소비자의 요청을 처리하는 클러스터를 구성한다. 파티션은 장애 허용성(Fault Tolerance)을 높이기 위해 여러 브로커에 복제될 수 있다. 하나의 복제본(Replica)이 해당 파티션의 현재 리더(Leader) 역할을 수행하고 다른 복제본은 카프카의 복제 메커니즘(Replication Mechanism)에 따라 데이터 사본을 유지한다. 리더를 호스팅하는 브로커를 사용할 수 없게 되면 적격 복제본(Eligible Replica)이 역할을 인계하여 리더십 변경 이후에도 클러스터가 해당 파티션에 대한 서비스를 계속 제공할 수 있다.

파티션 수(Partition Count), 복제 설정(Replication Configuration), 레코드 크기(Record Size), 보존 정책, 생산자 배치(Producer Batching), 승인 설정(Acknowledgement Setting), 소비자 처리 속도(Consumer Processing Speed)는 카프카 시스템의 동작을 함께 결정한다. 파티션 수를 증가시키면 병렬성을 높일 수 있지만 메타데이터(Metadata), 복제, 운영 오버헤드도 증가한다. 마찬가지로 긴 보존 기간은 재생 능력을 향상시키지만 더 많은 저장 공간을 요구한다. 따라서 카프카 아키텍처는 실제 이벤트 발생률(Event Rate), 장애 대응 요구사항, 처리 패턴을 기반으로 용량 계획(Capacity Planning)을 수행해야 한다.

로봇 플릿 환경에서 카프카는 운영 시스템과 데이터 집약적 서비스(Data-Intensive Service)를 연결하는 내구성 있는 이벤트 백본(Durable Event Backbone)으로 특히 효과적이다. 로봇이나 엣지 게이트웨이(Edge Gateway)는 정규화된 텔레메트리와 이벤트를 발행하고, 독립적인 소비자 그룹은 모니터링 시스템, 데이터 레이크(Data Lake), 디지털 트윈, AI 학습 파이프라인(AI Training Pipeline), 유지보수 분석(Maintenance Analytics)에 데이터를 공급할 수 있다. 대용량 바이너리 센서 페이로드(Binary Sensor Payload)는 객체 저장소(Object Storage)에 저장하고 카프카에서는 타임스탬프(Timestamp), 메타데이터, 이벤트 기술 정보(Event Descriptor), 저장된 객체에 대한 참조를 전달하는 방식이 일반적으로 더 적합하다.

카프카를 결정론적 로봇 제어 통신(Deterministic Robot-Control Communication)의 대체 수단으로 해석해서는 안 된다. 파티션, 오프셋, 소비자 그룹, 보존, 재생 기능은 하드 실시간 액추에이터 루프(Hard Real-Time Actuator Loop)가 아니라 확장 가능한 이벤트 이동과 처리를 위해 최적화되어 있다. 따라서 견고한 로보틱스 아키텍처에서는 ROS 2, 실시간 미들웨어(Real-Time Middleware), 필드 레벨 네트워크(Field-Level Network)를 즉각적인 제어에 사용하고, 카프카는 플릿, 엣지 인프라(Edge Infrastructure), 온프레미스 시스템(On-Premise System), 클라우드 분석(Cloud Analytics)을 연결하는 내구성 있는 비동기 이벤트 계층(Durable Asynchronous Event Layer)으로 활용할 수 있다.

##  

## 05.03 Kafka Cluster Setup and Topic Management [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

A Kafka cluster is a group of broker processes that cooperate to store, replicate, and serve event streams. Production deployments normally use multiple brokers so topic partitions can be distributed across machines and replicated for fault tolerance. Cluster design should begin with expected event throughput, retention volume, partition count, replication requirements, network capacity, and failure scenarios rather than simply selecting a fixed number of servers.

Modern Kafka deployments use KRaft, the Kafka Raft metadata architecture, to manage cluster metadata without requiring a separate ZooKeeper ensemble. Controller nodes participate in a metadata quorum that records broker registrations, topic definitions, partition assignments, configuration changes, and leadership information. Separating metadata coordination from ordinary event processing provides a clearer operational model and enables Kafka itself to manage the cluster control plane.

Before starting a cluster, each broker requires a defined identity, network listeners, storage directories, and role configuration. Administrators must determine which nodes operate as brokers, controllers, or combined broker-controller nodes. Development environments may combine roles for simplicity, whereas production architectures generally require careful controller placement, independent failure domains, predictable storage, and explicit network configuration to avoid creating unnecessary single points of failure.

Kafka listeners determine how brokers accept connections from clients and other cluster components. Listener configuration must account for internal broker communication, controller communication, local applications, external clients, encryption, and authentication. Incorrect advertised listener settings are a common source of connectivity problems because clients initially contact one broker and then use broker addresses returned through Kafka metadata to establish subsequent connections.

Storage planning is equally important because Kafka continuously writes partition logs to disk. Each partition is represented by log segments containing event records and associated indexes. Required capacity depends on incoming data rate, retention duration, replication factor, and operational headroom. High-throughput systems also require sufficient sequential disk performance and network bandwidth so replication and consumer traffic do not compete excessively with producer writes.

After the cluster is operational, topics define the logical event streams used by applications. Topic creation normally specifies a topic name, partition count, and replication factor, together with optional topic-level configurations. A robotics platform might create separate topics for fleet telemetry, mission events, diagnostics, alarms, robot state changes, or AI inference metadata rather than mixing unrelated event classes into a single undifferentiated stream.

The number of partitions influences both scalability and ordering. More partitions allow producer traffic and consumer processing to be distributed across more brokers and consumer instances, but they also increase metadata, open files, replication work, and administrative complexity. Partition counts should therefore reflect expected throughput and consumer parallelism. They should not be increased arbitrarily simply because partitioning is Kafka\'s primary scaling mechanism.

Replication determines how many copies of each partition are maintained across brokers. A replication factor greater than one allows another replica to preserve partition data if a broker becomes unavailable. One replica acts as the partition leader and handles normal reads and writes, while follower replicas synchronize with it. Production clusters commonly distribute replicas across brokers or failure domains so a single machine failure does not eliminate all copies of important event data.

Kafka tracks replicas that are sufficiently synchronized with the leader through the in-sync replica concept. Producer acknowledgement requirements and minimum in-sync replica settings can be combined to define stronger durability behavior. Stronger settings reduce the probability of acknowledged records being lost during failures, although they can also reduce availability when too few replicas remain synchronized. Robotics platforms should select this tradeoff according to event criticality.

Topic management extends beyond initial creation. Administrators need to inspect topic configurations, partition assignments, replication state, leader distribution, retention settings, and consumer behavior throughout system operation. Topic definitions should be treated as managed infrastructure rather than temporary application details. Naming conventions and ownership policies become increasingly important as a robot fleet grows and many teams begin publishing independent event streams.

Retention policies control how long Kafka keeps event records. Time-based retention can preserve records for a specified duration, while size-based limits can restrict the amount of storage consumed by a partition. Retention should reflect the purpose of each topic. High-frequency diagnostic data may require relatively short retention, whereas mission history or operational events may need longer periods to support analytics, debugging, incident investigation, and replay.

Kafka also supports log compaction for topics representing changing state rather than purely chronological history. Compaction attempts to retain the latest value associated with each message key while older superseded values can eventually be removed. This model can be useful for robot configuration, device metadata, fleet membership, or latest known state where reconstructing the most recent value for each entity is more important than preserving every intermediate update indefinitely.

Topic configuration should account for message size as well as retention. Large camera frames, LiDAR scans, maps, and AI artifacts can create excessive broker storage, network, replication, and consumer overhead when transported directly through Kafka. A more scalable robotics architecture commonly places large binary objects in object storage or specialized data stores and publishes compact Kafka records containing identifiers, timestamps, metadata, checksums, and storage references.

Partition keys require deliberate management because they determine data distribution and ordering boundaries. A robot identifier can keep events from one robot in the same partition, preserving their relative order, while distributing different robots across the cluster. However, an uneven key distribution can create hot partitions if one robot, site, or workload generates disproportionately large traffic. Partition strategy should therefore consider both semantic ordering and load balance.

Operational monitoring should observe broker availability, partition leadership, under-replicated partitions, replication lag, disk utilization, request latency, network throughput, controller health, and consumer lag. These metrics reveal different failure modes and capacity limitations. For example, healthy brokers do not necessarily imply healthy applications if consumers are falling behind continuously or if one partition receives substantially more traffic than others.

Topic changes should be managed cautiously in production. Increasing a topic\'s partition count can improve parallelism, but it can also change key-to-partition mapping and therefore affect assumptions about record ordering. Partition counts cannot simply be reduced as an ordinary configuration operation. Major changes to partitioning, retention, or data contracts should consequently be planned together with producers, consumers, schemas, and operational migration procedures.

Security configuration belongs to cluster setup rather than being added only after deployment. Kafka can protect connections using TLS and authenticate clients through mechanisms such as SASL or certificate-based authentication. Authorization policies can restrict which applications may create topics, publish records, consume data, or perform administrative operations. Robot identities, fleet services, analytics applications, and administrators should receive only the permissions required for their roles.

Backup and disaster-recovery planning must distinguish Kafka replication from independent recovery copies. Replication protects against broker failures within the designed cluster topology, but it does not automatically protect against accidental deletion, destructive configuration changes, site-wide failures, or operational mistakes. Critical robotics deployments may therefore replicate selected event streams to another cluster, region, or long-term storage system according to recovery objectives.

Automation becomes increasingly valuable as clusters and topic counts grow. Infrastructure-as-code, configuration management, deployment pipelines, and controlled topic provisioning can make cluster configuration reproducible and auditable. Rather than allowing every application to create arbitrary topics, organizations can define templates for partition counts, replication, retention, security, naming, and ownership, reducing configuration drift across development, testing, edge, on-premise, and cloud environments.

In a robot fleet architecture, a well-managed Kafka cluster becomes a durable asynchronous data backbone rather than a simple message server. Edge gateways and fleet services publish structured events, Kafka distributes and retains them, and independent consumers support monitoring, digital twins, predictive maintenance, data lakes, and AI pipelines. Correct cluster setup and disciplined topic management ensure that this event layer remains scalable, recoverable, observable, and operationally predictable as the robotic system expands.

카프카 클러스터(Kafka Cluster)는 이벤트 스트림(Event Stream)을 저장, 복제, 제공하기 위해 협력하는 여러 브로커 프로세스(Broker Process)의 집합이다. 운영 환경(Production Deployment)에서는 일반적으로 여러 브로커를 사용하여 토픽 파티션(Topic Partition)을 여러 시스템에 분산하고 장애 허용성(Fault Tolerance)을 위해 복제한다. 클러스터 설계는 단순히 고정된 서버 수를 선택하는 것이 아니라 예상 이벤트 처리량(Event Throughput), 보존 용량(Retention Volume), 파티션 수(Partition Count), 복제 요구사항(Replication Requirement), 네트워크 용량(Network Capacity), 장애 시나리오(Failure Scenario)를 기반으로 시작해야 한다.

현대적인 카프카 배포(Kafka Deployment)는 별도의 주키퍼 앙상블(ZooKeeper Ensemble) 없이 클러스터 메타데이터(Cluster Metadata)를 관리하기 위해 카프카 래프트 메타데이터 아키텍처(Kafka Raft Metadata Architecture)인 KRaft를 사용한다. 컨트롤러 노드(Controller Node)는 브로커 등록(Broker Registration), 토픽 정의(Topic Definition), 파티션 할당(Partition Assignment), 구성 변경(Configuration Change), 리더십 정보(Leadership Information)를 기록하는 메타데이터 쿼럼(Metadata Quorum)에 참여한다. 메타데이터 조정(Metadata Coordination)을 일반적인 이벤트 처리와 분리함으로써 더욱 명확한 운영 모델을 제공하고 카프카 자체가 클러스터 제어 평면(Cluster Control Plane)을 관리할 수 있도록 한다.

클러스터를 시작하기 전에 각 브로커에는 정의된 식별자(Identity), 네트워크 리스너(Network Listener), 저장 디렉터리(Storage Directory), 역할 구성(Role Configuration)이 필요하다. 관리자는 어떤 노드가 브로커(Broker), 컨트롤러(Controller), 또는 브로커-컨트롤러 결합 노드(Combined Broker-Controller Node)로 동작할지를 결정해야 한다. 개발 환경에서는 단순화를 위해 역할을 결합할 수 있지만, 운영 아키텍처에서는 불필요한 단일 장애점(Single Point of Failure)을 만들지 않도록 컨트롤러 배치, 독립적인 장애 영역(Failure Domain), 예측 가능한 저장 구조, 명시적인 네트워크 구성을 신중하게 설계해야 한다.

카프카 리스너(Kafka Listener)는 브로커가 클라이언트(Client)와 다른 클러스터 구성 요소의 연결을 수락하는 방법을 결정한다. 리스너 구성은 내부 브로커 통신(Internal Broker Communication), 컨트롤러 통신(Controller Communication), 로컬 애플리케이션(Local Application), 외부 클라이언트(External Client), 암호화(Encryption), 인증(Authentication)을 고려해야 한다. 잘못 설정된 광고 리스너(Advertised Listener)는 일반적인 연결 문제의 원인이 될 수 있는데, 클라이언트가 처음 하나의 브로커에 접속한 이후 카프카 메타데이터를 통해 반환된 브로커 주소를 사용하여 후속 연결을 설정하기 때문이다.

카프카는 파티션 로그(Partition Log)를 디스크에 지속적으로 기록하므로 저장소 계획(Storage Planning) 역시 중요하다. 각 파티션은 이벤트 레코드(Event Record)와 관련 인덱스(Index)를 포함하는 로그 세그먼트(Log Segment)로 구성된다. 필요한 용량은 입력 데이터 속도(Incoming Data Rate), 보존 기간(Retention Duration), 복제 계수(Replication Factor), 운영 여유 공간(Operational Headroom)에 따라 결정된다. 고처리량 시스템은 복제 및 소비자 트래픽이 생산자 쓰기 작업과 과도하게 경쟁하지 않도록 충분한 순차 디스크 성능(Sequential Disk Performance)과 네트워크 대역폭(Network Bandwidth)도 필요하다.

클러스터가 운영되기 시작하면 토픽(Topic)은 애플리케이션에서 사용하는 논리적 이벤트 스트림(Logical Event Stream)을 정의한다. 토픽 생성 시 일반적으로 토픽 이름(Topic Name), 파티션 수, 복제 계수를 지정하고 필요에 따라 토픽 수준 설정(Topic-Level Configuration)을 추가한다. 로봇 플랫폼에서는 서로 관련되지 않은 이벤트 유형을 하나의 구분되지 않은 스트림에 혼합하기보다 플릿 텔레메트리(Fleet Telemetry), 임무 이벤트(Mission Event), 진단(Diagnostics), 경보(Alarm), 로봇 상태 변경(Robot State Change), AI 추론 메타데이터(AI Inference Metadata) 등에 별도의 토픽을 생성할 수 있다.

파티션 수는 확장성(Scalability)과 순서 보장(Ordering) 모두에 영향을 준다. 파티션이 많으면 생산자 트래픽과 소비자 처리를 더 많은 브로커 및 소비자 인스턴스(Consumer Instance)에 분산할 수 있지만, 메타데이터, 열린 파일(Open File), 복제 작업, 관리 복잡성(Administrative Complexity)도 증가한다. 따라서 파티션 수는 예상 처리량과 소비자 병렬성(Consumer Parallelism)을 반영하여 결정해야 하며, 파티셔닝이 카프카의 주요 확장 메커니즘이라는 이유만으로 임의로 증가시켜서는 안 된다.

복제(Replication)는 각 파티션의 복사본이 여러 브로커에 몇 개 유지되는지를 결정한다. 1보다 큰 복제 계수를 사용하면 특정 브로커를 사용할 수 없게 되더라도 다른 복제본(Replica)이 파티션 데이터를 유지할 수 있다. 하나의 복제본은 파티션 리더(Partition Leader)로 동작하여 일반적인 읽기 및 쓰기를 처리하고, 팔로워 복제본(Follower Replica)은 리더와 동기화된다. 운영 클러스터에서는 일반적으로 복제본을 여러 브로커 또는 장애 영역에 분산하여 단일 시스템 장애로 중요한 이벤트 데이터의 모든 복사본이 손실되지 않도록 한다.

카프카는 리더와 충분히 동기화된 복제본을 동기화 복제본(In-Sync Replica) 개념을 통해 추적한다. 생산자 승인 요구사항(Producer Acknowledgement Requirement)과 최소 동기화 복제본(Minimum In-Sync Replica) 설정을 결합하여 더욱 강력한 내구성(Durability)을 정의할 수 있다. 강력한 설정은 장애 발생 시 이미 승인된 레코드가 손실될 가능성을 낮추지만, 동기화 상태를 유지하는 복제본 수가 부족할 경우 가용성(Availability)을 감소시킬 수 있다. 따라서 로봇 플랫폼에서는 이벤트 중요도(Event Criticality)에 따라 이러한 상충 관계를 선택해야 한다.

토픽 관리(Topic Management)는 최초 생성 이후에도 계속된다. 관리자는 시스템 운영 전반에 걸쳐 토픽 구성, 파티션 할당, 복제 상태(Replication State), 리더 분포(Leader Distribution), 보존 설정, 소비자 동작을 점검해야 한다. 토픽 정의는 일시적인 애플리케이션 세부사항이 아니라 관리되는 인프라(Managed Infrastructure)로 취급해야 한다. 로봇 플릿이 성장하고 여러 팀이 독립적인 이벤트 스트림을 발행하기 시작하면 명명 규칙(Naming Convention)과 소유권 정책(Ownership Policy)이 더욱 중요해진다.

보존 정책(Retention Policy)은 카프카가 이벤트 레코드를 얼마나 오랫동안 유지할지를 제어한다. 시간 기반 보존(Time-Based Retention)은 지정된 기간 동안 레코드를 유지하고, 크기 기반 제한(Size-Based Limit)은 파티션이 사용하는 저장 공간을 제한할 수 있다. 보존 정책은 각 토픽의 목적을 반영해야 한다. 고빈도 진단 데이터(High-Frequency Diagnostic Data)는 비교적 짧은 보존 기간이 적합할 수 있지만, 임무 이력(Mission History)이나 운영 이벤트(Operational Event)는 분석, 디버깅, 사고 조사(Incident Investigation), 재생을 위해 더 긴 기간 동안 보존해야 할 수 있다.

카프카는 단순한 시간순 이력(Chronological History)이 아니라 변화하는 상태(Changing State)를 표현하는 토픽을 위해 로그 압축(Log Compaction)도 지원한다. 압축은 각 메시지 키(Message Key)에 연결된 최신 값을 유지하고 이전 값은 이후 제거될 수 있도록 한다. 이러한 모델은 모든 중간 업데이트를 무기한 보존하는 것보다 각 엔터티(Entity)의 최신 값을 재구성하는 것이 중요한 로봇 구성(Robot Configuration), 장치 메타데이터(Device Metadata), 플릿 구성원 정보(Fleet Membership), 최신 상태(Latest Known State)에 유용할 수 있다.

토픽 구성은 보존 정책뿐 아니라 메시지 크기(Message Size)도 고려해야 한다. 대용량 카메라 프레임(Camera Frame), 라이다 스캔(LiDAR Scan), 지도(Map), AI 산출물(AI Artifact)을 카프카를 통해 직접 전달하면 브로커 저장소, 네트워크, 복제, 소비자 처리에 과도한 부하가 발생할 수 있다. 보다 확장 가능한 로보틱스 아키텍처에서는 일반적으로 대용량 바이너리 객체(Binary Object)를 객체 저장소(Object Storage) 또는 전문 데이터 저장소에 저장하고, 카프카에서는 식별자(Identifier), 타임스탬프(Timestamp), 메타데이터(Metadata), 체크섬(Checksum), 저장소 참조(Storage Reference)를 포함하는 작은 레코드를 발행한다.

파티션 키(Partition Key)는 데이터 분산(Data Distribution)과 순서 경계(Ordering Boundary)를 결정하기 때문에 신중하게 관리해야 한다. 로봇 식별자(Robot Identifier)를 사용하면 하나의 로봇에서 발생한 이벤트를 동일한 파티션에 유지하여 상대적인 순서를 보존하면서 서로 다른 로봇의 데이터를 클러스터 전체에 분산할 수 있다. 그러나 특정 로봇, 사이트 또는 워크로드가 지나치게 많은 트래픽을 생성하면 불균형한 키 분포로 인해 핫 파티션(Hot Partition)이 발생할 수 있다. 따라서 파티션 전략은 의미론적 순서(Semantic Ordering)와 부하 균형(Load Balance)을 함께 고려해야 한다.

운영 모니터링(Operational Monitoring)에서는 브로커 가용성(Broker Availability), 파티션 리더십(Partition Leadership), 복제 부족 파티션(Under-Replicated Partition), 복제 지연(Replication Lag), 디스크 사용률(Disk Utilization), 요청 지연시간(Request Latency), 네트워크 처리량(Network Throughput), 컨트롤러 상태(Controller Health), 소비자 지연(Consumer Lag)을 관찰해야 한다. 이러한 지표는 서로 다른 장애 유형과 용량 제한을 보여준다. 예를 들어 브로커가 정상 상태라고 하더라도 소비자가 지속적으로 뒤처지거나 하나의 파티션에 다른 파티션보다 훨씬 많은 트래픽이 집중된다면 애플리케이션 전체가 정상이라고 판단할 수 없다.

운영 환경에서는 토픽 변경(Topic Change)을 신중하게 관리해야 한다. 토픽의 파티션 수를 증가시키면 병렬성을 향상시킬 수 있지만 키와 파티션 간 매핑(Key-to-Partition Mapping)이 변경되어 레코드 순서에 대한 기존 가정에 영향을 줄 수 있다. 또한 파티션 수는 일반적인 구성 변경처럼 단순하게 감소시킬 수 없다. 따라서 파티셔닝, 보존 정책 또는 데이터 계약(Data Contract)에 대한 주요 변경은 생산자, 소비자, 스키마(Schema), 운영 마이그레이션 절차(Operational Migration Procedure)를 함께 고려하여 계획해야 한다.

보안 구성(Security Configuration)은 배포 이후 추가하는 기능이 아니라 클러스터 설정의 일부로 포함되어야 한다. 카프카는 TLS를 사용하여 연결을 보호하고 SASL 또는 인증서 기반 인증(Certificate-Based Authentication)과 같은 메커니즘으로 클라이언트를 인증할 수 있다. 권한 부여 정책(Authorization Policy)을 통해 어떤 애플리케이션이 토픽을 생성하고, 레코드를 발행하거나 소비하며, 관리 작업을 수행할 수 있는지 제한할 수 있다. 로봇 식별자(Robot Identity), 플릿 서비스(Fleet Service), 분석 애플리케이션, 관리자는 각자의 역할에 필요한 권한만 부여받아야 한다.

백업(Backup) 및 재해 복구(Disaster Recovery) 계획에서는 카프카 복제와 독립적인 복구 사본(Recovery Copy)을 구분해야 한다. 복제는 설계된 클러스터 토폴로지(Cluster Topology) 내부의 브로커 장애를 보호하지만 우발적인 삭제, 파괴적인 구성 변경, 사이트 전체 장애(Site-Wide Failure), 운영 실수로부터 자동으로 보호하는 것은 아니다. 따라서 중요한 로보틱스 배포 환경에서는 복구 목표(Recovery Objective)에 따라 선택된 이벤트 스트림을 다른 클러스터, 리전(Region), 장기 저장 시스템(Long-Term Storage System)으로 복제할 수 있다.

클러스터와 토픽 수가 증가할수록 자동화(Automation)의 가치도 높아진다. 코드형 인프라(Infrastructure-as-Code), 구성 관리(Configuration Management), 배포 파이프라인(Deployment Pipeline), 제어된 토픽 프로비저닝(Controlled Topic Provisioning)을 활용하면 클러스터 구성을 재현 가능하고 감사 가능한 형태로 관리할 수 있다. 모든 애플리케이션이 임의의 토픽을 생성하도록 허용하기보다 파티션 수, 복제, 보존, 보안, 명명 방식, 소유권에 대한 템플릿(Template)을 정의함으로써 개발, 테스트, 엣지(Edge), 온프레미스(On-Premise), 클라우드 환경 사이의 구성 편차(Configuration Drift)를 줄일 수 있다.

로봇 플릿 아키텍처(Robot Fleet Architecture)에서 잘 관리된 카프카 클러스터는 단순한 메시지 서버(Message Server)가 아니라 내구성 있는 비동기 데이터 백본(Durable Asynchronous Data Backbone)이 된다. 엣지 게이트웨이(Edge Gateway)와 플릿 서비스는 구조화된 이벤트(Structured Event)를 발행하고, 카프카는 이를 분산하고 보존하며, 독립적인 소비자는 모니터링, 디지털 트윈(Digital Twin), 예측 유지보수(Predictive Maintenance), 데이터 레이크(Data Lake), AI 파이프라인(AI Pipeline)을 지원한다. 올바른 클러스터 설정과 체계적인 토픽 관리를 통해 로봇 시스템이 확장되더라도 이벤트 계층(Event Layer)의 확장성, 복구 가능성(Recoverability), 관찰 가능성(Observability), 운영 예측 가능성(Operational Predictability)을 유지할 수 있다.

##  

## 05.04 Kafka Producer / Consumer Implementation [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

A Kafka producer is an application component that creates event records and publishes them to Kafka topics, while a consumer retrieves those records for processing. Together they form the primary application-facing data path of Kafka. In robotics, producers may run on edge gateways, fleet services, or backend applications, while consumers may implement monitoring, analytics, digital twins, maintenance, and AI data pipelines.

Producer implementation begins by defining the Kafka bootstrap servers and configuring serialization for message keys and values. Bootstrap servers provide the initial addresses used to discover the Kafka cluster rather than defining every broker permanently. The producer obtains cluster metadata after connecting and learns which brokers currently lead the partitions of the target topics, allowing records to be sent directly to appropriate partition leaders.

Before transmission, application data must be converted into a byte representation through serialization. Robot events are commonly represented using JSON, Protocol Buffers, Avro, or another structured schema format. Keys also require serialization when they are used for partition selection. A consistent serialization contract is essential because consumers must deserialize the same records correctly and interpret fields according to compatible data definitions.

A producer record typically contains a topic, optional partition, optional key, value, timestamp, and headers. Applications normally specify the topic and data while allowing Kafka\'s producer logic to select the partition. Headers can carry additional metadata such as schema versions, trace identifiers, source information, or processing context without embedding every transport-related property directly inside the application payload.

Partition selection affects both scalability and ordering. When a key is provided, records with the same key can be routed consistently to the same partition under a stable partitioning strategy. A robot fleet may therefore use robot_id as the key so events from one robot preserve their relative ordering. Records from different robots can simultaneously be distributed across other partitions, enabling parallel processing throughout the Kafka cluster.

Kafka producers improve throughput by buffering records and transmitting them in batches rather than performing an independent network operation for every event. Batch size and waiting time influence the tradeoff between throughput and latency. Larger batches can use network and broker resources more efficiently, while excessive waiting can increase event delivery delay. Robot applications should tune batching according to telemetry frequency and latency requirements.

Producer acknowledgement configuration determines how much confirmation is required before a send is considered successful. Stronger acknowledgement settings can require confirmation that records have reached the partition leader and appropriate replicas, improving durability during broker failures. This protection introduces additional coordination and potentially greater latency, so telemetry streams and critical operational events may reasonably use different reliability configurations.

Retries are necessary because temporary network failures, broker transitions, and transient cluster conditions can interrupt publishing. A robust producer should distinguish retryable failures from permanent errors and apply bounded retry and timeout policies. Idempotent producer capabilities can further reduce duplicate records caused by retries, which is important when the same event could otherwise trigger duplicate database changes, mission transitions, or downstream actions.

Asynchronous sending is generally preferred for high-throughput Kafka producers. The application submits records while the producer performs batching and network transmission in the background, and completion callbacks report success or failure. This prevents the robot application from blocking on every event. However, shutdown procedures must flush outstanding records so buffered events are not unintentionally discarded when a process terminates.

A Kafka consumer begins by connecting to bootstrap servers, subscribing to one or more topics, and normally joining a consumer group. Kafka then assigns topic partitions among active members of that group. Each consumer repeatedly polls for records, deserializes their keys and values, executes application logic, and advances its processing position. This poll-processing cycle forms the fundamental consumer implementation pattern.

Consumer groups allow multiple application instances to cooperate on the same workload. If a telemetry topic contains several partitions, several analytics consumers can join one group and process different partitions concurrently. Adding consumers can increase throughput until the available partitions are fully assigned. Additional consumers beyond the useful partition parallelism remain idle for that topic unless the partition structure or subscription changes.

Deserialization converts Kafka byte records back into application-level objects. Consumers must use formats compatible with producer serialization and should handle schema evolution carefully. When robot software is upgraded gradually across a fleet, old and new event versions may temporarily coexist. Versioned schemas and backward-compatible changes help consumers continue operating while producers are updated at different times across edge and backend environments.

Offset management determines the point from which a consumer resumes after restart or failure. Kafka records the committed processing position for consumer groups, allowing a restarted consumer to continue near the last confirmed point. Automatic offset commits simplify implementation but can separate message processing from progress recording. Manual commits provide greater control when applications need to coordinate offset advancement with successful business processing.

The timing of an offset commit affects delivery behavior. If an offset is committed before application processing completes, a crash can cause an unprocessed event to be skipped after restart. If the offset is committed only after successful processing, a crash before the commit may cause the event to be processed again. Consumers should therefore combine commit strategy with idempotent processing when repeated events cannot be safely ignored.

Consumer polling must be designed so processing does not block the group for excessive periods. Long AI inference, database transactions, file operations, or external API calls can delay polling and may cause Kafka to consider a consumer unhealthy. Heavy workloads can be separated from record retrieval through controlled worker processing, but applications must carefully manage ordering, offset commits, queue capacity, and backpressure when introducing additional concurrency.

Rebalancing occurs when consumers join or leave a group, fail, or change subscriptions. Kafka may then reassign partitions among available consumers. Applications should expect temporary ownership changes and ensure that partition-specific resources are initialized and released correctly. Frequent rebalances can reduce throughput and increase latency, making stable consumer membership and appropriate timeout configuration important for continuously operating robot services.

Error handling should separate malformed data, temporary processing failures, unavailable dependencies, and unrecoverable application errors. Repeatedly retrying a permanently invalid event can stop progress on a partition. Production pipelines therefore often use bounded retries, error topics, or dead-letter processing patterns so problematic records can be isolated for diagnosis while normal event processing continues according to defined operational policies.

Backpressure becomes important when consumers process data more slowly than producers generate it. Kafka can retain records while consumers fall behind, but consumer lag will continue increasing if processing capacity remains insufficient. Monitoring lag by topic, partition, and consumer group helps operators identify this condition. Additional consumers, more partitions, optimized processing, or reduced event volume may be required depending on the bottleneck.

Graceful lifecycle management is essential for both sides of the pipeline. Producers should stop accepting new work, flush pending records, and close network resources cleanly. Consumers should stop polling in a controlled manner, complete or safely abandon in-progress work, commit appropriate offsets, and leave the group. Predictable startup and shutdown behavior reduces duplicate processing, message loss, and unnecessary consumer-group rebalancing.

In a robotic event pipeline, producers and consumers should remain independent of hard real-time control loops. An edge gateway can publish telemetry, mission events, diagnostics, and AI metadata to Kafka while local ROS 2 or real-time control continues independently. Consumer groups can then feed monitoring, predictive maintenance, digital twins, data lakes, and AI training systems without coupling those services directly to individual robots.

A reliable Kafka producer-consumer implementation therefore combines serialization contracts, partition keys, batching, acknowledgements, retries, idempotence, consumer groups, offset management, error handling, backpressure, security, and observability. The objective is not merely to send and receive messages, but to create a predictable event-processing path that remains scalable and recoverable as robot counts, data rates, and downstream applications increase.

카프카 생산자(Kafka Producer)는 이벤트 레코드(Event Record)를 생성하여 카프카 토픽(Kafka Topic)에 발행하는 애플리케이션 구성 요소(Application Component)이며, 소비자(Consumer)는 이러한 레코드를 가져와 처리한다. 이들은 함께 카프카의 주요 애플리케이션 데이터 경로(Application-Facing Data Path)를 구성한다. 로보틱스에서는 생산자가 엣지 게이트웨이(Edge Gateway), 플릿 서비스(Fleet Service), 백엔드 애플리케이션(Backend Application)에서 실행될 수 있으며, 소비자는 모니터링, 분석, 디지털 트윈(Digital Twin), 유지보수, AI 데이터 파이프라인(AI Data Pipeline)을 구현할 수 있다.

생산자 구현(Producer Implementation)은 카프카 부트스트랩 서버(Kafka Bootstrap Server)를 정의하고 메시지 키(Message Key)와 값(Value)에 대한 직렬화(Serialization)를 구성하는 것에서 시작한다. 부트스트랩 서버는 모든 브로커를 영구적으로 지정하는 것이 아니라 카프카 클러스터(Kafka Cluster)를 발견하기 위해 사용하는 초기 주소를 제공한다. 생산자는 연결 이후 클러스터 메타데이터(Cluster Metadata)를 가져와 대상 토픽의 파티션 리더(Partition Leader)를 담당하는 브로커를 확인하고 적절한 파티션 리더로 직접 레코드를 전송한다.

전송하기 전에 애플리케이션 데이터는 직렬화를 통해 바이트 표현(Byte Representation)으로 변환되어야 한다. 로봇 이벤트는 일반적으로 JSON, 프로토콜 버퍼(Protocol Buffers), 아브로(Avro) 또는 다른 구조화된 스키마 형식(Structured Schema Format)으로 표현할 수 있다. 파티션 선택에 사용되는 키 역시 직렬화가 필요하다. 소비자가 동일한 레코드를 올바르게 역직렬화(Deserialize)하고 호환되는 데이터 정의에 따라 필드를 해석해야 하므로 일관된 직렬화 계약(Serialization Contract)이 필수적이다.

생산자 레코드(Producer Record)는 일반적으로 토픽, 선택적 파티션(Optional Partition), 선택적 키(Optional Key), 값, 타임스탬프(Timestamp), 헤더(Header)를 포함한다. 애플리케이션은 일반적으로 토픽과 데이터를 지정하고 파티션 선택은 카프카 생산자 로직(Kafka Producer Logic)에 맡긴다. 헤더에는 스키마 버전(Schema Version), 추적 식별자(Trace Identifier), 소스 정보(Source Information), 처리 컨텍스트(Processing Context) 등의 추가 메타데이터를 전달할 수 있으므로 모든 전송 관련 속성을 애플리케이션 페이로드(Application Payload)에 직접 포함할 필요가 없다.

파티션 선택(Partition Selection)은 확장성(Scalability)과 순서 보장(Ordering) 모두에 영향을 준다. 키가 제공되면 안정적인 파티셔닝 전략(Stable Partitioning Strategy)에서 동일한 키를 가진 레코드를 일관되게 같은 파티션으로 전달할 수 있다. 따라서 로봇 플릿(Robot Fleet)은 robot_id를 키로 사용하여 하나의 로봇에서 발생한 이벤트의 상대적인 순서를 유지할 수 있다. 동시에 서로 다른 로봇의 레코드는 다른 파티션으로 분산되어 카프카 클러스터 전체에서 병렬 처리가 가능하다.

카프카 생산자는 각각의 이벤트마다 독립적인 네트워크 작업을 수행하는 대신 레코드를 버퍼링(Buffering)하고 배치(Batch) 단위로 전송하여 처리량(Throughput)을 향상시킨다. 배치 크기(Batch Size)와 대기 시간(Waiting Time)은 처리량과 지연시간(Latency) 사이의 상충 관계에 영향을 준다. 큰 배치는 네트워크와 브로커 자원을 더욱 효율적으로 사용할 수 있지만 지나치게 긴 대기는 이벤트 전달 지연을 증가시킬 수 있다. 로봇 애플리케이션은 텔레메트리 발생 빈도와 지연시간 요구사항에 따라 배치 설정을 조정해야 한다.

생산자 승인 설정(Producer Acknowledgement Configuration)은 전송이 성공한 것으로 판단하기 전에 어느 정도의 확인이 필요한지를 결정한다. 더 강력한 승인 설정에서는 레코드가 파티션 리더와 적절한 복제본(Replica)에 도달했음을 확인하도록 요구할 수 있으므로 브로커 장애 상황에서 내구성(Durability)을 높일 수 있다. 이러한 보호는 추가적인 조정 작업과 지연시간 증가를 발생시킬 수 있으므로 텔레메트리 스트림(Telemetry Stream)과 중요한 운영 이벤트(Critical Operational Event)에 서로 다른 신뢰성 설정을 적용할 수 있다.

일시적인 네트워크 장애, 브로커 전환(Broker Transition), 일시적 클러스터 상태(Transient Cluster Condition)가 발행을 중단시킬 수 있으므로 재시도(Retry)가 필요하다. 견고한 생산자는 재시도 가능한 장애(Retryable Failure)와 영구적인 오류(Permanent Error)를 구분하고 제한된 재시도(Bounded Retry) 및 타임아웃 정책(Timeout Policy)을 적용해야 한다. 멱등 생산자(Idempotent Producer) 기능을 사용하면 재시도로 발생할 수 있는 중복 레코드를 더욱 줄일 수 있으며, 동일한 이벤트가 중복 데이터베이스 변경, 임무 상태 전환 또는 후속 동작을 발생시킬 수 있는 환경에서 특히 중요하다.

고처리량 카프카 생산자에서는 일반적으로 비동기 전송(Asynchronous Sending)이 선호된다. 애플리케이션이 레코드를 제출하면 생산자는 백그라운드에서 배치 처리와 네트워크 전송을 수행하고 완료 콜백(Completion Callback)을 통해 성공 또는 실패를 보고한다. 이를 통해 로봇 애플리케이션이 각각의 이벤트 전송을 기다리며 차단(Block)되는 것을 방지할 수 있다. 그러나 프로세스가 종료될 때 버퍼링된 이벤트가 의도하지 않게 손실되지 않도록 종료 절차에서 대기 중인 레코드를 플러시(Flush)해야 한다.

카프카 소비자(Kafka Consumer)는 부트스트랩 서버에 연결하고 하나 이상의 토픽을 구독한 다음 일반적으로 소비자 그룹(Consumer Group)에 참여하면서 동작을 시작한다. 이후 카프카는 해당 그룹의 활성 구성원 사이에 토픽 파티션을 할당한다. 각 소비자는 반복적으로 레코드를 폴링(Polling)하고 키와 값을 역직렬화한 후 애플리케이션 로직(Application Logic)을 실행하고 처리 위치를 진행시킨다. 이러한 폴링-처리 사이클(Poll-Processing Cycle)이 기본적인 소비자 구현 패턴을 구성한다.

소비자 그룹을 사용하면 여러 애플리케이션 인스턴스(Application Instance)가 동일한 워크로드를 협력하여 처리할 수 있다. 텔레메트리 토픽에 여러 파티션이 존재한다면 여러 분석 소비자(Analytics Consumer)가 하나의 그룹에 참여하여 서로 다른 파티션을 동시에 처리할 수 있다. 소비자를 추가하면 사용 가능한 파티션이 모두 할당될 때까지 처리량을 증가시킬 수 있다. 유효한 파티션 병렬성(Partition Parallelism)을 초과하여 추가된 소비자는 파티션 구조나 구독이 변경되지 않는 한 해당 토픽에 대해 유휴 상태(Idle)가 된다.

역직렬화(Deserialization)는 카프카의 바이트 레코드를 다시 애플리케이션 수준 객체(Application-Level Object)로 변환한다. 소비자는 생산자의 직렬화와 호환되는 형식을 사용해야 하며 스키마 진화(Schema Evolution)를 신중하게 처리해야 한다. 로봇 소프트웨어가 플릿 전체에서 점진적으로 업그레이드되면 이전 버전과 새로운 버전의 이벤트가 일시적으로 공존할 수 있다. 버전이 지정된 스키마(Versioned Schema)와 하위 호환 변경(Backward-Compatible Change)을 사용하면 엣지와 백엔드 환경의 생산자가 서로 다른 시점에 업데이트되더라도 소비자가 계속 동작할 수 있다.

오프셋 관리(Offset Management)는 소비자가 재시작하거나 장애에서 복구된 이후 어느 지점부터 처리를 재개할지를 결정한다. 카프카는 소비자 그룹에 대해 커밋된 처리 위치(Committed Processing Position)를 기록하므로 재시작된 소비자는 마지막으로 확인된 위치 근처에서 처리를 계속할 수 있다. 자동 오프셋 커밋(Automatic Offset Commit)은 구현을 단순화하지만 메시지 처리와 진행 상태 기록이 분리될 수 있다. 수동 커밋(Manual Commit)은 애플리케이션이 오프셋 진행을 성공적인 비즈니스 처리(Business Processing)와 조정해야 하는 경우 더 높은 제어 능력을 제공한다.

오프셋을 커밋하는 시점은 전달 동작(Delivery Behavior)에 영향을 준다. 애플리케이션 처리가 완료되기 전에 오프셋을 커밋하면 장애 발생 시 처리되지 않은 이벤트가 재시작 이후 건너뛰어질 수 있다. 반대로 성공적인 처리 이후에만 오프셋을 커밋하면 커밋 전에 장애가 발생했을 때 해당 이벤트가 다시 처리될 수 있다. 따라서 반복 이벤트를 안전하게 무시할 수 없는 환경에서는 소비자의 커밋 전략(Commit Strategy)을 멱등 처리(Idempotent Processing)와 결합해야 한다.

소비자 폴링(Consumer Polling)은 처리 작업으로 인해 그룹이 지나치게 오랫동안 차단되지 않도록 설계해야 한다. 장시간의 AI 추론(AI Inference), 데이터베이스 트랜잭션(Database Transaction), 파일 작업(File Operation), 외부 API 호출(External API Call)은 폴링을 지연시켜 카프카가 소비자를 비정상 상태로 판단하게 만들 수 있다. 무거운 워크로드는 제어된 작업자 처리(Controlled Worker Processing)를 통해 레코드 검색과 분리할 수 있지만, 추가적인 동시성을 도입할 경우 순서, 오프셋 커밋, 큐 용량(Queue Capacity), 백프레셔(Backpressure)를 신중하게 관리해야 한다.

리밸런싱(Rebalancing)은 소비자가 그룹에 참여하거나 이탈하고, 장애가 발생하거나 구독이 변경될 때 수행된다. 카프카는 이때 사용 가능한 소비자 사이에서 파티션을 다시 할당할 수 있다. 애플리케이션은 일시적인 소유권 변경(Ownership Change)을 예상하고 파티션별 자원(Partition-Specific Resource)이 올바르게 초기화되고 해제되도록 해야 한다. 빈번한 리밸런싱은 처리량을 감소시키고 지연시간을 증가시킬 수 있으므로 지속적으로 운영되는 로봇 서비스에서는 안정적인 소비자 멤버십(Stable Consumer Membership)과 적절한 타임아웃 설정이 중요하다.

오류 처리(Error Handling)는 잘못된 데이터(Malformed Data), 일시적인 처리 장애(Temporary Processing Failure), 사용할 수 없는 종속 서비스(Unavailable Dependency), 복구할 수 없는 애플리케이션 오류(Unrecoverable Application Error)를 구분해야 한다. 영구적으로 잘못된 이벤트를 계속 재시도하면 하나의 파티션에서 전체 처리가 중단될 수 있다. 따라서 운영 파이프라인에서는 일반적으로 제한된 재시도, 오류 토픽(Error Topic), 데드 레터 처리 패턴(Dead-Letter Processing Pattern)을 사용하여 문제가 있는 레코드를 진단 대상으로 격리하면서 정의된 운영 정책에 따라 정상적인 이벤트 처리를 계속한다.

소비자의 처리 속도보다 생산자의 데이터 생성 속도가 빠를 경우 백프레셔(Backpressure)가 중요해진다. 카프카는 소비자가 뒤처지는 동안 레코드를 보존할 수 있지만 처리 용량이 계속 부족하면 소비자 지연(Consumer Lag)은 지속적으로 증가한다. 토픽, 파티션, 소비자 그룹별 지연을 모니터링하면 운영자가 이러한 상태를 파악할 수 있다. 병목 지점(Bottleneck)에 따라 추가 소비자 배치, 파티션 증가, 처리 최적화 또는 이벤트 데이터 양 감소가 필요할 수 있다.

파이프라인 양쪽 모두에서 정상적인 생명주기 관리(Graceful Lifecycle Management)가 중요하다. 생산자는 새로운 작업 수락을 중지하고 대기 중인 레코드를 플러시한 후 네트워크 자원을 정상적으로 종료해야 한다. 소비자는 제어된 방식으로 폴링을 중지하고 진행 중인 작업을 완료하거나 안전하게 중단한 다음 적절한 오프셋을 커밋하고 그룹에서 이탈해야 한다. 예측 가능한 시작 및 종료 동작은 중복 처리, 메시지 손실, 불필요한 소비자 그룹 리밸런싱을 줄여준다.

로봇 이벤트 파이프라인(Robotic Event Pipeline)에서 생산자와 소비자는 하드 실시간 제어 루프(Hard Real-Time Control Loop)와 독립적으로 유지되어야 한다. 엣지 게이트웨이는 텔레메트리, 임무 이벤트, 진단, AI 메타데이터를 카프카에 발행하면서 로컬 ROS 2 또는 실시간 제어(Real-Time Control)를 독립적으로 계속 수행할 수 있다. 이후 소비자 그룹은 개별 로봇과 해당 서비스들을 직접 결합하지 않으면서 모니터링, 예측 유지보수(Predictive Maintenance), 디지털 트윈, 데이터 레이크(Data Lake), AI 학습 시스템(AI Training System)에 데이터를 공급할 수 있다.

따라서 신뢰할 수 있는 카프카 생산자-소비자 구현(Kafka Producer-Consumer Implementation)은 직렬화 계약, 파티션 키(Partition Key), 배치 처리, 승인, 재시도, 멱등성(Idempotence), 소비자 그룹, 오프셋 관리, 오류 처리, 백프레셔, 보안(Security), 관찰 가능성(Observability)을 종합적으로 결합해야 한다. 목표는 단순히 메시지를 송수신하는 것이 아니라 로봇 수, 데이터 발생률(Data Rate), 후속 애플리케이션이 증가하더라도 확장 가능하고 복구 가능하며 예측 가능한 이벤트 처리 경로(Event-Processing Path)를 구축하는 것이다.

##  

## 05.05 Kafka Streams: Robot Data Real-Time Processing [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

Kafka Streams is a client-side stream-processing library for building applications that continuously transform and analyze data stored in Kafka topics. Instead of moving robot events into a separate processing platform, applications can consume, process, aggregate, join, and republish Kafka records directly. This architecture is useful for robot fleets that continuously generate telemetry, diagnostics, mission events, localization updates, and AI inference results.

A Kafka Streams application represents continuous processing as a topology composed of sources, processing nodes, and sinks. Source processors read records from Kafka topics, intermediate processors perform operations such as filtering, mapping, aggregation, or joining, and sink processors publish results to destination topics. The topology therefore describes how raw robot events become operational information while remaining inside an event-driven Kafka architecture.

Kafka Streams provides two major programming abstractions: the Streams DSL and the Processor API. The Streams DSL offers high-level operations for common transformations, filtering, grouping, aggregation, windowing, and joins. The Processor API provides lower-level control over record processing and state. Robotics applications can use the DSL for standard telemetry pipelines while adopting lower-level processing when specialized event handling or state management is required.

A KStream represents a continuously evolving sequence of independent event records. Robot telemetry naturally fits this abstraction because every battery update, position measurement, diagnostic event, or mission transition can be treated as a new event. Operations can filter irrelevant records, transform measurement units, enrich metadata, or route specific event classes into new topics without modifying the original event stream.

A KTable represents a continuously updated view of the latest state associated with each key. If robot_id is used as the key, a table can represent the most recent battery level, operating mode, mission state, or health condition for every robot. New events update the corresponding key\'s current value, making KTable useful for fleet-state views where the latest known condition is more important than every historical update.

GlobalKTable provides a replicated table abstraction in which each application instance can maintain a local copy of the complete reference dataset. This can be useful when robot events need to be enriched using relatively small reference information such as robot configuration, site definitions, device capabilities, or fleet metadata. Local availability of reference data can reduce repeated remote lookups during continuous event processing.

Stateless transformations process each event without requiring information from previous records. Filtering can remove unnecessary telemetry, mapping can convert event structures, and branching can route alarms or diagnostics toward different processing paths. Stateless operations are relatively simple to scale because processing depends primarily on the current record rather than maintaining a historical context for each robot or event key.

Stateful processing maintains information across multiple records and enables more sophisticated robot analytics. Aggregations can calculate counts, averages, minimums, maximums, or other accumulated values for each key. A fleet application might continuously calculate average battery consumption, mission completion counts, repeated fault occurrences, or utilization statistics while new events arrive from robots and edge gateways.

Windowing adds time boundaries to stateful stream operations. Tumbling, hopping, sliding, or session-oriented windows can group events occurring within defined temporal intervals. A monitoring service might calculate the number of localization failures during the last minute, average motor temperature over a short interval, or repeated obstacle detections within a moving time window to identify conditions requiring further analysis.

Event time is especially important for robotic data because the time an event physically occurred may differ from the time Kafka receives or processes it. Wireless delays, intermittent connectivity, buffering, and edge processing can cause records to arrive late or out of order. Stream-processing logic should therefore use meaningful timestamps and define how late-arriving events are handled when performing time-windowed calculations.

Stream-table joins allow dynamic robot events to be combined with relatively stable reference state. A telemetry event containing robot_id can be joined with a table containing robot model, site, payload class, or configuration information. The enriched event can then be analyzed without requiring every producer to duplicate static metadata in each message, reducing payload size and centralizing reference-data management.

Stream-stream joins can correlate two continuously changing event sources within a defined time relationship. Mission events can be correlated with robot alarms, localization changes with obstacle detections, or charging commands with battery-state updates. Such correlations allow higher-level operational events to be derived from multiple independent data sources while maintaining the asynchronous characteristics of a distributed robotic system.

Kafka Streams supports local state stores for stateful operations. Aggregations, tables, windows, and joins may maintain state on the application instance responsible for the relevant partitions. Kafka-backed changelog topics can preserve state changes so another instance can reconstruct the required state after a failure or partition reassignment. This mechanism combines local processing efficiency with distributed recovery capability.

Parallelism follows Kafka\'s partitioning model. Multiple instances of the same Kafka Streams application can run simultaneously, and Kafka partitions are distributed among processing tasks. Increasing application instances can therefore increase processing capacity until available partition parallelism is exhausted. Choosing appropriate topic partition counts remains important because partition design influences both event ordering and achievable processing scalability.

Kafka Streams applications participate in Kafka consumer-group coordination, allowing processing tasks to move when instances start, stop, or fail. Rebalancing redistributes partitions and associated processing responsibilities across available instances. Stateful applications may require state restoration after reassignment, so deployment architecture should consider local storage performance, changelog volume, restart behavior, and recovery time.

Processing guarantees are important when stream results update operational systems. Kafka Streams can coordinate input offsets, state-store updates, and output records to provide stronger processing semantics when configured appropriately. Nevertheless, external side effects such as database updates, robot commands, or third-party API calls require additional application design because transaction guarantees inside Kafka do not automatically make external physical actions exactly once.

Real-time robot analytics should distinguish operational event processing from hard real-time control. Kafka Streams can detect trends, correlate events, calculate fleet statistics, identify abnormal patterns, and generate alerts with low processing delay, but it is not intended to replace deterministic motor-control or safety loops. Immediate actuator control should remain in ROS 2, real-time middleware, embedded controllers, or field-level networks.

A practical fleet pipeline may begin with edge gateways publishing robot telemetry to Kafka. Kafka Streams can normalize records, remove invalid measurements, enrich events with fleet metadata, aggregate values over time windows, and detect threshold conditions. Processed topics can then feed dashboards, maintenance systems, digital twins, data lakes, or AI pipelines without forcing each downstream application to independently repeat the same transformations.

Backpressure and consumer lag still require monitoring even though Kafka Streams manages much of the consumer coordination internally. If processing becomes slower than incoming event generation, records accumulate in Kafka and processing delay increases. Operators should monitor task throughput, consumer lag, processing latency, state-store size, restore duration, exceptions, and broker conditions to understand whether the streaming application can sustain fleet data rates.

Scaling should be based on partition distribution, event rate, state size, processing complexity, and required latency rather than application-instance count alone. CPU-intensive transformations, large joins, long windows, or extensive state stores may require more compute and storage resources. Increasing instances provides limited benefit if the source topics contain too few partitions or if a downstream dependency remains the dominant bottleneck.

Fault handling should prevent a malformed event from unnecessarily stopping an entire robot-data pipeline. Applications need policies for serialization failures, invalid schemas, unexpected values, processing exceptions, and unavailable dependencies. Problematic records can be isolated into dedicated error topics for later diagnosis while valid events continue through the topology, provided the operational requirements permit such separation.

For Physical AI and autonomous robot fleets, Kafka Streams can serve as the real-time transformation layer between raw operational events and higher-level intelligence. It can convert distributed robot observations into normalized state, temporal aggregates, correlated events, alerts, and AI-ready data streams. Combined with Kafka topics and durable event storage, this creates a scalable processing backbone connecting edge robots with fleet management, analytics, digital twins, and learning systems.

카프카 스트림즈(Kafka Streams)는 카프카 토픽(Kafka Topic)에 저장된 데이터를 지속적으로 변환하고 분석하는 애플리케이션을 구축하기 위한 클라이언트 측 스트림 처리 라이브러리(Client-Side Stream-Processing Library)이다. 로봇 이벤트를 별도의 처리 플랫폼으로 이동시키는 대신 애플리케이션이 카프카 레코드(Kafka Record)를 직접 소비, 처리, 집계, 조인(Join), 재발행할 수 있다. 이러한 아키텍처는 텔레메트리(Telemetry), 진단(Diagnostics), 임무 이벤트(Mission Event), 위치추정 업데이트(Localization Update), AI 추론 결과(AI Inference Result)를 지속적으로 생성하는 로봇 플릿(Robot Fleet)에 유용하다.

카프카 스트림즈 애플리케이션(Kafka Streams Application)은 연속적인 처리를 소스(Source), 처리 노드(Processing Node), 싱크(Sink)로 구성된 토폴로지(Topology)로 표현한다. 소스 프로세서(Source Processor)는 카프카 토픽에서 레코드를 읽고, 중간 프로세서는 필터링(Filtering), 매핑(Mapping), 집계(Aggregation), 조인 등의 작업을 수행하며, 싱크 프로세서(Sink Processor)는 결과를 목적지 토픽(Destination Topic)에 발행한다. 따라서 토폴로지는 이벤트 기반 카프카 아키텍처(Event-Driven Kafka Architecture) 내부에서 원시 로봇 이벤트(Raw Robot Event)가 운영 정보(Operational Information)로 변환되는 과정을 정의한다.

카프카 스트림즈는 스트림즈 DSL(Streams DSL)과 프로세서 API(Processor API)라는 두 가지 주요 프로그래밍 추상화(Programming Abstraction)를 제공한다. 스트림즈 DSL은 일반적인 변환, 필터링, 그룹화(Grouping), 집계, 윈도잉(Windowing), 조인을 위한 고수준 연산을 제공한다. 프로세서 API는 레코드 처리와 상태(State)에 대한 보다 낮은 수준의 제어를 제공한다. 로봇 애플리케이션에서는 표준 텔레메트리 파이프라인에 DSL을 사용하고, 특수한 이벤트 처리나 상태 관리가 필요한 경우 보다 저수준의 처리 방식을 사용할 수 있다.

KStream은 지속적으로 변화하는 독립적인 이벤트 레코드의 시퀀스(Sequence)를 표현한다. 모든 배터리 업데이트(Battery Update), 위치 측정(Position Measurement), 진단 이벤트, 임무 상태 전환(Mission Transition)을 새로운 이벤트로 처리할 수 있으므로 로봇 텔레메트리는 이러한 추상화에 자연스럽게 대응된다. 연산을 통해 불필요한 레코드를 필터링하고, 측정 단위를 변환하며, 메타데이터(Metadata)를 보강하거나 특정 이벤트 유형을 원래 이벤트 스트림을 변경하지 않고 새로운 토픽으로 전달할 수 있다.

KTable은 각 키(Key)에 연결된 최신 상태를 지속적으로 갱신하는 뷰(View)를 나타낸다. robot_id를 키로 사용하면 테이블은 각 로봇의 가장 최근 배터리 수준, 동작 모드(Operating Mode), 임무 상태(Mission State), 상태 조건(Health Condition)을 표현할 수 있다. 새로운 이벤트는 해당 키의 현재 값을 갱신하므로 KTable은 모든 과거 업데이트보다 가장 최근에 알려진 상태(Latest Known Condition)가 중요한 플릿 상태 뷰(Fleet-State View)에 유용하다.

GlobalKTable은 각 애플리케이션 인스턴스(Application Instance)가 전체 참조 데이터셋(Reference Dataset)의 로컬 사본(Local Copy)을 유지할 수 있는 복제 테이블 추상화(Replicated Table Abstraction)를 제공한다. 로봇 이벤트를 로봇 구성(Robot Configuration), 사이트 정의(Site Definition), 장치 기능(Device Capability), 플릿 메타데이터(Fleet Metadata)와 같이 상대적으로 작은 참조 정보로 보강해야 하는 경우 유용할 수 있다. 참조 데이터를 로컬에서 사용할 수 있으므로 연속적인 이벤트 처리 과정에서 반복적인 원격 조회(Remote Lookup)를 줄일 수 있다.

무상태 변환(Stateless Transformation)은 이전 레코드의 정보를 요구하지 않고 각각의 이벤트를 처리한다. 필터링은 불필요한 텔레메트리를 제거하고, 매핑은 이벤트 구조(Event Structure)를 변환하며, 분기(Branching)는 경보 또는 진단 데이터를 서로 다른 처리 경로로 전달할 수 있다. 무상태 연산은 각 로봇이나 이벤트 키의 과거 컨텍스트(Historical Context)를 유지할 필요 없이 현재 레코드에 주로 의존하므로 비교적 쉽게 확장할 수 있다.

상태 기반 처리(Stateful Processing)는 여러 레코드에 걸쳐 정보를 유지함으로써 더욱 정교한 로봇 분석을 가능하게 한다. 집계는 각 키에 대한 개수(Count), 평균(Average), 최솟값(Minimum), 최댓값(Maximum) 또는 다른 누적 값(Accumulated Value)을 계산할 수 있다. 플릿 애플리케이션은 로봇과 엣지 게이트웨이(Edge Gateway)에서 새로운 이벤트가 도착하는 동안 평균 배터리 소비량, 임무 완료 횟수, 반복 장애 발생 횟수, 활용률(Utilization Statistics)을 지속적으로 계산할 수 있다.

윈도잉(Windowing)은 상태 기반 스트림 연산에 시간 경계(Time Boundary)를 추가한다. 텀블링 윈도(Tumbling Window), 호핑 윈도(Hopping Window), 슬라이딩 윈도(Sliding Window), 세션 기반 윈도(Session-Oriented Window)는 정의된 시간 간격 내에서 발생한 이벤트를 그룹화할 수 있다. 모니터링 서비스는 최근 1분 동안의 위치추정 실패 횟수, 짧은 구간의 평균 모터 온도, 이동 시간창(Moving Time Window) 내 반복적인 장애물 감지를 계산하여 추가 분석이 필요한 상태를 식별할 수 있다.

이벤트 시간(Event Time)은 이벤트가 물리적으로 발생한 시간과 카프카가 이를 수신하거나 처리하는 시간이 서로 다를 수 있기 때문에 로봇 데이터에서 특히 중요하다. 무선 통신 지연(Wireless Delay), 간헐적인 연결(Intermittent Connectivity), 버퍼링(Buffering), 엣지 처리(Edge Processing)로 인해 레코드가 늦게 도착하거나 순서가 뒤바뀔 수 있다. 따라서 스트림 처리 로직(Stream-Processing Logic)은 의미 있는 타임스탬프(Timestamp)를 사용하고 시간창 계산을 수행할 때 지연 도착 이벤트(Late-Arriving Event)를 어떻게 처리할지 정의해야 한다.

스트림-테이블 조인(Stream-Table Join)을 사용하면 동적으로 변화하는 로봇 이벤트와 상대적으로 안정적인 참조 상태(Reference State)를 결합할 수 있다. robot_id를 포함하는 텔레메트리 이벤트를 로봇 모델, 사이트, 페이로드 등급(Payload Class), 구성 정보를 포함하는 테이블과 조인할 수 있다. 이렇게 보강된 이벤트(Enriched Event)를 사용하면 모든 생산자(Producer)가 각 메시지에 정적 메타데이터를 반복해서 포함하지 않아도 분석할 수 있으므로 페이로드 크기를 줄이고 참조 데이터 관리를 중앙화할 수 있다.

스트림-스트림 조인(Stream-Stream Join)은 정의된 시간 관계(Time Relationship) 안에서 지속적으로 변화하는 두 이벤트 소스를 연관시킬 수 있다. 임무 이벤트와 로봇 경보, 위치추정 변화와 장애물 감지, 충전 명령(Charging Command)과 배터리 상태 업데이트를 서로 연관시킬 수 있다. 이러한 상관관계(Correlation)를 이용하면 분산 로봇 시스템의 비동기적 특성을 유지하면서 여러 독립적인 데이터 소스로부터 더 높은 수준의 운영 이벤트를 생성할 수 있다.

카프카 스트림즈는 상태 기반 연산을 위해 로컬 상태 저장소(Local State Store)를 지원한다. 집계, 테이블, 윈도, 조인은 관련 파티션을 담당하는 애플리케이션 인스턴스에서 상태를 유지할 수 있다. 카프카 기반 변경 로그 토픽(Kafka-Backed Changelog Topic)은 상태 변경을 보존하여 장애 또는 파티션 재할당(Partition Reassignment)이 발생한 이후 다른 인스턴스가 필요한 상태를 재구성할 수 있도록 한다. 이러한 메커니즘은 로컬 처리 효율성과 분산 복구 능력(Distributed Recovery Capability)을 결합한다.

병렬성(Parallelism)은 카프카의 파티셔닝 모델(Partitioning Model)을 따른다. 동일한 카프카 스트림즈 애플리케이션의 여러 인스턴스를 동시에 실행할 수 있으며 카프카 파티션은 처리 태스크(Processing Task) 사이에 분배된다. 따라서 애플리케이션 인스턴스를 증가시키면 사용 가능한 파티션 병렬성이 소진될 때까지 처리 능력을 높일 수 있다. 파티션 설계는 이벤트 순서와 달성 가능한 처리 확장성 모두에 영향을 주므로 적절한 토픽 파티션 수를 선택하는 것이 중요하다.

카프카 스트림즈 애플리케이션은 카프카 소비자 그룹 조정(Kafka Consumer-Group Coordination)에 참여하므로 인스턴스가 시작, 종료 또는 장애 상태가 될 때 처리 태스크를 이동할 수 있다. 리밸런싱(Rebalancing)은 사용 가능한 인스턴스 사이에서 파티션과 관련 처리 책임을 다시 분배한다. 상태 기반 애플리케이션은 재할당 이후 상태 복원(State Restoration)이 필요할 수 있으므로 배포 아키텍처에서는 로컬 저장소 성능, 변경 로그 데이터량(Changelog Volume), 재시작 동작(Restart Behavior), 복구 시간(Recovery Time)을 고려해야 한다.

스트림 처리 결과가 운영 시스템을 갱신하는 경우 처리 보장(Processing Guarantee)이 중요하다. 카프카 스트림즈는 적절하게 구성하면 입력 오프셋(Input Offset), 상태 저장소 갱신(State-Store Update), 출력 레코드(Output Record)를 조정하여 더욱 강력한 처리 의미론(Processing Semantics)을 제공할 수 있다. 그러나 데이터베이스 갱신, 로봇 명령, 외부 API 호출과 같은 외부 부수 효과(External Side Effect)는 별도의 애플리케이션 설계가 필요하다. 카프카 내부의 트랜잭션 보장이 외부의 물리적 동작까지 자동으로 정확히 한 번(Exactly Once) 수행되도록 만드는 것은 아니기 때문이다.

실시간 로봇 분석(Real-Time Robot Analytics)은 운영 이벤트 처리와 하드 실시간 제어(Hard Real-Time Control)를 구분해야 한다. 카프카 스트림즈는 추세를 감지하고, 이벤트를 연관시키며, 플릿 통계를 계산하고, 이상 패턴(Abnormal Pattern)을 식별하며, 낮은 처리 지연으로 경보를 생성할 수 있지만 결정론적 모터 제어(Deterministic Motor Control) 또는 안전 루프(Safety Loop)를 대체하기 위한 기술은 아니다. 즉각적인 액추에이터 제어(Actuator Control)는 ROS 2, 실시간 미들웨어(Real-Time Middleware), 임베디드 제어기(Embedded Controller), 필드 레벨 네트워크(Field-Level Network)에서 수행해야 한다.

실용적인 플릿 파이프라인(Fleet Pipeline)은 엣지 게이트웨이가 로봇 텔레메트리를 카프카에 발행하는 것에서 시작할 수 있다. 카프카 스트림즈는 레코드를 정규화(Normalization)하고 잘못된 측정값을 제거하며 플릿 메타데이터로 이벤트를 보강하고 시간창 단위로 값을 집계하며 임계값 조건(Threshold Condition)을 감지할 수 있다. 이후 처리된 토픽은 각 후속 애플리케이션이 동일한 변환을 독립적으로 반복하지 않고도 대시보드(Dashboard), 유지보수 시스템, 디지털 트윈, 데이터 레이크(Data Lake), AI 파이프라인에 데이터를 제공할 수 있다.

카프카 스트림즈가 소비자 조정의 상당 부분을 내부적으로 관리하더라도 백프레셔(Backpressure)와 소비자 지연(Consumer Lag)은 계속 모니터링해야 한다. 처리 속도가 입력 이벤트 생성 속도보다 느려지면 레코드가 카프카에 누적되고 처리 지연시간이 증가한다. 운영자는 태스크 처리량(Task Throughput), 소비자 지연, 처리 지연시간(Processing Latency), 상태 저장소 크기(State-Store Size), 복원 시간(Restore Duration), 예외(Exception), 브로커 상태(Broker Condition)를 모니터링하여 스트리밍 애플리케이션이 플릿 데이터 발생률을 지속적으로 처리할 수 있는지 판단해야 한다.

확장(Scaling)은 단순히 애플리케이션 인스턴스 수만을 기준으로 하는 것이 아니라 파티션 분포(Partition Distribution), 이벤트 발생률(Event Rate), 상태 크기(State Size), 처리 복잡도(Processing Complexity), 요구 지연시간을 기반으로 결정해야 한다. CPU 집약적인 변환(CPU-Intensive Transformation), 대규모 조인, 긴 윈도, 큰 상태 저장소는 더 많은 컴퓨팅 및 저장 자원을 요구할 수 있다. 소스 토픽의 파티션 수가 너무 적거나 후속 종속 시스템(Downstream Dependency)이 주요 병목이라면 인스턴스를 증가시켜도 얻을 수 있는 효과는 제한적이다.

장애 처리(Fault Handling)는 하나의 잘못된 이벤트가 전체 로봇 데이터 파이프라인을 불필요하게 중단시키지 않도록 해야 한다. 애플리케이션에는 직렬화 실패(Serialization Failure), 잘못된 스키마(Invalid Schema), 예상하지 못한 값(Unexpected Value), 처리 예외(Processing Exception), 사용할 수 없는 종속 서비스에 대한 정책이 필요하다. 운영 요구사항이 허용한다면 문제가 있는 레코드를 전용 오류 토픽(Error Topic)으로 격리하여 이후 진단할 수 있도록 하고, 정상 이벤트는 토폴로지를 통해 계속 처리되도록 구성할 수 있다.

피지컬 AI(Physical AI)와 자율 로봇 플릿(Autonomous Robot Fleet)에서 카프카 스트림즈는 원시 운영 이벤트(Raw Operational Event)와 상위 수준 지능(Higher-Level Intelligence) 사이를 연결하는 실시간 변환 계층(Real-Time Transformation Layer)으로 활용할 수 있다. 분산된 로봇 관측 데이터를 정규화된 상태(Normalized State), 시간 기반 집계(Temporal Aggregate), 상관 이벤트(Correlated Event), 경보(Alert), AI 준비 데이터 스트림(AI-Ready Data Stream)으로 변환할 수 있다. 이를 카프카 토픽과 내구성 있는 이벤트 저장(Durable Event Storage) 기능과 결합하면 엣지 로봇을 플릿 관리(Fleet Management), 분석, 디지털 트윈, 학습 시스템(Learning System)과 연결하는 확장 가능한 처리 백본(Scalable Processing Backbone)을 구축할 수 있다.

##  

## 05.06 NATS JetStream: Lightweight High-Performance Messaging [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

NATS is a lightweight, high-performance messaging system designed for distributed applications that require simple communication, low latency, and efficient resource usage. It supports publish-subscribe, request-reply, and queue-based communication through a compact protocol and lightweight server architecture. In robotics, NATS can connect robot services, edge computers, fleet applications, AI components, and cloud systems without introducing a heavy messaging infrastructure.

The basic communication model of NATS is organized around subjects rather than traditional queues or complex routing exchanges. Publishers send messages to named subjects, and subscribers express interest in those subjects. A subject can represent robot telemetry, mission commands, diagnostics, localization events, or service requests. Publishers and subscribers remain logically decoupled, allowing individual components to be added, removed, restarted, or scaled independently.

NATS subjects can be organized hierarchically using dot-separated names such as robot.amr01.telemetry or fleet.site01.alerts. Subscribers can use wildcard patterns to receive groups of related subjects without explicitly subscribing to every individual stream. This hierarchical naming model is particularly useful for robot fleets because subjects can naturally represent sites, robot identifiers, subsystems, sensors, missions, and event categories.

Core NATS provides extremely lightweight publish-subscribe messaging intended for applications where speed and simplicity are primary requirements. Messages are normally delivered to currently connected subscribers without being stored as a durable historical log. If a subscriber is unavailable when a Core NATS message is published, that subscriber generally cannot retrieve the earlier message later. This behavior makes Core NATS suitable for transient real-time information.

Request-reply communication extends the publish-subscribe model by allowing one service to send a request and receive a response from another service. NATS implements this pattern efficiently using reply subjects rather than requiring a separate protocol. A fleet controller can request robot status, an edge service can request AI inference, or a diagnostic application can query another service while maintaining loose coupling between communicating components.

Queue groups provide load-balanced message distribution among multiple subscribers. Several service instances can subscribe to the same subject using a shared queue group, and NATS distributes messages among the available members rather than sending every message to every instance. This model enables horizontal scaling of stateless robot services such as event processors, command validators, telemetry transformers, and inference gateways.

JetStream extends NATS with persistence, replay, acknowledgement, retention, and durable consumer capabilities. Instead of treating every message as transient, JetStream can store selected subject traffic inside persistent streams. Consumers can then process stored messages immediately or later, recover after disconnection, and replay earlier events. This allows one NATS deployment to support both lightweight transient communication and durable event processing.

A JetStream stream defines which NATS subjects should be captured and how their messages should be retained. For example, a stream can store robot telemetry, mission events, safety alerts, or diagnostic records while ordinary service discovery traffic remains transient. Stream configuration can specify storage type, retention behavior, replication, message limits, age limits, and other policies according to operational requirements.

JetStream supports memory and file storage depending on performance and durability requirements. Memory storage provides very fast access but is primarily appropriate when persistence across process or host failures is not required at the same level. File-backed storage preserves messages on disk and is more suitable for durable operational data. Robotics architectures can select storage policies independently for different event categories.

Consumers represent applications that retrieve messages from JetStream streams. A consumer maintains delivery and acknowledgement state so processing can continue from an appropriate position after restart or disconnection. Durable consumers preserve their identity and progress across sessions, making them suitable for monitoring, analytics, maintenance, or integration services that must reliably process robot events over extended periods.

JetStream supports both push-based and pull-based consumption models. Push consumers receive messages as the server delivers them, which can simplify event-driven applications requiring immediate delivery. Pull consumers explicitly request available messages and can control processing rate more directly. Pull-based processing is useful when robot analytics or AI services need to regulate workloads according to available CPU, GPU, memory, or downstream capacity.

Acknowledgements allow consumers to indicate whether delivered messages have been processed successfully. If acknowledgement is not received according to the configured policy, JetStream can make messages available for redelivery. Applications must therefore expect possible duplicate delivery and should use idempotent processing where repeated events could cause undesirable database changes, duplicate commands, or repeated operational actions.

Retention policies determine when messages can be removed from a stream. JetStream can retain messages according to limits, interest, or work-queue-oriented semantics depending on the intended processing model. Age, message count, and storage limits can further constrain retained data. These mechanisms allow short-lived edge event buffers and more durable operational streams to be implemented within the same messaging technology.

Message replay is valuable for debugging, recovery, testing, and reconstruction of robot behavior. A service can begin consuming from newly arriving messages, from an earlier sequence position, or according to available historical data. Recorded mission events or diagnostics can therefore be reprocessed after an application update, while a digital-twin or analytics service can reconstruct selected historical operating sequences.

JetStream can replicate stream data across multiple NATS servers to improve availability and fault tolerance. A clustered deployment distributes messaging services and can preserve durable data when individual nodes become unavailable. Replication introduces additional storage and network costs, so the number of replicas should reflect the importance of the stream, expected failure conditions, and required balance between durability and operational efficiency.

NATS is particularly attractive at the edge because the server and client architecture is relatively lightweight. A small robot site or edge computer can operate messaging services without requiring the resource footprint associated with larger data-streaming platforms. NATS servers can also be connected through distributed topologies, enabling communication across local edge environments, regional infrastructure, and centralized systems.

Leaf nodes provide a mechanism for extending a NATS system into remote or edge locations while preserving local communication. Robots and local services can communicate through a nearby server, while selected traffic is exchanged with an upstream NATS infrastructure. This architecture can reduce dependence on continuous wide-area connectivity and is useful for factories, hospitals, warehouses, campuses, and outdoor robot deployments.

Subject design should follow clear naming and authorization boundaries. A fleet may organize subjects according to site, robot, subsystem, and event type, while separating command subjects from telemetry subjects. Consistent naming improves routing, observability, security, and operational management. Excessively dynamic or ambiguous subject structures can make authorization policies and fleet-wide monitoring significantly more difficult.

Security can be applied through encrypted connections, authentication, accounts, users, credentials, and subject-level permissions. Production robotics systems should restrict which components can publish commands, consume sensitive diagnostics, or access fleet-wide information. Separating robot, edge, fleet, and administrative permissions helps prevent a compromised or incorrectly configured service from obtaining unnecessary messaging authority.

Observability should include connection counts, message rates, subscription behavior, stream storage, consumer acknowledgement status, pending messages, redelivery, and processing delay. JetStream consumer backlog can indicate that downstream services are unable to keep pace with incoming robot events. Monitoring these signals allows operators to distinguish network problems, consumer bottlenecks, storage pressure, and abnormal event-generation rates.

NATS and JetStream should not replace deterministic field-level or hard real-time robot communication. They are appropriate for service communication, telemetry, commands at supervisory levels, fleet coordination, event distribution, and asynchronous processing, but motor-control and safety-critical loops generally require bounded real-time mechanisms. A layered architecture can therefore combine real-time control locally with NATS-based distributed service messaging.

For a Physical AI robot fleet, Core NATS can provide fast communication among edge services while JetStream preserves selected telemetry, mission events, diagnostics, and AI results. Consumers can feed fleet monitoring, predictive maintenance, digital twins, data platforms, and AI pipelines. This combination creates a lightweight messaging backbone that can begin at the robot edge and expand toward distributed on-premise and cloud infrastructure.

내츠(NATS)는 단순한 통신, 낮은 지연시간(Low Latency), 효율적인 자원 사용(Resource Usage)이 필요한 분산 애플리케이션(Distributed Application)을 위해 설계된 경량 고성능 메시징 시스템(Lightweight High-Performance Messaging System)이다. 간결한 프로토콜(Compact Protocol)과 경량 서버 아키텍처(Lightweight Server Architecture)를 기반으로 발행-구독(Publish-Subscribe), 요청-응답(Request-Reply), 큐 기반 통신(Queue-Based Communication)을 지원한다. 로보틱스에서는 무거운 메시징 인프라를 도입하지 않고도 로봇 서비스, 엣지 컴퓨터(Edge Computer), 플릿 애플리케이션(Fleet Application), AI 구성 요소, 클라우드 시스템을 연결할 수 있다.

내츠의 기본 통신 모델(Basic Communication Model)은 전통적인 큐나 복잡한 라우팅 익스체인지(Routing Exchange) 대신 서브젝트(Subject)를 중심으로 구성된다. 발행자(Publisher)는 이름이 지정된 서브젝트에 메시지를 전송하고 구독자(Subscriber)는 해당 서브젝트에 대한 관심을 등록한다. 서브젝트는 로봇 텔레메트리(Robot Telemetry), 임무 명령(Mission Command), 진단(Diagnostics), 위치추정 이벤트(Localization Event), 서비스 요청(Service Request)을 표현할 수 있다. 발행자와 구독자는 논리적으로 분리되어 개별 구성 요소를 독립적으로 추가, 제거, 재시작 또는 확장할 수 있다.

내츠 서브젝트(NATS Subject)는 robot.amr01.telemetry 또는 fleet.site01.alerts와 같이 점으로 구분된 이름(Dot-Separated Name)을 사용하여 계층적으로 구성할 수 있다. 구독자는 모든 개별 스트림을 명시적으로 구독하지 않고 와일드카드 패턴(Wildcard Pattern)을 사용하여 관련된 서브젝트 그룹을 수신할 수 있다. 이러한 계층적 명명 모델(Hierarchical Naming Model)은 사이트, 로봇 식별자(Robot Identifier), 서브시스템(Subsystem), 센서, 임무, 이벤트 유형을 자연스럽게 표현할 수 있어 로봇 플릿(Robot Fleet)에 특히 유용하다.

코어 내츠(Core NATS)는 속도와 단순성을 주요 요구사항으로 하는 애플리케이션을 위해 매우 가벼운 발행-구독 메시징(Publish-Subscribe Messaging)을 제공한다. 일반적으로 메시지는 내구성 있는 과거 로그(Durable Historical Log)로 저장되지 않고 현재 연결된 구독자에게 전달된다. 코어 내츠 메시지가 발행되는 시점에 구독자가 연결되어 있지 않다면 해당 구독자는 일반적으로 이전 메시지를 나중에 다시 가져올 수 없다. 이러한 특성으로 인해 코어 내츠는 일시적인 실시간 정보(Transient Real-Time Information)에 적합하다.

요청-응답 통신(Request-Reply Communication)은 하나의 서비스가 요청을 전송하고 다른 서비스로부터 응답을 받을 수 있도록 발행-구독 모델을 확장한다. 내츠는 별도의 프로토콜을 요구하는 대신 응답 서브젝트(Reply Subject)를 사용하여 이 패턴을 효율적으로 구현한다. 플릿 제어기(Fleet Controller)는 로봇 상태를 요청하고, 엣지 서비스(Edge Service)는 AI 추론을 요청하며, 진단 애플리케이션은 다른 서비스를 조회할 수 있으면서도 통신 구성 요소 사이의 느슨한 결합(Loose Coupling)을 유지할 수 있다.

큐 그룹(Queue Group)은 여러 구독자 사이에서 부하가 균형화된 메시지 분배(Load-Balanced Message Distribution)를 제공한다. 여러 서비스 인스턴스가 공유 큐 그룹을 사용하여 동일한 서브젝트를 구독하면 내츠는 모든 인스턴스에 동일한 메시지를 전송하는 대신 사용 가능한 구성원 사이에 메시지를 분배한다. 이러한 모델은 이벤트 프로세서(Event Processor), 명령 검증기(Command Validator), 텔레메트리 변환기(Telemetry Transformer), 추론 게이트웨이(Inference Gateway)와 같은 무상태 로봇 서비스(Stateless Robot Service)의 수평 확장(Horizontal Scaling)을 가능하게 한다.

제트스트림(JetStream)은 내츠에 영속성(Persistence), 재생(Replay), 승인(Acknowledgement), 보존(Retention), 내구성 소비자(Durable Consumer) 기능을 추가한다. 모든 메시지를 일시적인 데이터로 취급하는 대신 선택된 서브젝트 트래픽을 영속적인 스트림(Persistent Stream)에 저장할 수 있다. 소비자는 저장된 메시지를 즉시 또는 나중에 처리하고 연결이 끊어진 후 복구하며 이전 이벤트를 재생할 수 있다. 따라서 하나의 내츠 배포 환경에서 경량 일시적 통신과 내구성 있는 이벤트 처리(Durable Event Processing)를 동시에 지원할 수 있다.

제트스트림 스트림(JetStream Stream)은 어떤 내츠 서브젝트를 수집하고 해당 메시지를 어떻게 보존할지를 정의한다. 예를 들어 스트림은 로봇 텔레메트리, 임무 이벤트(Mission Event), 안전 경보(Safety Alert), 진단 기록(Diagnostic Record)을 저장하면서 일반적인 서비스 탐색 트래픽(Service Discovery Traffic)은 일시적인 형태로 유지할 수 있다. 스트림 구성(Stream Configuration)은 운영 요구사항에 따라 저장 유형(Storage Type), 보존 동작(Retention Behavior), 복제(Replication), 메시지 제한(Message Limit), 기간 제한(Age Limit) 등의 정책을 지정할 수 있다.

제트스트림은 성능과 내구성 요구사항에 따라 메모리 저장소(Memory Storage)와 파일 저장소(File Storage)를 지원한다. 메모리 저장소는 매우 빠른 접근 성능을 제공하지만 프로세스 또는 호스트 장애 이후에도 동일한 수준의 영속성이 요구되지 않는 경우에 주로 적합하다. 파일 기반 저장소(File-Backed Storage)는 메시지를 디스크에 보존하므로 내구성 있는 운영 데이터에 더 적합하다. 로보틱스 아키텍처에서는 서로 다른 이벤트 유형마다 독립적으로 저장 정책(Storage Policy)을 선택할 수 있다.

소비자(Consumer)는 제트스트림 스트림에서 메시지를 가져오는 애플리케이션을 나타낸다. 소비자는 전달 및 승인 상태(Delivery and Acknowledgement State)를 유지하므로 재시작 또는 연결 중단 이후 적절한 위치부터 처리를 계속할 수 있다. 내구성 소비자는 여러 세션(Session)에 걸쳐 자신의 식별 정보와 처리 진행 상태를 유지하므로 장기간에 걸쳐 로봇 이벤트를 안정적으로 처리해야 하는 모니터링, 분석, 유지보수, 통합 서비스(Integration Service)에 적합하다.

제트스트림은 푸시 기반 소비(Push-Based Consumption)와 풀 기반 소비(Pull-Based Consumption)를 모두 지원한다. 푸시 소비자(Push Consumer)는 서버가 전달하는 메시지를 수신하므로 즉각적인 전달이 필요한 이벤트 기반 애플리케이션(Event-Driven Application)을 단순하게 구성할 수 있다. 풀 소비자(Pull Consumer)는 사용 가능한 메시지를 명시적으로 요청하여 처리 속도를 보다 직접적으로 제어할 수 있다. 풀 기반 처리는 로봇 분석이나 AI 서비스가 사용 가능한 CPU, GPU, 메모리 또는 후속 처리 용량(Downstream Capacity)에 따라 워크로드를 조절해야 하는 경우 유용하다.

승인(Acknowledgement)을 사용하면 소비자는 전달된 메시지가 성공적으로 처리되었는지를 표시할 수 있다. 설정된 정책에 따라 승인이 수신되지 않으면 제트스트림은 해당 메시지를 재전달(Redelivery)할 수 있다. 따라서 애플리케이션은 중복 전달(Duplicate Delivery)이 발생할 가능성을 고려해야 하며 반복되는 이벤트가 바람직하지 않은 데이터베이스 변경, 중복 명령(Duplicate Command), 반복적인 운영 동작을 발생시킬 수 있다면 멱등 처리(Idempotent Processing)를 사용해야 한다.

보존 정책(Retention Policy)은 스트림에서 메시지를 언제 제거할 수 있는지를 결정한다. 제트스트림은 의도된 처리 모델에 따라 제한 기반(Limits-Based), 관심 기반(Interest-Based), 작업 큐 기반(Work-Queue-Oriented) 의미론을 이용하여 메시지를 보존할 수 있다. 메시지 기간(Age), 메시지 수(Message Count), 저장 용량 제한(Storage Limit)을 통해 보존 데이터를 추가로 제한할 수 있다. 이러한 메커니즘을 사용하면 하나의 메시징 기술 안에서 단기간 엣지 이벤트 버퍼(Edge Event Buffer)와 더욱 내구성 있는 운영 스트림을 함께 구현할 수 있다.

메시지 재생(Message Replay)은 로봇 동작의 디버깅(Debugging), 복구(Recovery), 테스트(Testing), 재구성(Reconstruction)에 유용하다. 서비스는 새롭게 도착하는 메시지부터 소비를 시작하거나 이전 시퀀스 위치(Sequence Position) 또는 사용 가능한 과거 데이터를 기준으로 처리를 시작할 수 있다. 따라서 기록된 임무 이벤트나 진단 데이터를 애플리케이션 업데이트 이후 다시 처리할 수 있으며, 디지털 트윈(Digital Twin) 또는 분석 서비스가 선택된 과거 운영 시퀀스를 재구성할 수도 있다.

제트스트림은 여러 내츠 서버에 스트림 데이터를 복제하여 가용성(Availability)과 장애 허용성(Fault Tolerance)을 향상시킬 수 있다. 클러스터 배포(Clustered Deployment)는 메시징 서비스를 분산하고 개별 노드를 사용할 수 없게 되더라도 내구성 있는 데이터를 보존할 수 있다. 복제에는 추가적인 저장소 및 네트워크 비용이 발생하므로 복제본 수(Number of Replicas)는 스트림 중요도, 예상 장애 조건, 내구성과 운영 효율성 사이에서 요구되는 균형을 고려하여 결정해야 한다.

내츠는 서버와 클라이언트 아키텍처(Server and Client Architecture)가 상대적으로 가볍기 때문에 엣지(Edge) 환경에서 특히 유용하다. 소규모 로봇 사이트 또는 엣지 컴퓨터에서도 대규모 데이터 스트리밍 플랫폼보다 적은 자원 사용량(Resource Footprint)으로 메시징 서비스를 운영할 수 있다. 또한 내츠 서버를 분산 토폴로지(Distributed Topology)로 연결하여 로컬 엣지 환경, 지역 인프라(Regional Infrastructure), 중앙 집중식 시스템(Centralized System) 사이의 통신을 구현할 수 있다.

리프 노드(Leaf Node)는 로컬 통신을 유지하면서 내츠 시스템을 원격 또는 엣지 위치로 확장하기 위한 메커니즘을 제공한다. 로봇과 로컬 서비스는 가까운 서버를 통해 통신하고 선택된 트래픽만 상위 내츠 인프라(Upstream NATS Infrastructure)와 교환할 수 있다. 이러한 아키텍처는 지속적인 광역 네트워크 연결(Wide-Area Connectivity)에 대한 의존성을 줄일 수 있으므로 공장, 병원, 물류창고, 캠퍼스, 실외 로봇 배포 환경에 유용하다.

서브젝트 설계(Subject Design)는 명확한 명명 규칙(Naming Convention)과 권한 부여 경계(Authorization Boundary)를 따라야 한다. 플릿은 사이트, 로봇, 서브시스템, 이벤트 유형에 따라 서브젝트를 구성하면서 명령 서브젝트(Command Subject)와 텔레메트리 서브젝트(Telemetry Subject)를 분리할 수 있다. 일관된 명명 방식은 라우팅(Routing), 관찰 가능성(Observability), 보안(Security), 운영 관리를 향상시킨다. 지나치게 동적이거나 모호한 서브젝트 구조는 권한 정책과 플릿 전체 모니터링을 크게 어렵게 만들 수 있다.

보안은 암호화된 연결(Encrypted Connection), 인증(Authentication), 계정(Account), 사용자(User), 자격 증명(Credential), 서브젝트 수준 권한(Subject-Level Permission)을 통해 적용할 수 있다. 운영 로보틱스 시스템에서는 어떤 구성 요소가 명령을 발행하고 민감한 진단 정보를 소비하며 플릿 전체 정보에 접근할 수 있는지를 제한해야 한다. 로봇, 엣지, 플릿, 관리 권한(Administrative Permission)을 분리하면 침해되거나 잘못 구성된 서비스가 불필요한 메시징 권한을 획득하는 것을 방지하는 데 도움이 된다.

관찰 가능성(Observability)은 연결 수(Connection Count), 메시지 발생률(Message Rate), 구독 동작(Subscription Behavior), 스트림 저장 용량(Stream Storage), 소비자 승인 상태(Consumer Acknowledgement Status), 대기 메시지(Pending Message), 재전달, 처리 지연(Processing Delay)을 포함해야 한다. 제트스트림 소비자 백로그(Consumer Backlog)는 후속 서비스가 입력되는 로봇 이벤트의 속도를 따라가지 못하고 있음을 나타낼 수 있다. 이러한 신호를 모니터링하면 운영자는 네트워크 문제, 소비자 병목(Consumer Bottleneck), 저장 공간 압박(Storage Pressure), 비정상적인 이벤트 발생률을 구분할 수 있다.

내츠와 제트스트림은 결정론적 필드 레벨 통신(Deterministic Field-Level Communication) 또는 하드 실시간 로봇 통신(Hard Real-Time Robot Communication)을 대체해서는 안 된다. 서비스 통신, 텔레메트리, 상위 제어 수준의 명령(Supervisory-Level Command), 플릿 조정(Fleet Coordination), 이벤트 분배(Event Distribution), 비동기 처리(Asynchronous Processing)에는 적합하지만 모터 제어와 안전 필수 루프(Safety-Critical Loop)는 일반적으로 시간 경계가 보장되는 실시간 메커니즘(Bounded Real-Time Mechanism)을 필요로 한다. 따라서 계층화된 아키텍처(Layered Architecture)는 로컬 실시간 제어와 내츠 기반 분산 서비스 메시징을 결합할 수 있다.

피지컬 AI(Physical AI) 로봇 플릿에서 코어 내츠는 엣지 서비스 사이의 빠른 통신을 제공하고 제트스트림은 선택된 텔레메트리, 임무 이벤트, 진단, AI 결과를 보존할 수 있다. 소비자는 플릿 모니터링(Fleet Monitoring), 예측 유지보수(Predictive Maintenance), 디지털 트윈, 데이터 플랫폼(Data Platform), AI 파이프라인(AI Pipeline)에 데이터를 공급할 수 있다. 이러한 조합을 통해 로봇 엣지에서 시작하여 분산된 온프레미스(On-Premise) 및 클라우드 인프라까지 확장할 수 있는 경량 메시징 백본(Lightweight Messaging Backbone)을 구축할 수 있다.

##  

## 05.07 RabbitMQ Architecture and Exchange Types [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

RabbitMQ is a message broker designed to receive messages from producers, route them according to defined rules, and deliver them through queues to consumers. Its architecture emphasizes flexible routing, reliable delivery, acknowledgement, and workload distribution. In robotics, RabbitMQ can connect robot gateways, fleet services, enterprise applications, monitoring systems, maintenance platforms, and asynchronous backend processes.

The RabbitMQ messaging model separates producers from consumers through exchanges and queues. Producers normally publish messages to an exchange rather than directly selecting a consumer. The exchange evaluates routing rules and determines which queue or queues should receive each message. Consumers then retrieve messages from those queues, allowing message generation, routing, buffering, and processing to evolve independently.

A connection represents a long-lived TCP communication path between an application and RabbitMQ. Applications typically create multiple lightweight channels inside one connection instead of opening a separate network connection for every operation. Channels provide logical communication sessions for publishing, consuming, acknowledgements, and administrative operations while reducing the resource overhead associated with large numbers of TCP connections.

An exchange is the routing component that receives published messages and determines their destinations. RabbitMQ supports several exchange types whose behavior is defined by routing rules. The most important built-in types are direct, topic, fanout, and headers exchanges. Selecting an exchange type determines whether messages are routed by exact keys, patterns, broadcast behavior, or message-header attributes.

A queue stores messages until they can be delivered to consumers according to the configured messaging behavior. Queues can buffer temporary differences between producer and consumer processing rates, allowing a robot event producer to continue publishing while downstream services process messages independently. Queue properties can control durability, exclusivity, expiration, size limits, priorities, and other operational characteristics.

Bindings connect exchanges to queues and define the conditions under which messages are routed between them. A binding can include a routing key or other arguments depending on the exchange type. Because exchanges, queues, and bindings are independent objects, RabbitMQ can implement complex communication topologies without requiring producers to know the addresses or identities of individual consumers.

A direct exchange routes a message to queues whose binding key exactly matches the message routing key. This provides predictable one-to-one or selective one-to-many routing. A robotics system might publish messages using keys such as robot.command, robot.diagnostics, or maintenance.request, while queues bind only to the exact categories required by their corresponding backend services.

Direct exchanges are useful when message categories are clearly defined and exact routing behavior is preferred over pattern matching. For example, separate queues can receive charging commands, mission requests, or maintenance jobs. Multiple queues may use the same binding key when several independent services require identical message categories, while competing consumers can share one queue when workload distribution is required.

A topic exchange extends routing by interpreting routing keys as dot-separated words and matching them against binding patterns. The asterisk wildcard matches one word, while the hash wildcard can match zero or more words. A fleet architecture can therefore use routing keys such as site01.amr07.telemetry or site02.amr15.alert and allow services to subscribe to meaningful groups of robot events.

Topic exchanges are particularly suitable for hierarchical robot event classification. A monitoring service could receive events from every robot at one site, while a diagnostic service could receive diagnostic events from all sites. This allows routing policies to follow site, robot, subsystem, mission, or event type without requiring a separate exchange for every combination of communication requirements.

A fanout exchange ignores routing keys and distributes every published message to all queues bound to that exchange. It therefore implements broadcast-style communication. In robotics, fanout routing can distribute common configuration notifications, fleet-wide state changes, service announcements, or operational events to several independent systems that must each receive their own copy of the same message.

Fanout exchanges should be distinguished from multiple consumers attached to one queue. When several consumers share a single queue, messages are normally distributed among those consumers as competing workers. When several queues are bound to a fanout exchange, each queue receives a copy of the published message. This distinction determines whether the architecture implements workload balancing or independent broadcast delivery.

A headers exchange routes messages using attributes stored in message headers instead of relying primarily on routing keys. Bindings can specify header values and matching rules, enabling decisions based on properties such as robot type, site, priority, mission class, or event category. Headers exchanges can be useful when routing conditions do not fit naturally into a single hierarchical routing-key structure.

Header-based routing provides flexibility but can introduce additional complexity compared with direct or topic exchanges. If robot events can be represented clearly using stable routing keys, simpler exchange types are often easier to operate and observe. Headers exchanges become more valuable when routing requires combinations of independent attributes and when those attributes already form part of the message metadata.

The default exchange is a special built-in direct exchange that provides straightforward routing to queues using queue names as routing keys. It simplifies basic messaging because applications can address a queue without explicitly creating a custom exchange. However, larger robotic systems generally benefit from deliberately designed exchanges and bindings that separate producers from specific queue names and support future topology changes.

Consumer acknowledgements allow RabbitMQ to determine whether a delivered message has been processed successfully. With manual acknowledgements, a consumer confirms completion after processing. If the consumer fails before acknowledgement, RabbitMQ can make the message available for redelivery. This mechanism improves reliability but requires applications to expect duplicates and design idempotent processing where repeated operations could have undesirable effects.

Publisher confirms provide reliability on the publishing side by allowing RabbitMQ to acknowledge that messages have been accepted by the broker according to the applicable configuration. They help producers distinguish successfully accepted messages from uncertain transmissions caused by connection failures. Robot gateways and backend services can combine publisher confirms with retry policies to reduce the risk of silently losing important operational events.

Message durability depends on several related settings rather than a single option. Durable queues and durable exchanges can survive broker restarts, while persistent message handling can improve the ability of messages to survive failures. Reliability requirements should therefore be designed across publishers, exchanges, queues, acknowledgements, storage, and cluster configuration instead of assuming that one durability flag guarantees complete protection.

RabbitMQ can distribute work among multiple consumers connected to the same queue. Prefetch configuration controls how many unacknowledged messages may be delivered to a consumer, helping prevent one worker from receiving excessive work while others remain available. This is useful for robot backend tasks such as image-processing jobs, report generation, mission validation, database operations, or asynchronous AI processing.

Dead-letter exchanges provide a controlled destination for messages that cannot remain in their original queues. Messages may be dead-lettered after rejection, expiration, queue-length conditions, or other configured situations. A robotics platform can route failed commands or malformed events to dedicated diagnostic queues where operators or recovery services inspect them without blocking normal message processing.

RabbitMQ clustering and replicated queue technologies can improve availability when individual broker nodes fail. Production design must consider node placement, storage, network reliability, queue replication, client reconnection, and failure domains. High availability does not eliminate application-level responsibilities such as retry handling, duplicate detection, acknowledgement strategy, and graceful behavior during temporary broker unavailability.

Security should be integrated into the messaging architecture through TLS, authentication, virtual hosts, users, and permissions. Virtual hosts can isolate applications or environments, while access controls determine which resources a user may configure, publish to, or consume from. Robot command paths should generally receive stricter permissions than ordinary telemetry or monitoring paths to enforce least-privilege communication.

RabbitMQ observability should cover connections, channels, exchanges, queues, message rates, acknowledgements, unacknowledged messages, consumer counts, queue depth, memory, disk usage, and node health. Growing queue depth may indicate that consumers cannot keep pace with producers, while repeated redelivery can reveal processing failures. Monitoring these signals is essential for maintaining predictable robot backend services.

In a Physical AI robot architecture, RabbitMQ is particularly useful for command workflows, task distribution, enterprise integration, and asynchronous service coordination where flexible routing is important. Direct exchanges can route explicit commands, topic exchanges can classify fleet events, fanout exchanges can broadcast shared notifications, and headers exchanges can implement attribute-based routing, creating a flexible messaging layer between robots and backend services.

래빗엠큐(RabbitMQ)는 생산자(Producer)로부터 메시지를 수신하고, 정의된 규칙에 따라 메시지를 라우팅(Routing)하며, 큐(Queue)를 통해 소비자(Consumer)에게 전달하도록 설계된 메시지 브로커(Message Broker)이다. 래빗엠큐의 아키텍처는 유연한 라우팅, 신뢰성 있는 전달(Reliable Delivery), 승인(Acknowledgement), 워크로드 분산(Workload Distribution)을 강조한다. 로보틱스에서는 로봇 게이트웨이(Robot Gateway), 플릿 서비스(Fleet Service), 기업 애플리케이션(Enterprise Application), 모니터링 시스템, 유지보수 플랫폼, 비동기 백엔드 프로세스(Asynchronous Backend Process)를 연결할 수 있다.

래빗엠큐 메시징 모델(RabbitMQ Messaging Model)은 익스체인지(Exchange)와 큐를 통해 생산자와 소비자를 분리한다. 생산자는 일반적으로 특정 소비자를 직접 선택하는 대신 익스체인지에 메시지를 발행한다. 익스체인지는 라우팅 규칙(Routing Rule)을 평가하여 어떤 큐가 메시지를 수신해야 하는지를 결정한다. 이후 소비자가 해당 큐에서 메시지를 가져가므로 메시지 생성, 라우팅, 버퍼링(Buffering), 처리를 서로 독립적으로 발전시킬 수 있다.

연결(Connection)은 애플리케이션과 래빗엠큐 사이의 장기간 유지되는 TCP 통신 경로를 의미한다. 애플리케이션은 일반적으로 각각의 작업마다 별도의 네트워크 연결을 생성하는 대신 하나의 연결 내부에 여러 개의 경량 채널(Channel)을 생성한다. 채널은 발행(Publishing), 소비(Consuming), 승인, 관리 작업(Administrative Operation)을 위한 논리적 통신 세션(Logical Communication Session)을 제공하면서 많은 TCP 연결에서 발생하는 자원 오버헤드(Resource Overhead)를 줄인다.

익스체인지는 발행된 메시지를 수신하고 목적지를 결정하는 라우팅 구성 요소(Routing Component)이다. 래빗엠큐는 라우팅 규칙에 따라 동작이 정의되는 여러 익스체인지 유형(Exchange Type)을 지원한다. 가장 중요한 기본 유형은 다이렉트(Direct), 토픽(Topic), 팬아웃(Fanout), 헤더(Headers) 익스체인지이다. 익스체인지 유형에 따라 메시지를 정확한 키(Key), 패턴(Pattern), 브로드캐스트(Broadcast), 메시지 헤더 속성(Message-Header Attribute) 중 어떤 기준으로 라우팅할지가 결정된다.

큐는 설정된 메시징 동작(Messaging Behavior)에 따라 소비자에게 전달될 때까지 메시지를 저장한다. 큐는 생산자와 소비자의 처리 속도 차이를 일시적으로 버퍼링할 수 있으므로 로봇 이벤트 생산자가 메시지를 계속 발행하는 동안 후속 서비스(Downstream Service)는 독립적인 속도로 메시지를 처리할 수 있다. 큐 속성(Queue Property)을 통해 내구성(Durability), 배타성(Exclusivity), 만료(Expiration), 크기 제한(Size Limit), 우선순위(Priority) 등의 운영 특성을 제어할 수 있다.

바인딩(Binding)은 익스체인지와 큐를 연결하고 메시지가 이들 사이에서 라우팅되는 조건을 정의한다. 바인딩에는 익스체인지 유형에 따라 라우팅 키(Routing Key) 또는 다른 인수(Argument)가 포함될 수 있다. 익스체인지, 큐, 바인딩이 서로 독립적인 객체이기 때문에 래빗엠큐는 생산자가 개별 소비자의 주소나 식별 정보를 알지 못하더라도 복잡한 통신 토폴로지(Communication Topology)를 구현할 수 있다.

다이렉트 익스체인지(Direct Exchange)는 메시지의 라우팅 키와 정확하게 일치하는 바인딩 키(Binding Key)를 가진 큐로 메시지를 전달한다. 이를 통해 예측 가능한 일대일(One-to-One) 또는 선택적인 일대다(Selective One-to-Many) 라우팅을 구현할 수 있다. 로봇 시스템에서는 robot.command, robot.diagnostics, maintenance.request와 같은 키를 사용하여 메시지를 발행하고 각각의 백엔드 서비스(Backend Service)가 필요한 정확한 범주에 해당하는 큐만 바인딩할 수 있다.

다이렉트 익스체인지는 메시지 범주가 명확하게 정의되고 패턴 매칭(Pattern Matching)보다 정확한 라우팅 동작이 필요한 경우 유용하다. 예를 들어 충전 명령(Charging Command), 임무 요청(Mission Request), 유지보수 작업(Maintenance Job)을 각각 별도의 큐로 전달할 수 있다. 여러 독립적인 서비스가 동일한 메시지 범주를 필요로 하는 경우 여러 큐에서 같은 바인딩 키를 사용할 수 있으며, 워크로드 분산이 필요하면 여러 경쟁 소비자(Competing Consumer)가 하나의 큐를 공유할 수 있다.

토픽 익스체인지(Topic Exchange)는 라우팅 키를 점으로 구분된 단어(Dot-Separated Word)로 해석하고 이를 바인딩 패턴(Binding Pattern)과 비교하여 라우팅 기능을 확장한다. 별표 와일드카드(Asterisk Wildcard)는 하나의 단어와 일치하고 해시 와일드카드(Hash Wildcard)는 0개 이상의 단어와 일치할 수 있다. 따라서 플릿 아키텍처에서는 site01.amr07.telemetry 또는 site02.amr15.alert와 같은 라우팅 키를 사용하고 서비스가 의미 있는 로봇 이벤트 그룹을 구독하도록 구성할 수 있다.

토픽 익스체인지는 계층적인 로봇 이벤트 분류(Hierarchical Robot Event Classification)에 특히 적합하다. 모니터링 서비스는 특정 사이트에 있는 모든 로봇의 이벤트를 수신할 수 있고, 진단 서비스는 모든 사이트에서 발생하는 진단 이벤트를 수신할 수 있다. 이를 통해 각각의 통신 요구사항 조합마다 별도의 익스체인지를 생성하지 않고도 사이트, 로봇, 서브시스템(Subsystem), 임무, 이벤트 유형에 따라 라우팅 정책을 구성할 수 있다.

팬아웃 익스체인지(Fanout Exchange)는 라우팅 키를 무시하고 해당 익스체인지에 바인딩된 모든 큐에 발행된 메시지를 전달한다. 따라서 브로드캐스트 방식의 통신(Broadcast-Style Communication)을 구현한다. 로보틱스에서는 공통 구성 알림(Common Configuration Notification), 플릿 전체 상태 변경(Fleet-Wide State Change), 서비스 공지(Service Announcement), 운영 이벤트를 동일한 메시지의 복사본을 각각 받아야 하는 여러 독립 시스템으로 분배할 수 있다.

팬아웃 익스체인지는 하나의 큐에 여러 소비자가 연결된 구조와 구분해야 한다. 여러 소비자가 하나의 큐를 공유하면 일반적으로 메시지는 경쟁 작업자(Competing Worker) 역할을 하는 소비자 사이에 분배된다. 반면 여러 큐가 팬아웃 익스체인지에 바인딩되어 있으면 각각의 큐가 발행된 메시지의 복사본을 수신한다. 이러한 차이에 따라 아키텍처가 워크로드 균형화(Workload Balancing)를 구현하는지 또는 독립적인 브로드캐스트 전달(Independent Broadcast Delivery)을 구현하는지가 결정된다.

헤더 익스체인지(Headers Exchange)는 주로 라우팅 키를 사용하는 대신 메시지 헤더(Message Header)에 저장된 속성을 기준으로 메시지를 라우팅한다. 바인딩에서 헤더 값(Header Value)과 매칭 규칙(Matching Rule)을 지정할 수 있으므로 로봇 유형(Robot Type), 사이트, 우선순위, 임무 등급(Mission Class), 이벤트 범주 등의 속성을 기준으로 라우팅을 결정할 수 있다. 헤더 익스체인지는 라우팅 조건을 하나의 계층적인 라우팅 키 구조로 자연스럽게 표현하기 어려운 경우 유용하다.

헤더 기반 라우팅(Header-Based Routing)은 높은 유연성을 제공하지만 다이렉트 또는 토픽 익스체인지와 비교하면 추가적인 복잡성이 발생할 수 있다. 로봇 이벤트를 안정적인 라우팅 키를 사용하여 명확하게 표현할 수 있다면 보다 단순한 익스체인지 유형이 운영과 관찰 측면에서 더 쉬울 수 있다. 헤더 익스체인지는 여러 독립적인 속성의 조합을 기준으로 라우팅해야 하고 해당 속성이 이미 메시지 메타데이터(Metadata)의 일부로 구성되어 있을 때 더욱 유용하다.

기본 익스체인지(Default Exchange)는 큐 이름(Queue Name)을 라우팅 키로 사용하여 큐로 직접 메시지를 전달할 수 있는 특별한 내장 다이렉트 익스체인지(Built-In Direct Exchange)이다. 별도의 사용자 정의 익스체인지(Custom Exchange)를 명시적으로 생성하지 않고도 애플리케이션이 큐를 지정할 수 있으므로 기본적인 메시징을 단순화한다. 그러나 대규모 로봇 시스템에서는 생산자를 특정 큐 이름으로부터 분리하고 향후 토폴로지 변경을 지원할 수 있도록 의도적으로 설계된 익스체인지와 바인딩을 사용하는 것이 일반적으로 더 유리하다.

소비자 승인(Consumer Acknowledgement)을 사용하면 래빗엠큐가 전달된 메시지가 성공적으로 처리되었는지를 판단할 수 있다. 수동 승인(Manual Acknowledgement)을 사용하는 경우 소비자는 처리를 완료한 이후 성공 여부를 확인한다. 소비자가 승인하기 전에 장애가 발생하면 래빗엠큐는 해당 메시지를 다시 전달 가능한 상태로 만들 수 있다. 이러한 메커니즘은 신뢰성을 향상시키지만 애플리케이션은 중복 가능성을 고려해야 하며 반복 작업이 바람직하지 않은 결과를 발생시킬 수 있다면 멱등 처리(Idempotent Processing)를 설계해야 한다.

발행자 확인(Publisher Confirm)은 래빗엠큐가 해당 구성에 따라 메시지를 브로커에서 수락했다는 사실을 생산자에게 확인할 수 있도록 함으로써 발행 측의 신뢰성을 제공한다. 이를 통해 생산자는 성공적으로 수락된 메시지와 연결 장애로 인해 전송 상태를 확신할 수 없는 메시지를 구분할 수 있다. 로봇 게이트웨이와 백엔드 서비스는 발행자 확인과 재시도 정책(Retry Policy)을 결합하여 중요한 운영 이벤트가 조용히 손실될 위험을 줄일 수 있다.

메시지 내구성(Message Durability)은 하나의 옵션이 아니라 서로 관련된 여러 설정에 의해 결정된다. 내구성 큐(Durable Queue)와 내구성 익스체인지(Durable Exchange)는 브로커가 재시작된 이후에도 유지될 수 있으며, 영속 메시지 처리(Persistent Message Handling)는 장애 상황에서 메시지를 유지할 가능성을 높일 수 있다. 따라서 신뢰성 요구사항은 하나의 내구성 플래그(Durability Flag)가 모든 보호를 보장한다고 가정하지 말고 생산자, 익스체인지, 큐, 승인, 저장소, 클러스터 구성을 종합적으로 고려하여 설계해야 한다.

래빗엠큐는 동일한 큐에 연결된 여러 소비자 사이에서 작업을 분배할 수 있다. 프리페치 설정(Prefetch Configuration)은 소비자가 아직 승인하지 않은 상태에서 전달받을 수 있는 메시지 수를 제어하여 하나의 작업자가 과도한 작업을 수신하는 동안 다른 작업자가 유휴 상태가 되는 것을 방지하는 데 도움을 준다. 이는 이미지 처리 작업(Image-Processing Job), 보고서 생성, 임무 검증(Mission Validation), 데이터베이스 작업, 비동기 AI 처리(Asynchronous AI Processing)와 같은 로봇 백엔드 작업에 유용하다.

데드 레터 익스체인지(Dead-Letter Exchange)는 원래 큐에 계속 유지할 수 없는 메시지를 제어된 목적지로 전달한다. 메시지는 거부(Rejection), 만료, 큐 길이 조건(Queue-Length Condition) 또는 기타 설정된 상황에 따라 데드 레터 처리(Dead-Lettering)될 수 있다. 로보틱스 플랫폼에서는 실패한 명령이나 잘못된 이벤트를 전용 진단 큐(Diagnostic Queue)로 전달하여 정상적인 메시지 처리를 방해하지 않으면서 운영자 또는 복구 서비스가 이를 검사하도록 할 수 있다.

래빗엠큐 클러스터링(RabbitMQ Clustering)과 복제 큐 기술(Replicated Queue Technology)은 개별 브로커 노드(Broker Node)에 장애가 발생했을 때 가용성(Availability)을 향상시킬 수 있다. 운영 설계에서는 노드 배치(Node Placement), 저장소, 네트워크 신뢰성(Network Reliability), 큐 복제(Queue Replication), 클라이언트 재연결(Client Reconnection), 장애 영역(Failure Domain)을 고려해야 한다. 고가용성(High Availability)을 구성하더라도 재시도 처리, 중복 탐지(Duplicate Detection), 승인 전략, 일시적인 브로커 장애 상황에서의 정상 동작과 같은 애플리케이션 수준의 책임이 사라지는 것은 아니다.

보안(Security)은 TLS, 인증(Authentication), 가상 호스트(Virtual Host), 사용자(User), 권한(Permission)을 통해 메시징 아키텍처에 통합해야 한다. 가상 호스트는 애플리케이션이나 환경을 서로 격리할 수 있으며 접근 제어(Access Control)는 사용자가 어떤 자원을 구성하고 메시지를 발행하거나 소비할 수 있는지를 결정한다. 로봇 명령 경로(Robot Command Path)는 일반적인 텔레메트리 또는 모니터링 경로보다 엄격한 권한을 적용하여 최소 권한 통신(Least-Privilege Communication)을 구현하는 것이 일반적으로 적절하다.

래빗엠큐 관찰 가능성(RabbitMQ Observability)은 연결, 채널, 익스체인지, 큐, 메시지 발생률(Message Rate), 승인, 미승인 메시지(Unacknowledged Message), 소비자 수(Consumer Count), 큐 깊이(Queue Depth), 메모리, 디스크 사용량(Disk Usage), 노드 상태(Node Health)를 포함해야 한다. 큐 깊이가 지속적으로 증가하면 소비자가 생산자의 처리 속도를 따라가지 못하고 있음을 의미할 수 있으며 반복적인 재전달은 처리 장애를 나타낼 수 있다. 이러한 신호를 모니터링하는 것은 예측 가능한 로봇 백엔드 서비스를 유지하는 데 필수적이다.

피지컬 AI(Physical AI) 로봇 아키텍처에서 래빗엠큐는 유연한 라우팅이 중요한 명령 워크플로(Command Workflow), 작업 분배(Task Distribution), 기업 시스템 통합(Enterprise Integration), 비동기 서비스 조정(Asynchronous Service Coordination)에 특히 유용하다. 다이렉트 익스체인지는 명시적인 명령을 라우팅하고, 토픽 익스체인지는 플릿 이벤트를 분류하며, 팬아웃 익스체인지는 공통 알림을 브로드캐스트하고, 헤더 익스체인지는 속성 기반 라우팅(Attribute-Based Routing)을 구현함으로써 로봇과 백엔드 서비스 사이에 유연한 메시징 계층(Flexible Messaging Layer)을 구성할 수 있다.

##  

## 05.08 Message Broker Security: SASL, TLS, ACL [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

Message broker security must protect communication among robots, edge systems, backend services, and cloud applications without disrupting the reliability and performance expected from distributed messaging. A secure architecture normally combines transport encryption, client authentication, authorization, credential management, auditing, and network isolation. SASL, TLS, and ACL mechanisms address different parts of this security model and should be designed as complementary controls.

Transport Layer Security, or TLS, protects network communication between messaging clients and brokers. Encryption prevents an observer on the network from easily reading telemetry, commands, credentials, or operational events while they are transmitted. TLS also provides integrity protection, helping detect modification of protected traffic. For robot fleets operating across Wi-Fi, Ethernet, cellular, or external networks, encrypted broker connections should be treated as a fundamental security requirement.

TLS depends on digital certificates and trusted certificate authorities to establish server identity. A client verifies the certificate presented by the broker before establishing a protected session, reducing the risk of connecting to an unauthorized endpoint. Certificate validation should include the appropriate trust chain and broker identity. Disabling certificate verification for convenience can undermine the protection that TLS is intended to provide.

Mutual TLS, commonly called mTLS, extends certificate authentication to both sides of the connection. The broker presents its certificate to the client, while the client also presents a certificate that identifies the connecting application or device. This approach is useful for robot fleets because individual robots, gateways, or services can receive distinct identities without relying entirely on shared usernames and passwords.

Certificate lifecycle management becomes important when hundreds or thousands of robots participate in the messaging infrastructure. Certificates require secure issuance, distribution, storage, renewal, rotation, and revocation. Long-lived credentials increase exposure when a robot or edge computer is compromised. Automated certificate management can reduce operational errors while allowing compromised or retired devices to lose access without changing credentials for the entire fleet.

Simple Authentication and Security Layer, or SASL, provides a framework for authentication between clients and messaging services. SASL does not define one universal authentication algorithm; instead, it supports multiple mechanisms that can be selected according to platform requirements. Message brokers such as Kafka can combine SASL authentication with TLS so that credentials and authentication exchanges occur through an encrypted communication channel.

Username-and-password mechanisms are relatively easy to deploy but require careful protection of stored secrets. Credentials should not be embedded directly in application source code, container images, or configuration repositories. Robot applications should obtain secrets from protected configuration or secret-management systems where practical. Password rotation, account separation, and strong access policies reduce the impact of leaked or reused credentials.

SASL mechanisms can also support stronger identity systems than static passwords. Depending on the broker and deployment environment, authentication may use mechanisms based on challenge-response credentials, tokens, Kerberos, or OAuth-oriented identity flows. The appropriate mechanism depends on infrastructure, device capabilities, identity providers, and operational complexity. Authentication design should remain consistent with the broader enterprise identity architecture.

Authentication establishes who a client is, but it does not determine what that client is allowed to do. Authorization must therefore be applied after identity verification. Access Control Lists, or ACLs, define permissions for messaging resources and operations. An authenticated robot might be allowed to publish its own telemetry while being prohibited from publishing commands to other robots or reading fleet-wide administrative events.

ACL design should follow the principle of least privilege. Each robot, edge service, analytics process, and operator application should receive only the permissions required for its intended function. Broad wildcard permissions may simplify initial deployment but can significantly increase the impact of compromised credentials. Security policies should therefore distinguish telemetry publishing, command consumption, administrative operations, and access to sensitive event streams.

Resource naming is closely related to authorization. Kafka topics, RabbitMQ exchanges and queues, and NATS subjects should use consistent naming structures that allow security policies to be expressed clearly. Names can represent sites, robot identifiers, subsystems, applications, or data classifications. A predictable hierarchy makes it easier to restrict one robot to its own communication namespace while allowing authorized fleet services to operate across multiple robots.

Command channels require particularly strict authorization because unauthorized publication can influence physical robot behavior. A telemetry producer generally should not possess permission to publish motion commands, mission instructions, charging requests, or safety-related control messages. Command publishers should use dedicated identities and narrowly defined permissions. Critical commands may also require validation by an independent application layer before execution.

Broker administrative privileges should be separated from ordinary messaging privileges. Applications that publish or consume operational data rarely require permission to create users, modify security policies, delete topics, alter queues, or change broker configuration. Dedicated administrative identities reduce the possibility that compromise of a robot application becomes compromise of the entire messaging infrastructure.

Network segmentation provides another security boundary around message brokers. Robot networks, edge servers, management systems, and external cloud connections can be separated through firewalls, virtual networks, or other network controls. Only required broker ports and approved communication paths should be exposed. Authentication and ACLs remain necessary because network location alone should not be considered sufficient proof that a client is trustworthy.

Security should also protect broker-to-broker communication. Kafka clusters, RabbitMQ clusters, NATS clusters, gateways, and distributed edge deployments may exchange messages and metadata across multiple nodes. Internal traffic can contain sensitive operational information and control data. Encryption and authenticated node identities help prevent unauthorized systems from joining the messaging infrastructure or intercepting inter-node communication.

Credential storage on robots requires special attention because physical devices may operate outside tightly controlled data centers. Private keys, passwords, and tokens should be protected using operating-system permissions, secure storage, hardware-backed security where available, or dedicated secret-management mechanisms. Diagnostic logs should avoid exposing credentials, tokens, certificate private keys, or complete authentication information.

Credential rotation should be designed before deployment rather than added after credentials expire or become compromised. Systems need procedures for introducing new certificates, passwords, or tokens while maintaining service continuity. Overlapping validity periods can support controlled transitions. Emergency revocation procedures are equally important because a stolen robot, compromised gateway, or leaked credential may require immediate access termination.

Audit logging provides evidence of security-relevant broker activity. Useful records include authentication successes and failures, authorization denials, administrative changes, unusual connection attempts, and access to sensitive messaging resources. Audit events should include sufficient identity and timing information for investigation while avoiding unnecessary exposure of secrets. Centralized analysis can correlate broker activity with robot and infrastructure events.

Monitoring can identify behavior that is technically authenticated but operationally abnormal. Examples include a robot suddenly accessing unexpected topics, a service generating unusually high message rates, repeated authorization failures, or connections originating from unexpected network locations. Security monitoring should therefore combine broker metrics, audit logs, network observations, and application context instead of relying only on successful authentication.

Denial-of-service resistance is also part of broker security. A compromised or malfunctioning client can consume broker resources by creating excessive connections, publishing messages at abnormal rates, or generating oversized payloads. Connection limits, message-size limits, quotas, rate controls, queue limits, and resource monitoring can reduce the effect of such behavior and prevent one component from degrading fleet-wide messaging.

Security configuration should be automated and version controlled where appropriate. Infrastructure-as-code and configuration-management practices can define TLS settings, authentication mechanisms, ACL policies, broker listeners, and network rules consistently across development, test, and production environments. Automated validation can detect accidental anonymous access, overly broad permissions, expired certificates, or insecure protocol settings before deployment.

Different message brokers implement these concepts through different mechanisms. Kafka commonly combines TLS, SASL, and resource ACLs; RabbitMQ provides TLS, users, permissions, and virtual-host isolation; NATS supports TLS, accounts, users, credentials, and subject permissions. The syntax differs, but the architectural objective remains consistent: establish trusted identities, encrypt communication, and authorize only required messaging operations.

For Physical AI systems, broker security should be integrated from robot identity to edge infrastructure and centralized fleet services. A practical security chain is device identity, encrypted TLS communication, authenticated broker access, least-privilege authorization, protected credentials, and continuous auditing. This layered approach allows messaging platforms to support distributed robotic intelligence while limiting the ability of compromised components to affect other robots or critical services.

메시지 브로커 보안(Message Broker Security)은 분산 메시징(Distributed Messaging)에 요구되는 신뢰성과 성능을 저해하지 않으면서 로봇, 엣지 시스템(Edge System), 백엔드 서비스(Backend Service), 클라우드 애플리케이션(Cloud Application) 사이의 통신을 보호해야 한다. 안전한 아키텍처는 일반적으로 전송 암호화(Transport Encryption), 클라이언트 인증(Client Authentication), 권한 부여(Authorization), 자격 증명 관리(Credential Management), 감사(Auditing), 네트워크 격리(Network Isolation)를 결합한다. SASL, TLS, ACL 메커니즘은 이러한 보안 모델의 서로 다른 영역을 담당하며 상호 보완적인 제어 수단으로 설계되어야 한다.

전송 계층 보안(Transport Layer Security, TLS)은 메시징 클라이언트(Messaging Client)와 브로커(Broker) 사이의 네트워크 통신을 보호한다. 암호화(Encryption)는 네트워크상의 관찰자가 전송 중인 텔레메트리(Telemetry), 명령(Command), 자격 증명(Credential), 운영 이벤트(Operational Event)를 쉽게 읽지 못하도록 한다. TLS는 무결성 보호(Integrity Protection)도 제공하여 보호된 트래픽의 변경을 탐지하는 데 도움을 준다. Wi-Fi, 이더넷(Ethernet), 셀룰러(Cellular), 외부 네트워크에서 동작하는 로봇 플릿(Robot Fleet)에서는 암호화된 브로커 연결을 기본적인 보안 요구사항으로 취급해야 한다.

TLS는 서버 신원(Server Identity)을 확립하기 위해 디지털 인증서(Digital Certificate)와 신뢰할 수 있는 인증 기관(Certificate Authority)에 의존한다. 클라이언트는 보호된 세션(Protected Session)을 설정하기 전에 브로커가 제시한 인증서를 검증하여 승인되지 않은 엔드포인트(Unauthorized Endpoint)에 연결될 위험을 줄인다. 인증서 검증(Certificate Validation)에는 적절한 신뢰 체인(Trust Chain)과 브로커 신원 확인이 포함되어야 한다. 편의를 위해 인증서 검증을 비활성화하면 TLS가 제공하려는 보호 기능 자체를 약화시킬 수 있다.

상호 TLS(Mutual TLS), 일반적으로 mTLS라고 부르는 방식은 인증서 기반 인증(Certificate Authentication)을 연결 양쪽으로 확장한다. 브로커가 클라이언트에 인증서를 제시하는 동시에 클라이언트도 연결하는 애플리케이션이나 장치를 식별하는 인증서를 제시한다. 이러한 방식은 공유 사용자 이름과 비밀번호에 전적으로 의존하지 않고 개별 로봇, 게이트웨이(Gateway), 서비스에 서로 다른 신원(Identity)을 부여할 수 있기 때문에 로봇 플릿에 유용하다.

수백 또는 수천 대의 로봇이 메시징 인프라(Messaging Infrastructure)에 참여하면 인증서 수명주기 관리(Certificate Lifecycle Management)가 중요해진다. 인증서는 안전한 발급(Issuance), 배포(Distribution), 저장(Storage), 갱신(Renewal), 교체(Rotation), 폐기(Revocation)가 필요하다. 장기간 유지되는 자격 증명은 로봇이나 엣지 컴퓨터가 침해되었을 때 노출 위험을 증가시킨다. 자동화된 인증서 관리(Automated Certificate Management)는 운영 오류를 줄이면서 침해되거나 폐기된 장치의 접근을 전체 플릿의 자격 증명을 변경하지 않고 차단할 수 있다.

단순 인증 및 보안 계층(Simple Authentication and Security Layer, SASL)은 클라이언트와 메시징 서비스 사이의 인증을 위한 프레임워크(Authentication Framework)를 제공한다. SASL은 하나의 범용 인증 알고리즘을 정의하는 것이 아니라 플랫폼 요구사항에 따라 선택할 수 있는 여러 인증 메커니즘(Authentication Mechanism)을 지원한다. 카프카(Kafka)와 같은 메시지 브로커는 SASL 인증과 TLS를 결합하여 자격 증명과 인증 교환(Authentication Exchange)이 암호화된 통신 채널을 통해 이루어지도록 할 수 있다.

사용자 이름과 비밀번호 기반 메커니즘(Username-and-Password Mechanism)은 비교적 쉽게 배포할 수 있지만 저장된 비밀정보(Stored Secret)를 신중하게 보호해야 한다. 자격 증명을 애플리케이션 소스 코드(Application Source Code), 컨테이너 이미지(Container Image), 구성 저장소(Configuration Repository)에 직접 포함해서는 안 된다. 가능한 경우 로봇 애플리케이션은 보호된 구성 또는 비밀정보 관리 시스템(Secret-Management System)에서 자격 증명을 가져와야 한다. 비밀번호 교체(Password Rotation), 계정 분리(Account Separation), 강력한 접근 정책은 자격 증명의 유출이나 재사용으로 인한 영향을 줄인다.

SASL 메커니즘은 정적인 비밀번호(Static Password)보다 강력한 신원 시스템(Identity System)도 지원할 수 있다. 브로커와 배포 환경에 따라 인증에는 챌린지-응답 자격 증명(Challenge-Response Credential), 토큰(Token), 커버로스(Kerberos), OAuth 기반 신원 흐름(OAuth-Oriented Identity Flow)을 이용하는 메커니즘이 사용될 수 있다. 적절한 메커니즘은 인프라, 장치 성능, 신원 제공자(Identity Provider), 운영 복잡성에 따라 달라진다. 인증 설계는 보다 광범위한 기업 신원 아키텍처(Enterprise Identity Architecture)와 일관성을 유지해야 한다.

인증(Authentication)은 클라이언트가 누구인지를 확인하지만 해당 클라이언트가 무엇을 할 수 있는지는 결정하지 않는다. 따라서 신원 검증 이후 권한 부여(Authorization)를 적용해야 한다. 접근 제어 목록(Access Control List, ACL)은 메시징 자원과 작업에 대한 권한을 정의한다. 인증된 로봇은 자신의 텔레메트리를 발행할 수 있지만 다른 로봇에 명령을 발행하거나 플릿 전체 관리 이벤트(Fleet-Wide Administrative Event)를 읽는 것은 금지하도록 구성할 수 있다.

ACL 설계는 최소 권한 원칙(Principle of Least Privilege)을 따라야 한다. 각 로봇, 엣지 서비스, 분석 프로세스(Analytics Process), 운영자 애플리케이션(Operator Application)은 자신의 기능을 수행하는 데 필요한 권한만 가져야 한다. 광범위한 와일드카드 권한(Wildcard Permission)은 초기 배포를 단순화할 수 있지만 자격 증명이 침해되었을 때 피해 범위를 크게 증가시킬 수 있다. 따라서 보안 정책은 텔레메트리 발행, 명령 소비, 관리 작업, 민감한 이벤트 스트림(Sensitive Event Stream)에 대한 접근을 구분해야 한다.

자원 명명(Resource Naming)은 권한 부여와 밀접하게 관련된다. 카프카 토픽(Kafka Topic), 래빗엠큐 익스체인지 및 큐(RabbitMQ Exchange and Queue), 내츠 서브젝트(NATS Subject)는 보안 정책을 명확하게 표현할 수 있도록 일관된 명명 구조(Naming Structure)를 사용해야 한다. 이름에는 사이트, 로봇 식별자(Robot Identifier), 서브시스템(Subsystem), 애플리케이션, 데이터 분류(Data Classification)를 표현할 수 있다. 예측 가능한 계층 구조는 개별 로봇을 자신의 통신 네임스페이스(Communication Namespace)로 제한하면서 승인된 플릿 서비스에는 여러 로봇에 대한 접근을 허용하기 쉽게 만든다.

명령 채널(Command Channel)은 승인되지 않은 메시지 발행이 물리적인 로봇 동작에 영향을 미칠 수 있기 때문에 특히 엄격한 권한 부여가 필요하다. 일반적인 텔레메트리 생산자는 모션 명령(Motion Command), 임무 지시(Mission Instruction), 충전 요청(Charging Request), 안전 관련 제어 메시지(Safety-Related Control Message)를 발행할 권한을 가져서는 안 된다. 명령 발행자는 전용 신원(Dedicated Identity)과 좁게 정의된 권한을 사용해야 한다. 중요한 명령은 실행 전에 독립적인 애플리케이션 계층(Application Layer)의 추가 검증을 거칠 수도 있다.

브로커 관리 권한(Broker Administrative Privilege)은 일반적인 메시징 권한과 분리해야 한다. 운영 데이터를 발행하거나 소비하는 애플리케이션은 일반적으로 사용자를 생성하거나 보안 정책을 변경하고, 토픽을 삭제하거나 큐를 변경하며, 브로커 구성을 수정할 권한이 필요하지 않다. 전용 관리자 신원(Dedicated Administrative Identity)을 사용하면 로봇 애플리케이션의 침해가 전체 메시징 인프라의 침해로 확대될 가능성을 줄일 수 있다.

네트워크 분할(Network Segmentation)은 메시지 브로커 주변에 또 하나의 보안 경계(Security Boundary)를 제공한다. 로봇 네트워크, 엣지 서버, 관리 시스템, 외부 클라우드 연결은 방화벽(Firewall), 가상 네트워크(Virtual Network) 또는 기타 네트워크 제어를 통해 분리할 수 있다. 필요한 브로커 포트와 승인된 통신 경로만 외부에 노출해야 한다. 그러나 네트워크 위치 자체만으로 클라이언트가 신뢰할 수 있다고 판단해서는 안 되므로 인증과 ACL은 여전히 필요하다.

보안은 브로커 간 통신(Broker-to-Broker Communication)도 보호해야 한다. 카프카 클러스터(Kafka Cluster), 래빗엠큐 클러스터(RabbitMQ Cluster), 내츠 클러스터(NATS Cluster), 게이트웨이, 분산 엣지 배포(Distributed Edge Deployment)는 여러 노드 사이에서 메시지와 메타데이터(Metadata)를 교환할 수 있다. 내부 트래픽에도 민감한 운영 정보와 제어 데이터가 포함될 수 있다. 암호화와 인증된 노드 신원(Authenticated Node Identity)은 승인되지 않은 시스템이 메시징 인프라에 참여하거나 노드 간 통신을 가로채는 것을 방지하는 데 도움을 준다.

로봇의 자격 증명 저장(Credential Storage)은 물리적 장치가 엄격하게 통제되는 데이터센터 외부에서 동작할 수 있기 때문에 특별한 주의가 필요하다. 개인 키(Private Key), 비밀번호, 토큰은 운영체제 권한(Operating-System Permission), 보안 저장소(Secure Storage), 가능한 경우 하드웨어 기반 보안(Hardware-Backed Security), 전용 비밀정보 관리 메커니즘을 이용해 보호해야 한다. 진단 로그(Diagnostic Log)에는 자격 증명, 토큰, 인증서 개인 키 또는 전체 인증 정보가 노출되지 않도록 해야 한다.

자격 증명 교체(Credential Rotation)는 인증서가 만료되거나 자격 증명이 침해된 이후 추가하는 것이 아니라 배포 전에 설계해야 한다. 시스템은 서비스 연속성(Service Continuity)을 유지하면서 새로운 인증서, 비밀번호, 토큰을 도입할 수 있는 절차가 필요하다. 유효 기간 중첩(Overlapping Validity Period)을 사용하면 통제된 전환을 지원할 수 있다. 도난당한 로봇, 침해된 게이트웨이 또는 유출된 자격 증명의 접근을 즉시 차단해야 할 수 있으므로 긴급 폐기 절차(Emergency Revocation Procedure) 역시 중요하다.

감사 로깅(Audit Logging)은 보안과 관련된 브로커 활동의 증거를 제공한다. 유용한 기록에는 인증 성공 및 실패, 권한 거부(Authorization Denial), 관리 변경(Administrative Change), 비정상적인 연결 시도, 민감한 메시징 자원에 대한 접근이 포함된다. 감사 이벤트에는 조사에 필요한 충분한 신원 및 시간 정보(Identity and Timing Information)가 포함되어야 하지만 비밀정보가 불필요하게 노출되어서는 안 된다. 중앙 집중식 분석(Centralized Analysis)을 통해 브로커 활동을 로봇 및 인프라 이벤트와 연계할 수 있다.

모니터링(Monitoring)은 기술적으로는 인증되었지만 운영 관점에서는 비정상적인 동작을 탐지할 수 있다. 예를 들어 로봇이 갑자기 예상하지 못한 토픽에 접근하거나 서비스가 비정상적으로 높은 메시지 발생률을 보이고, 권한 거부가 반복되거나 예상하지 못한 네트워크 위치에서 연결이 발생할 수 있다. 따라서 보안 모니터링(Security Monitoring)은 성공적인 인증만 확인하는 대신 브로커 메트릭(Broker Metric), 감사 로그, 네트워크 관찰(Network Observation), 애플리케이션 컨텍스트(Application Context)를 결합해야 한다.

서비스 거부 공격 저항성(Denial-of-Service Resistance) 역시 브로커 보안의 일부이다. 침해되거나 오작동하는 클라이언트는 과도한 연결을 생성하고 비정상적인 속도로 메시지를 발행하거나 지나치게 큰 페이로드(Payload)를 생성하여 브로커 자원을 소모할 수 있다. 연결 제한(Connection Limit), 메시지 크기 제한(Message-Size Limit), 할당량(Quota), 전송률 제어(Rate Control), 큐 제한(Queue Limit), 자원 모니터링을 통해 이러한 동작의 영향을 줄이고 하나의 구성 요소가 전체 플릿 메시징 성능을 저하시키는 것을 방지할 수 있다.

보안 구성(Security Configuration)은 가능한 경우 자동화되고 버전 관리(Version Control)되어야 한다. 코드형 인프라(Infrastructure as Code)와 구성 관리(Configuration Management)를 이용하면 TLS 설정, 인증 메커니즘, ACL 정책, 브로커 리스너(Broker Listener), 네트워크 규칙을 개발, 테스트, 운영 환경에 일관되게 정의할 수 있다. 자동 검증(Automated Validation)은 배포 전에 실수로 허용된 익명 접근(Anonymous Access), 지나치게 넓은 권한, 만료된 인증서, 안전하지 않은 프로토콜 설정을 탐지할 수 있다.

서로 다른 메시지 브로커는 이러한 개념을 서로 다른 메커니즘으로 구현한다. 카프카는 일반적으로 TLS, SASL, 자원 ACL(Resource ACL)을 결합하며, 래빗엠큐는 TLS, 사용자, 권한, 가상 호스트 격리(Virtual-Host Isolation)를 제공하고, 내츠는 TLS, 계정, 사용자, 자격 증명, 서브젝트 권한(Subject Permission)을 지원한다. 구체적인 문법과 구성 방식은 다르지만 신뢰할 수 있는 신원을 확립하고 통신을 암호화하며 필요한 메시징 작업만 승인한다는 아키텍처 목표는 동일하다.

피지컬 AI(Physical AI) 시스템에서 브로커 보안은 로봇 신원(Robot Identity)부터 엣지 인프라와 중앙 집중식 플릿 서비스(Centralized Fleet Service)에 이르기까지 통합되어야 한다. 실용적인 보안 체계는 장치 신원(Device Identity), 암호화된 TLS 통신, 인증된 브로커 접근(Authenticated Broker Access), 최소 권한 기반 권한 부여(Least-Privilege Authorization), 보호된 자격 증명(Protected Credential), 지속적인 감사(Continuous Auditing)로 연결할 수 있다. 이러한 계층적 접근(Layered Approach)은 메시징 플랫폼이 분산 로봇 지능(Distributed Robotic Intelligence)을 지원하면서 침해된 구성 요소가 다른 로봇이나 핵심 서비스에 영향을 미칠 수 있는 범위를 제한한다.

##  

## 05.09 Robot Event Streaming Architecture: Broker Selection

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

Robot event streaming architecture provides a continuous communication backbone for transferring operational events from robots and edge systems to fleet services, analytics platforms, storage systems, and AI pipelines. Unlike simple request-response communication, event streaming treats robot activity as an ongoing sequence of events. Telemetry, mission transitions, diagnostics, localization updates, alerts, and AI inference results can therefore be processed independently by multiple downstream applications.

A practical architecture begins by classifying robot data according to communication behavior rather than sending every message through one mechanism. High-frequency transient status, durable operational events, commands, large sensor data, and enterprise workflow messages have different requirements. Event rate, payload size, ordering, retention, replay, latency, reliability, and consumer behavior should therefore be identified before selecting a message broker.

Robot and edge systems form the producer layer of the event architecture. Individual robots can generate navigation status, battery information, sensor health, mission events, safety alerts, and perception results, while edge gateways aggregate or transform selected information. Local preprocessing can reduce unnecessary traffic by filtering repetitive events, calculating summaries, compressing data, or publishing references to externally stored large sensor objects.

The broker layer separates event producers from downstream consumers. Producers publish according to topics, subjects, routing keys, or other broker-specific addressing models without needing to know which applications will eventually process the events. Monitoring, maintenance, digital twins, AI pipelines, and data platforms can then subscribe independently, allowing new consumers to be introduced without redesigning robot-side software.

Durability requirements strongly influence broker selection. Some events are useful only while they represent the current robot state, whereas mission transitions, faults, safety events, and maintenance records may need persistent storage and later replay. A streaming architecture should explicitly determine which events may be discarded after delivery, which require temporary buffering, and which must remain available for historical processing or recovery.

Ordering is another important architectural requirement. Many robot events are meaningful only when their sequence is preserved, but global ordering across an entire fleet is rarely necessary. Partitioning events by robot identifier, mission identifier, or another stable key can preserve relevant local ordering while allowing parallel processing. The selected broker should provide an ordering model that matches the actual operational dependency of the event stream.

Kafka is well suited to durable, high-throughput event streams that require retention, replay, partition-based scalability, and multiple independent consumer groups. A fleet can retain telemetry, mission events, diagnostics, and AI metadata as an operational history while monitoring, analytics, maintenance, and machine-learning services consume the same streams independently. Kafka is therefore particularly useful as a centralized or regional event backbone.

Kafka becomes especially valuable when robot data must be reprocessed after the original event occurred. Historical streams can support algorithm updates, fleet analytics, incident investigation, digital-twin reconstruction, or AI dataset generation. Its partitioned log architecture supports large event volumes, but cluster operation, storage planning, partition management, and consumer-lag monitoring introduce more infrastructure complexity than lightweight messaging systems.

NATS is attractive when lightweight deployment, low latency, simple subject-based communication, and efficient edge operation are major requirements. Core NATS supports transient publish-subscribe, request-reply, and queue-group communication with a small operational footprint. It can therefore connect robot services and edge applications where immediate communication is more important than maintaining a long historical event log.

JetStream extends NATS when persistence, acknowledgement, replay, and durable consumption are required. This allows an architecture to use Core NATS for transient service communication while selected operational events are stored by JetStream. Leaf-node and distributed deployment patterns can also connect remote robot sites to regional or centralized infrastructure, making NATS particularly useful for edge-oriented distributed robot systems.

RabbitMQ is strong when event delivery depends on flexible routing, work queues, acknowledgements, and enterprise-style asynchronous workflows. Direct, topic, fanout, and headers exchanges allow messages to be routed according to application-specific rules. Robot commands, maintenance jobs, integration requests, report-generation tasks, and backend processing workloads can therefore be distributed to appropriate services without requiring a persistent event-log model.

RabbitMQ is often more natural for task-oriented messaging than for retaining very large historical event streams. Competing consumers can distribute work from queues, publisher confirms improve publishing reliability, and dead-letter mechanisms can isolate failed messages. These capabilities are useful when robot events initiate discrete backend actions rather than becoming a long-lived replayable record consumed repeatedly by independent analytical applications.

MQTT brokers are appropriate when constrained devices, intermittent networks, lightweight telemetry, and device-to-cloud communication dominate the architecture. Topic-based publish-subscribe, quality-of-service levels, retained messages, and persistent-session features can support distributed IoT-style robot connectivity. MQTT is frequently useful at device or gateway boundaries, although large-scale historical stream processing usually requires additional backend infrastructure.

Broker selection should not be reduced to a single throughput benchmark. Latency distribution, message rate, payload size, connection count, persistence cost, recovery behavior, ordering guarantees, consumer scalability, operational complexity, security, observability, and deployment footprint all affect suitability. The same broker can perform very differently depending on acknowledgement policy, replication, storage configuration, network conditions, and message size.

Large camera images, LiDAR point clouds, video, maps, and other sensor payloads should generally be evaluated separately from ordinary broker messages. Repeatedly transporting very large binary objects through a broker can consume network, memory, and storage resources needed for operational events. A common design stores large objects in object or file storage and publishes metadata, timestamps, robot identifiers, checksums, and object references through the event stream.

Edge and centralized brokers can be combined instead of forcing every robot to communicate directly with one remote cluster. A local broker can support low-latency robot and site communication while selected events are forwarded to regional or cloud infrastructure. This architecture can continue local operation during temporary WAN disruption and reduce external bandwidth by filtering, aggregating, or prioritizing events before upstream transmission.

A multi-broker architecture may be appropriate when communication requirements are fundamentally different. NATS can provide lightweight edge service messaging, Kafka can maintain durable fleet-wide event history, RabbitMQ can coordinate enterprise workflows, and MQTT can connect constrained devices. The advantage is functional specialization, but additional brokers increase integration, security, monitoring, schema-management, and operational complexity.

Event schemas should remain consistent regardless of broker technology. Robot identifiers, timestamps, event types, schema versions, correlation identifiers, mission context, and payload metadata should follow defined conventions. Schema evolution must allow producers and consumers to change independently without breaking the event pipeline. Protobuf, Avro, JSON, or other serialization formats can be selected according to interoperability and performance requirements.

Backpressure must be considered whenever consumers can process events more slowly than producers generate them. Durable brokers may accumulate backlog, while transient systems may drop or delay messages depending on their design and configuration. Monitoring queue depth, consumer lag, acknowledgement delay, storage growth, and message rate helps identify overload before it affects fleet services or consumes excessive infrastructure resources.

Reliability requirements should be defined per event class. A periodic battery update may tolerate loss because another update will soon replace it, while a safety alert, mission completion event, or maintenance fault may require durable delivery and acknowledgement. Applying maximum durability to every message increases cost and latency unnecessarily, whereas treating all messages as transient can cause important operational information to disappear during failures.

Security requirements also affect broker placement and selection. Robot identity, TLS encryption, authentication, least-privilege authorization, credential rotation, and audit logging should extend across edge and centralized messaging layers. Command channels require stricter controls than ordinary telemetry. Broker technologies should therefore be evaluated not only for performance but also for their ability to express and operate the required security boundaries.

Observability should provide an end-to-end view from event generation to final consumption. Operators need to understand broker health, connection status, event rates, partition or queue backlog, consumer lag, failed deliveries, redelivery, storage utilization, and processing latency. Correlation identifiers and consistent timestamps allow one robot event to be traced across gateways, brokers, processing services, and downstream applications.

Broker evaluation should use representative robot workloads rather than generic synthetic tests alone. Test scenarios should reproduce expected event rates, burst behavior, message sizes, consumer counts, retention policies, replication, network delays, and failure conditions. Measurements should include normal operation, broker restart, consumer failure, network interruption, recovery, and backlog processing so architectural decisions reflect actual fleet behavior.

For Physical AI systems, broker selection is therefore an architectural mapping problem rather than a search for one universally superior product. Transient edge communication, durable fleet event history, asynchronous business workflows, and constrained device connectivity may require different mechanisms. A well-designed robot event architecture assigns each event class to an appropriate messaging path while maintaining common schemas, security, observability, and governance across the complete fleet.

로봇 이벤트 스트리밍 아키텍처(Robot Event Streaming Architecture)는 로봇과 엣지 시스템(Edge System)에서 발생하는 운영 이벤트(Operational Event)를 플릿 서비스(Fleet Service), 분석 플랫폼(Analytics Platform), 저장 시스템(Storage System), AI 파이프라인(AI Pipeline)으로 지속적으로 전달하는 통신 백본(Communication Backbone)을 제공한다. 단순한 요청-응답 통신(Request-Response Communication)과 달리 이벤트 스트리밍(Event Streaming)은 로봇의 활동을 연속적인 이벤트 시퀀스(Event Sequence)로 취급한다. 따라서 텔레메트리(Telemetry), 임무 전환(Mission Transition), 진단(Diagnostics), 위치추정 업데이트(Localization Update), 경보(Alert), AI 추론 결과(AI Inference Result)를 여러 후속 애플리케이션(Downstream Application)이 독립적으로 처리할 수 있다.

실용적인 아키텍처는 모든 메시지를 하나의 메커니즘으로 전송하는 대신 로봇 데이터를 통신 동작(Communication Behavior)에 따라 분류하는 것에서 시작한다. 고주파 일시적 상태(High-Frequency Transient Status), 내구성 운영 이벤트(Durable Operational Event), 명령(Command), 대용량 센서 데이터(Large Sensor Data), 기업 워크플로 메시지(Enterprise Workflow Message)는 서로 다른 요구사항을 가진다. 따라서 메시지 브로커(Message Broker)를 선택하기 전에 이벤트 발생률(Event Rate), 페이로드 크기(Payload Size), 순서 보장(Ordering), 보존(Retention), 재생(Replay), 지연시간(Latency), 신뢰성(Reliability), 소비자 동작(Consumer Behavior)을 정의해야 한다.

로봇과 엣지 시스템은 이벤트 아키텍처(Event Architecture)의 생산자 계층(Producer Layer)을 구성한다. 개별 로봇은 내비게이션 상태(Navigation Status), 배터리 정보, 센서 상태(Sensor Health), 임무 이벤트(Mission Event), 안전 경보(Safety Alert), 인식 결과(Perception Result)를 생성할 수 있으며, 엣지 게이트웨이(Edge Gateway)는 선택된 정보를 집계하거나 변환할 수 있다. 로컬 전처리(Local Preprocessing)를 통해 반복 이벤트 필터링, 요약 계산, 데이터 압축 또는 외부 저장소에 저장된 대용량 센서 객체(Large Sensor Object)의 참조 발행 등을 수행하여 불필요한 트래픽을 줄일 수 있다.

브로커 계층(Broker Layer)은 이벤트 생산자(Event Producer)와 후속 소비자(Downstream Consumer)를 분리한다. 생산자는 최종적으로 어떤 애플리케이션이 이벤트를 처리할지 알 필요 없이 토픽(Topic), 서브젝트(Subject), 라우팅 키(Routing Key) 또는 기타 브로커별 주소 지정 모델(Broker-Specific Addressing Model)에 따라 이벤트를 발행한다. 이후 모니터링, 유지보수, 디지털 트윈(Digital Twin), AI 파이프라인, 데이터 플랫폼(Data Platform)이 독립적으로 구독할 수 있으므로 로봇 측 소프트웨어를 재설계하지 않고 새로운 소비자를 추가할 수 있다.

내구성 요구사항(Durability Requirement)은 브로커 선택에 큰 영향을 미친다. 일부 이벤트는 현재 로봇 상태를 나타내는 동안에만 의미가 있지만 임무 전환, 장애(Fault), 안전 이벤트(Safety Event), 유지보수 기록(Maintenance Record)은 영속적인 저장과 이후 재생이 필요할 수 있다. 스트리밍 아키텍처는 전달 이후 폐기 가능한 이벤트, 임시 버퍼링(Temporary Buffering)이 필요한 이벤트, 과거 처리 또는 복구를 위해 계속 사용할 수 있어야 하는 이벤트를 명확하게 구분해야 한다.

순서 보장(Ordering) 역시 중요한 아키텍처 요구사항이다. 많은 로봇 이벤트는 발생 순서가 유지되어야 의미가 있지만 전체 플릿에 대한 전역 순서(Global Ordering)는 일반적으로 필요하지 않다. 로봇 식별자(Robot Identifier), 임무 식별자(Mission Identifier) 또는 다른 안정적인 키(Stable Key)를 기준으로 이벤트를 파티셔닝(Partitioning)하면 관련된 로컬 순서(Local Ordering)를 유지하면서 병렬 처리(Parallel Processing)를 수행할 수 있다. 선택된 브로커는 실제 이벤트 스트림의 운영 의존성(Operational Dependency)에 적합한 순서 보장 모델을 제공해야 한다.

카프카(Kafka)는 보존, 재생, 파티션 기반 확장성(Partition-Based Scalability), 여러 독립적인 소비자 그룹(Consumer Group)이 필요한 내구성 고처리량 이벤트 스트림(Durable High-Throughput Event Stream)에 적합하다. 플릿은 텔레메트리, 임무 이벤트, 진단, AI 메타데이터(AI Metadata)를 운영 이력(Operational History)으로 보존하면서 모니터링, 분석, 유지보수, 머신러닝 서비스(Machine-Learning Service)가 동일한 스트림을 독립적으로 소비하도록 할 수 있다. 따라서 카프카는 중앙 집중형 또는 지역 이벤트 백본(Centralized or Regional Event Backbone)에 특히 유용하다.

카프카는 원래 이벤트가 발생한 이후 로봇 데이터를 다시 처리해야 할 때 특히 유용하다. 과거 스트림(Historical Stream)은 알고리즘 업데이트(Algorithm Update), 플릿 분석(Fleet Analytics), 사고 조사(Incident Investigation), 디지털 트윈 재구성(Digital-Twin Reconstruction), AI 데이터셋 생성(AI Dataset Generation)을 지원할 수 있다. 파티션 로그 아키텍처(Partitioned Log Architecture)는 대규모 이벤트를 지원하지만 클러스터 운영(Cluster Operation), 저장소 계획(Storage Planning), 파티션 관리(Partition Management), 소비자 지연 모니터링(Consumer-Lag Monitoring)은 경량 메시징 시스템보다 높은 인프라 복잡성을 발생시킨다.

내츠(NATS)는 경량 배포(Lightweight Deployment), 낮은 지연시간(Low Latency), 단순한 서브젝트 기반 통신(Subject-Based Communication), 효율적인 엣지 운영(Edge Operation)이 주요 요구사항인 경우 유용하다. 코어 내츠(Core NATS)는 작은 운영 자원 사용량(Operational Footprint)으로 일시적 발행-구독(Transient Publish-Subscribe), 요청-응답(Request-Reply), 큐 그룹 통신(Queue-Group Communication)을 지원한다. 따라서 장기간의 이벤트 로그를 유지하는 것보다 즉각적인 통신이 중요한 로봇 서비스와 엣지 애플리케이션을 연결하는 데 적합하다.

제트스트림(JetStream)은 영속성(Persistence), 승인(Acknowledgement), 재생, 내구성 소비(Durable Consumption)가 필요한 경우 내츠의 기능을 확장한다. 이를 통해 코어 내츠는 일시적인 서비스 통신에 사용하고 선택된 운영 이벤트는 제트스트림에 저장하는 아키텍처를 구성할 수 있다. 리프 노드(Leaf Node)와 분산 배포 패턴(Distributed Deployment Pattern)은 원격 로봇 사이트를 지역 또는 중앙 인프라와 연결할 수 있으므로 내츠는 엣지 중심의 분산 로봇 시스템(Distributed Robot System)에 특히 유용하다.

래빗엠큐(RabbitMQ)는 이벤트 전달에 유연한 라우팅(Flexible Routing), 작업 큐(Work Queue), 승인, 기업형 비동기 워크플로(Enterprise-Style Asynchronous Workflow)가 필요한 경우 강점을 가진다. 다이렉트(Direct), 토픽(Topic), 팬아웃(Fanout), 헤더(Headers) 익스체인지(Exchange)를 통해 애플리케이션별 규칙에 따라 메시지를 라우팅할 수 있다. 따라서 로봇 명령, 유지보수 작업, 통합 요청(Integration Request), 보고서 생성 작업, 백엔드 처리 워크로드(Backend Processing Workload)를 적절한 서비스에 분배할 수 있다.

래빗엠큐는 매우 큰 과거 이벤트 스트림을 장기간 보존하는 것보다 작업 중심 메시징(Task-Oriented Messaging)에 더욱 자연스럽게 적용되는 경우가 많다. 경쟁 소비자(Competing Consumer)는 큐의 작업을 분산하고, 발행자 확인(Publisher Confirm)은 발행 신뢰성을 향상시키며, 데드 레터 메커니즘(Dead-Letter Mechanism)은 실패한 메시지를 격리할 수 있다. 이러한 기능은 로봇 이벤트가 여러 독립적인 분석 애플리케이션에서 반복적으로 소비되는 장기 재생 기록이 되기보다 개별 백엔드 작업을 시작하는 경우 유용하다.

MQTT 브로커(MQTT Broker)는 제약된 장치(Constrained Device), 불안정한 네트워크(Intermittent Network), 경량 텔레메트리(Lightweight Telemetry), 장치-클라우드 통신(Device-to-Cloud Communication)이 아키텍처의 중심인 경우 적합하다. 토픽 기반 발행-구독(Topic-Based Publish-Subscribe), 서비스 품질 수준(Quality-of-Service Level), 보존 메시지(Retained Message), 영속 세션(Persistent Session) 기능을 통해 분산 IoT 방식의 로봇 연결을 지원할 수 있다. MQTT는 장치 또는 게이트웨이 경계에서 유용하지만 대규모 과거 스트림 처리에는 일반적으로 추가적인 백엔드 인프라가 필요하다.

브로커 선택(Broker Selection)은 하나의 처리량 벤치마크(Throughput Benchmark)만으로 결정해서는 안 된다. 지연시간 분포(Latency Distribution), 메시지 발생률, 페이로드 크기, 연결 수(Connection Count), 영속성 비용(Persistence Cost), 복구 동작(Recovery Behavior), 순서 보장, 소비자 확장성(Consumer Scalability), 운영 복잡성, 보안, 관찰 가능성(Observability), 배포 자원 사용량(Deployment Footprint)이 모두 적합성에 영향을 준다. 동일한 브로커라도 승인 정책, 복제(Replication), 저장소 구성, 네트워크 조건, 메시지 크기에 따라 성능이 크게 달라질 수 있다.

대용량 카메라 이미지, 라이다 포인트 클라우드(LiDAR Point Cloud), 비디오, 지도(Map), 기타 센서 페이로드는 일반적인 브로커 메시지와 별도로 평가하는 것이 적절하다. 매우 큰 바이너리 객체(Binary Object)를 브로커를 통해 반복적으로 전송하면 운영 이벤트에 필요한 네트워크, 메모리, 저장소 자원을 소비할 수 있다. 일반적인 설계에서는 대용량 객체를 객체 저장소(Object Storage) 또는 파일 저장소(File Storage)에 저장하고 메타데이터, 타임스탬프(Timestamp), 로봇 식별자, 체크섬(Checksum), 객체 참조(Object Reference)를 이벤트 스트림으로 발행한다.

모든 로봇이 하나의 원격 클러스터(Remote Cluster)와 직접 통신하도록 강제하는 대신 엣지 브로커(Edge Broker)와 중앙 집중식 브로커(Centralized Broker)를 결합할 수 있다. 로컬 브로커는 낮은 지연시간의 로봇 및 사이트 통신을 지원하고 선택된 이벤트만 지역 또는 클라우드 인프라로 전달할 수 있다. 이러한 아키텍처는 일시적인 광역 네트워크(WAN) 장애 중에도 로컬 운영을 지속할 수 있으며 이벤트를 상위 시스템으로 전송하기 전에 필터링, 집계(Aggregation), 우선순위화를 수행하여 외부 대역폭 사용량을 줄일 수 있다.

통신 요구사항이 근본적으로 서로 다른 경우 멀티 브로커 아키텍처(Multi-Broker Architecture)를 적용할 수 있다. 내츠는 경량 엣지 서비스 메시징을 제공하고, 카프카는 내구성 있는 플릿 전체 이벤트 이력(Fleet-Wide Event History)을 유지하며, 래빗엠큐는 기업 워크플로를 조정하고, MQTT는 제약된 장치를 연결할 수 있다. 기능별 전문화(Functional Specialization)가 장점이지만 브로커 수가 증가하면 통합, 보안, 모니터링, 스키마 관리(Schema Management), 운영 복잡성도 증가한다.

이벤트 스키마(Event Schema)는 브로커 기술과 관계없이 일관성을 유지해야 한다. 로봇 식별자, 타임스탬프, 이벤트 유형(Event Type), 스키마 버전(Schema Version), 상관관계 식별자(Correlation Identifier), 임무 컨텍스트(Mission Context), 페이로드 메타데이터(Payload Metadata)는 정의된 규칙을 따라야 한다. 스키마 진화(Schema Evolution)는 이벤트 파이프라인을 중단하지 않으면서 생산자와 소비자가 독립적으로 변경될 수 있도록 지원해야 한다. 프로토버프(Protobuf), 아브로(Avro), JSON 또는 기타 직렬화 형식(Serialization Format)은 상호운용성(Interoperability)과 성능 요구사항에 따라 선택할 수 있다.

소비자가 생산자의 이벤트 생성 속도보다 느리게 처리할 가능성이 있다면 백프레셔(Backpressure)를 반드시 고려해야 한다. 내구성 브로커는 백로그(Backlog)를 누적할 수 있으며 일시적 시스템은 설계와 구성에 따라 메시지를 삭제하거나 지연시킬 수 있다. 큐 깊이(Queue Depth), 소비자 지연(Consumer Lag), 승인 지연(Acknowledgement Delay), 저장소 증가량(Storage Growth), 메시지 발생률을 모니터링하면 과부하가 플릿 서비스에 영향을 주거나 과도한 인프라 자원을 소비하기 전에 문제를 식별할 수 있다.

신뢰성 요구사항(Reliability Requirement)은 이벤트 유형별로 정의해야 한다. 주기적인 배터리 업데이트는 곧 새로운 업데이트가 발생하기 때문에 일부 손실을 허용할 수 있지만 안전 경보, 임무 완료 이벤트(Mission Completion Event), 유지보수 장애(Maintenance Fault)는 내구성 있는 전달과 승인이 필요할 수 있다. 모든 메시지에 최대 수준의 내구성을 적용하면 불필요하게 비용과 지연시간이 증가하고, 반대로 모든 메시지를 일시적으로 취급하면 장애 상황에서 중요한 운영 정보가 사라질 수 있다.

보안 요구사항(Security Requirement) 역시 브로커 배치와 선택에 영향을 준다. 로봇 신원(Robot Identity), TLS 암호화(TLS Encryption), 인증(Authentication), 최소 권한 기반 권한 부여(Least-Privilege Authorization), 자격 증명 교체(Credential Rotation), 감사 로깅(Audit Logging)은 엣지와 중앙 메시징 계층 전체에 적용되어야 한다. 명령 채널(Command Channel)은 일반 텔레메트리보다 엄격한 제어가 필요하다. 따라서 브로커 기술은 성능뿐 아니라 필요한 보안 경계(Security Boundary)를 표현하고 운영할 수 있는 능력도 함께 평가해야 한다.

관찰 가능성은 이벤트 생성부터 최종 소비까지 종단 간 관점(End-to-End View)을 제공해야 한다. 운영자는 브로커 상태(Broker Health), 연결 상태(Connection Status), 이벤트 발생률, 파티션 또는 큐 백로그(Partition or Queue Backlog), 소비자 지연, 전달 실패(Failed Delivery), 재전달(Redelivery), 저장소 사용량, 처리 지연시간(Processing Latency)을 파악할 수 있어야 한다. 상관관계 식별자와 일관된 타임스탬프를 사용하면 하나의 로봇 이벤트가 게이트웨이, 브로커, 처리 서비스, 후속 애플리케이션을 통과하는 전체 경로를 추적할 수 있다.

브로커 평가(Broker Evaluation)는 일반적인 합성 테스트(Synthetic Test)에만 의존하지 않고 실제 로봇을 대표하는 워크로드(Representative Robot Workload)를 사용해야 한다. 테스트 시나리오는 예상 이벤트 발생률, 버스트 동작(Burst Behavior), 메시지 크기, 소비자 수, 보존 정책, 복제, 네트워크 지연, 장애 조건을 재현해야 한다. 정상 운영뿐 아니라 브로커 재시작, 소비자 장애, 네트워크 중단, 복구, 백로그 처리 상황을 측정하여 실제 플릿 동작을 반영하는 아키텍처 결정을 내려야 한다.

피지컬 AI(Physical AI) 시스템에서 브로커 선택은 하나의 보편적으로 우수한 제품을 찾는 문제가 아니라 아키텍처 매핑 문제(Architectural Mapping Problem)이다. 일시적인 엣지 통신(Transient Edge Communication), 내구성 있는 플릿 이벤트 이력, 비동기 비즈니스 워크플로(Asynchronous Business Workflow), 제약된 장치 연결(Constrained Device Connectivity)은 서로 다른 메커니즘을 요구할 수 있다. 잘 설계된 로봇 이벤트 아키텍처는 각각의 이벤트 유형을 적절한 메시징 경로(Messaging Path)에 할당하면서 전체 플릿에 걸쳐 공통 스키마(Common Schema), 보안, 관찰 가능성, 거버넌스(Governance)를 일관되게 유지한다.

##  

## 05.10 Message Broker Performance Benchmark Methodology

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Message broker performance benchmarking should reproduce the communication conditions expected in the target robot system rather than measure only the maximum message rate of an isolated broker. Robotics workloads combine periodic telemetry, burst events, commands, diagnostics, and asynchronous processing. A useful methodology therefore evaluates throughput, latency, reliability, resource usage, scalability, and recovery under representative operating conditions.

The benchmark environment must be documented before measurements begin. Broker version, operating system, CPU, memory, storage, network interface, virtualization, container configuration, and client hardware can significantly affect results. Broker configuration should also record replication, persistence, acknowledgement, compression, batching, queue or partition settings, and security options so that measurements can be reproduced and compared fairly.

Workload modeling begins with the actual message classes generated by robots. A test can include frequent small telemetry messages, medium-sized mission and diagnostic events, command traffic, and occasional references to large sensor objects. Message-size distributions should represent real operation instead of relying on one fixed payload size, because serialization, network transfer, storage, and memory behavior change substantially as payload size increases.

Message rate should be tested across multiple operating regions. A baseline load represents normal fleet activity, while progressively higher rates reveal the point at which latency, queue depth, or consumer lag begins to increase. Burst tests are also important because many robots can simultaneously generate events after a mission transition, network recovery, emergency condition, software restart, or synchronized operational schedule.

Throughput measures how much useful messaging work the system completes during a defined interval. It can be expressed as messages per second, bytes per second, or both. Producer throughput alone is insufficient because a broker may accept messages faster than consumers can process them. End-to-end throughput should therefore confirm that messages pass through producers, brokers, and consumers without uncontrolled backlog accumulation.

Latency should be measured as a distribution rather than represented only by an average value. Median latency describes typical behavior, while higher percentiles such as p95, p99, and p99.9 expose slow events that may affect operational responsiveness. Robot systems can be sensitive to tail latency because an occasional delayed command, alert, or mission event may matter more than a small improvement in average message delivery time.

End-to-end latency should include the complete path from producer publication to consumer reception or processing completion. Broker-only latency can still be measured for diagnosis, but it does not represent application experience. Timestamps should identify publication, broker interaction, reception, and processing stages. When measurements span multiple machines, synchronized clocks are necessary to prevent clock error from being interpreted as communication delay.

Benchmark clients can themselves become performance bottlenecks. Producer and consumer machines must have sufficient CPU, memory, network bandwidth, and concurrency to generate and process the intended load. Client-side serialization, logging, disk access, thread scheduling, or application code can limit measured throughput before the broker reaches capacity. Client resource metrics should therefore be collected together with broker metrics.

Persistence changes benchmark behavior significantly. In-memory or transient messaging should not be compared directly with durable storage without identifying the difference. Kafka replication and acknowledgement policies, JetStream file storage, RabbitMQ durable queues and persistent messages, or MQTT persistence mechanisms introduce different storage and reliability costs. Benchmark reports should associate every performance result with the corresponding durability configuration.

Acknowledgement settings create another important tradeoff between latency and delivery assurance. Waiting for stronger confirmation can increase message delivery time but reduce uncertainty during failures. Tests should evaluate the acknowledgement mode intended for production rather than selecting the fastest setting solely to improve benchmark numbers. Reliability and performance should be measured as linked properties instead of independent characteristics.

Scalability testing determines how performance changes as system size increases. Tests can vary the number of producers, consumers, robot identities, topics, subjects, queues, partitions, connections, or broker nodes. A scalable architecture should increase capacity predictably without causing disproportionate latency or coordination overhead. Scaling limits may originate from storage, networking, broker metadata, partitioning, consumers, or application design.

Consumer scalability requires separate analysis because increasing broker capacity does not guarantee faster downstream processing. Kafka consumers are influenced by partition count, RabbitMQ workers by queue and prefetch behavior, and NATS or JetStream consumers by subscription and delivery configuration. Tests should measure consumer lag, pending messages, acknowledgement delay, and processing throughput while the number of consumer instances changes.

Backpressure tests intentionally create situations where consumers process data more slowly than producers generate it. The objective is to observe how queues, logs, or streams accumulate pending data and how the system behaves as backlog grows. Measurements should include storage growth, memory use, latency, message rejection, flow control, and recovery time. This reveals whether overload remains controlled or propagates into other robot services.

Resource utilization should be recorded throughout every benchmark phase. CPU usage, memory consumption, disk throughput, disk latency, network bandwidth, connection count, thread activity, and storage growth help explain why performance changes. A broker reaching high throughput with permanently saturated resources may provide little operational safety margin for bursts, failover, maintenance activity, or future fleet expansion.

Network conditions should reflect the intended deployment topology. Local data-center tests may underestimate latency and loss experienced by robots communicating through Wi-Fi, 5G, VPNs, WAN links, or remote edge sites. Controlled experiments can introduce delay, jitter, bandwidth limits, packet loss, and temporary disconnections. Edge-to-cloud messaging should be evaluated under both normal connectivity and degraded network conditions.

Failure testing is essential because benchmark results from steady-state operation describe only part of system behavior. Broker processes can be restarted, nodes can be removed, consumers can fail, storage can become slow, or network paths can be interrupted. The benchmark should measure message loss, duplicate delivery, failover time, unavailable duration, backlog growth, and recovery behavior while the configured reliability mechanisms respond to these failures.

Recovery performance should be measured separately from failure detection. After connectivity or broker service returns, accumulated messages may create a large replay workload. The system must process this backlog while simultaneously accepting new robot events. Recovery tests should measure catch-up throughput, consumer lag reduction, resource saturation, and the time required to return to normal operating latency without destabilizing other services.

Security features should remain enabled when they will be used in production. TLS encryption, SASL authentication, ACL checks, certificate validation, and encrypted broker-to-broker traffic consume computational and network resources. Benchmarking an unsecured configuration and applying security only afterward can produce unrealistic capacity estimates. Secure and insecure tests may be compared for analysis, but production sizing should use the secured configuration.

Large sensor payloads require dedicated experiments. Camera frames, point clouds, video segments, and maps can distort ordinary messaging benchmarks because they consume substantial bandwidth and memory. Tests should compare direct broker transport with an architecture that stores large objects externally and publishes references through the broker. This helps determine payload thresholds beyond which object storage becomes more efficient.

Benchmark duration should be long enough to expose behavior that short tests can hide. Brief measurements may emphasize cache effects or temporary buffering while failing to reveal storage growth, memory pressure, garbage collection, log maintenance, consumer lag, or thermal throttling. Warm-up, steady-state, overload, recovery, and cool-down phases should be separated so that transient startup effects are not confused with sustainable performance.

Comparing Kafka, NATS, JetStream, RabbitMQ, and MQTT requires equivalent application-level requirements rather than superficially identical configuration values. Each technology has different messaging semantics and architectural strengths. Tests should normalize workload, durability objectives, consumer behavior, security, and failure expectations as closely as practical while documenting unavoidable differences instead of declaring a winner from one synthetic metric.

Performance results should be presented with both measurements and configuration context. Throughput, latency percentiles, error rates, message loss, duplicate delivery, consumer lag, resource utilization, and recovery time can be combined into a multidimensional profile. Capacity planning should then include operational headroom rather than sizing the production system exactly at the highest load achieved during laboratory testing.

For Physical AI robot fleets, the most useful benchmark is an end-to-end scenario that resembles actual deployment. Robots or realistic generators publish mixed event traffic through edge and centralized brokers while multiple services consume, store, analyze, and replay data. By testing normal load, bursts, backpressure, failures, security, and recovery together, engineers can select and size messaging infrastructure according to measurable fleet requirements rather than isolated headline performance.

메시지 브로커 성능 벤치마킹(Message Broker Performance Benchmarking)은 격리된 브로커의 최대 메시지 처리율만 측정하는 것이 아니라 목표 로봇 시스템에서 예상되는 통신 조건을 재현해야 한다. 로보틱스 워크로드(Robotics Workload)는 주기적인 텔레메트리(Telemetry), 버스트 이벤트(Burst Event), 명령(Command), 진단(Diagnostics), 비동기 처리(Asynchronous Processing)를 결합한다. 따라서 유용한 방법론은 대표적인 운영 조건에서 처리량(Throughput), 지연시간(Latency), 신뢰성(Reliability), 자원 사용량(Resource Usage), 확장성(Scalability), 복구(Recovery)를 평가해야 한다.

측정을 시작하기 전에 벤치마크 환경(Benchmark Environment)을 문서화해야 한다. 브로커 버전(Broker Version), 운영체제(Operating System), CPU, 메모리, 저장소(Storage), 네트워크 인터페이스(Network Interface), 가상화(Virtualization), 컨테이너 구성(Container Configuration), 클라이언트 하드웨어(Client Hardware)는 결과에 상당한 영향을 줄 수 있다. 또한 측정 결과를 재현하고 공정하게 비교할 수 있도록 복제(Replication), 영속성(Persistence), 승인(Acknowledgement), 압축(Compression), 배칭(Batching), 큐 또는 파티션 설정, 보안 옵션을 포함한 브로커 구성도 기록해야 한다.

워크로드 모델링(Workload Modeling)은 로봇이 실제로 생성하는 메시지 유형(Message Class)에서 시작한다. 테스트에는 빈번한 소형 텔레메트리 메시지, 중간 크기의 임무 및 진단 이벤트, 명령 트래픽(Command Traffic), 대용량 센서 객체(Large Sensor Object)에 대한 간헐적인 참조가 포함될 수 있다. 직렬화(Serialization), 네트워크 전송, 저장소, 메모리 동작은 페이로드 크기가 증가함에 따라 크게 달라지므로 하나의 고정된 페이로드 크기에 의존하지 않고 실제 운영을 반영하는 메시지 크기 분포(Message-Size Distribution)를 사용해야 한다.

메시지 발생률(Message Rate)은 여러 운영 영역(Operating Region)에 걸쳐 테스트해야 한다. 기준 부하(Baseline Load)는 정상적인 플릿 활동(Fleet Activity)을 나타내며, 점진적으로 더 높은 발생률을 적용하면 지연시간, 큐 깊이(Queue Depth), 소비자 지연(Consumer Lag)이 증가하기 시작하는 지점을 확인할 수 있다. 또한 많은 로봇이 임무 전환(Mission Transition), 네트워크 복구(Network Recovery), 비상 상황(Emergency Condition), 소프트웨어 재시작, 동기화된 운영 일정 이후 동시에 이벤트를 생성할 수 있으므로 버스트 테스트(Burst Test)도 중요하다.

처리량(Throughput)은 정의된 시간 동안 시스템이 완료하는 유효한 메시징 작업의 양을 측정한다. 초당 메시지 수(Messages per Second), 초당 바이트 수(Bytes per Second) 또는 두 가지 모두로 표현할 수 있다. 생산자 처리량(Producer Throughput)만으로는 충분하지 않은데, 브로커가 소비자가 처리할 수 있는 속도보다 빠르게 메시지를 받아들일 수 있기 때문이다. 따라서 종단 간 처리량(End-to-End Throughput)은 제어되지 않는 백로그(Backlog) 누적 없이 메시지가 생산자, 브로커, 소비자를 통과하는지를 확인해야 한다.

지연시간(Latency)은 평균값 하나로 표현하기보다 분포(Distribution)로 측정해야 한다. 중앙값 지연시간(Median Latency)은 일반적인 동작을 설명하며, p95, p99, p99.9와 같은 상위 백분위수(Higher Percentile)는 운영 응답성(Operational Responsiveness)에 영향을 줄 수 있는 느린 이벤트를 드러낸다. 로봇 시스템에서는 간헐적으로 지연되는 명령, 경보, 임무 이벤트가 평균 메시지 전달시간의 작은 개선보다 중요할 수 있으므로 꼬리 지연시간(Tail Latency)에 민감할 수 있다.

종단 간 지연시간(End-to-End Latency)은 생산자의 발행(Publication)부터 소비자의 수신 또는 처리 완료까지 전체 경로를 포함해야 한다. 진단 목적으로 브로커 자체 지연시간(Broker-Only Latency)을 별도로 측정할 수 있지만 이는 실제 애플리케이션 경험을 나타내지는 않는다. 타임스탬프(Timestamp)를 통해 발행, 브로커 상호작용(Broker Interaction), 수신, 처리 단계를 식별해야 한다. 여러 시스템에 걸쳐 측정하는 경우 시계 오차(Clock Error)가 통신 지연으로 잘못 해석되지 않도록 시계 동기화(Clock Synchronization)가 필요하다.

벤치마크 클라이언트(Benchmark Client) 자체가 성능 병목(Performance Bottleneck)이 될 수도 있다. 생산자와 소비자 시스템은 목표 부하를 생성하고 처리할 수 있는 충분한 CPU, 메모리, 네트워크 대역폭(Network Bandwidth), 동시성(Concurrency)을 갖추어야 한다. 클라이언트 측 직렬화(Client-Side Serialization), 로깅(Logging), 디스크 접근, 스레드 스케줄링(Thread Scheduling), 애플리케이션 코드가 브로커의 한계에 도달하기 전에 측정 처리량을 제한할 수 있다. 따라서 브로커 메트릭(Broker Metric)과 함께 클라이언트 자원 메트릭(Client Resource Metric)도 수집해야 한다.

영속성(Persistence)은 벤치마크 동작을 크게 변화시킨다. 메모리 기반(In-Memory) 또는 일시적 메시징(Transient Messaging)은 차이를 명시하지 않고 내구성 저장(Durable Storage)과 직접 비교해서는 안 된다. 카프카(Kafka)의 복제 및 승인 정책, 제트스트림(JetStream)의 파일 저장소(File Storage), 래빗엠큐(RabbitMQ)의 내구성 큐(Durable Queue)와 영속 메시지(Persistent Message), MQTT의 영속성 메커니즘은 서로 다른 저장 및 신뢰성 비용을 발생시킨다. 따라서 모든 성능 결과는 해당 결과에 사용된 내구성 구성(Durability Configuration)과 함께 제시해야 한다.

승인 설정(Acknowledgement Setting)은 지연시간과 전달 보장(Delivery Assurance) 사이에서 또 다른 중요한 트레이드오프(Trade-Off)를 만든다. 더 강력한 확인을 기다리면 메시지 전달시간이 증가할 수 있지만 장애 발생 시 불확실성을 줄일 수 있다. 벤치마크 수치를 높이기 위해 가장 빠른 설정만 선택하는 대신 실제 운영 환경에서 사용할 승인 모드(Acknowledgement Mode)를 평가해야 한다. 신뢰성과 성능은 독립적인 특성이 아니라 서로 연결된 속성으로 측정해야 한다.

확장성 테스트(Scalability Testing)는 시스템 규모가 증가할 때 성능이 어떻게 변화하는지를 확인한다. 테스트에서는 생산자, 소비자, 로봇 식별자(Robot Identity), 토픽(Topic), 서브젝트(Subject), 큐(Queue), 파티션(Partition), 연결(Connection), 브로커 노드(Broker Node)의 수를 변화시킬 수 있다. 확장 가능한 아키텍처는 불균형한 지연시간 증가나 조정 오버헤드(Coordination Overhead)를 발생시키지 않으면서 예측 가능하게 용량을 증가시켜야 한다. 확장 한계는 저장소, 네트워크, 브로커 메타데이터(Broker Metadata), 파티셔닝(Partitioning), 소비자 또는 애플리케이션 설계에서 발생할 수 있다.

소비자 확장성(Consumer Scalability)은 별도로 분석해야 하는데 브로커 용량 증가가 반드시 후속 처리 속도의 향상을 의미하지는 않기 때문이다. 카프카 소비자는 파티션 수의 영향을 받고, 래빗엠큐 작업자(Worker)는 큐와 프리페치 동작(Prefetch Behavior)의 영향을 받으며, 내츠(NATS) 또는 제트스트림 소비자는 구독(Subscription)과 전달 구성(Delivery Configuration)의 영향을 받는다. 테스트에서는 소비자 인스턴스 수를 변경하면서 소비자 지연, 대기 메시지(Pending Message), 승인 지연(Acknowledgement Delay), 처리량을 측정해야 한다.

백프레셔 테스트(Backpressure Test)는 의도적으로 소비자가 생산자의 데이터 생성 속도보다 느리게 처리하는 상황을 만든다. 목적은 큐, 로그(Log), 스트림(Stream)에 대기 데이터가 어떻게 축적되고 백로그가 증가함에 따라 시스템이 어떻게 동작하는지를 관찰하는 것이다. 저장소 증가량(Storage Growth), 메모리 사용량, 지연시간, 메시지 거부(Message Rejection), 흐름 제어(Flow Control), 복구 시간을 측정해야 한다. 이를 통해 과부하(Overload)가 통제된 상태로 유지되는지 또는 다른 로봇 서비스로 전파되는지를 확인할 수 있다.

모든 벤치마크 단계에서 자원 활용률(Resource Utilization)을 기록해야 한다. CPU 사용률, 메모리 소비량, 디스크 처리량(Disk Throughput), 디스크 지연시간(Disk Latency), 네트워크 대역폭, 연결 수, 스레드 활동(Thread Activity), 저장소 증가량은 성능이 변화하는 원인을 설명하는 데 도움을 준다. 높은 처리량을 달성하더라도 자원이 지속적으로 포화(Saturation)되는 브로커는 버스트, 장애 조치(Failover), 유지보수 작업 또는 향후 플릿 확장을 위한 운영 여유(Operation Headroom)가 거의 없을 수 있다.

네트워크 조건(Network Condition)은 목표 배포 토폴로지(Deployment Topology)를 반영해야 한다. 로컬 데이터센터(Local Data Center) 테스트는 Wi-Fi, 5G, VPN, WAN 링크 또는 원격 엣지 사이트(Remote Edge Site)를 통해 통신하는 로봇에서 발생하는 지연과 손실을 과소평가할 수 있다. 제어된 실험에서는 지연(Delay), 지터(Jitter), 대역폭 제한(Bandwidth Limit), 패킷 손실(Packet Loss), 일시적인 연결 중단을 적용할 수 있다. 엣지-클라우드 메시징(Edge-to-Cloud Messaging)은 정상 연결 상태와 저하된 네트워크 상태(Degraded Network Condition) 모두에서 평가해야 한다.

장애 테스트(Failure Testing)는 정상 상태(Steady-State) 운영의 벤치마크 결과만으로는 시스템 동작의 일부만 파악할 수 있기 때문에 필수적이다. 브로커 프로세스를 재시작하거나 노드를 제거하고, 소비자 장애를 발생시키거나 저장소 성능을 저하시키며, 네트워크 경로를 중단할 수 있다. 설정된 신뢰성 메커니즘(Reliability Mechanism)이 이러한 장애에 대응하는 동안 메시지 손실(Message Loss), 중복 전달(Duplicate Delivery), 장애 조치 시간(Failover Time), 서비스 불가 시간(Unavailable Duration), 백로그 증가량, 복구 동작을 측정해야 한다.

복구 성능(Recovery Performance)은 장애 탐지(Failure Detection)와 별도로 측정해야 한다. 연결이나 브로커 서비스가 복구된 이후 누적된 메시지는 대규모 재생 워크로드(Replay Workload)를 생성할 수 있다. 시스템은 새로운 로봇 이벤트를 계속 받아들이면서 동시에 이러한 백로그를 처리해야 한다. 복구 테스트에서는 따라잡기 처리량(Catch-Up Throughput), 소비자 지연 감소, 자원 포화, 다른 서비스를 불안정하게 만들지 않으면서 정상 운영 지연시간으로 복귀하는 데 필요한 시간을 측정해야 한다.

운영 환경에서 사용할 보안 기능(Security Feature)은 벤치마크에서도 활성화해야 한다. TLS 암호화(TLS Encryption), SASL 인증(SASL Authentication), ACL 검사(ACL Check), 인증서 검증(Certificate Validation), 암호화된 브로커 간 통신(Encrypted Broker-to-Broker Communication)은 연산 및 네트워크 자원을 소비한다. 보안이 비활성화된 구성만 벤치마킹한 후 나중에 보안을 적용하면 비현실적인 용량 추정(Capacity Estimate)이 발생할 수 있다. 분석을 위해 보안 적용 전후를 비교할 수 있지만 운영 용량 산정(Production Sizing)은 보안이 적용된 구성을 기준으로 해야 한다.

대용량 센서 페이로드(Large Sensor Payload)는 별도의 실험이 필요하다. 카메라 프레임(Camera Frame), 포인트 클라우드(Point Cloud), 비디오 세그먼트(Video Segment), 지도(Map)는 상당한 네트워크 대역폭과 메모리를 소비하기 때문에 일반적인 메시징 벤치마크 결과를 왜곡할 수 있다. 테스트에서는 대용량 객체를 브로커를 통해 직접 전송하는 방식과 외부 저장소에 저장하고 객체 참조(Object Reference)만 브로커로 발행하는 아키텍처를 비교해야 한다. 이를 통해 어느 페이로드 크기부터 객체 저장소(Object Storage)가 더 효율적인지 판단할 수 있다.

벤치마크 지속시간(Benchmark Duration)은 짧은 테스트에서 숨겨질 수 있는 동작을 확인할 만큼 충분히 길어야 한다. 짧은 측정은 캐시 효과(Cache Effect)나 일시적 버퍼링(Temporary Buffering)을 강조하면서 저장소 증가, 메모리 압박(Memory Pressure), 가비지 컬렉션(Garbage Collection), 로그 유지관리(Log Maintenance), 소비자 지연, 열 스로틀링(Thermal Throttling)을 발견하지 못할 수 있다. 워밍업(Warm-Up), 정상 상태, 과부하, 복구, 쿨다운(Cool-Down) 단계를 분리하여 일시적인 시작 효과가 지속 가능한 성능(Sustainable Performance)으로 잘못 해석되지 않도록 해야 한다.

카프카, 내츠, 제트스트림, 래빗엠큐, MQTT를 비교할 때는 표면적으로 동일한 구성값보다 동등한 애플리케이션 수준 요구사항(Equivalent Application-Level Requirement)을 기준으로 해야 한다. 각 기술은 서로 다른 메시징 의미론(Messaging Semantics)과 아키텍처적 강점을 가진다. 워크로드, 내구성 목표(Durability Objective), 소비자 동작, 보안, 장애 기대조건(Failure Expectation)을 가능한 한 동일하게 설정하면서 피할 수 없는 차이점을 명확하게 기록해야 하며 하나의 합성 성능 지표만으로 우열을 결정해서는 안 된다.

성능 결과(Performance Result)는 측정값과 구성 컨텍스트(Configuration Context)를 함께 제시해야 한다. 처리량, 지연시간 백분위수(Latency Percentile), 오류율(Error Rate), 메시지 손실, 중복 전달, 소비자 지연, 자원 활용률, 복구 시간을 결합하여 다차원 성능 프로파일(Multidimensional Performance Profile)을 구성할 수 있다. 이후 용량 계획(Capacity Planning)에서는 실험실 테스트에서 달성한 최대 부하에 운영 시스템을 정확히 맞추는 대신 충분한 운영 여유를 포함해야 한다.

피지컬 AI(Physical AI) 로봇 플릿에서 가장 유용한 벤치마크는 실제 배포 환경과 유사한 종단 간 시나리오(End-to-End Scenario)이다. 로봇 또는 현실적인 트래픽 생성기(Traffic Generator)가 엣지 및 중앙 브로커를 통해 혼합 이벤트 트래픽(Mixed Event Traffic)을 발행하고 여러 서비스가 데이터를 소비, 저장, 분석, 재생하도록 구성해야 한다. 정상 부하, 버스트, 백프레셔, 장애, 보안, 복구를 함께 테스트함으로써 엔지니어는 단편적인 최고 성능 수치가 아니라 측정 가능한 플릿 요구사항(Measurable Fleet Requirement)을 기준으로 메시징 인프라를 선택하고 용량을 설계할 수 있다.
