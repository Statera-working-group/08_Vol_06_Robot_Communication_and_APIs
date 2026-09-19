**Volume 06 Robot Communication and APIs**

# 06. Robot API Design

## 06.01 Robot API Design Principles: Abstraction / Safety

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 API(Robot API)는 모터, 센서, 제어기(Controller), 미들웨어 노드(Middleware Node), 하드웨어 드라이버(Hardware Driver)의 내부 구현을 직접 노출하기보다 안정적인 추상화(Abstraction)를 통해 로봇의 기능을 제공해야 한다. 응용 프로그램(Application)은 이동(Motion), 내비게이션(Navigation), 임무 실행(Mission Execution), 충전(Charging), 조작(Manipulation), 진단(Diagnostics)과 같은 개념을 중심으로 상호작용해야 한다. 이러한 분리를 통해 외부 클라이언트(Client)를 변경하지 않고도 로봇의 하드웨어와 소프트웨어 구성요소를 발전시킬 수 있다.

추상화(Abstraction)는 API를 운용 의도(Operational Intent)를 중심으로 정의하는 것에서 시작한다. 일반적으로 클라이언트는 휠 속도(Wheel Velocity), 액추에이터 전류(Actuator Current), 저수준 조향 명령(Low-Level Steering Command)을 직접 계산하는 대신 "이 목적지로 이동하라"와 같은 요청을 전달해야 한다. 로봇 플랫폼은 이러한 고수준 의도(High-Level Intent)를 경로 계획(Planning)과 제어(Control) 동작으로 변환하는 책임을 가진다. 이러한 경계는 결합도(Coupling)를 낮추고 서로 다른 이동 메커니즘과 컴퓨팅 아키텍처를 가진 로봇에서도 일관된 인터페이스(Interface)를 제공한다.

그러나 추상화 수준(Abstraction Level)은 클라이언트가 보유한 제어 권한(Authority)에 맞아야 한다. 플릿 관리 시스템(Fleet Management System)은 일반적으로 임무(Mission), 경로(Route), 충전(Charging), 운용 상태(Operational State) 인터페이스가 필요하지만, 엔지니어링 또는 유지보수 도구는 더 낮은 수준의 진단 기능이 필요할 수 있다. 따라서 직접적인 액추에이터 제어(Actuator Control)는 일반 응용 API와 분리해야 한다. 임무, 기능(Capability), 진단, 엔지니어링 인터페이스를 분리하면 편의 기능이 의도하지 않게 제한 없는 제어 채널(Control Channel)로 변하는 것을 방지할 수 있다.

로봇 API는 명령이 물리적인 결과(Physical Consequence)를 발생시킬 수 있다는 점에서 일반적인 정보 서비스 API와 다르다. 잘못된 데이터베이스 질의(Database Query)는 데이터를 손상시킬 수 있지만, 부적절한 로봇 명령은 이동, 충돌, 장비 손상 또는 사람의 부상을 초래할 수 있다. 따라서 안전성(Safety)은 API 계약(API Contract)의 아키텍처적 속성(Architectural Property)으로 다루어야 한다. 인증(Authentication)된 클라이언트도 현재 로봇 상태에 적합하지 않은 명령을 전송할 수 있으므로 인증만으로는 충분하지 않다.

모든 명령(Command)은 실행 전에 명시적인 사전 조건(Precondition)에 따라 평가되어야 한다. 이동 요청(Motion Request)은 로봇이 정상 운용 상태이고, 위치 추정(Localization)이 유효하며, 중대한 고장(Critical Fault)이 없고, 현재 안전 상태(Safety State)에서 이동이 허용되는지를 확인해야 할 수 있다. 조작 명령(Manipulation Command)은 유효한 도구 구성(Tool Configuration)과 허용 가능한 작업 공간 조건(Workspace Condition)을 요구할 수 있다. API는 안전하지 않은 명령을 하위 제어 계층으로 전달하거나 요청을 암묵적으로 변경하는 대신 사전 조건 실패(Failed Precondition)를 명확하게 보고해야 한다.

명령 권한(Command Authority)은 최소 권한 원칙(Principle of Least Privilege)을 따라야 한다. 모니터링 클라이언트(Monitoring Client)에는 읽기 전용(Read-Only) 접근 권한을 제공하고, 플릿 시스템(Fleet System)에는 임무 수준 명령 권한을 부여하며, 유지보수 응용 프로그램에는 진단 접근 권한을 제공할 수 있다. 특수 엔지니어링 도구에는 제한적인 저수준 제어 권한을 부여할 수 있다. 따라서 인가(Authorization)는 단순히 특정 신원(Identity)이 로봇에 접속할 수 있는지를 판단하는 것을 넘어 어떤 명령을 실행할 수 있는지를 정의해야 한다.

상태 인식(State Awareness)도 중요하다. 로봇 명령은 현재 상황(Context)과 결합될 때 의미를 가지기 때문이다. API는 로봇이 유휴(Idle), 실행 중(Executing), 일시 정지(Paused), 충전 중(Charging), 성능 저하(Degraded), 고장(Faulted), 비상 제한(Emergency Restriction) 상태인지 클라이언트가 이해할 수 있도록 충분한 운용 상태 정보를 제공해야 한다. 명령 수락(Command Acceptance)은 정의된 상태 전이(State Transition)와 연결되어야 하며, 서로 모순되는 요청이 물리적 동작을 임의로 변경하지 못하도록 해야 한다. 이를 통해 API는 로봇 운용 상태 머신(Operational State Machine)의 일부로 기능하게 된다.

로봇 동작에는 비동기 실행(Asynchronous Execution)에 대한 명확한 처리도 필요하다. 내비게이션, 도킹(Docking), 검사(Inspection), 조작 작업은 수초에서 수분까지 걸릴 수 있으므로 API에서 명령이 수락되었다는 사실과 실제 물리적 동작이 완료되었다는 사실을 구분해야 한다. 잘 설계된 인터페이스는 명령 수신(Command Receipt), 검증(Validation), 수락(Acceptance), 실행(Execution), 완료(Completion), 취소(Cancellation), 시간 초과(Timeout), 실패(Failure)를 구분한다. 이를 통해 클라이언트는 원시 텔레메트리(Raw Telemetry)를 반복적으로 추정하지 않고도 작업의 생명주기(Operation Lifecycle)를 파악할 수 있다.

실패 의미 체계(Failure Semantics)는 정상적인 동작만큼 신중하게 설계해야 한다. 오류(Error)는 잘못된 매개변수(Invalid Parameter), 인증 실패(Authentication Failure), 권한 부족(Insufficient Authority), 사용할 수 없는 기능(Unavailable Capability), 로봇 상태 충돌(Conflicting Robot State), 안전 인터록(Safety Interlock), 통신 장애(Communication Failure), 실행 고장(Execution Fault)을 구분해야 한다. 사람이 읽을 수 있는 설명(Human-Readable Description)이 변경되더라도 기계 판독형 오류 식별자(Machine-Readable Error Identifier)는 안정적으로 유지되어야 한다. 이를 통해 플릿 제어기와 자동화 시스템은 결정론적 복구 동작(Deterministic Recovery Behavior)을 구현할 수 있다.

안전 중요 명령(Safety-Critical Command)은 일반적인 요청보다 더욱 엄격하게 처리해야 한다. 비상 정지(Emergency Stop), 보호 정지(Protective Stop), 이동 활성화(Motion Enable), 안전 리셋(Safety Reset), 수동 오버라이드(Manual Override)는 일반 비즈니스 서비스 API 동작과 동일한 수준으로 모델링해서는 안 된다. 소프트웨어 API는 안전 관련 동작을 요청하거나 상태를 보고할 수 있지만, 인증된 안전 기능(Certified Safety Function)은 일반적으로 전용 안전 제어기(Safety Controller), 안전 회로(Safety Circuit), 안전 등급 통신 경로(Safety-Rated Communication Path)를 통해 강제되어야 한다. API가 이러한 안전 메커니즘을 네트워크 소프트웨어가 대체할 수 있다는 아키텍처적 가정을 만들어서는 안 된다.

시간적 유효성(Temporal Validity) 역시 중요한 설계 요소다. 몇 초 전에 합리적이었던 로봇 명령도 위치 추정이 변경되거나, 장애물이 출현하거나, 통신이 중단되거나, 다른 제어기가 제어 권한을 획득하면 더 이상 안전하지 않을 수 있다. 따라서 명령에는 적절한 만료(Expiration), 시간 초과(Timeout), 취소(Cancellation), 최신성(Freshness) 의미 체계를 적용해야 한다. 또한 작업이 실행 중인 상태에서 클라이언트 연결이 끊어졌을 때 어떤 동작을 수행할지 명시적으로 정의해야 하며, 이를 구현상의 우연한 동작에 의존해서는 안 된다.

동시성(Concurrency)을 처리하기 위해서는 명확한 소유권 규칙(Ownership Rule)이 필요하다. 하나의 로봇이 플릿 관리자(Fleet Manager), 로컬 운영자 인터페이스(Local Operator Interface), 유지보수 터미널(Maintenance Terminal), 클라우드 서비스(Cloud Service), 자율 AI 구성요소(Autonomous AI Component)로부터 동시에 요청을 받을 수 있다. API 아키텍처는 우선순위(Priority), 임대 권한(Lease), 명령 소유권(Command Ownership), 중재(Arbitration), 선점(Preemption) 정책을 정의해야 한다. 이러한 규칙이 없으면 각각의 명령이 개별적으로 유효하더라도 서로 충돌하여 예측하기 어려운 로봇 동작을 발생시킬 수 있다.

기능 탐색(Capability Discovery)은 이기종 로봇 플릿(Heterogeneous Robot Fleet) 전체에서 추상화를 유지하는 데 도움이 된다. 모든 로봇이 동일한 기능을 지원한다고 가정하는 대신 클라이언트가 지원 기능(Supported Capability), 운용 한계(Operational Limit), 인터페이스 버전(Interface Version), 선택 기능(Optional Feature)을 조회할 수 있도록 설계할 수 있다. 이를 통해 자율이동로봇(AMR), 모바일 매니퓰레이터(Mobile Manipulator), 사족보행 로봇(Quadruped), 무인항공기(UAV)가 공통 API 생태계(API Ecosystem)에 참여하면서 필요한 경우에만 플랫폼별 확장 기능(Platform-Specific Extension)을 제공할 수 있다.

API 스키마(API Schema)는 단위(Unit), 좌표계(Coordinate Frame), 기준 시스템(Reference System), 제약조건(Constraint)을 명확하게 표현해야 한다. 좌표계가 없는 위치(Position)나 단위가 없는 속도(Velocity)는 모호하며 잠재적으로 위험하다. 필드는 미터(Meter), 라디안(Radian), 초(Second), 타임스탬프(Timestamp), 지도 식별자(Map Identifier), 프레임 식별자(Frame Identifier), 허용 오차(Tolerance), 허용 범위(Allowable Range)를 명확하게 정의해야 한다. 잘못된 물리적 매개변수가 계획기(Planner)나 제어기(Controller)에 도달하기 전에 API 경계(API Boundary)에서 검증되어야 한다.

관측 가능성(Observability)은 디버깅 단계에서 추가되는 기능이 아니라 API 계약 자체에 포함되어야 한다. 중요한 명령에는 상관관계 식별자(Correlation Identifier)를 부여하고, 요청자 신원(Requester Identity), 명령 매개변수(Command Parameter), 검증 결과(Validation Result), 상태 전이(State Transition), 실행 결과(Execution Outcome), 관련 고장 정보(Fault Information)를 추적 가능한 이벤트(Traceable Event)로 기록해야 한다. 이러한 기록은 외부 요청과 실제 로봇 동작 사이의 인과관계(Causal Relationship)를 유지하면서 진단, 사고 조사(Incident Investigation), 플릿 분석(Fleet Analytics), 시스템 검증(System Verification)을 지원한다.

견고한 로봇 API(Robust Robot API)는 궁극적으로 디지털 의도(Digital Intent)와 물리적 동작(Physical Action) 사이에 통제된 경계(Controlled Boundary)를 형성한다. 추상화(Abstraction)는 클라이언트를 불필요한 구현 세부사항으로부터 보호하고, 안전 규칙(Safety Rule)은 이러한 추상화가 제한 없는 제어 권한으로 변하는 것을 방지한다. 안정적인 기능 모델(Capability Model), 상태 인식 검증(State-Aware Validation), 명확한 생명주기 의미 체계(Lifecycle Semantics), 인가(Authorization), 시간적 제약(Temporal Constraint), 중재(Arbitration), 관측 가능성(Observability)을 함께 적용함으로써 단일 로봇에서 이기종 플릿까지 확장하면서도 예측 가능한 물리적 동작을 유지하는 인터페이스를 구축할 수 있다.

## 06.02 Robot Command API: Atomicity, Idempotency, Timeout [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 명령 API(Robot Command API)는 디지털 요청(Digital Request)이 물리적 동작(Physical Action)으로 전환되는 경계이므로 일반적인 정보 서비스보다 명령 의미 체계(Command Semantics)가 훨씬 중요하다. 네트워크 지연(Network Delay), 재시도(Retry), 중복 메시지(Duplicated Message), 부분 장애(Partial Failure), 동시 요청(Concurrent Request)이 이동이나 조작 동작의 통제되지 않은 반복을 발생시켜서는 안 된다. 따라서 원자성(Atomicity), 멱등성(Idempotency), 시간 초과(Timeout) 의미 체계는 신뢰할 수 있는 로봇 명령 실행을 위한 핵심 속성이 된다.

원자성(Atomicity)은 요청 클라이언트(Requesting Client)의 관점에서 하나의 명령을 논리적으로 일관된 단일 작업으로 처리하는 것을 의미한다. 명령은 유효한 실행 상태(Execution State)로 수락되거나 의도하지 않은 부분적 영향(Partial Effect)을 남기지 않고 거부되어야 한다. 예를 들어 도킹 요청(Docking Request)이 실제 동작에 필요한 제어 자원을 확보하지 못한 상태에서 임무 상태만 "도킹 중(Docking)"으로 변경되어서는 안 된다. 내부 구현은 여러 단계로 이루어질 수 있지만 외부 API 계약(API Contract)은 일관된 상태 전이(State Transition)를 제공해야 한다.

물리적 동작(Physical Action)은 항상 되돌릴 수 있는 것이 아니기 때문에 기존 트랜잭션(Transaction)의 개념을 로봇에 그대로 적용하기는 어렵다. 데이터베이스 트랜잭션(Database Transaction)은 이전 데이터를 복원할 수 있지만, 이미 2미터를 이동한 로봇의 물리적 이력을 단순히 삭제할 수는 없다. 따라서 로봇 API의 원자성은 완전한 물리적 롤백(Physical Rollback)을 가정하기보다 명령 수락(Command Acceptance), 자원 예약(Resource Reservation), 상태 전이, 실행 소유권(Execution Ownership)의 일관성에 초점을 맞춰야 한다. 기존 트랜잭션의 역연산보다 보상 동작(Compensation Action)이나 안전 복구(Safe Recovery)가 현실적인 방법인 경우가 많다.

명령 검증(Command Validation)은 실행 권한(Execution Authority)이 확정되기 전에 이루어져야 한다. 매개변수(Parameter), 좌표계(Coordinate Frame), 운용 상태(Operational State), 안전 조건(Safety Condition), 요구 기능(Required Capability), 자원 가용성(Resource Availability), 인가(Authorization)를 하나의 수락 과정(Acceptance Process)으로 검증해야 한다. 필수 조건 중 하나라도 충족되지 않으면 가능한 경우 되돌릴 수 없는 물리적 동작이 시작되기 전에 명령을 거부해야 한다. 이를 통해 단순히 요청이 도착한 상태와 로봇이 명령을 공식적으로 수락한 상태 사이에 명확한 경계를 형성할 수 있다.

멱등성(Idempotency)은 분산 시스템(Distributed System)에서 발생하는 또 다른 일반적인 문제를 해결한다. 클라이언트는 자신이 전송한 요청이 서버에 실제로 수신되었는지를 항상 확인할 수 있는 것은 아니다. 로봇이 명령을 수락한 이후 응답(Acknowledgment)이 클라이언트에 도달하기 전에 네트워크 연결이 실패할 수 있다. 이때 클라이언트가 동일한 요청을 다시 전송하고 API가 이를 새로운 명령으로 해석하면 로봇은 같은 동작을 두 번 실행할 수 있다. 물리 시스템(Physical System)에서는 이러한 중복 실행이 단순한 데이터 중복 처리보다 훨씬 위험할 수 있다.

멱등성을 지원하는 명령 인터페이스(Idempotent Command Interface)는 동일한 논리적 요청(Logical Request)이 반복적으로 제출되어도 여러 개의 독립적인 실행을 생성하지 않도록 한다. 클라이언트는 의도된 작업을 고유하게 표현하는 멱등성 키(Idempotency Key), 명령 식별자(Command Identifier), 요청 식별자(Request Identifier)를 제공할 수 있다. 로봇 서비스는 이 식별 정보를 기록하고 명령 생명주기(Command Lifecycle)와 연결한다. 동일한 식별자가 다시 전달되면 새로운 물리적 동작을 자동으로 시작하는 대신 기존 명령의 상태나 결과를 반환한다.

멱등성은 모든 로봇 동작을 반복했을 때 자연스럽게 동일한 물리적 결과가 발생한다는 의미는 아니다. "앞으로 1미터 이동하라"와 같은 명령은 "위치 X로 이동하라"와 같은 상태 지향 요청(State-Oriented Request)과 본질적으로 다르다. 전자의 명령을 반복하면 로봇이 추가로 이동할 수 있지만, 후자는 동일한 목표 상태(Desired Goal)를 의미할 수 있다. 따라서 API 설계자는 본질적으로 멱등성을 갖는 동작과 명시적인 중복 억제(Duplicate Suppression)가 필요한 명령을 구분해야 한다.

명령 식별자(Command Identifier)는 정의된 보존 기간(Retention Period) 동안 유효하게 유지되어야 한다. 중복 탐지 기록(Duplicate-Detection Record)이 너무 빨리 삭제되면 지연된 재시도 요청이 새로운 작업으로 잘못 처리될 수 있다. 반대로 기록을 무기한 유지하면 저장 공간과 식별자 관리가 불필요하게 복잡해진다. 보존 정책(Retention Policy)은 예상되는 네트워크 장애 시간, 클라이언트의 재시도 동작, 임무 지속 시간(Mission Duration), 운용 위험(Operational Risk)을 고려해야 하며, 식별자를 재사용할 수 있는지도 API 계약에 명확하게 정의해야 한다.

시간 초과 의미 체계(Timeout Semantics)는 명령의 각 단계가 얼마 동안 유효한지를 정의한다. 요청 시간 초과(Request Timeout)는 클라이언트가 API 응답을 기다리는 시간을 제한하고, 수락 시간 초과(Acceptance Timeout)는 서버가 명령을 검증하고 스케줄링(Scheduling)하는 데 사용할 수 있는 시간을 제한할 수 있다. 실행 시간 초과(Execution Timeout)는 물리적 작업이 허용되는 실행 시간을 정의한다. 각각 서로 다른 명령 생명주기 단계를 제어하므로 하나의 모호한 시간 초과 값으로 표현해서는 안 된다.

클라이언트 측 시간 초과(Client-Side Timeout)가 발생했다고 해서 로봇이 해당 명령의 실행을 중단했다는 의미는 아니다. 클라이언트는 응답 대기를 중단했지만 서버에서는 내비게이션(Navigation), 도킹(Docking), 조작(Manipulation)이 계속 실행될 수 있다. 이러한 차이는 매우 중요하며, 통신 시간 초과 후 무조건적인 재시도는 서로 충돌하는 작업을 생성할 수 있다. 응답 상태가 불확실한 경우 클라이언트는 안정적인 명령 식별자를 이용하여 해당 명령이 거부(Rejected), 수락(Accepted), 실행 중(Executing), 완료(Completed), 취소(Cancelled), 실패(Failed) 중 어느 상태인지 조회해야 한다.

실행 시간 초과(Execution Timeout)가 발생했을 때의 동작도 명확하게 정의해야 한다. 내비게이션 명령이 허용된 실행 시간을 초과하면 로봇이 해당 작업을 취소할 것인지, 안전 정지(Safe Stop) 상태로 전환할 것인지, 복구 가능한 실패 상태(Recoverable Failure State)로 진입할 것인지, 또는 다른 감독 정책(Supervisory Policy)에 따라 계속 실행할 것인지 정의해야 한다. 시간 초과 처리는 단순히 API 핸들러(API Handler)를 종료하는 것이 아니라 모션 제어기(Motion Controller), 임무 관리자(Mission Manager), 안전 시스템(Safety System), 자원 관리자(Resource Manager)와 연계되어야 한다.

취소(Cancellation)는 시간 초과와 밀접하게 관련되어 있지만 별도의 동작으로 유지해야 한다. 취소 요청(Cancellation Request)은 기존 명령의 종료를 시스템에 요청하는 것이지만 실제 물리적 취소 과정에는 제어된 감속(Controlled Deceleration), 매니퓰레이터 안정화(Manipulator Stabilization), 자원 해제(Resource Release), 안전 상태(Safe State)로의 전환이 필요할 수 있다. 따라서 API는 "취소 요청됨(Cancellation Requested)"과 "취소 완료(Cancelled)"를 구분해야 한다. 이를 통해 외부 응용 프로그램이 취소 메시지가 수신되는 즉시 물리적 동작까지 완전히 중단되었다고 잘못 판단하는 것을 방지할 수 있다.

여러 클라이언트가 동시에 명령을 전송하는 동시성(Concurrency) 환경에서는 추가적인 원자성 요구사항이 발생한다. 두 개의 유효한 명령이 동일한 이동 서브시스템(Mobility Subsystem), 매니퓰레이터(Manipulator), 충전 인터페이스(Charging Interface), 임무 자원(Mission Resource)을 동시에 요구할 수 있다. 따라서 명령 수락 과정에서는 적절한 추상화 수준에서 소유권(Ownership)과 잠금(Locking)을 조정해야 한다. 개별적으로는 유효하지만 동시에 안전하게 실행할 수 없는 독립적인 요청들이 함께 수락되는 상황을 API가 방지해야 한다.

재시도(Retry)는 제한 없는 반복이 아니라 명시적인 정책(Explicit Policy)에 따라 수행되어야 한다. 일시적인 통신 장애(Temporary Communication Failure)는 지수 백오프(Exponential Backoff)와 제한된 재시도(Bounded Retry)를 적용할 수 있지만, 실행 실패(Execution Failure)는 다시 시도하기 전에 로봇 상태를 평가해야 하는 경우가 많다. 예를 들어 충전 스테이션이 사용 중이어서 도킹에 실패한 로봇이 즉시 동일한 동작을 반복하는 것이 항상 적절한 것은 아니다. 재시도 정책은 오류 분류(Error Classification), 명령 식별 정보, 로봇 상태, 안전 조건, 원래 실행에서 발생한 부분적 물리 효과를 함께 고려해야 한다.

명령 상태 모델(Command Status Model)은 원자성, 멱등성, 시간 초과 동작을 통합하기 위해 필요한 정보를 제공한다. 실용적인 생명주기는 수신(Received), 검증 중(Validating), 수락(Accepted), 대기(Queued), 실행 중(Executing), 취소 중(Cancelling), 완료(Completed), 실패(Failed), 거부(Rejected), 만료(Expired), 취소 완료(Cancelled) 상태를 구분할 수 있다. 모든 구현이 동일한 상태 이름을 사용할 필요는 없지만 상태 전이는 결정론적(Deterministic)이고 외부에서 명확하게 이해할 수 있어야 한다. 최종 상태(Terminal State)는 클라이언트가 불확실한 통신 결과를 확인할 수 있도록 충분한 기간 동안 조회 가능해야 한다.

신뢰할 수 있는 명령 API는 지속적인 감사 가능성(Auditability)도 필요로 한다. 각 명령에는 식별자, 요청자 신원(Requester Identity), 타임스탬프(Timestamp), 매개변수, 검증 결과, 상태 전이, 재시도 관계(Retry Relationship), 시간 초과 이벤트(Timeout Event), 취소 이력(Cancellation History), 최종 결과(Final Result)가 기록되어야 한다. 이러한 기록을 통해 예상하지 못한 물리적 동작이 새로운 요청, 중복 전달(Duplicate Delivery), 지연된 네트워크 메시지, 복구 절차(Recovery Procedure), 내부 실행 장애 중 어디에서 발생했는지 재구성할 수 있다.

플릿 환경(Fleet Environment)에서는 명령이 플릿 관리자(Fleet Manager), 게이트웨이(Gateway), 메시지 브로커(Message Broker), 엣지 컴퓨터(Edge Computer), 로봇 로컬 서비스(Robot-Local Service)를 거쳐 전달될 수 있으므로 이러한 원칙이 더욱 중요해진다. 가능한 경우 종단 간 명령 식별 정보(End-to-End Command Identity)가 이러한 경계를 통과해도 유지되어야 한다. 전송 계층의 메시지 전달 보장(Message Delivery Guarantee)만으로 물리적 정확히 한 번 실행(Physical Exactly-Once Execution)을 보장할 수 없다. 응용 계층(Application Layer)에서 고유 명령 식별, 중복 탐지, 상태 관리, 통제된 실행 소유권을 결합해야 한다.

견고한 로봇 명령 API(Robust Robot Command API)는 원자성(Atomicity), 멱등성(Idempotency), 시간 초과(Timeout)를 서로 연결된 안전 및 신뢰성 메커니즘(Safety and Reliability Mechanism)으로 다룬다. 원자성은 명령 상태의 일관성을 보호하고, 멱등성은 의도하지 않은 중복 실행을 방지하며, 시간 초과 의미 체계는 통신 장애를 물리적 실행 종료와 혼동하지 않으면서 명령의 시간적 유효성을 제한한다. 명시적인 취소(Cancellation), 동시성 제어(Concurrency Control), 지속적인 명령 상태(Persistent Command Status), 감사 로깅(Audit Logging)을 함께 적용하면 네트워크와 분산 구성요소에 장애가 발생하는 상황에서도 예측 가능한 로봇 동작을 구현할 수 있다.

## 06.03 Robot Status API: Real-Time Feed / Event Design [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 상태 API(Robot Status API)는 외부 시스템이 로봇의 운용 상태(Operational Condition)를 지속적으로 해석할 수 있도록 표현하는 인터페이스를 제공한다. 요청된 동작을 표현하는 명령 API(Command API)와 달리 상태 인터페이스(Status Interface)는 로봇이 현재 무엇을 수행하고 있는지, 어떤 자원을 사용할 수 있는지, 비정상 상태가 존재하는지를 설명한다. 이러한 설계는 불필요한 내부 구현 세부사항을 노출하지 않으면서 운영자(Operator), 플릿 관리자(Fleet Manager), 모니터링 응용 프로그램(Monitoring Application), 자율 서비스(Autonomous Service)를 지원해야 한다.

로봇 상태(Robot Status)는 정리되지 않은 센서 값들의 집합이 아니라 구조화된 상태(Structured State)로 모델링해야 한다. 일반적인 정보에는 운용 모드(Operational Mode), 임무 상태(Mission State), 내비게이션 상태(Navigation State), 위치 추정 품질(Localization Quality), 배터리 상태(Battery Condition), 충전 상태(Charging State), 속도(Velocity), 자세 및 위치(Pose), 안전 상태(Safety State), 연결 상태(Connectivity), 활성 고장(Active Fault), 서브시스템 건전성(Subsystem Health) 등이 포함된다. 각 필드에는 정의된 의미 체계(Semantics), 단위(Unit), 좌표계(Coordinate Frame), 타임스탬프(Timestamp), 유효 조건(Validity Condition)이 있어야 모든 클라이언트가 동일한 데이터를 일관되게 해석할 수 있다.

유용한 설계에서는 비교적 지속적인 상태(Persistent State)와 빠르게 변화하는 텔레메트리(Telemetry)를 분리한다. 로봇 식별 정보(Robot Identity), 기능(Capability), 소프트웨어 버전(Software Version), 운용 모드, 현재 임무(Current Mission)는 상대적으로 드물게 변경되지만, 자세 및 위치, 속도, 배터리 전류(Battery Current), 위치 추정 신뢰도(Localization Confidence), 센서 건전성(Sensor Health)은 지속적으로 변화할 수 있다. 모든 필드를 하나의 고주파 메시지(High-Frequency Message)에 포함하면 대역폭과 처리 자원이 낭비되므로 상태 정보는 갱신 주기(Update Frequency), 중요도(Importance), 소비자 요구사항(Consumer Requirement)에 따라 분리해야 한다.

실시간 상태 전달(Real-Time Status Delivery)은 지속적인 폴링(Polling)보다 푸시 기반 통신(Push-Based Communication)을 이용하는 것이 일반적으로 효과적이다. 웹소켓(WebSocket), gRPC 스트리밍(gRPC Streaming), MQTT 또는 기타 이벤트 지향 전송(Event-Oriented Transport)을 통해 로봇이나 엣지 서비스(Edge Service)에서 필요한 소비자에게 실시간 피드(Live Feed)를 유지할 수 있다. REST 엔드포인트(REST Endpoint)는 스냅샷(Snapshot), 초기화(Initialization), 복구(Recovery), 관리 질의(Administrative Query)에 유용하다. 스냅샷 API와 스트리밍 인터페이스(Streaming Interface)를 결합하면 클라이언트가 알려진 상태를 먼저 확보한 다음 증분 변경(Incremental Change)을 효율적으로 수신할 수 있다.

"실시간(Real Time)"의 의미는 즉각적인 통신을 의미한다고 가정하기보다 운용 요구사항(Operational Requirement)에 따라 정의해야 한다. 플릿 대시보드(Fleet Dashboard)는 1초 간격의 갱신을 허용할 수 있지만, 모션 감독(Motion Supervision)이나 안전 관련 모니터링(Safety-Related Monitoring)은 훨씬 짧은 주기와 결정론적인 로컬 메커니즘(Deterministic Local Mechanism)을 요구할 수 있다. API 설계자는 예상 갱신 주기(Expected Update Period), 최대 허용 데이터 수명(Maximum Acceptable Age), 지연 목표(Latency Target), 혼잡 상황에서의 동작을 정의해야 한다. 안전 중요 제어(Safety-Critical Control)는 일반적인 클라우드 지향 상태 피드(Cloud-Oriented Status Feed)에만 의존해서는 안 된다.

이벤트(Event)는 의미 있는 변화나 발생 상황을 표현한다는 점에서 주기적인 상태(Status)와 다르다. 예를 들어 임무 시작(Mission Started), 웨이포인트 도달(Waypoint Reached), 도킹 완료(Docking Completed), 배터리 임계값 도달(Battery Threshold Crossed), 장애물 감지(Obstacle Detected), 위치 추정 상실(Localization Lost), 보호 정지 활성화(Protective Stop Activated), 통신 복구(Communication Restored), 고장 발생(Fault Raised) 등이 이벤트가 될 수 있다. 이벤트 기반 설계(Event-Driven Design)는 불필요한 통신량을 줄이고 소비자가 연속적인 상태 스냅샷을 반복 비교하지 않고도 중요한 상태 전이에 반응할 수 있도록 한다.

각 이벤트는 독립적으로 활용할 수 있을 만큼 충분한 상황 정보(Context)를 포함해야 한다. 실용적인 이벤트 엔벌로프(Event Envelope)는 이벤트 식별자(Event Identifier), 이벤트 유형(Event Type), 로봇 식별자(Robot Identifier), 타임스탬프, 시퀀스 번호(Sequence Number), 심각도(Severity), 발생 서브시스템(Source Subsystem), 관련 명령 또는 임무 식별자(Related Command or Mission Identifier), 구조화된 페이로드(Structured Payload)를 포함할 수 있다. 안정적인 이벤트 유형과 스키마(Schema)를 사용하면 모니터링, 분석(Analytics), 플릿 오케스트레이션(Fleet Orchestration), 사고 관리 시스템(Incident-Management System)이 사람이 읽는 메시지의 불안정한 해석에 의존하지 않고 이벤트를 자동으로 처리할 수 있다.

타임스탬프(Timestamp)는 분산 로봇 시스템(Distributed Robotic System)에서 특히 중요하다. 이벤트가 물리적으로 발생한 시점과 이벤트가 감지, 전송, 수신 또는 저장된 시점은 서로 다를 수 있다. 상태 및 이벤트 API는 타임스탬프 출처(Timestamp Provenance)를 명확하게 정의하고 시간적 상관관계(Temporal Correlation)가 중요한 경우 동기화된 시계(Synchronized Clock)를 사용해야 한다. 로봇, 센서, 엣지 컴퓨터(Edge Computer), 플릿 서버(Fleet Server)가 정보를 교환할 때 일관된 시간 기준(Time Reference)을 사용하면 여러 구성요소에 걸친 인과적 순서(Causal Sequence)를 재구성할 수 있다.

시퀀스 번호(Sequence Number)는 타임스탬프를 보완하여 클라이언트가 누락, 중복 또는 순서가 뒤바뀐 갱신을 식별하도록 지원한다. 네트워크가 다시 연결되거나 메시지 브로커(Message Broker)가 메시지를 재전송하거나 버퍼링된 데이터(Buffered Data)가 더 최신 정보보다 늦게 도착할 수 있다. 단조 증가 시퀀스(Monotonically Increasing Sequence) 또는 스트림별 리비전(Stream-Specific Revision)을 사용하면 소비자가 데이터의 공백(Gap)을 감지하고 새로운 스냅샷을 요청할지 판단할 수 있다. 이는 완전한 로봇 상태를 반복적으로 전송하는 대신 증분 상태 갱신(Incremental Status Update)을 사용하는 경우 특히 유용하다.

상태 최신성(Status Freshness)은 명시적으로 표현해야 한다. 특정 필드의 형식이 유효하더라도 센서 갱신이 중단되거나 통신이 끊기면 해당 정보는 오래된 상태(Stale State)가 될 수 있다. 따라서 API는 값 자체와 해당 값의 최신성 또는 유효성(Freshness or Validity)을 구분해야 한다. 타임스탬프, 데이터 수명 제한(Age Limit), 유효성 플래그(Validity Flag), 품질 지표(Quality Indicator), 오래된 상태 표시(Stale-State Marker)를 사용하면 클라이언트가 과거의 위치 추정, 배터리, 안전, 임무 정보를 현재 로봇 상태로 잘못 해석하는 것을 방지할 수 있다.

이벤트 전달 의미 체계(Event Delivery Semantics)도 정의해야 한다. 일부 소비자는 시각화 데이터(Visualization Data)에 대해 최선형 전달(Best-Effort Delivery)을 허용할 수 있지만, 운용 이벤트(Operational Event)는 확인 응답(Acknowledgment), 영속성(Persistence), 재생(Replay), 최소 한 번 전달(At-Least-Once Delivery)을 요구할 수 있다. 중복 전달이 가능한 경우 이벤트 식별자를 이용하여 소비자가 중복 제거(Deduplication)를 수행할 수 있다. 분산 플릿 전체에서 정확히 한 번 처리(Exactly-Once Behavior)를 보장하려는 방식은 비용과 복잡성이 높을 수 있으므로 응용 계층의 식별 정보(Application-Level Identity)와 멱등 처리(Idempotent Processing)를 결합하는 것이 더욱 현실적인 경우가 많다.

재연결 동작(Reconnection Behavior)은 실시간 피드 설계의 핵심 요소다. 연결이 끊어진 이후 클라이언트는 이전에 가지고 있던 로컬 상태(Local State)가 여전히 정확하다고 가정해서는 안 된다. 견고한 방식은 연결을 다시 설정한 후 현재 상태 스냅샷(Current Status Snapshot)을 획득하고, 최신 스트림 위치(Stream Position) 또는 시퀀스 번호를 확인한 다음 이벤트 수신을 재개하는 것이다. 이벤트 재생(Event Replay)을 지원한다면 클라이언트는 정상적인 실시간 처리로 복귀하기 전에 연결이 끊어진 기간 동안 생성된 이벤트를 요청할 수 있다.

플릿 규모(Fleet Scale)가 증가할수록 대역폭 관리(Bandwidth Management)의 중요성도 커진다. 수백 대의 로봇이 고주파의 전체 상태 객체(Complete Status Object)를 전송하면 게이트웨이(Gateway), 메시지 브로커, 무선 네트워크(Wireless Network), 대시보드에 과도한 부하를 발생시킬 수 있다. API는 설정 가능한 갱신 속도(Configurable Update Rate), 필드 선택(Field Selection), 구독(Subscription), 델타 갱신(Delta Update), 이벤트 필터링(Event Filtering), 집계(Aggregation), 압축(Compression), 우선순위 클래스(Priority Class)를 통해 이러한 부하를 줄일 수 있다. 중요도가 낮은 텔레메트리가 제한되는 상황에서도 핵심 운용 정보(Critical Operational Information)는 계속 사용할 수 있어야 한다.

구독 모델(Subscription Model)을 사용하면 각 소비자가 필요한 정보만 수신할 수 있다. 플릿 스케줄러(Fleet Scheduler)는 임무, 가용성(Availability), 배터리 이벤트를 구독할 수 있고, 유지보수 시스템(Maintenance System)은 온도, 장치 건전성(Device Health), 고장 상태 전이(Fault Transition)가 필요할 수 있다. 시각화 클라이언트(Visualization Client)는 더 높은 빈도의 자세 및 위치 갱신이 필요할 수 있다. 토픽 구조(Topic Structure), 스트림 필터(Stream Filter), 구독 매개변수(Subscription Parameter)는 일관된 의미 체계를 유지하면서 모든 소비자가 모든 로봇 신호를 수신하는 비효율을 방지해야 한다.

고장 및 건전성 보고(Fault and Health Reporting)는 현재 상태와 과거 이벤트를 구분해야 한다. 활성 고장(Active Fault)은 현재 로봇에 영향을 주고 있는 조건을 의미하지만, 고장 발생 이벤트(Fault-Raised Event)는 해당 조건이 언제 발생했는지를 기록하고 고장 해제 이벤트(Fault-Cleared Event)는 언제 문제가 해소되었는지를 기록한다. 이러한 분리를 통해 클라이언트가 재연결되거나 재시작되었을 때 발생할 수 있는 모호성을 방지할 수 있다. 스냅샷은 현재 무엇이 잘못되어 있는지를 보여주고, 이벤트 스트림(Event Stream)은 로봇이 언제 어떻게 현재 상태에 도달했는지를 설명한다.

상태 API는 로봇 전체를 하나의 불리언 값(Boolean Value)으로 단순화하기보다 계층적인 건전성 정보(Hierarchical Health Information)를 제공해야 한다. 전체 가용성(Overall Availability)은 위치 추정, 이동성(Mobility), 인지(Perception), 통신(Communication), 컴퓨팅(Compute), 배터리, 조작(Manipulation), 안전 서브시스템(Safety Subsystem)의 상태에 따라 달라질 수 있다. 성능 저하 상태(Degraded State)의 로봇도 선택적 기능(Optional Capability) 하나를 사용할 수 없을 뿐 제한된 임무를 계속 수행할 수 있다. 구조화된 서브시스템 건전성(Structured Subsystem Health)을 제공하면 플릿 오케스트레이션 소프트웨어가 모든 비정상 상태를 완전한 고장으로 취급하지 않고 기능 인식형 의사결정(Capability-Aware Decision)을 수행할 수 있다.

읽기 중심의 상태 인터페이스(Read-Oriented Status Interface)에서도 보안(Security)과 인가(Authorization)는 필요하다. 로봇 위치(Robot Location), 임무 정보(Mission Information), 카메라 기반 이벤트(Camera-Derived Event), 운용 일정(Operational Schedule), 진단 정보(Diagnostic Information)는 민감한 운용 정보를 노출할 수 있다. 따라서 클라이언트는 자신의 역할(Role)에 적합한 상태 필드와 이벤트 스트림만 수신해야 한다. 인증(Authentication), 인가, 암호화 전송(Encrypted Transport), 구독 제어(Subscription Control), 감사 로깅(Audit Logging), 속도 제한(Rate Limiting)을 적용하되 불필요하게 지연을 증가시키거나 지속적인 모니터링을 방해하지 않도록 설계해야 한다.

잘 설계된 로봇 상태 API(Robot Status API)는 궁극적으로 스냅샷(Snapshot), 실시간 피드(Real-Time Feed), 이벤트(Event)를 하나의 일관된 상태 관찰 모델(State-Observation Model)로 통합한다. 스냅샷은 권위 있는 현재 상태(Authoritative Current State)를 설정하고, 스트리밍 갱신(Streaming Update)은 적시 상태 인식(Timely Awareness)을 유지하며, 이벤트는 의미 있는 상태 전이(Meaningful Transition)를 전달한다. 명확한 타임스탬프, 시퀀스 번호, 최신성 의미 체계(Freshness Semantics), 재연결 규칙(Reconnection Rule), 확장 가능한 구독(Scalable Subscription), 구조화된 건전성 정보, 안전한 접근(Secure Access)을 결합하면 단일 로봇에서 대규모 이기종 플릿(Heterogeneous Fleet)에 이르기까지 신뢰할 수 있는 모니터링 인터페이스를 구축할 수 있다.

## 06.04 Robot Mission API: Create, Cancel, Progress [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 임무 API(Robot Mission API)는 개별 액추에이터(Actuator) 또는 모션 명령(Motion Command)이 아니라 완전한 운용 목표(Operational Objective)를 표현하기 위한 상위 수준 인터페이스(High-Level Interface)를 제공한다. 임무(Mission)는 자재 운송, 검사 지점 방문, 순찰 수행, 충전을 위한 도킹(Docking), 또는 일련의 조작 작업(Manipulation Task) 실행 등을 의미할 수 있다. API는 이러한 목표를 통제된 로봇 동작으로 변환하면서 생성부터 완료, 취소 또는 실패까지 명확한 생명주기(Lifecycle)를 유지해야 한다.

임무 생성(Mission Creation)은 의도(Intent)를 구조적으로 정의하는 것에서 시작한다. 요청에는 임무 식별자(Mission Identifier), 임무 유형(Mission Type), 대상 로봇 또는 로봇 그룹, 목적지(Destination), 웨이포인트(Waypoint), 동작(Action), 우선순위(Priority), 시간 제약(Timing Constraint), 페이로드 정보(Payload Information), 실행 정책(Execution Policy) 등이 포함될 수 있다. 인터페이스는 외부 응용 프로그램이 내부 플래너(Planner), 제어기(Controller), ROS2 노드(ROS2 Node), 하드웨어 드라이버(Hardware Driver), 플랫폼별 구현 세부사항을 이해하도록 요구하지 않으면서 무엇을 수행해야 하는지를 표현해야 한다.

새롭게 생성된 임무가 실행 가능한 상태가 되기 전에 검증(Validation)이 수행되어야 한다. 시스템은 필수 매개변수(Required Parameter), 좌표계(Coordinate Frame), 로봇 기능(Robot Capability), 목적지 유효성(Destination Validity), 자원 가용성(Resource Availability), 인가(Authorization), 운용 상태(Operational State), 관련 안전 제약(Safety Constraint)을 확인해야 한다. 문법적으로 올바른 임무라도 운용 측면에서는 실행이 불가능할 수 있다. 따라서 API는 잘못 구성된 요청(Malformed Request)과 구조적으로는 유효하지만 현재 실행을 수락할 수 없는 임무를 구분해야 한다.

임무 생성은 전체 생명주기 동안 해당 작업과 연결되는 안정적인 임무 식별자(Stable Mission Identifier)를 반환해야 한다. 이 식별자를 이용하여 클라이언트(Client)는 상태를 조회하고, 이벤트(Event)를 연계하고, 취소를 요청하고, 오류를 확인하며, 최종 결과를 획득할 수 있다. 클라이언트가 생성한 요청 식별자(Request Identifier) 또는 멱등성 키(Idempotency Key)를 추가로 사용하면 네트워크 장애로 인해 응답이 불확실해져 요청을 다시 전송해야 하는 경우 중복 임무 생성(Duplicate Mission Creation)을 방지할 수 있다.

임무 생명주기(Mission Lifecycle)는 모호한 텍스트 설명 대신 명시적인 상태(Explicit State)를 사용해야 한다. 일반적인 상태에는 생성됨(Created), 검증 중(Validating), 수락됨(Accepted), 대기 중(Queued), 할당됨(Assigned), 실행 중(Executing), 일시 정지됨(Paused), 취소 중(Cancelling), 취소됨(Cancelled), 완료됨(Completed), 실패함(Failed), 만료됨(Expired) 등이 포함될 수 있다. 구현에 따라 정확한 용어는 달라질 수 있지만 상태 전이(State Transition)는 결정론적(Deterministic)이어야 한다. 클라이언트는 임무가 자원을 기다리는 중인지, 물리적으로 실행 중인지, 종료 과정에 있는지, 또는 이미 최종 상태(Terminal State)에 도달했는지를 판단할 수 있어야 한다.

플릿 환경(Fleet Environment)에서는 임무 할당(Mission Assignment)과 임무 생성이 서로 구분된다. 창고 또는 병원 시스템은 특정 자율이동로봇(AMR)이 선정되기 전에 운송 임무를 생성할 수 있다. 이후 플릿 관리자(Fleet Manager)는 위치(Location), 배터리 수준(Battery Level), 적재 용량(Payload Capacity), 가용성(Availability), 기능(Capability), 교통 상황(Traffic Condition), 스케줄링 정책(Scheduling Policy)에 따라 적절한 로봇을 선택할 수 있다. 임무 의도와 로봇 할당을 분리하면 이기종 플릿(Heterogeneous Fleet)과 변화하는 운용 조건에서 유연한 오케스트레이션(Orchestration)이 가능하다.

복잡한 임무(Complex Mission)는 일반적으로 단계(Stage), 작업(Task), 또는 동작(Action)으로 분해된다. 운송 임무는 픽업 지점까지의 내비게이션(Navigation), 페이로드 확인(Payload Confirmation), 목적지까지 이동, 배송 확인(Delivery Confirmation), 복귀 또는 대기 동작으로 구성될 수 있다. 각 단계는 자체 상태를 유지하면서 전체 임무 상태에 기여할 수 있다. 계층적 표현(Hierarchical Representation)을 사용하면 모든 저수준 경로(Trajectory), 제어 명령 또는 내부 실행 세부사항을 노출하지 않고도 클라이언트가 진행 상황을 이해할 수 있다.

진행 상황 보고(Progress Reporting)는 단순한 백분율 값만 제공하기보다 의미 있는 운용 진행 상태(Operational Advancement)를 전달해야 한다. 작업량을 자연스럽게 측정할 수 있는 경우 완료율(Percentage Completion)이 유용하지만, 많은 로봇 임무에는 수행 시간이 서로 다르고 실행 시간을 예측하기 어려운 작업이 포함된다. 더욱 강력한 진행 모델(Progress Model)은 현재 단계(Current Stage), 완료된 단계(Completed Stage), 활성 웨이포인트(Active Waypoint), 남은 작업(Remaining Task), 경과 시간(Elapsed Time), 가능한 경우 예상 잔여 시간(Estimated Remaining Time), 주요 실행 이벤트(Significant Execution Event)를 제공할 수 있다.

임무 진행 상황은 질의(Query) 인터페이스와 스트리밍(Streaming) 인터페이스를 모두 통해 전달할 수 있다. REST 방식 요청(REST-Style Request)은 권위 있는 스냅샷(Authoritative Snapshot)을 조회하는 데 유용하며, 웹소켓(WebSocket), gRPC 스트리밍(gRPC Streaming), MQTT 또는 이벤트 피드(Event Feed)는 지속적인 갱신을 제공할 수 있다. 클라이언트는 먼저 현재 임무 표현(Current Mission Representation)을 조회한 후 변경 사항을 구독(Subscribe)할 수 있다. 이러한 하이브리드 방식(Hybrid Approach)은 과도한 폴링(Polling) 없이 대시보드(Dashboard), 플릿 관리자, 워크플로 시스템(Workflow System), 모바일 응용 프로그램을 지원할 수 있다.

진행 상황 갱신(Progress Update)에는 타임스탬프(Timestamp)와 시퀀스 정보(Sequence Information)가 포함되어야 하며, 이를 통해 클라이언트는 오래되거나 누락되거나 중복되거나 순서가 변경된 데이터를 식별할 수 있다. 통신이 중단되더라도 클라이언트는 임무 진행이 중단되었다고 가정해서는 안 된다. 재연결(Reconnection) 이후에는 권위 있는 임무 상태를 다시 조회한 다음 적절한 위치에서 스트리밍을 재개할 수 있다. 이벤트 재생(Event Replay)을 지원하면 연결이 끊어진 기간에 발생한 중요한 상태 전이도 복구할 수 있다.

임무 취소(Mission Cancellation)는 단순히 레코드(Record)를 삭제하거나 데이터베이스 플래그(Database Flag)를 변경하는 것 이상의 처리가 필요하다. 취소 요청이 도착했을 때 로봇은 이미 이동 중이거나, 페이로드를 운반하거나, 물체를 조작하거나, 엘리베이터에 진입하거나, 도킹을 수행하고 있을 수 있다. 따라서 임무 API는 취소를 현재 작업을 적절한 종료 상태(Termination State)로 전환하기 위해 플래너, 제어기, 자원 관리자(Resource Manager), 안전 메커니즘(Safety Mechanism)과 협력하는 통제된 요청(Controlled Request)으로 처리해야 한다.

API는 취소 요청(Cancellation Request)과 취소 완료(Completed Cancellation)를 구분해야 한다. 로봇이 제어된 감속(Controlled Deceleration), 예약 해제(Reservation Release), 매니퓰레이터 안정화(Manipulator Stabilization), 교통 자원 해제(Traffic Resource Release), 안전한 중간 상태(Safe Intermediate State)로의 전환을 수행하는 동안 임무는 실행 중(Executing)에서 취소 중(Cancelling)으로 전환될 수 있다. 이러한 절차가 완료된 후에만 임무가 취소됨(Cancelled) 상태가 되어야 한다. 이를 통해 취소 요청이 수락된 즉시 물리적 동작까지 중단되었다고 클라이언트가 잘못 판단하는 것을 방지할 수 있다.

취소 정책(Cancellation Policy)은 임무 상황(Mission Context)에 따라 달라질 수 있다. 일부 동작은 즉시 중단할 수 있지만 다른 동작은 종료 전에 원자적 구간(Atomic Section)이나 안전 중요 구간(Safety-Critical Section)을 완료해야 할 수 있다. 엘리베이터 내부에 있거나 불안정한 물체를 운반하거나 도킹 동작을 수행하는 로봇에는 특수한 처리 방식이 필요할 수 있다. API는 취소가 수락(Accepted), 지연(Deferred), 거부(Rejected), 또는 이미 완료(Completed)되었는지를 표현하고 즉시 종료할 수 없는 경우 기계 판독형 사유(Machine-Readable Reason)를 제공해야 한다.

임무 실패(Mission Failure)는 취소와 명확하게 구분되어야 한다. 취소는 일반적으로 의도적인 외부 또는 감독 결정(Supervisory Decision)에 의해 발생하지만, 실패는 실행 과정에서 임무 목표를 달성할 수 없었음을 의미한다. 위치 추정 상실(Localization Loss), 경로 차단(Blocked Route), 배터리 부족(Depleted Battery), 페이로드 문제(Payload Problem), 서브시스템 고장(Subsystem Fault), 인프라 사용 불가(Unavailable Infrastructure), 실행 제한 초과(Exceeded Execution Limit) 등이 실패의 원인이 될 수 있다. 이러한 구분을 유지하면 플릿 시스템이 적절한 복구(Recovery), 재할당(Reassignment), 에스컬레이션(Escalation), 재시도(Retry) 정책을 적용할 수 있다.

복구 동작(Recovery Behavior)은 무기한 실행되는 임무 내부에 숨겨두기보다 명시적으로 표현해야 한다. 일시적인 장애물은 재계획(Replanning)을 유발할 수 있고, 통신 단절은 대기 상태를 발생시킬 수 있으며, 도킹 실패에는 제한된 재시도(Bounded Retry)가 허용될 수 있다. 임무 API는 원래 임무 식별자를 유지하면서 복구 상태(Recovery State) 또는 복구 이벤트(Recovery Event)를 보고할 수 있다. 이를 통해 클라이언트는 정상적으로 진행되는 복구 과정과 실질적인 진행이 중단된 임무를 구분할 수 있다.

자원 관리(Resource Management)는 임무 실행과 밀접하게 연결되어 있다. 임무는 엘리베이터(Elevator), 충전 스테이션(Charging Station), 적재 구역(Loading Area), 출입문(Door), 교통 구역(Traffic Zone), 도구(Tool), 공유 조작 자원(Shared Manipulation Resource)을 예약할 수 있다. 예약(Reservation)은 임무 식별자와 연결되어야 하며 완료, 취소, 만료 또는 실패 이후 안정적으로 해제되어야 한다. 그렇지 않으면 원래 자원을 예약했던 로봇이 더 이상 사용하지 않는데도 버려진 예약이 이후의 임무를 차단할 수 있다.

여러 임무가 로봇과 인프라 자원을 놓고 경쟁하는 경우 우선순위(Priority)와 선점 정책(Preemption Policy)이 중요해진다. 긴급 검사(Urgent Inspection), 안전 대응(Safety Response), 충전 요구(Charging Requirement)는 낮은 우선순위의 작업을 중단해야 할 수 있다. API는 임의의 클라이언트가 운용 거버넌스(Operational Governance)를 무시하지 못하도록 하면서 우선순위를 표현해야 한다. 선점(Preemption)이 발생하면 영향을 받은 임무가 정의된 상태를 거쳐 전이되도록 하여 이후의 재개(Continuation), 취소 또는 재할당 상태를 명확하게 이해할 수 있어야 한다.

임무 결과(Mission Result)는 단순한 최종 성공 여부 불리언 값(Boolean) 이상의 정보를 포함해야 한다. 완료된 운송 임무는 전달된 페이로드 정보, 목적지, 완료 타임스탬프(Completion Timestamp), 이동 거리(Distance Traveled), 관련 작업 결과(Task Result)를 보고할 수 있다. 실패한 임무는 구조화된 실패 코드(Failure Code), 실행이 중단된 단계, 복구 가능성(Recoverability) 정보를 제공해야 한다. 취소된 임무는 운용상 필요한 경우 누가 또는 무엇이 취소를 요청했는지와 최종 종료 조건(Final Termination Condition)을 기록해야 한다.

임무는 긴 일련의 물리적 동작을 발생시킬 수 있으므로 감사 가능성(Auditability)이 필수적이다. 시스템은 임무 생성 데이터, 요청자 신원(Requester Identity), 검증 결과(Validation Outcome), 로봇 할당, 상태 전이, 생성된 명령 또는 작업, 진행 이벤트, 재시도, 취소 요청, 고장, 자원 사용(Resource Usage), 최종 결과를 보존해야 한다. 상관관계 식별자(Correlation Identifier)를 사용하면 플릿 서비스, 로봇 소프트웨어, 메시지 브로커(Message Broker), 모니터링 시스템의 정보를 하나의 운용 이력(Operational History)으로 재구성할 수 있다.

견고한 로봇 임무 API(Robust Robot Mission API)는 생성(Create), 취소(Cancel), 진행 상황(Progress) 기능을 서로 독립적인 엔드포인트(Endpoint)가 아니라 하나의 지속적인 임무 생명주기(Persistent Mission Lifecycle)를 구성하는 요소로 다룬다. 생성은 검증된 운용 의도(Validated Operational Intent)를 확립하고, 진행 상황 보고는 의미 있는 실행 상태를 제공하며, 취소는 물리적 동작을 통제된 방식으로 종료한다. 안정적인 식별자(Stable Identity), 결정론적 상태, 구조화된 이벤트(Structured Event), 자원 조정(Resource Coordination), 복구 의미 체계(Recovery Semantics), 감사 기록(Audit Record)을 결합하면 단일 로봇부터 대규모 이기종 플릿까지 신뢰할 수 있는 임무 오케스트레이션(Mission Orchestration)을 구현할 수 있다.

## 06.05 Robot Diagnostics API: Health / Error Code Standard [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 진단 API(Robot Diagnostics API)는 모든 내부 구현 세부사항을 노출하지 않으면서 로봇 건전성(Robot Health)을 관찰하고, 고장(Fault)을 식별하며, 유지보수 의사결정(Maintenance Decision)을 지원하기 위한 표준화된 인터페이스(Standardized Interface)를 제공한다. 일반적인 상태 API(Status API)와 달리 진단(Diagnostics)은 구성요소와 서브시스템(Subsystem)이 정상적으로 동작하는지, 성능 저하(Degradation)가 발생한 원인이 무엇인지, 어떤 기술적 증거(Technical Evidence)를 활용할 수 있는지에 초점을 맞춘다. 여러 종류의 로봇이 하나의 플릿 플랫폼(Fleet Platform)을 공유할수록 일관된 진단 모델(Diagnostic Model)은 더욱 중요해진다.

로봇 건전성(Robot Health)은 단순한 정상 또는 비정상 플래그(Flag) 하나로 축소하기보다 계층적으로 표현해야 한다. 전체 로봇 건전성(Overall Robot Health)은 이동성(Mobility), 위치 추정(Localization), 인지(Perception), 컴퓨팅(Compute), 통신(Communication), 배터리(Battery), 충전(Charging), 조작(Manipulation), 안전(Safety), 주변장치(Peripheral) 서브시스템의 상태에 따라 결정될 수 있다. 각 서브시스템은 다시 개별 구성요소(Component)와 각각의 상태를 포함할 수 있다. 이러한 계층 구조(Hierarchy)를 통해 응용 프로그램은 성능 저하의 발생 위치와 로봇이 제한된 운용을 계속할 수 있는지를 판단할 수 있다.

건전성 상태(Health State)는 소프트웨어 버전과 로봇 플랫폼이 변경되어도 안정적으로 유지되는 명확한 의미 체계(Semantics)를 사용해야 한다. 실용적인 모델에서는 정상(Normal), 정보(Informational), 경고(Warning), 성능 저하(Degraded), 심각(Critical), 사용 불가(Unavailable), 알 수 없음(Unknown) 등의 상태를 구분할 수 있다. 이러한 상태는 단순히 내부 로그 심각도(Log Severity)를 그대로 반영하는 것이 아니라 실제 운용 영향(Operational Impact)을 나타내야 한다. 예를 들어 선택적 카메라(Optional Camera)의 고장은 전체 모바일 로봇을 사용 불가능하게 만들지 않으면서 인지 기능의 성능 저하를 발생시킬 수 있다.

진단(Diagnostics)은 건전성 상태(Health State)와 고장 이벤트(Fault Event)를 분리해야 한다. 건전성은 구성요소 또는 서브시스템의 현재 상태를 설명하지만, 고장 이벤트는 특정 비정상 상황의 발생과 그 생명주기(Lifecycle)를 기록한다. 모터 제어기(Motor Controller)가 성능 저하 상태를 계속 유지하는 동안 문제 해결 과정에서 여러 진단 이벤트(Diagnostic Event)가 발생할 수 있다. 이러한 개념을 분리하면 모니터링 응용 프로그램(Monitoring Application)이 지속적인 상태와 과거 이벤트를 혼동하지 않고 "현재 무엇이 잘못되어 있는가?"와 "이전에 어떤 일이 발생했는가?"를 모두 파악할 수 있다.

표준화된 오류 코드(Standardized Error Code)는 기계 판독형 진단(Machine-Readable Diagnostics)의 핵심 요소다. 사람이 읽을 수 있는 메시지(Human-Readable Message)는 기술자에게 유용하지만 언어, 소프트웨어 버전, 구현 방식에 따라 표현이 변경될 수 있으므로 안정적인 식별자로 사용하기 어렵다. 따라서 각각의 고장에는 정의된 범주(Category), 발생원(Source), 심각도(Severity), 설명(Description), 운용 영향, 권장 대응(Recommended Response)과 연결된 안정적인 오류 코드(Error Code)가 있어야 한다. 이를 통해 외부 시스템은 필터링, 에스컬레이션(Escalation), 복구(Recovery), 유지보수 워크플로(Maintenance Workflow)를 자동화할 수 있다.

오류 코드 네임스페이스(Error-Code Namespace)는 로봇 플랫폼이 확장되더라도 코드 충돌(Collision)이 발생하지 않도록 설계해야 한다. 코드는 이동성, 내비게이션(Navigation), 위치 추정, 인지, 컴퓨팅, 통신, 전원(Power), 충전, 조작, 안전, 인프라(Infrastructure) 등의 도메인(Domain)과 서브시스템에 따라 구성할 수 있다. 추가 필드를 이용하여 고장이 발생한 구성요소와 상세 고장 유형(Failure Type)을 식별할 수도 있다. 새로운 하드웨어와 소프트웨어 모듈(Module)이 추가되어도 기존 코드의 의미를 다시 정의하지 않도록 코드 구조는 확장 가능(Extensible)해야 한다.

오류 코드는 모든 구현별 세부사항(Implementation-Specific Detail)을 하나의 영구적인 숫자 값으로 직접 인코딩해서는 안 된다. 지나치게 경직된 체계(Rigid Scheme)는 하드웨어 세대와 소프트웨어 아키텍처가 발전함에 따라 유지하기 어려워진다. 더욱 강력한 진단 계약(Diagnostic Contract)은 안정적인 상위 수준 고장 식별자(High-Level Fault Identifier)와 구성요소, 인스턴스(Instance), 펌웨어 버전(Firmware Version), 측정값(Measured Value), 임계값(Threshold), 공급업체별 세부정보(Vendor-Specific Detail)를 설명하는 구조화된 메타데이터(Structured Metadata)를 결합한다. 이를 통해 충분한 엔지니어링 정보를 유지하면서 상호운용성(Interoperability)을 확보할 수 있다.

심각도(Severity)와 운용 영향(Operational Impact)도 서로 분리하여 모델링해야 한다. 특정 진단 조건은 기술적으로 심각하더라도 중복성(Redundancy)에 의해 격리될 수 있지만, 비교적 사소해 보이는 고장이라도 필수 기능(Required Capability)을 사용할 수 없게 만들어 임무 수행을 방해할 수 있다. 따라서 API는 고장 심각도와 함께 영향 없음(No Impact), 기능 제한(Reduced Capability), 임무 제한(Mission Restricted), 정지 필요(Stop Required), 정비 필요(Service Required) 등의 로봇 수준 결과를 제공할 수 있다. 이를 통해 플릿 시스템은 보다 지능적인 운용 의사결정을 수행할 수 있다.

진단 기록(Diagnostic Record)은 문제 해결(Troubleshooting)에 도움이 되는 경우 상황별 측정값(Contextual Measurement)을 포함해야 한다. 배터리 과열 고장(Battery Overtemperature Fault)은 측정 온도(Measured Temperature), 임계값, 전압(Voltage), 전류(Current), 운용 모드(Operating Mode), 타임스탬프(Timestamp)가 함께 제공될 때 더욱 유용하다. 위치 추정 실패(Localization Failure)는 신뢰도(Confidence), 센서 가용성(Sensor Availability), 지도 식별자(Map Identifier), 최근 품질 지표(Quality Metric)를 포함할 수 있다. 구조화된 상황 정보(Structured Context)는 별도의 로그 검색(Log Search)에 대한 의존성을 줄이고 자동화된 근본 원인 분석(Root-Cause Analysis)을 가속한다.

타임스탬프(Timestamp)는 분산된 로봇 구성요소 전체에서 고장을 연계하는 데 필수적이다. 진단 데이터는 필요에 따라 조건이 감지된 시점, 활성화된 시점, 보고된 시점, 해제된 시점을 구분해야 한다. 동기화된 시계(Synchronized Clock)를 사용하면 모터 고장, 통신 중단, 위치 추정 성능 저하, 임무 실패를 인과적 순서(Causal Order)에 따라 재구성할 수 있다. 시퀀스 번호(Sequence Number)를 추가하면 누락되거나 순서가 뒤바뀐 진단 이벤트를 식별하는 데 도움이 된다.

고장 생명주기 관리(Fault Lifecycle Management)는 필요한 경우 감지됨(Detected), 활성(Active), 확인됨(Acknowledged), 복구 중(Recovering), 해제됨(Cleared), 래치됨(Latched) 등의 상태 전이를 명확하게 표현해야 한다. 유지보수 시스템이 반복되는 이상 상태를 파악해야 할 수 있으므로 해제된 고장도 단순히 이력에서 사라져서는 안 된다. 래치된 고장(Latched Fault)은 원래의 물리적 문제가 사라진 이후에도 검사 또는 권한이 부여된 리셋(Authorized Reset)을 요구할 수 있다. API는 현재 활성 고장(Active Fault)과 과거 진단 기록(Historical Diagnostic Record)을 명확하게 구분해야 한다.

확인(Acknowledgment)은 고장 해결(Fault Resolution)과 혼동해서는 안 된다. 운영자가 경보(Alarm)를 확인했다는 것은 해당 조건을 인지했거나 처리를 위해 접수했다는 의미일 뿐 근본적인 문제가 사라졌음을 증명하지 않는다. 마찬가지로 오류 코드를 리셋(Reset)하는 과정에서 엔지니어링 분석에 필요한 증거를 삭제해서는 안 된다. 진단 API는 대시보드(Dashboard)와 자동화 시스템이 확인된 고장을 정상 상태로 잘못 분류하지 않도록 이러한 차이를 유지해야 한다.

복구 정보(Recovery Information)를 제공하면 진단 데이터를 실제 대응이 가능한 정보로 만들 수 있다. 고장 정의에는 자동 복구(Automatic Recovery)가 허용되는지, 재시도(Retry)가 적절한지, 서브시스템 재시작(Subsystem Restart)이 가능한지, 또는 사람의 유지보수(Human Maintenance)가 필요한지를 나타낼 수 있다. 그러나 진단 API가 임의의 클라이언트에게 제한 없는 복구 동작을 허용해서는 안 된다. 읽기 전용 진단(Read-Only Diagnostics), 확인, 리셋, 보정(Calibration), 재시작(Restart), 서비스 기능(Service Function)은 인가(Authorization)와 운용 위험에 따라 분리되어야 한다.

건전성 집계(Health Aggregation)에는 명확한 규칙이 필요하다. 선택적 장치(Optional Device)나 중복 서브시스템이 고장 나더라도 운용을 계속할 수 있기 때문에 전체 로봇 건전성을 단순히 모든 구성요소에서 보고된 가장 높은 심각도와 동일하게 설정해서는 안 된다. 집계 과정에서는 기능 의존성(Capability Dependency), 중복성, 임무 요구사항(Mission Requirement), 안전 관련성(Safety Relevance)을 고려해야 한다. 결과적으로 로봇 수준 건전성은 플랫폼이 완전히 사용 가능(Fully Available), 성능이 저하되었지만 사용 가능(Degraded but Usable), 제한됨(Restricted), 또는 운용 불가(Unavailable) 상태인지를 표현할 수 있다.

진단은 스냅샷(Snapshot)과 이벤트 기반 피드(Event-Driven Feed)를 모두 지원해야 한다. 스냅샷을 이용하면 클라이언트가 현재 로봇 건전성, 활성 고장, 서브시스템 상태, 관련 측정값을 조회할 수 있다. 웹소켓(WebSocket), gRPC, MQTT 등의 스트리밍 메커니즘(Streaming Mechanism)은 고장 발생(Fault-Raised), 고장 갱신(Fault-Updated), 고장 확인(Fault-Acknowledged), 복구 시작(Recovery-Started), 고장 해제(Fault-Cleared) 이벤트를 배포할 수 있다. 두 방식을 결합하면 초기 동기화(Initial Synchronization), 지속적인 모니터링, 재연결(Reconnection), 과거 이력 재구성을 지원할 수 있다.

플릿 규모 진단(Fleet-Scale Diagnostics)은 이기종 로봇(Heterogeneous Robot) 전체에서 일관된 의미 체계를 필요로 한다. 자율이동로봇(AMR), 매니퓰레이터(Manipulator), 사족보행 로봇(Quadruped), 무인항공기(UAV)는 매우 다른 하드웨어를 사용할 수 있지만, 플릿 응용 프로그램은 전원 성능 저하, 통신 손실, 위치 추정 문제, 컴퓨팅 과부하(Compute Overload), 안전 제한(Safety Restriction) 등의 공통 개념을 이해할 수 있어야 한다. 플랫폼별 확장(Platform-Specific Extension)은 상세 정보를 제공하고, 공통 건전성 범주(Common Health Category)와 오류 의미 체계(Error Semantics)는 플랫폼 간 상호운용성을 유지한다.

진단 API는 관측 가능성 시스템(Observability System)과도 통합되어야 한다. 메트릭(Metric)은 온도 상승이나 통신 지연 증가와 같은 추세를 보여주고, 로그(Log)는 상세한 소프트웨어 상황을 제공하며, 트레이스(Trace)는 분산 서비스 간 상호작용을 연결하고, 진단 이벤트는 운용상 중요한 조건을 식별한다. 상관관계 식별자(Correlation Identifier)를 이용하여 이러한 정보원을 임무(Mission)와 명령(Command)에 연결하면 엔지니어는 플릿 수준의 경보에서 시작하여 고장의 원인을 설명하는 상세한 기술적 증거까지 추적할 수 있다.

과거 진단 데이터(Historical Diagnostic Data)는 즉각적인 문제 해결뿐 아니라 예지 정비(Predictive Maintenance)와 신뢰성 엔지니어링(Reliability Engineering)에도 활용할 수 있다. 반복적인 모터 과전류(Motor Overcurrent), 배터리 온도 이상(Battery Temperature Excursion), 센서 데이터 손실(Sensor Dropout), 컴퓨팅 스로틀링(Compute Throttling), 충전 실패(Charging Failure)는 완전한 고장이 발생하기 전에 성능 저하의 징후를 나타낼 수 있다. 표준화된 진단 기록을 사용하면 대규모 플릿에서 통계 분석(Statistical Analysis), 이상 탐지(Anomaly Detection), 잔여 수명 추정(Remaining-Life Estimation), 유지보수 일정 계획(Maintenance Scheduling)을 수행할 수 있다.

진단 인터페이스(Diagnostic Interface)는 하드웨어 구성, 소프트웨어 버전, 네트워크 상태, 고장, 내부 시스템 구조와 같은 정보를 노출할 수 있기 때문에 보안(Security)이 특히 중요하다. 인증(Authentication)과 역할 기반 인가(Role-Based Authorization)를 통해 상세한 엔지니어링 정보에 접근할 수 있는 사용자와 서비스를 제한해야 한다. 고장 리셋, 서브시스템 재시작, 보정, 유지보수 모드 활성화(Maintenance-Mode Activation)와 같은 민감한 작업은 일반적인 건전성 조회보다 더 높은 권한, 감사 로깅(Audit Logging), 검증(Validation)을 요구해야 한다.

견고한 로봇 진단 API(Robust Robot Diagnostics API)는 궁극적으로 구성요소, 소프트웨어 계층(Software Layer), 플릿 시스템 전체에서 로봇 건전성을 표현하는 공통 언어(Common Language)를 구축한다. 계층적 건전성 모델(Hierarchical Health Model)은 현재 상태를 설명하고, 표준화된 오류 코드는 비정상 상황을 식별하며, 구조화된 상황 정보는 기술적 증거를 제공하고, 생명주기 이벤트(Lifecycle Event)는 운용 이력을 보존한다. 일관된 심각도, 영향, 복구, 보안, 관측 가능성 의미 체계를 함께 적용하면 개별 로봇부터 이기종 플릿까지 신뢰할 수 있는 유지보수 체계를 구축할 수 있다.

## 06.06 API Backward Compatibility: Deprecation Policy

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

API 하위 호환성(API Backward Compatibility)은 기존 클라이언트(Client), 로봇 소프트웨어(Robot Software), 플릿 서비스(Fleet Service), 운용 워크플로(Operational Workflow)를 예기치 않게 중단시키지 않으면서 인터페이스(Interface)를 발전시킬 수 있는 능력이다. 로봇 시스템에서는 배치된 로봇이 수년간 운용되는 동안 클라우드 플랫폼(Cloud Platform), 플릿 관리자(Fleet Manager), 모바일 응용 프로그램(Mobile Application), 온보드 소프트웨어(Onboard Software)가 서로 다른 속도로 발전할 수 있기 때문에 특히 중요하다. 따라서 호환성(Compatibility)은 일시적인 개발 편의가 아니라 아키텍처 계약(Architectural Contract)으로 관리해야 한다.

하위 호환 API(Backward-Compatible API)는 이전 계약(Contract)을 기준으로 개발된 클라이언트가 더 새로운 호환 서버(Compatible Server)와 통신하더라도 정상적으로 계속 동작할 수 있도록 한다. 호환성을 유지한다고 해서 모든 내부 구현(Internal Implementation)을 변경하지 않아야 하는 것은 아니다. 서버는 기존 클라이언트가 의존하는 외부 관찰 가능 계약(Externally Observable Contract)의 의미와 동작을 유지하는 한 알고리즘(Algorithm), 미들웨어(Middleware), 데이터베이스(Database), 하드웨어 통합(Hardware Integration)을 변경할 수 있다.

API 계약(API Contract)은 엔드포인트 이름(Endpoint Name)이나 메시지 스키마(Message Schema)만을 의미하지 않는다. 필드 의미 체계(Field Semantics), 단위(Unit), 좌표계(Coordinate Frame), 기본값(Default Value), 검증 규칙(Validation Rule), 순서 가정(Ordering Assumption), 타이밍 동작(Timing Behavior), 오류 응답(Error Response), 명령 생명주기 의미 체계(Command Lifecycle Semantics), 인가 요구사항(Authorization Requirement)도 포함한다. 필드 이름을 유지한 채 단위를 미터에서 밀리미터로 변경하는 것은 해당 필드를 제거하는 것보다 더 심각한 문제를 일으킬 수 있다. 따라서 호환성 검토(Compatibility Review)는 문법뿐 아니라 동작까지 확인해야 한다.

추가형 진화(Additive Evolution)는 일반적으로 가장 안전한 접근 방식이다. 기존 동작이 계속 유효하다면 새로운 선택적 필드(Optional Field), 엔드포인트, 이벤트 유형(Event Type), 기능(Capability), 질의 매개변수(Query Parameter)를 기존 클라이언트에 영향을 주지 않고 추가할 수 있다. 직렬화 기술(Serialization Technology)이 허용하는 경우 클라이언트는 알 수 없는 필드(Unknown Field)를 허용하도록 설계해야 한다. 서버도 선택적 정보가 누락된 경우와 잘못된 정보가 입력된 경우를 구분하여 이전 클라이언트가 새로운 기능을 알지 못한다는 이유만으로 요청을 거부하지 않아야 한다.

기존 필드를 제거하거나 이름을 변경하는 것은 컴파일된 응용 프로그램(Compiled Application), 스크립트(Script), 대시보드(Dashboard), 통합 시스템(Integration)이 여전히 해당 필드를 참조할 수 있으므로 일반적으로 호환성을 깨뜨리는 변경(Breaking Change)에 해당한다. 필수 필드(Required Field)를 선택적 필드로 변경하는 것도 무해해 보이지만 다운스트림 소프트웨어(Downstream Software)의 가정을 변경할 수 있으며, 선택적 필드를 필수 필드로 변경하면 기존 요청이 즉시 거부될 수 있다. 따라서 스키마 진화 규칙(Schema Evolution Rule)은 추가, 제거, 자료형 변경(Type Change), 필수 조건 변경(Requirement Change), 의미 변경(Semantic Change)을 호환성 영향에 따라 명확하게 분류해야 한다.

열거형 값(Enumeration Value)은 새로운 값을 추가할 경우 기존 값의 집합이 완전하다고 가정하는 클라이언트를 중단시킬 수 있으므로 특별한 주의가 필요하다. 로봇 상태(Robot State), 임무 상태(Mission State), 고장 범주(Fault Category), 기능 유형(Capability Type)은 플랫폼이 발전하면서 자주 확장된다. 클라이언트는 익숙하지 않은 값이 나타났을 때 실패하는 대신 알 수 없음(Unknown) 또는 지원하지 않음(Unsupported)을 처리할 수 있는 경로를 포함해야 한다. 서버는 기존 열거형 값의 의미가 이미 운용 로직(Operational Logic)에 사용되고 있을 수 있으므로 이를 조용히 재정의해서는 안 된다.

버전 관리(Versioning)는 하위 호환성을 유지할 수 없는 변경을 도입하기 위한 통제된 메커니즘(Controlled Mechanism)을 제공한다. API 기술에 따라 버전은 URL 경로(URL Path), 헤더(Header), 미디어 유형(Media Type), 서비스 정의(Service Definition), 패키지 네임스페이스(Package Namespace), 프로토콜 스키마(Protocol Schema) 등에 표현할 수 있다. 중요한 것은 버전을 어떤 형식으로 표시하는가가 아니라 어떤 계약이 사용되고 있는지를 식별하고 지원 생명주기(Supported Lifetime) 동안 예측 가능한 동작을 유지할 수 있는가이다.

주 버전(Major Version)과 부 버전(Minor Version) 개념을 사용하면 호환성에 대한 기대를 효과적으로 전달할 수 있다. 부 버전 변경은 기존 동작을 유지하면서 호환 가능한 기능을 추가할 수 있고, 주 버전 변경은 클라이언트 마이그레이션(Client Migration)이 필요한 계약 변경을 의도적으로 포함할 수 있다. 패치 수준 업데이트(Patch-Level Update)는 일반적으로 공개 의미 체계(Public Semantics)를 재정의하지 않고 구현 결함(Implementation Defect)을 수정해야 한다. 조직은 버전 번호가 단순한 릴리스 라벨(Release Label)이 아니라 운용상 기대 수준을 전달할 수 있도록 이러한 의미를 문서화해야 한다.

로봇 API(Robot API)는 불필요한 버전 파편화(Version Fragmentation)를 피해야 한다. 다수의 활성 API 버전을 유지하면 특히 이기종 플릿(Heterogeneous Fleet) 환경에서 테스트(Testing), 문서화(Documentation), 보안(Security), 배포(Deployment), 지원(Support) 비용이 증가한다. 더 나은 전략은 소수의 명확하게 정의된 계약을 유지하면서 가능한 경우 추가형 방식으로 발전시키는 것이다. 주 버전은 내부 소프트웨어 구성요소가 변경될 때마다 도입하는 것이 아니라 아키텍처적 이점(Architectural Benefit)이 마이그레이션 비용(Migration Cost)을 정당화할 때 도입해야 한다.

사용 중단 예정(Deprecation)은 API 기능이 일정 기간 계속 사용 가능하지만 새로운 개발에서는 더 이상 사용해서는 안 된다는 것을 알리는 통제된 과정(Controlled Process)이다. 사용 중단 예정으로 지정된 엔드포인트, 필드, 이벤트, 명령 또는 동작은 공지된 전환 기간(Transition Period) 동안 문서화된 계약에 따라 계속 동작해야 한다. 따라서 사용 중단 예정과 제거(Removal)는 서로 다르다. 즉각적인 제거는 아직 업그레이드되지 않은 클라이언트의 마이그레이션 문제(Migration Problem)를 실제 운용 장애(Operational Failure)로 바꿀 수 있다.

사용 중단 공지(Deprecation Notice)는 무엇이 사용 중단 예정인지, 변경 이유가 무엇인지, 어떤 대체 기능(Replacement)을 사용해야 하는지, 사용 중단 기간이 언제 시작되는지, 지원 종료(End of Support)가 언제 예정되어 있는지를 명확하게 식별해야 한다. 마이그레이션 지침(Migration Guidance)은 필요한 경우 매개변수(Parameter), 응답(Response), 상태(State), 이벤트, 오류 처리(Error Handling)의 차이를 설명해야 한다. 단순히 "이 API는 사용 중단 예정입니다"라는 경고만 제공해서는 충분하지 않으며, 클라이언트 개발자가 대체 인터페이스로 이동할 수 있는 결정론적인 전환 경로(Deterministic Migration Path)를 제공해야 한다.

사용 중단 기간(Deprecation Period)은 로봇 시스템의 실제 배포 환경(Deployment Reality)을 반영해야 한다. 웹 서비스(Web Service)는 몇 분 만에 업데이트할 수 있지만 공장, 병원, 캠퍼스 또는 원격 현장에 배치된 수백 대의 로봇을 업그레이드하려면 단계적 검증(Staged Validation)과 물리적 접근(Physical Access)이 필요할 수 있다. 안전 인증(Safety Certification), 고객 승인(Customer Acceptance), 유지보수 시간대(Maintenance Window), 네트워크 제한(Network Limitation)도 배포를 지연시킬 수 있다. 따라서 사용 중단 일정은 단순한 소프트웨어 릴리스 주기(Software Release Cadence)가 아니라 현실적인 마이그레이션 능력(Migration Capability)을 기준으로 설정해야 한다.

사용 중단 정보(Deprecation Information)는 여러 채널(Channel)을 통해 확인할 수 있어야 한다. API 문서(API Documentation), 스키마 주석(Schema Annotation), SDK 경고(SDK Warning), 릴리스 노트(Release Note), 개발자 포털(Developer Portal), 런타임 텔레메트리(Runtime Telemetry)를 통해 인터페이스의 지원 종료가 가까워지고 있음을 알릴 수 있다. 런타임 경고(Runtime Warning)는 로그나 통신 네트워크를 과도하게 발생시키지 않도록 제어해야 한다. 기계 판독형 사용 중단 메타데이터(Machine-Readable Deprecation Metadata)를 사용하면 개발 도구와 플릿 관리 시스템이 오래된 의존성(Obsolete Dependency)을 자동으로 식별할 수도 있다.

사용량 텔레메트리(Usage Telemetry)는 어떤 클라이언트, 로봇, 서비스 또는 소프트웨어 버전이 여전히 이전 인터페이스에 의존하고 있는지를 보여줌으로써 사용 중단 의사결정(Deprecation Decision)을 개선할 수 있다. 제거 전에 유지보수 담당자는 사용 중단 예정 엔드포인트가 여전히 트래픽(Traffic)을 받고 있는지, 중요 고객이 계속 의존하고 있는지를 확인할 수 있다. 텔레메트리는 적절한 보안 및 개인정보 보호 통제(Security and Privacy Control)와 함께 수집되어야 하지만, 모든 사용자가 문서의 요청에 따라 마이그레이션했다고 가정하는 것보다 실제 운용 증거(Operational Evidence)를 활용하는 것이 일반적으로 더 신뢰할 수 있다.

이전 계약과 새로운 계약이 공존해야 하는 경우 호환성 계층(Compatibility Layer)을 사용하여 마이그레이션 위험(Migration Risk)을 줄일 수 있다. 어댑터(Adapter)는 레거시 요청(Legacy Request)을 현재 내부 모델(Current Internal Model)로 변환하고 응답을 다시 레거시 표현(Legacy Representation)으로 변환할 수 있다. 이를 통해 오래된 의미 체계를 핵심 로봇 소프트웨어(Core Robot Software)로부터 격리하면서 클라이언트에 추가적인 마이그레이션 시간을 제공할 수 있다. 그러나 임시 어댑터가 제거하기 어려운 영구적인 아키텍처 의존성(Architectural Dependency)이 되지 않도록 호환성 계층을 명시적으로 관리해야 한다.

기능 협상(Capability Negotiation)은 서로 다른 소프트웨어 세대를 실행하는 로봇과 클라이언트가 상호작용할 때 유용하다. 모든 로봇이 최신 인터페이스를 지원한다고 가정하는 대신 클라이언트는 API 버전, 지원 기능(Supported Feature), 명령 유형(Command Type), 이벤트 스키마(Event Schema), 선택적 기능(Optional Capability)을 조회할 수 있다. 이후 응용 프로그램은 호환 가능한 동작을 선택하거나 요청된 기능을 사용할 수 없음을 보고할 수 있다. 이러한 방식은 대규모 이기종 플릿을 롤링 업그레이드(Rolling Upgrade)할 때 특히 유용하다.

계약 테스트(Contract Testing)는 주요 릴리스 직전에만 수행하는 것이 아니라 하위 호환성을 지속적으로 검증해야 한다. 저장된 스키마(Stored Schema), 대표적인 레거시 요청, SDK 테스트, 소비자 주도 계약(Consumer-Driven Contract), 기록된 상호작용 시나리오(Recorded Interaction Scenario)를 새로운 서버 버전에 대해 실행할 수 있다. 테스트에서는 정상적인 작업뿐 아니라 검증 동작, 오류, 상태 전이(State Transition), 이벤트까지 확인해야 한다. 자동화된 호환성 검사(Automated Compatibility Check)를 사용하면 소프트웨어가 실제 로봇이나 고객 통합 시스템에 배포되기 전에 의도하지 않은 호환성 파괴를 발견할 수 있다.

SDK는 개발자가 기반 API를 감싸는 클래스(Class), 메서드(Method), 예외(Exception), 데이터 모델(Data Model)에 의존할 수 있으므로 자체적인 호환성 정책(Compatibility Policy)이 필요하다. 서버가 프로토콜 호환성(Protocol Compatibility)을 유지하더라도 SDK 업데이트가 응용 프로그램 소스 코드(Application Source Code)를 중단시킬 수 있다. 따라서 SDK 릴리스 정책은 API 호환성과 소스 및 바이너리 호환성(Source and Binary Compatibility)을 구분해야 한다. 마이그레이션 문서는 서버 버전, SDK 버전, 지원되는 조합(Supported Combination)을 연계하여 클라이언트가 예측 가능한 방식으로 업그레이드할 수 있도록 해야 한다.

보안 변경(Security Change)으로 인해 호환성을 의도적으로 중단해야 하는 경우도 있다. 취약한 인증 메커니즘(Vulnerable Authentication Mechanism), 안전하지 않은 명령 동작(Unsafe Command Behavior), 노출된 진단 기능(Exposed Diagnostic Operation)을 레거시 클라이언트가 사용한다는 이유만으로 무기한 유지해서는 안 된다. 정책에는 안전 또는 보안 위험이 호환성 유지 비용보다 큰 경우 사용 중단 기간을 단축하거나 즉시 비활성화할 수 있는 긴급 경로(Emergency Path)가 정의되어야 한다. 이러한 예외는 정당한 경우에만 제한적으로 적용하고 문서화, 공지, 감사(Audit)를 수행해야 한다.

제거(Removal)는 정의된 사용 중단 절차가 완료되었거나 승인된 긴급 예외(Emergency Exception)가 적용되는 경우에만 수행해야 한다. 제거 전에 팀은 마이그레이션 문서, 대체 기능의 가용성(Replacement Availability), 사용량 텔레메트리, 호환성 테스트, 고객 의존성(Customer Dependency), 운용 준비 상태(Operational Readiness)를 확인해야 한다. 제거 이후 지원되지 않는 요청(Unsupported Request)에 대해서는 예측할 수 없는 동작 대신 명확하고 안정적인 오류를 반환하여 남아 있는 레거시 클라이언트가 문제의 원인과 필요한 마이그레이션 작업을 식별할 수 있도록 해야 한다.

성숙한 하위 호환성 및 사용 중단 정책(Backward Compatibility and Deprecation Policy)은 궁극적으로 로봇 소프트웨어 생태계(Robot Software Ecosystem) 전체에 예측 가능한 진화(Predictable Evolution)를 제공한다. 안정적인 계약(Stable Contract)은 배치된 시스템을 보호하고, 추가형 변경(Additive Change)은 불필요한 마이그레이션을 줄이며, 명시적 버전 관리(Explicit Versioning)는 피할 수 없는 호환성 파괴 변경을 격리하고, 구조화된 사용 중단 절차(Structured Deprecation)는 전환에 필요한 시간과 지침을 제공한다. 여기에 텔레메트리, 기능 협상, 호환성 테스트, SDK 거버넌스(SDK Governance), 통제된 제거(Controlled Removal)를 결합하면 운용 신뢰성(Operational Reliability)을 희생하지 않으면서 로봇 API를 지속적으로 발전시킬 수 있다.

## 06.07 Robot API SDK Design: Python / C++ Wrappers [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 API SDK(Robot API SDK)는 저수준 프로토콜 동작(Low-Level Protocol Operation)을 개발자가 편리하게 사용할 수 있는 프로그래밍 인터페이스(Programming Interface)로 변환하는 개발자 지향 소프트웨어 계층(Developer-Oriented Software Layer)을 제공한다. 응용 프로그램이 HTTP 요청, gRPC 메시지, 웹소켓 세션(WebSocket Session), MQTT 토픽(Topic)을 직접 구성하도록 요구하는 대신 SDK는 로봇 중심 객체(Robot-Oriented Object)와 메서드(Method)를 제공한다. 파이썬(Python)과 C++ 래퍼(Wrapper)는 동일한 기반 API 계약(API Contract)을 표현하면서 각 프로그래밍 언어의 관례와 특성을 따라야 한다.

SDK 아키텍처(SDK Architecture)는 공개 개발자 인터페이스(Public Developer Interface)를 전송(Transport) 및 직렬화(Serialization) 세부사항과 분리해야 한다. 응용 프로그램 코드는 RobotClient, MissionClient, CommandClient, StatusClient, DiagnosticsClient와 같은 객체를 사용할 수 있으며, 내부 모듈(Internal Module)은 인증(Authentication), 연결 설정(Connection Establishment), 요청 인코딩(Request Encoding), 응답 디코딩(Response Decoding), 재시도(Retry), 프로토콜 오류(Protocol Error)를 관리할 수 있다. 이러한 분리를 통해 전송 기술이 발전하더라도 사용자 응용 프로그램에 큰 변경을 요구하지 않을 수 있다.

공통 API 모델(Common API Model)은 파이썬과 C++ 구현 모두에서 단일 기준(Source of Truth)으로 유지되어야 한다. 로봇 식별자(Robot Identifier), 자세(Pose), 명령(Command), 임무 상태(Mission State), 진단 기록(Diagnostic Record), 오류 코드(Error Code), 기능(Capability), 이벤트 정의(Event Definition)는 언어가 달라도 동일한 의미 체계(Semantics)를 유지해야 한다. 래퍼는 서로 다른 문법(Syntax), 메모리 모델(Memory Model), 동시성 메커니즘(Concurrency Mechanism)을 사용할 수 있지만 특정 언어에서 자연스럽게 보이게 하기 위해 기반 로봇 동작의 의미를 재정의해서는 안 된다.

파이썬 래퍼(Python Wrapper)는 가독성(Readability), 신속한 통합(Rapid Integration), 효율적인 실험(Productive Experimentation)을 중점적으로 지원해야 한다. 파이썬 개발자는 최소한의 상용구 코드(Boilerplate)만으로 로봇에 연결하고 상태를 확인하며 임무를 생성하고 이벤트를 구독하거나 진단을 요청할 수 있어야 한다. 타입 힌트(Type Hint), 데이터 클래스(Dataclass) 또는 구조화 모델(Structured Model), 컨텍스트 관리자(Context Manager), 반복자(Iterator), 명확한 예외(Exception)를 사용하면 로봇 API 계약에 정의된 검증 및 동작 규칙을 유지하면서 인터페이스를 쉽게 이해할 수 있다.

C++ 래퍼(C++ Wrapper)는 결정론적 자원 관리(Deterministic Resource Management)와 성능에 민감한 로봇 환경(Performance-Sensitive Robotics Environment)을 고려하면서 파이썬과 동등한 기능을 제공해야 한다. RAII 기반 연결 객체(RAII-Based Connection Object), 강력한 형식의 식별자(Strongly Typed Identifier), 명시적인 결과 유형(Explicit Result Type), 소유권이 공유되는 경우 스마트 포인터(Smart Pointer), 이동 의미론(Move Semantics)을 사용하면 객체 수명(Lifetime)과 소유권(Ownership)의 모호성을 줄일 수 있다. SDK는 고주파 경로(High-Frequency Path)에서 불필요한 메모리 할당과 복사를 피하면서 공개 인터페이스를 응용 프로그램 개발자가 이해하기 쉽게 유지해야 한다.

동기식(Synchronous) 및 비동기식(Asynchronous) 동작 모델은 명확한 의도를 가지고 설계해야 한다. 단순한 상태 질의(Status Query)는 블로킹 호출(Blocking Call)이 적합할 수 있지만 임무 실행, 명령 완료, 이벤트 구독(Event Subscription), 텔레메트리 스트림(Telemetry Stream)은 본질적으로 비동기 동작이 필요하다. 파이썬은 동기식 편의 메서드와 함께 async 및 await 인터페이스를 제공할 수 있으며, C++는 지원되는 언어 표준과 런타임 아키텍처(Runtime Architecture)에 따라 퓨처(Future), 콜백(Callback), 코루틴(Coroutine), 실행기 기반 메커니즘(Executor-Based Mechanism)을 사용할 수 있다.

명령 래퍼(Command Wrapper)는 단순히 즉각적인 성공 값을 반환하는 대신 명령 식별자(Command Identity)와 생명주기 의미 체계(Lifecycle Semantics)를 유지해야 한다. 예를 들어 내비게이션 명령(Navigation Command)을 전송하는 호출은 명령 식별자와 상태 조회, 완료 대기, 취소 요청, 실패 정보 확인을 위한 메서드를 포함하는 명령 핸들(Command Handle)을 반환할 수 있다. 이를 통해 SDK는 요청 수락(Request Acceptance)과 이에 대응하는 실제 로봇 물리 동작의 완료를 구분하는 중요한 의미를 숨기지 않게 된다.

임무 래퍼(Mission Wrapper)는 임무 생성, 진행 상황 모니터링(Progress Monitoring), 취소(Cancellation), 결과(Result)를 위한 상위 수준 추상화(High-Level Abstraction)를 제공할 수 있다. 임무 핸들(Mission Handle)은 서버에서 사용하는 안정적인 임무 식별자(Stable Mission Identifier)를 유지하면서 현재 상태, 활성 단계(Active Stage), 진행 정보(Progress Information), 이벤트, 최종 결과(Terminal Result)를 제공할 수 있다. 편의 함수(Convenience Function)는 대기 중(Queued), 실행 중(Executing), 취소 중(Cancelling), 실패(Failed), 완료(Completed)와 같은 중요한 상태를 감추지 않으면서 반복적인 응용 프로그램 코드를 줄여야 한다.

상태 및 텔레메트리 래퍼(Status and Telemetry Wrapper)는 지속적으로 변화하는 데이터가 일반적인 요청-응답(Request-Response) 동작과 다르기 때문에 신중하게 처리해야 한다. SDK는 권위 있는 현재 상태(Authoritative Current State)를 얻기 위한 스냅샷 메서드(Snapshot Method)와 지속적인 갱신을 위한 구독 인터페이스(Subscription Interface)를 제공할 수 있다. 언어에 따라 반복자, 콜백, 비동기 생성기(Asynchronous Generator), 옵저버 패턴(Observer Pattern)을 사용하여 스트림(Stream)을 표현할 수 있다. 느린 소비자(Slow Consumer)로 인해 데이터가 제한 없이 누적되지 않도록 버퍼링(Buffering)과 역압력 정책(Backpressure Policy)을 문서화해야 한다.

진단 래퍼(Diagnostic Wrapper)는 구조화된 건전성 및 고장 정보(Structured Health and Fault Information)를 일관된 언어 수준 모델(Language-Level Model)로 변환해야 한다. 응용 프로그램은 공급업체별 페이로드(Vendor-Specific Payload)를 직접 분석하지 않고도 전체 건전성(Overall Health)을 조회하고, 서브시스템 상태를 검사하고, 활성 고장(Active Fault)을 조회하며, 진단 이벤트(Diagnostic Event)를 구독할 수 있어야 한다. 안정적인 오류 코드 객체(Error-Code Object)는 선택적인 플랫폼별 진단 메타데이터(Platform-Specific Diagnostic Metadata)를 유지하면서 범주(Category), 심각도(Severity), 운용 영향(Operational Impact), 상황 정보(Context), 권장 대응(Recommended Response)을 포함할 수 있다.

오류 처리(Error Handling)는 통신 실패(Communication Failure), API 거부(API Rejection), 물리적 실행 실패(Physical Execution Failure)를 구분해야 한다. 네트워크 타임아웃(Network Timeout), 인증 실패(Authentication Failure), 잘못된 매개변수(Invalid Parameter), 지원되지 않는 기능(Unsupported Capability), 거부된 명령(Rejected Command), 실패한 임무(Failed Mission)는 근본적으로 서로 다른 상황이다. 파이썬은 이러한 범주를 문서화된 예외 계층(Exception Hierarchy)에 매핑할 수 있으며, C++는 SDK 설계 정책에 따라 예외, Expected 스타일 결과 객체(Expected-Style Result Object), 상태 객체(Status Object)를 사용할 수 있다. 동일한 조건은 두 언어 모두에서 일관되게 식별할 수 있어야 한다.

재시도(Retry)는 API 의미 체계상 안전한 경우에만 구현해야 한다. SDK는 일시적인 읽기 작업(Transient Read Operation)이나 멱등 요청(Idempotent Request)을 자동으로 재시도할 수 있지만 실행 상태가 불확실한 물리적 명령을 무조건 반복해서는 안 된다. 명령 식별자, 임무 식별자, 멱등성 키(Idempotency Key)는 재시도 로직(Retry Logic) 전체에 전달되어야 한다. 결과가 모호한 경우 SDK는 잠재적으로 중복된 물리 동작을 다시 생성하는 대신 권위 있는 서버 상태(Authoritative Server State)를 조회해야 한다.

연결 관리(Connection Management)는 일반적인 복잡성을 숨기면서도 연결 상태 자체를 보이지 않게 만들어서는 안 된다. SDK 클라이언트는 엔드포인트 설정(Endpoint Configuration), 인증 토큰(Authentication Token), TLS, 연결 유지(Keepalive), 재연결(Reconnection), 연결 풀(Connection Pool)을 내부적으로 관리할 수 있다. 그러나 응용 프로그램은 통신 상태를 계속 관찰할 수 있어야 하며 통신이 불가능해지거나 복구되었을 때 의미 있는 알림(Notification)을 받을 수 있어야 한다. 재연결 과정에서는 명확하게 문서화된 정책에 따라 구독을 복원하고 상태를 동기화해야 한다.

기능 탐색(Capability Discovery)은 응용 프로그램이 이기종 로봇(Heterogeneous Robot) 전체에서 동작할 수 있도록 래퍼 설계에 통합되어야 한다. 클라이언트는 기능을 호출하기 전에 지원되는 API 버전, 임무 유형(Mission Type), 명령, 센서, 페이로드 기능(Payload Function), 선택적 기능(Optional Capability)을 조회할 수 있다. 편의 메서드는 기능 가용성(Capability Availability)을 자동으로 확인할 수 있지만 지원되지 않는 기능은 예측할 수 없는 방식으로 실패하는 대신 명시적인 결과를 반환해야 한다. 이를 통해 공통 응용 프로그램이 자율이동로봇(AMR), 매니퓰레이터(Manipulator), 사족보행 로봇(Quadruped), 무인항공기(UAV), 특수 플랫폼(Specialized Platform)에서 동작할 수 있다.

스레드 안전성(Thread Safety)과 동시성 동작(Concurrency Behavior)은 명확하게 문서화해야 한다. C++ 응용 프로그램은 여러 작업자 스레드(Worker Thread)에서 하나의 클라이언트에 접근할 수 있으며, 파이썬 응용 프로그램은 스레드, 비동기 이벤트 루프(Asynchronous Event Loop), 백그라운드 콜백(Background Callback)을 함께 사용할 수 있다. SDK는 어떤 객체가 스레드 안전(Thread-Safe)한지, 콜백이 내부 실행기(Internal Executor) 또는 사용자 실행기(User Executor) 중 어디에서 실행되는지, 종료(Shutdown)가 활성 요청(Active Request)과 어떻게 상호작용하는지를 정의해야 한다. 정의되지 않은 동시성 가정은 진단하기 매우 어려운 간헐적인 장애(Intermittent Failure)를 발생시킬 수 있다.

기반 API가 프로토콜 버퍼(Protocol Buffers) 또는 OpenAPI와 같은 공식 스키마(Formal Schema)를 사용하는 경우 생성 코드(Generated Code)를 활용하여 파이썬과 C++ SDK 간의 불일치를 줄일 수 있다. 생성된 전송 모델(Transport Model), 직렬화 로직, 기본 서비스 스텁(Service Stub)은 공통 기반을 제공할 수 있다. 그러나 생성 코드만으로는 일반적으로 고품질 로보틱스 SDK(Robotics SDK)를 구성하기 어렵다. 정제된 래퍼 계층(Curated Wrapper Layer)은 생성된 프로토콜 계층 위에서 로봇 도메인 추상화(Robot-Domain Abstraction), 검증, 생명주기 처리, 오류 변환(Error Translation), 문서화, 안전한 기본값(Safe Default)을 제공해야 한다.

SDK 버전 관리(SDK Versioning)는 서버 API 호환성(Server API Compatibility)과 조정되어야 한다. 각 릴리스는 지원되는 API 버전, 최소 서버 요구사항(Minimum Server Requirement), 사용 중단 예정 메서드(Deprecated Method), 마이그레이션 지침(Migration Guidance)을 문서화해야 한다. 새로운 SDK 버전은 가능한 경우 자주 사용되는 클라이언트 객체와 데이터 모델의 소스 호환성(Source Compatibility)을 유지해야 한다. 호환성을 깨는 변경(Breaking Change)을 피할 수 없는 경우에는 기반 로봇 API에 적용되는 것과 동일한 통제된 버전 관리 및 사용 중단 원칙(Deprecation Principle)을 따라야 한다.

테스트(Testing)는 실제 하드웨어와 독립적으로 래퍼 동작을 검증할 수 있어야 한다. 단위 테스트(Unit Test)는 데이터 모델, 오류 변환, 재시도, 직렬화 경계(Serialization Boundary), 생명주기 처리를 검증할 수 있으며, 모의 서버(Mock Server)와 로봇 시뮬레이터(Robot Simulator)를 이용하여 전체 상호작용을 시험할 수 있다. 동일한 계약 시나리오(Contract Scenario)를 파이썬과 C++ 구현에 대해 실행하여 의미적 동등성(Semantic Equivalence)을 확인해야 한다. 이후 실제 로봇과의 통합 테스트(Integration Test)는 전송 타이밍(Transport Timing), 물리적 동작, 배포별 조건(Deployment-Specific Condition)에 집중할 수 있다.

문서화(Documentation)는 단순히 생성된 메서드 목록을 나열하는 것이 아니라 개발자 워크플로(Developer Workflow)를 중심으로 설계해야 한다. 예제에서는 로봇 연결, 기능 확인, 임무 생성 및 취소, 명령 전송, 진행 상황 모니터링, 상태 구독, 고장 처리, 정상적인 종료(Clean Shutdown)를 보여주어야 한다. 동일한 기능에 대한 파이썬과 C++ 예제를 제공하면 언어 차이를 명확하게 보여주면서 두 래퍼가 동일한 로봇 API 의미 체계와 운용 안전 규칙(Operational Safety Rule)을 구현한다는 점을 강화할 수 있다.

잘 설계된 파이썬 및 C++ 로봇 API SDK는 궁극적으로 응용 프로그램 개발자와 분산 로봇 시스템(Distributed Robotic System)의 복잡성 사이에서 안정적인 경계(Stable Boundary) 역할을 한다. 공유 계약(Shared Contract)은 언어 간 일관성을 유지하고, 언어 고유 래퍼(Language-Native Wrapper)는 사용성을 향상시키며, 명확한 생명주기, 동시성, 재시도, 스트리밍, 오류 의미 체계는 물리적 동작을 보호한다. 이러한 아키텍처를 통해 응용 프로그램은 개별 로봇 및 이기종 플릿과의 예측 가능한 상호작용을 유지하면서 독립적으로 발전할 수 있다.

## 06.08 Robot API Simulator Mock Environment [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 API 시뮬레이터(Robot API Simulator)는 응용 프로그램이 실제 시스템에서 사용하는 것과 동일한 API 계약(API Contract)을 통해 가상 로봇(Virtual Robot)과 상호작용할 수 있도록 하는 소프트웨어 환경(Software Environment)을 제공한다. 주요 목적은 모든 물리 현상을 완벽하게 재현하는 것이 아니라 개발과 테스트에 충분한 수준으로 외부에서 관찰 가능한 API 동작(Externally Observable API Behavior)을 재현하는 것이다. 개발자는 실제 하드웨어가 준비되기 전에도 임무를 생성하고, 명령을 전송하고, 상태를 조회하고, 이벤트를 수신하고, 진단 정보를 확인할 수 있다.

시뮬레이터(Simulator)는 실제 운용 로봇 API(Production Robot API)에 정의된 것과 동일한 식별자(Identifier), 메시지 스키마(Message Schema), 생명주기 상태(Lifecycle State), 오류 의미 체계(Error Semantics), 타이밍 규칙(Timing Rule), 기능 모델(Capability Model)을 유지해야 한다. 응용 프로그램은 연결 대상 엔드포인트(Endpoint)를 시뮬레이션 로봇에서 실제 로봇으로 변경하더라도 거의 또는 전혀 수정할 필요가 없어야 한다. 이러한 계약 동등성(Contract Equivalence)을 통해 응용 프로그램 로직(Application Logic)을 하드웨어 가용성과 독립적으로 개발할 수 있으며, 시뮬레이션 전용 인터페이스가 별도의 소프트웨어 의존성(Software Dependency)으로 발전하는 것을 방지할 수 있다.

모의 환경(Mock Environment)은 시뮬레이션과 관련되어 있지만 서로 다른 목적을 수행한다. 모의 객체(Mock)는 일반적으로 결정론적 소프트웨어 테스트(Deterministic Software Test)를 위해 통제된 응답을 반환하는 반면, 시뮬레이터는 변화하는 로봇 상태를 유지하고 시간에 따른 동작을 생성한다. 모의 객체는 개별 클라이언트 메서드(Client Method)와 예외 상황(Exceptional Condition)을 테스트하는 데 유용하며, 상태 기반 시뮬레이터(Stateful Simulator)는 임무, 명령 생명주기(Command Lifecycle), 텔레메트리 스트림(Telemetry Stream), 복구 절차(Recovery Procedure), 여러 서비스 간 상호작용을 테스트하는 데 더욱 적합하다.

시뮬레이션 충실도(Simulation Fidelity)는 테스트 목적에 따라 선택해야 한다. API 수준 시뮬레이션(API-Level Simulation)은 논리적 상태(Logical State)와 대략적인 타이밍만 모델링할 수 있으며, 운동학 시뮬레이터(Kinematic Simulator)는 로봇의 움직임, 자세(Pose), 속도(Velocity), 내비게이션 진행 상황(Navigation Progress)을 재현할 수 있다. 더욱 발전된 환경에서는 물리 엔진(Physics Engine), 센서 시뮬레이션(Sensor Simulation), 지도(Map), 교통(Traffic), 매니퓰레이터(Manipulator), 디지털 트윈(Digital Twin)을 통합할 수 있다. 응용 프로그램이 아키텍처 변경 없이 여러 환경 사이를 이동할 수 있도록 API 계층은 이러한 충실도 수준 전체에서 일관성을 유지해야 한다.

가상 로봇 상태(Virtual Robot State)는 임의의 스크립트 응답(Scripted Response)이 아니라 명시적인 상태 머신(State Machine)에 의해 관리되어야 한다. 로봇은 정의된 규칙에 따라 유휴(Idle), 준비(Ready), 실행 중(Executing), 일시 정지(Paused), 충전 중(Charging), 성능 저하(Degraded), 고장(Faulted), 비상(Emergency), 사용 불가(Unavailable) 상태 사이를 전이할 수 있다. 명령과 임무는 이러한 상태를 기준으로 수락되거나 거부되어야 한다. 이를 통해 개발자는 모든 요청이 항상 성공한다고 가정하지 않고 현실적인 운용 제약(Operational Constraint)에 응용 프로그램이 올바르게 대응하는지 검증할 수 있다.

명령 시뮬레이션(Command Simulation)은 명령 수락(Command Acceptance)과 실행(Execution)을 서로 다른 단계로 재현해야 한다. 클라이언트가 내비게이션 또는 조작 명령(Manipulation Command)을 제출하면 시뮬레이터는 요청을 검증하고, 명령 식별자(Command Identifier)를 할당하며, 대기 중(Queued)과 실행 중(Executing) 상태를 거쳐 진행 상황을 발행하고 최종적으로 완료 또는 실패를 보고할 수 있다. 취소(Cancellation), 타임아웃(Timeout), 멱등성(Idempotency), 중복 요청(Duplicate Request), 자원 충돌(Resource Conflict)도 실제 운용 API에서 기대되는 것과 동일한 의미 체계를 따라야 한다.

임무 시뮬레이션(Mission Simulation)은 즉시 성공 결과를 반환하는 대신 완전한 임무 생명주기(Mission Lifecycle)를 지원해야 한다. 운송 임무(Transport Mission)는 할당(Assignment), 픽업 지점까지의 내비게이션, 적재 확인(Loading Confirmation), 이동, 배송(Delivery), 완료 등의 단계를 포함할 수 있다. 설정 가능한 지연(Configurable Delay)과 실행 결과를 통해 현실적인 동작을 표현할 수 있다. 따라서 응용 프로그램은 실제 로봇이 동일한 시나리오를 반복적으로 수행하지 않아도 진행 상황 시각화(Progress Visualization), 취소, 임무 재할당(Mission Reassignment), 복구, 최종 결과 처리를 테스트할 수 있다.

상태 및 텔레메트리 시뮬레이션(Status and Telemetry Simulation)은 서로 독립적인 임의 값(Random Value)이 아니라 일관된 데이터(Coherent Data)를 생성해야 한다. 자세는 속도에 따라 일관되게 변화해야 하고, 배터리 수준(Battery Level)은 운용 상태에 따라 감소해야 하며, 충전은 에너지 상태(Energy State)를 증가시키고, 임무 상태는 현재 실행 상태와 일치해야 한다. 응용 프로그램의 오류는 여러 상태 변수가 상호작용할 때 자주 나타나기 때문에 필드 간 관계(Relationship Between Fields)가 중요하다. 반복 가능한 테스트(Repeatable Testing)를 위해 일반적으로 통제되지 않은 무작위성보다 결정론적 모델(Deterministic Model)이 적합하다.

이벤트 스트림(Event Stream)은 실제 API에서 정의한 순서(Ordering), 타임스탬프(Timestamp), 시퀀스 번호(Sequence Number), 생명주기 전이(Lifecycle Transition)를 재현해야 한다. 시뮬레이터는 명령 수락(Command Accepted), 임무 시작(Mission Started), 웨이포인트 도달(Waypoint Reached), 충전 시작(Charging Started), 고장 발생(Fault Raised), 복구 시작(Recovery Started), 임무 완료(Mission Completed) 등의 이벤트를 생성할 수 있다. 설정 가능한 네트워크 효과(Network Effect)를 통해 지연(Delay), 중복(Duplication), 순서 변경(Reordering), 일시적인 연결 단절(Temporary Disconnection)을 추가하여 클라이언트가 분산 시스템 동작(Distributed-System Behavior)을 올바르게 처리하는지 테스트할 수도 있다.

고장 주입(Fault Injection)은 실제 하드웨어에서 재현하기 어렵거나 위험한 여러 중요 고장을 안전하게 테스트할 수 있기 때문에 로봇 API 시뮬레이터의 가장 중요한 기능 중 하나이다. 테스트 시나리오는 위치 추정 상실(Localization Loss), 장애물에 의한 경로 차단(Obstacle Blockage), 배터리 부족(Low Battery), 모터 고장(Motor Fault), 센서 고장(Sensor Failure), 통신 중단(Communication Interruption), 컴퓨팅 과부하(Compute Overload), 충전 실패(Charging Failure), 인프라 사용 불가(Unavailable Infrastructure)를 발생시킬 수 있다. 각각의 주입된 조건은 표준화된 건전성 상태(Health State), 오류 코드(Error Code), 이벤트, 운용 결과(Operational Consequence)를 생성해야 한다.

고장 시나리오(Fault Scenario)는 일시적 조건(Transient Condition)과 지속적 조건(Persistent Condition)을 모두 지원해야 한다. 일시적인 통신 단절은 정의된 시간이 지나면 자동으로 복구될 수 있지만, 시뮬레이션된 모터 제어기 고장(Motor Controller Failure)은 유지보수 동작이 수행될 때까지 활성 상태로 유지될 수 있다. 고장은 특정 임무 단계에서 예약하여 발생시키거나 배터리 임계값(Battery Threshold) 또는 위치와 같은 조건에 의해 발생하도록 구성할 수도 있다. 결정론적 트리거링(Deterministic Triggering)을 통해 개발자는 자동화된 회귀 테스트(Automated Regression Testing)에서 고장을 안정적으로 재현할 수 있다.

시간 제어(Time Control)는 자동화된 개발 환경에서 시뮬레이션의 활용성을 크게 향상시킬 수 있다. 환경은 시나리오에 따라 실시간(Real Time), 가속 시간(Accelerated Time), 감속 시간(Slowed Time), 단계 제어 실행(Step-Controlled Execution)으로 동작할 수 있다. 긴 충전 작업이나 장시간 임무는 압축하여 실행할 수 있으며, 경쟁 상태(Race Condition)는 통제된 진행을 통해 세밀하게 분석할 수 있다. 타임스탬프, 타임아웃, 클라이언트 동작을 명확하게 해석할 수 있도록 시뮬레이션 시간(Simulated Time)은 실제 시계 시간(Wall-Clock Time)과 명확하게 구분되어야 한다.

시나리오 설정(Scenario Configuration)은 개발자가 시뮬레이터 코드를 다시 작성하지 않고 환경을 정의할 수 있도록 해야 한다. 하나의 시나리오는 로봇 유형(Robot Type), 초기 자세(Initial Pose), 배터리 수준, 기능(Capability), 지도, 임무, 장애물(Obstacle), 인프라 자원(Infrastructure Resource), 네트워크 특성(Network Characteristic), 예정된 고장(Scheduled Fault)을 정의할 수 있다. 기계 판독형 설정(Machine-Readable Configuration)을 사용하면 시나리오를 버전 관리 시스템(Version Control)에 저장하고 소스 코드와 함께 검토하며 로컬 개발(Local Development), 지속적 통합(Continuous Integration), 릴리스 검증(Release Validation)에서 반복적으로 실행할 수 있다.

다중 로봇 시뮬레이션(Multi-Robot Simulation)은 자원이 공유될 때 많은 API 문제가 발생하기 때문에 플릿 응용 프로그램(Fleet Application)에서 중요하다. 가상 로봇은 임무, 교통 구역(Traffic Zone), 엘리베이터(Elevator), 충전기(Charger), 적재 스테이션(Loading Station), 기타 인프라를 놓고 경쟁할 수 있다. 환경은 대기열(Queueing), 예약 충돌(Reservation Conflict), 우선순위 변경(Priority Change), 재할당(Reassignment)을 재현할 수 있다. 단순화된 플릿 시뮬레이터(Fleet Simulator)만으로도 고객 현장 배포 과정에서 발견하면 많은 비용이 발생할 수 있는 오케스트레이션 결함(Orchestration Defect)을 사전에 발견할 수 있다.

모의 서버(Mock Server)는 완전한 시뮬레이션 세계를 구성하지 않고도 경계 조건(Edge Case)을 쉽게 생성할 수 있도록 해야 한다. 테스트에서는 서버가 인증 실패(Authentication Failure), 잘못된 응답(Malformed Response), 지원되지 않는 기능(Unsupported Capability), 오래된 상태(Stale Status), 속도 제한(Rate Limit), 타임아웃, 특정 HTTP 및 gRPC 오류를 반환하도록 설정할 수 있다. 이를 통해 SDK와 응용 프로그램 개발자는 방어 코드 경로(Defensive Code Path)를 체계적으로 시험할 수 있다. 테스트가 실제 계약과 연결된 상태를 유지하도록 모의 계층(Mock Layer)도 실제 운용 스키마(Production Schema)를 사용해야 한다.

시뮬레이터는 이기종 로봇(Heterogeneous Robot)을 위한 기능 프로파일(Capability Profile)을 지원해야 한다. 자율이동로봇(AMR)은 내비게이션과 도킹(Docking)을 제공할 수 있고, 매니퓰레이터는 관절 또는 작업 기능(Joint or Task Capability)을 제공할 수 있으며, 무인항공기(UAV)는 비행 전용 기능(Flight-Specific Function)을 제공할 수 있다. 응용 프로그램은 실제 운용 환경과 동일한 메커니즘을 통해 이러한 기능을 탐색할 수 있다. 서로 다른 API 버전과 선택적 기능(Optional Feature)도 표현할 수 있으므로 실제 배치된 플릿에 롤링 업그레이드(Rolling Upgrade)를 적용하기 전에 혼합 소프트웨어 세대(Mixed Software Generation)의 호환성을 테스트할 수 있다.

SDK 통합(SDK Integration)은 간단하고 직접적이어야 한다. 파이썬(Python) 및 C++ 클라이언트는 실제 로봇에서 사용하는 것과 동일한 RobotClient, MissionClient, CommandClient, StatusClient, DiagnosticsClient 추상화(Abstraction)를 이용하여 시뮬레이션 엔드포인트에 연결해야 한다. 로컬 개발에서는 인증을 단순화할 수 있지만 실제 운용 환경과 유사한 보안 모드(Production-Like Security Mode)도 테스트할 수 있어야 한다. 이를 통해 단위 테스트(Unit Test)에서 시뮬레이션, 하드웨어 인 더 루프(Hardware-in-the-Loop), 스테이징(Staging), 실제 물리 배포(Physical Deployment)까지 연속적인 개발 경로를 구성할 수 있다.

자동화 인터페이스(Automation Interface)는 지속적 통합 및 지속적 전달(Continuous Integration and Continuous Delivery)에 필수적이다. 테스트는 수동 개입 없이 시뮬레이터를 시작하고, 시나리오를 로드하고, 준비 상태를 기다리고, 클라이언트 상호작용을 실행하고, 결과를 확인한 후 환경을 종료할 수 있어야 한다. 종료 코드(Exit Code), 구조화 로그(Structured Log), 이벤트 기록(Event Record), 테스트 산출물(Test Artifact)은 장애 원인을 진단할 수 있도록 구성해야 한다. 컨테이너화된 시뮬레이터 인스턴스(Containerized Simulator Instance)를 사용하면 병렬 CI 작업(Parallel CI Job)을 위한 격리되고 재현 가능한 환경을 제공할 수 있다.

관측 가능성(Observability)은 개발자가 시나리오가 특정 방식으로 동작한 이유를 이해할 수 있도록 시뮬레이터 내부에 구축되어야 한다. 로그(Log)는 API 요청과 상태 전이(State Transition)를 기록하고, 메트릭(Metric)은 시뮬레이션 타이밍과 부하를 나타내며, 트레이스(Trace)는 클라이언트 호출과 내부 시뮬레이터 처리를 연결할 수 있다. 시나리오 식별자(Scenario Identifier), 로봇 식별자(Robot Identifier), 명령 식별자(Command Identifier), 임무 식별자(Mission Identifier)는 이러한 기록 전체에서 서로 연계되어야 하며, 이를 통해 실패한 자동화 테스트를 더욱 쉽게 재현하고 분석할 수 있다.

시뮬레이터는 또한 자체적인 한계(Limitation)를 명확하게 문서화해야 한다. API 모의 환경(API Mock Environment)에서 성공적으로 실행되었다고 해서 실제 로봇의 위치 추정, 인지(Perception), 제어(Control), 네트워크(Networking), 기계 시스템(Mechanical System)이 올바르게 동작한다는 것을 보장하지는 않는다. 고충실도 시뮬레이션(High-Fidelity Simulation)과 하드웨어 인 더 루프 테스트는 이러한 차이를 일부 줄일 수 있지만 실제 환경 검증(Physical Validation)은 여전히 필요하다. 따라서 시뮬레이터는 실제 환경 테스트를 대체하는 수단이 아니라 더 광범위한 검증 전략(Verification Strategy)을 구성하는 하나의 계층으로 활용해야 한다.

잘 설계된 로봇 API 시뮬레이터 및 모의 환경(Robot API Simulator and Mock Environment)은 궁극적으로 API 설계, 응용 프로그램 개발, 실제 물리 배포 사이에 재현 가능한 연결 구조(Reproducible Bridge)를 구축한다. 계약과 동등한 인터페이스(Contract-Equivalent Interface)는 통합을 검증하고, 상태 기반 시뮬레이션은 생명주기 동작을 시험하며, 모의 객체는 경계 조건을 격리하고, 고장 주입은 복구 로직(Recovery Logic)을 검증한다. 여기에 시나리오 제어, 이기종 로봇 프로파일, CI 자동화(CI Automation), 관측 가능성을 결합하면 변경 사항이 실제 로봇에 적용되기 전에 더욱 안전하고 빠르게 로봇 소프트웨어를 발전시킬 수 있다.

## 06.09 Robot API Contract Testing: Consumer-Driven [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

소비자 주도 계약 테스트(Consumer-Driven Contract Testing)는 실제로 로봇 API에 의존하는 응용 프로그램(Application)과 서비스(Service)의 관점에서 로봇 API를 검증한다. 제공자(Provider)가 자체 명세(Specification)를 준수하는지만 테스트하는 대신, 소비자(Consumer)가 자신에게 필요한 요청(Request), 응답(Response), 이벤트(Event), 상태(State), 오류 동작(Error Behavior)을 정의한다. 이러한 기대사항은 로봇 API 서비스, SDK, 플릿 시스템(Fleet System), 지원 구성요소가 변경될 때마다 검증할 수 있는 실행 가능한 계약(Executable Contract)이 된다.

로봇 소프트웨어 생태계(Robot Software Ecosystem)에서 소비자는 플릿 관리자(Fleet Manager), 임무 플래너(Mission Planner), 운영자 대시보드(Operator Dashboard), 모바일 응용 프로그램(Mobile Application), 클라우드 서비스(Cloud Service), 유지보수 도구(Maintenance Tool), 분석 플랫폼(Analytics Platform), 제3자 통합(Third-Party Integration) 등을 포함할 수 있다. 제공자는 로봇 게이트웨이(Robot Gateway), 임무 서비스(Mission Service), 명령 서비스(Command Service), 상태 서비스(Status Service), 진단 서비스(Diagnostics Service)가 될 수 있다. 소비자 주도 테스트는 응용 프로그램 코드 내부에 숨겨진 가정 대신 명시적인 기대사항을 통해 독립적으로 발전하는 이러한 구성요소를 연결한다.

계약(Contract)은 내부 구현 세부사항(Internal Implementation Detail)이 아니라 외부에서 관찰 가능한 상호작용(Observable Interaction)을 설명해야 한다. 소비자는 임무 생성 시 안정적인 임무 식별자(Stable Mission Identifier)가 반환되고, 잘못된 목적지(Invalid Destination)에 대해 정의된 오류가 발생하며, 취소 시 예측 가능한 생명주기 전이(Lifecycle Transition)가 생성될 것을 요구할 수 있다. 특정 데이터베이스(Database), ROS2 노드 구조(ROS2 Node Structure), 계획 알고리즘(Planning Algorithm), 내부 클래스(Internal Class)가 의도적으로 공개 인터페이스(Public Interface)의 일부가 아닌 경우 이를 계약에서 요구해서는 안 된다.

계약 테스트(Contract Test)는 API 스키마 검증(API Schema Validation)을 대체하는 것이 아니라 보완한다. OpenAPI, 프로토콜 버퍼(Protocol Buffers), JSON 스키마(JSON Schema)는 필드 이름, 자료형(Type), 필수 속성(Required Property), 직렬화 구조(Serialization Structure)를 검증할 수 있지만 많은 호환성 실패(Compatibility Failure)는 동작에서 발생한다. 응답이 문법적으로 유효하더라도 예상하지 못한 상태를 반환하거나 오류 의미 체계(Error Semantics)를 변경하거나 생명주기 이벤트를 누락하거나 타임아웃 동작(Timeout Behavior)을 변경할 수 있다. 따라서 계약 테스트는 구조적 기대사항과 동작적 기대사항(Behavioral Expectation)을 모두 검증한다.

소비자 계약(Consumer Contract)은 일반적으로 대표적인 상호작용 시나리오(Interaction Scenario)를 기반으로 생성된다. 임무 관리 응용 프로그램(Mission-Management Application)은 운송 임무를 생성할 때 사용하는 요청과 필요한 최소 응답 필드를 정의할 수 있다. 모니터링 서비스(Monitoring Service)는 예상되는 상태 및 진단 이벤트(Diagnostic Event)를 정의할 수 있다. 각각의 계약에는 소비자가 실제로 의존하는 동작만 포함해야 한다. 지나치게 광범위한 계약은 제공자의 발전을 불필요하게 제한하고 문제가 없는 변경까지 호환되지 않는 변경으로 판단하게 만든다.

제공자 검증(Provider Verification)은 API 구현(API Implementation) 또는 통제된 제공자 환경(Controlled Provider Environment)에 대해 소비자 계약을 실행한다. 각각의 상호작용에서 제공자는 필요한 초기 상태(Initial State)를 구성하고, 정의된 요청을 수신한 다음, 응답 또는 이벤트가 소비자의 기대사항을 충족하는지 검증한다. 새로운 서버 릴리스(Server Release)가 기존 소비자에게 필요한 동작을 변경하면 배포 전에 검증이 실패하여 잠재적인 통합 호환성 파괴 변경(Breaking Integration Change)을 조기에 확인할 수 있다.

제공자 상태 관리(Provider State Management)는 로보틱스 계약(Robotics Contract)에서 특히 중요하다. 임무 취소를 테스트하려면 시뮬레이션 로봇(Simulated Robot)에 활성 임무(Active Mission)가 존재해야 하며, 배터리 부족으로 인한 요청 거부를 테스트하려면 알려진 배터리 상태(Battery Condition)가 필요하다. 따라서 계약 프레임워크(Contract Framework)는 로봇 상태, 기능(Capability), 고장(Fault), 임무 상태, 자원 가용성(Resource Availability)을 결정론적으로 설정할 수 있어야 한다. 신뢰할 수 있는 제공자 상태는 테스트가 예측할 수 없는 물리적 조건에 의존하는 것을 방지한다.

로봇 명령 계약(Robot Command Contract)은 즉각적인 요청 수락만을 테스트해서는 안 된다. 소비자는 명령 식별자(Command Identifier), 검증 결과(Validation Result), 생명주기 상태(Lifecycle State), 취소 동작(Cancellation Behavior), 멱등성(Idempotency), 타임아웃 의미 체계(Timeout Semantics), 구조화된 실패 정보(Structured Failure Information)에 의존할 수 있다. 계약은 동일한 멱등성 키(Idempotency Key)를 사용하는 중복 요청이 여러 개의 논리적 명령(Logical Command)을 생성하지 않는지를 검증할 수 있다. 이러한 테스트는 API 요청이 실제 물리적 동작을 발생시킬 수 있는 환경에서 중요한 응용 프로그램의 가정을 보호한다.

임무 API 계약(Mission API Contract)은 생성(Creation), 할당(Assignment), 진행 상황(Progress), 취소(Cancellation), 완료(Completion), 실패(Failure)의 의미 체계를 검증할 수 있다. 소비자는 모든 선택적 서버 필드를 요구하지 않으면서 특정 최종 상태(Terminal State) 또는 진행 상황 필드에 의존할 수 있다. 이벤트 중심 계약(Event-Oriented Contract)은 중요한 상태 전이가 임무 식별자, 타임스탬프(Timestamp), 시퀀스 정보(Sequence Information)를 포함하는 정의된 이벤트 유형(Event Type)을 생성하는지도 검증할 수 있다. 이를 통해 제공자가 임무 메타데이터(Mission Metadata)를 확장할 수 있는 유연성을 유지하면서 워크플로 통합(Workflow Integration)을 보호할 수 있다.

상태 및 텔레메트리 계약(Status and Telemetry Contract)은 소비자가 필요로 하는 정보와 데이터 최신성(Freshness) 및 스트리밍(Streaming)의 의미 체계에 집중해야 한다. 대시보드는 배터리 수준(Battery Level), 운용 모드(Operating Mode), 활성 임무, 연결 상태(Connectivity State)를 요구할 수 있으며 다른 서비스는 자세 갱신(Pose Update)과 타임스탬프에 의존할 수 있다. 테스트는 모든 고주파 텔레메트리 샘플(High-Frequency Telemetry Sample)을 검증하려고 하기보다 스냅샷 구조(Snapshot Structure), 이벤트 구독(Event Subscription), 오래된 데이터 표시(Stale-Data Indicator), 재연결 동작(Reconnection Behavior)을 검증할 수 있다.

진단 계약(Diagnostic Contract)은 소프트웨어 릴리스 전체에서 표준화된 건전성 및 오류 의미 체계(Standardized Health and Error Semantics)를 보호할 수 있다. 유지보수 응용 프로그램(Maintenance Application)은 안정적인 오류 코드(Error Code), 심각도(Severity), 운용 영향(Operational Impact), 서브시스템 식별자(Subsystem Identifier), 고장 생명주기 상태(Fault Lifecycle State)에 의존할 수 있다. 제공자 검증을 통해 기존 오류 코드가 조용히 재정의되지 않는지 확인하고, 고장 발생(Fault-Raised) 및 고장 해제(Fault-Cleared) 이벤트가 예상되는 구조를 유지하는지 검증할 수 있다. 이는 여러 로봇 플랫폼이 하나의 유지보수 시스템을 공유하는 경우 특히 유용하다.

소비자 주도 계약은 호환 가능한 API 진화(Compatible API Evolution)를 허용해야 한다. 소비자가 세 개의 응답 필드를 요구한다면 제공자는 일반적으로 검증 실패 없이 네 번째 선택적 필드(Optional Field)를 추가할 수 있어야 한다. 매칭 규칙(Matching Rule)을 사용하면 바이트 단위의 완전한 동일성(Byte-for-Byte Equality)을 요구하는 대신 허용 가능한 값, 자료형, 패턴(Pattern), 범위(Range), 부분집합(Subset)을 정의할 수 있다. 유연한 매칭(Flexible Matching)은 응용 프로그램이 실제로 필요로 하는 의미 체계를 보호하면서 계약 테스트가 지나치게 취약해지는 것을 방지한다.

소비자의 기대사항이 정당하게 달라지는 경우 버전 관리(Versioning)가 중요해진다. 계약에는 API 버전(API Version), 소비자 버전(Consumer Version), 제공자 버전(Provider Version), 관련 기능 가정(Capability Assumption)을 식별할 수 있어야 한다. 마이그레이션(Migration) 기간 동안 제공자는 레거시 소비자(Legacy Consumer)와 현재 소비자(Current Consumer)의 계약을 모두 검증할 수 있다. 이를 통해 로봇, 플릿 서비스, SDK, 고객 응용 프로그램이 일시적으로 서로 다른 소프트웨어 세대(Software Generation)로 동작하는 동안에도 롤링 업그레이드(Rolling Upgrade)가 안전한지를 확인할 수 있다.

계약 저장소 또는 브로커(Contract Repository or Broker)는 여러 개발팀 간 계약을 조정할 수 있다. 소비자는 자신의 기대사항을 게시하고, 제공자는 적용 가능한 계약을 가져오며, 검증 결과(Verification Result)는 어떤 제공자 버전이 어떤 소비자 버전을 만족하는지를 기록한다. 이를 통해 단순히 릴리스 문서(Release Documentation)에 의존하는 것보다 유용한 호환성 맵(Compatibility Map)을 구축할 수 있다. 배포 파이프라인(Deployment Pipeline)은 검증 결과를 이용하여 후보 API 빌드(Candidate API Build)가 알려진 소비자들과 호환되는지를 확인할 수 있다.

지속적 통합(Continuous Integration)은 API 동작에 영향을 미치는 변경이 발생할 때마다 계약 검증을 실행해야 한다. 소비자 변경은 갱신된 계약을 게시할 수 있으며 제공자 변경은 관련된 모든 소비자 기대사항에 대한 검증을 실행할 수 있다. 계약 실패(Contract Failure)는 영향을 받은 상호작용과 예상 동작을 명확하게 식별해야 한다. 이러한 짧은 피드백 루프(Short Feedback Loop)를 통해 실제 로봇이나 고객 환경에 배포된 이후가 아니라 개발 과정에서 통합 문제(Integration Problem)를 발견할 수 있다.

계약 테스트는 부정적 및 예외적 상호작용(Negative and Exceptional Interaction)도 포함해야 한다. 잘못된 매개변수(Invalid Parameter), 지원되지 않는 기능(Unsupported Capability), 인증 실패(Authentication Failure), 자원 충돌(Resource Conflict), 오래된 요청(Stale Request), 중복 명령(Duplicate Command), 타임아웃, 사용 불가능한 인프라(Unavailable Infrastructure), 고장 조건(Fault Condition)은 성공 사례보다 더 중요할 수도 있다. 이러한 상황에 대한 명시적인 계약을 정의하면 소비자가 문서화되지 않은 오류 메시지나 우연한 제공자 동작에 의존하지 않고 결정론적 복구(Deterministic Recovery)를 구현할 수 있다.

비동기 API(Asynchronous API)는 요청-응답 메시지뿐 아니라 이벤트에 대한 계약도 필요하다. 소비자는 명령 수락 이벤트(Command-Accepted Event) 이후 실행 및 완료 이벤트가 발생하는 것에 의존하거나, 안정적인 상관관계 식별자(Correlation Identifier)를 포함하는 고장 이벤트에 의존할 수 있다. 비동기 상호작용(Asynchronous Interaction)을 테스트하려면 결정론적 이벤트 생성(Deterministic Event Generation), 순서 규칙(Ordering Rule), 타이밍 허용 오차(Timing Tolerance), 상관관계(Correlation)가 필요하다. 목적은 비현실적으로 정확한 타이밍에 테스트를 의존시키는 것이 아니라 의미 있는 이벤트 의미 체계를 검증하는 것이다.

모의 객체(Mock)와 시뮬레이터(Simulator)는 소비자 계약을 기반으로 생성하거나 구성하여 응용 프로그램 개발을 가속할 수 있다. 소비자 팀은 실제 로봇 API 구현이 완료되기 전에 모의 제공자(Mock Provider)를 대상으로 테스트할 수 있으며, 이후 제공자는 동일한 계약을 자신의 구현에 대해 검증할 수 있다. 로봇 API 시뮬레이터(Robot API Simulator)는 상태를 유지하고, 임무 생명주기를 실행하고, 텔레메트리와 고장을 생성하면서 클라이언트가 기대하는 계약을 유지함으로써 이러한 접근 방식을 더욱 확장할 수 있다.

계약 테스트는 완전한 종단간 물리 검증(End-to-End Physical Validation)과 분리하여 유지해야 한다. 계약 테스트는 두 소프트웨어 구성요소가 외부에서 관찰 가능한 API 동작에 합의하고 있음을 검증할 수 있지만, 로봇이 요구된 거리 내에서 정지하는지, 정확하게 위치를 추정하는지, 물체를 올바르게 파지하는지, 혼잡한 환경에서 안전하게 주행하는지를 증명할 수는 없다. 따라서 계약 검증(Contract Verification)은 단위 테스트(Unit Testing)와 보다 광범위한 통합 테스트(Integration Testing), 시뮬레이션, 하드웨어 인 더 루프(Hardware-in-the-Loop), 실제 물리 시스템 테스트(Physical System Testing) 사이에 위치해야 한다.

소비자의 수가 증가하면 거버넌스(Governance)가 필요하다. 팀은 계약의 소유자(Owner)가 누구인지, 오래된 소비자 버전을 어떻게 폐기할 것인지, 호환성을 깨는 기대사항(Breaking Expectation)을 어떻게 검토할 것인지, 사용 중단된 API 동작(Deprecated API Behavior)을 어떻게 제거할 것인지를 정의해야 한다. 계약은 무기한 누적되는 것이 아니라 현재 활성화된 의존성(Active Dependency)을 표현해야 한다. 계약 거버넌스를 API 버전 관리 및 사용 중단 정책(Deprecation Policy)과 연계하면 과거의 테스트가 정당한 아키텍처 발전을 방해하는 것을 방지할 수 있다.

성숙한 소비자 주도 계약 테스트 전략(Consumer-Driven Contract Testing Strategy)은 로봇 API 제공자와 소비자 사이에 실행 가능한 호환성 경계(Executable Compatibility Boundary)를 구축한다. 소비자 기대사항은 실제 의존성을 포착하고, 제공자 검증은 호환되지 않는 변경을 탐지하며, 유연한 매칭은 추가형 진화(Additive Evolution)를 허용하고, CI 자동화(CI Automation)는 호환성을 지속적으로 확인한다. 여기에 결정론적 시뮬레이터 상태(Deterministic Simulator State), 버전 거버넌스(Version Governance), 비동기 이벤트 테스트(Asynchronous Event Testing), 실제 물리 검증을 결합하면 통합 위험(Integration Risk)을 크게 낮추면서 로봇 API를 지속적으로 발전시킬 수 있다.

## 06.10 Robot API Design Case: AMR API Standard

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Hills AMR API 표준(Hills AMR API Standard)은 실내 및 실외 환경에서 이기종 자율이동로봇(Heterogeneous Autonomous Mobile Robot)을 제어, 모니터링, 통합하기 위한 통합 소프트웨어 인터페이스(Unified Software Interface)를 정의한다. 목적은 응용 프로그램(Application)을 로봇별 하드웨어, ROS2 노드(Node), 제어기(Controller), 공급업체 구현(Vendor Implementation)으로부터 분리하는 것이다. 플릿 서비스(Fleet Service)는 개별 플랫폼 소프트웨어에 직접 의존하는 대신 안정적인 로봇, 명령, 임무, 상태, 진단, 기능 인터페이스(Capability Interface)를 통해 상호작용한다.

아키텍처(Architecture)는 Hills AMR 플랫폼과 RMS, FMS, 운영자 콘솔(Operator Console), 클라우드 서비스(Cloud Service), 유지보수 도구(Maintenance Tool), 피지컬 AI 시스템(Physical AI System)과 같은 상위 응용 프로그램 사이에 API 추상화 계층(API Abstraction Layer)을 배치한다. 실내 물류 로봇, 의료 로봇, 실외 순찰 AMR, 향후 중대형 플랫폼은 서로 다른 센서와 제어기를 사용할 수 있지만 외부 API는 일관된 운용 개념(Operational Concept)을 제공한다. 플랫폼 어댑터(Platform Adapter)는 이러한 공통 개념을 로봇별 ROS2 인터페이스와 제어 기능으로 변환한다.

각 로봇은 영구적인 로봇 식별자(Persistent Robot Identifier)와 기능 프로파일(Capability Profile)로 표현된다. 프로파일은 지원되는 내비게이션(Navigation), 도킹(Docking), 충전(Charging), 페이로드(Payload), 센싱(Sensing), 통신(Communication), 조작(Manipulation), 임무 기능과 API 및 소프트웨어 버전을 설명한다. 응용 프로그램은 모든 Hills 로봇이 동일한 기능을 제공한다고 가정하지 않고 기능을 조회한다. 이를 통해 하나의 관리 플랫폼이 여러 로봇 세대를 조정하면서 특정 플랫폼에서 지원되지 않는 작업을 안전하게 거부할 수 있다.

명령 API(Command API)는 정지(Stop), 일시 정지(Pause), 재개(Resume), 내비게이션(Navigate), 도킹, 도킹 해제(Undock), 승인된 서브시스템 초기화(Reset), 지원되는 페이로드 기능 활성화와 같이 범위가 명확한 로봇 동작을 표현한다. 각 요청은 고유한 명령 식별자(Command Identifier)를 부여받고 실행 전에 검증을 거친다. 수락(Acceptance)은 요청이 인식되어 처리 대상으로 승인되었음을 의미하며 물리적 동작이 완료되었다는 의미는 아니다. 응용 프로그램은 완료, 실패, 취소, 거부 또는 만료까지 명령 생명주기(Command Lifecycle)를 추적한다.

명령 실행(Command Execution)은 명시적인 원자성(Atomicity), 멱등성(Idempotency), 타임아웃(Timeout), 소유권(Ownership) 규칙을 따른다. 멱등성 키(Idempotency Key)는 불확실한 네트워크 재시도(Network Retry)가 중복 논리 동작을 생성하는 것을 방지하며, 자원 소유권(Resource Ownership)은 서로 충돌하는 클라이언트가 동일한 이동 기능을 동시에 제어하는 것을 방지한다. 요청 타임아웃(Request Timeout)과 물리적 실행 타임아웃(Physical Execution Timeout)은 서로 다른 개념으로 유지된다. 수락 이후 통신이 중단되면 클라이언트는 이동 요청을 무조건 다시 전송하지 않고 권위 있는 명령 상태(Authoritative Command State)를 조회한다.

임무 API(Mission API)는 자재 운송(Material Transport), 의약품 배송(Medicine Delivery), 순찰(Patrol), 점검(Inspection), 충전, 기지 복귀(Return-to-Base)와 같은 운용 목표를 위한 상위 수준 인터페이스를 제공한다. 임무에는 안정적인 식별자, 임무 유형(Mission Type), 대상 로봇 또는 기능 요구사항(Capability Requirement), 우선순위(Priority), 매개변수(Parameter), 선택적 시간 제약(Time Constraint)이 포함된다. 임무 실행은 생성(Created), 검증(Validated), 대기(Queued), 할당(Assigned), 실행(Executing), 취소 진행(Cancelling), 완료(Completed), 실패(Failed), 취소(Cancelled)와 같은 정의된 상태를 거치므로 RMS와 FMS가 작업을 일관되게 조정할 수 있다.

임무 진행 상황(Mission Progress)은 단순히 백분율 값에만 의존하지 않고 의미 있는 운용 단계(Operational Stage)로 표현된다. 운송 임무는 픽업 지점으로 이동, 도착, 적재, 목적지 이동, 하역, 완료 상태를 보고할 수 있다. 실외 점검 임무(Outdoor Inspection Mission)는 경로 구간(Route Segment), 웨이포인트(Waypoint), 점검 작업(Inspection Task), 복귀 진행 상황을 제공할 수 있다. 진행 이벤트(Progress Event)는 임무 식별자와 타임스탬프(Timestamp)를 포함하므로 대시보드, 분석 시스템, 자율 감독 시스템(Autonomous Supervisor)이 로봇을 지속적으로 폴링(Polling)하지 않고도 실행 과정을 재구성할 수 있다.

상태 API(Status API)는 지속적으로 변화하는 정보에 대해 권위 있는 스냅샷(Authoritative Snapshot)과 실시간 피드(Real-Time Feed)를 함께 제공한다. 공통 필드에는 운용 모드(Operating Mode), 자세(Pose), 속도(Velocity), 위치 추정 품질(Localization Quality), 배터리 상태(Battery State), 충전 상태(Charging State), 연결 상태(Connectivity), 활성 명령(Active Command), 활성 임무(Active Mission), 안전 상태(Safety State), 전체 건전성(Overall Health)이 포함된다. 고주파 텔레메트리(High-Frequency Telemetry)는 지속적인 운용 상태와 분리하여 전달할 수 있으므로 모니터링 응용 프로그램이 모든 센서 주기의 데이터를 처리하지 않고도 필요한 정보를 얻을 수 있다.

실시간 통신(Real-Time Communication)은 배포 아키텍처에 따라 웹소켓(WebSocket), gRPC 스트리밍(gRPC Streaming), MQTT를 사용할 수 있으며, REST 또는 gRPC 요청-응답 인터페이스(Request-Response Interface)는 스냅샷과 관리 작업을 제공할 수 있다. 이벤트는 임무 시작, 웨이포인트 도달, 도킹 완료, 충전 시작, 명령 실패, 고장 발생과 같은 의미 있는 상태 전이(State Transition)를 표현한다. 타임스탬프, 시퀀스 번호(Sequence Number), 로봇 식별자, 명령 식별자, 임무 식별자는 분산 서비스(Distributed Service) 전체에서 순서와 상관관계(Correlation)를 제공한다.

진단 API(Diagnostics API)는 Hills 플랫폼 전반에서 공통 건전성 모델(Common Health Model)을 정의한다. 건전성은 이동성(Mobility), 위치 추정(Localization), 인지(Perception), 컴퓨팅(Compute), 통신, 배터리, 충전, 안전, 선택적 페이로드 시스템에 대해 로봇, 서브시스템(Subsystem), 구성요소(Component) 수준에서 평가할 수 있다. 표준화된 오류 코드(Standardized Error Code)는 안정적인 고장 의미를 식별하고, 구조화된 메타데이터(Structured Metadata)는 측정값, 임계값(Threshold), 구성요소 인스턴스(Component Instance), 공급업체별 세부정보(Vendor-Specific Detail)를 제공한다. 이러한 구조는 사람의 문제 해결과 자동화된 유지보수 처리(Automated Maintenance Processing)를 모두 지원한다.

운용 영향(Operational Impact)은 기술적 심각도(Technical Severity)와 분리된다. 선택적 센서의 고장은 제한적인 운용을 허용하면서 인지 성능 저하(Degraded Perception)를 발생시킬 수 있지만, 필수 위치 추정 기능의 상실은 자율주행을 완전히 중단시킬 수 있다. 따라서 진단 기록(Diagnostic Record)은 심각도, 영향을 받는 기능(Affected Capability), 운용 결과(Operational Consequence), 고장 생명주기(Fault Lifecycle), 권장 복구 범주(Recommended Recovery Category)를 나타낸다. 이를 통해 RMS와 유지보수 시스템은 로봇이 임무를 계속 수행할 수 있는지, 정비를 위해 복귀해야 하는지, 즉시 정지해야 하는지를 판단할 수 있다.

안전(Safety)은 일반적인 응용 프로그램 제어보다 하위 계층에 위치하며 독립적으로 유지된다. API는 이동 또는 운용 상태 전이를 요청할 수 있지만 인증된 비상 정지(Certified Emergency Stop), 안전 제어기(Safety Controller), 보호 영역(Protective Field), 충돌 방지(Collision Prevention), 하드웨어 인터록(Hardware Interlock)을 대체하지 않는다. 명령을 수락하기 전에 로봇 게이트웨이(Robot Gateway)는 운용 상태, 권한(Authorization), 기능 가용성(Capability Availability), 자원 소유권, 관련 안전 전제조건(Safety Precondition)을 확인한다. 안전과 관련된 거부는 일반적인 통신 오류 뒤에 숨기지 않고 명시적으로 보고한다.

보안(Security)은 인증된 신원(Authenticated Identity)과 역할 기반 권한 부여(Role-Based Authorization)를 기반으로 한다. 플릿 서비스는 임무를 생성하고 로봇을 모니터링할 수 있으며, 운영자는 승인된 운용 작업을 수행하고, 유지보수 담당자는 보다 상세한 진단 또는 승인된 초기화 기능에 접근할 수 있다. 민감한 작업(Sensitive Action)은 읽기 전용 모니터링(Read-Only Monitoring)과 분리되고 감사 로그(Audit Log)에 기록된다. 전송 암호화(Transport Encryption)와 자격 증명 관리(Credential Management)는 로봇, 엣지(Edge), 온프레미스(On-Premise), 클라우드 환경 사이의 통신을 보호한다.

Hills 표준은 동일한 API 계약으로부터 파생된 파이썬(Python) 및 C++ SDK를 지원해야 한다. RobotClient, CommandClient, MissionClient, StatusClient, DiagnosticsClient 추상화는 동일한 의미 체계를 유지하면서 각 언어에 자연스러운 접근 방식을 제공한다. SDK는 직렬화(Serialization), 인증(Authentication), 연결 처리(Connection Handling), 재시도(Retry), 구독(Subscription), 오류 변환(Error Translation)을 관리하지만 명령 또는 임무 생명주기를 숨기지는 않는다. 따라서 개발자는 전송 프로토콜별 세부사항(Transport-Specific Protocol Detail)을 직접 관리하지 않고도 응용 프로그램을 개발할 수 있다.

시뮬레이션(Simulation)은 별도의 개발 인터페이스가 아니라 API 아키텍처의 일부로 취급된다. 동일한 SDK는 모의 서버(Mock Server), 상태 기반 로봇 API 시뮬레이터(Stateful Robot API Simulator), 하드웨어 인 더 루프 환경(Hardware-in-the-Loop Environment), 실제 AMR에 연결할 수 있다. 시뮬레이션 로봇은 실제 운용 계약(Production Contract)을 사용하여 명령 생명주기, 임무, 상태, 텔레메트리, 진단, 고장 이벤트를 재현한다. 이를 통해 실제 로봇이나 고객 현장을 지속적으로 사용할 수 없는 상황에서도 응용 프로그램 개발과 회귀 테스트(Regression Testing)를 진행할 수 있다.

소비자 주도 계약 테스트(Consumer-Driven Contract Testing)는 로봇 API, RMS/FMS 서비스, SDK, 대시보드, 외부 응용 프로그램 사이의 통합을 보호한다. 소비자는 자신이 의존하는 외부 관찰 가능 상호작용(Observable Interaction)을 정의하고, 제공자 검증(Provider Verification)은 새로운 API 구현이 이러한 기대사항을 계속 충족하는지 확인한다. 스키마 검증(Schema Validation)은 메시지 구조를 확인하며, 동작 계약(Behavioral Contract)은 생명주기 상태, 오류, 이벤트, 멱등성, 취소와 같이 스키마 정의만으로 보호할 수 없는 의미 체계를 검증한다.

하위 호환성(Backward Compatibility)을 통해 배포된 Hills 로봇과 관리 소프트웨어가 서로 다른 속도로 발전할 수 있다. 호환 가능한 변경(Compatible Change)은 기존 의미 체계를 재정의하지 않고 선택적 필드 또는 기능을 추가하는 방식을 우선해야 한다. 호환성을 깨는 변경(Breaking Change)은 명시적인 버전 관리(Versioning), 마이그레이션 지침(Migration Guidance), 통제된 사용 중단(Controlled Deprecation)을 필요로 한다. 기능 협상(Capability Negotiation)을 통해 클라이언트는 각 로봇이 지원하는 기능과 API 버전을 확인할 수 있으며, 이는 하나의 플릿에 여러 하드웨어 세대와 소프트웨어 릴리스가 함께 존재할 때 특히 중요하다.

플릿 규모(Fleet Scale)에서 이 표준은 실내 및 실외 Hills AMR 전체에 공통 운용 언어(Common Operational Language)를 구축한다. RMS 또는 FMS 응용 프로그램은 일관된 계약을 통해 로봇을 탐색하고, 임무를 할당하고, 진행 상황을 모니터링하고, 건전성을 관찰하고, 충전을 관리하고, 고장에 대응할 수 있으며 플랫폼 어댑터가 하드웨어 차이를 처리한다. 이를 통해 중복된 통합 로직(Integration Logic)을 줄이고 새로운 로봇 유형이 추가될 때 모든 응용 프로그램에서 또 다른 공급업체별 제어 경로(Vendor-Specific Control Path)를 구현할 필요가 없어진다.

관측 가능성(Observability)은 서로 연계된 로그(Log), 메트릭(Metric), 트레이스(Trace), 명령, 임무, 진단 이벤트를 통해 API 동작과 로봇 운용을 연결한다. 하나의 임무 식별자는 플릿 요청(Fleet Request)을 명령 실행, 내비게이션 동작, 서브시스템 경고(Subsystem Warning), 최종 결과와 연결할 수 있다. 이러한 추적 가능성(Traceability)은 현장 디버깅(Field Debugging), 성능 분석(Performance Analysis), 신뢰성 엔지니어링(Reliability Engineering), 예지 정비(Predictive Maintenance)를 지원하는 동시에 API 회귀 테스트와 운용 사고 분석(Operational Incident Analysis)을 위한 근거를 제공한다.

Hills AMR API 표준은 궁극적으로 피지컬 AI 응용 프로그램(Physical AI Application)과 이기종 로봇 기계(Heterogeneous Robotic Machine) 사이에 안정적인 디지털 경계(Stable Digital Boundary)를 구축한다. 기능 탐색(Capability Discovery), 명령, 임무, 상태, 이벤트, 진단, 보안, SDK, 시뮬레이션, 테스트, 호환성을 위한 통합 계약(Unified Contract)을 통해 소프트웨어와 하드웨어가 독립적으로 발전할 수 있다. 이러한 아키텍처는 명시적인 제어 의미 체계(Control Semantics), 운용 추적 가능성, 안전 경계(Safety Boundary), 예측 가능한 물리적 동작(Predictable Physical Behavior)을 유지하면서 확장 가능한 플릿 통합(Scalable Fleet Integration)을 지원한다.
