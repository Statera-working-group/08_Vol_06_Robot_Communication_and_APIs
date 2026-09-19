**Volume 06 Robot Communication and APIs**

# 02. gRPC

## 02.01 gRPC Architecture: Protocol Buffers / HTTP2

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC는 분산 소프트웨어 구성요소(distributed software components) 간의 통신을 위해 설계된 고성능 원격 프로시저 호출(Remote Procedure Call, RPC) 프레임워크이다. REST 방식처럼 URL을 통해 자원(resource)을 노출하는 대신, gRPC는 클라이언트(client)가 원격으로 호출하는 서비스 메서드(service method)를 중심으로 통신을 모델링한다. 이러한 접근 방식은 엣지 컴퓨터(edge computer), 플릿 서버(fleet server), AI 서비스(AI service), 클라우드 구성요소(cloud component)가 낮은 지연시간(low latency)으로 구조화된 데이터를 교환해야 하는 로봇 시스템에 특히 적합하다.

gRPC의 아키텍처(architecture)는 클라이언트(client)와 서버(server)가 공유하는 명시적인 서비스 계약(service contract)을 기반으로 한다. 서비스(service)는 호출 가능한 연산(operation)을 정의하고, 메시지(message)는 해당 연산을 통해 교환되는 정보를 정의한다. 따라서 애플리케이션(application) 관점에서 원격 연산(remote operation)은 로컬 함수(local function)를 호출하는 것과 유사하게 보일 수 있지만, 네트워크 전송(network transport), 직렬화(serialization), 연결 관리(connection management), 장애(failure), 분산 실행(distributed execution)은 여전히 중요한 아키텍처 고려사항이다.

일반적으로 프로토버프(Protobuf)라고 불리는 프로토콜 버퍼(Protocol Buffers)는 gRPC에서 주로 사용하는 인터페이스 정의 언어(Interface Definition Language, IDL)와 바이너리 직렬화(binary serialization) 메커니즘을 제공한다. 개발자는 .proto 정의를 사용하여 서비스(service), 메서드(method), 요청 메시지(request message), 응답 메시지(response message)를 기술한다. 이러한 정의는 언어 중립적인 계약(language-neutral contract)을 형성하며, 이를 기반으로 C++, Python, Java, Go, C# 등 다양한 로봇 소프트웨어 환경에서 사용하는 언어의 클라이언트 및 서버 코드를 생성할 수 있다.

프로토버프 메시지(Protobuf message)는 안정적인 숫자형 필드 번호(field number)로 식별되는 강타입 필드(strongly typed field)로 구성된다. 일반적인 JSON처럼 텍스트 형태의 필드 이름을 반복적으로 전송하는 대신, 바이너리 표현(binary representation)은 이러한 식별자와 효율적인 기본 자료형 표현을 사용하여 데이터를 압축된 형태로 인코딩한다. 이는 로봇이 상태(state), 위치추정 결과(localization result), 진단 정보(diagnostics), 임무 정보(mission information), 인지 메타데이터(perception metadata), AI 추론 결과(AI inference result)를 지속적으로 교환할 때 페이로드 크기(payload size)와 직렬화 오버헤드(serialization overhead)를 줄이는 데 유용하다.

스키마 진화(schema evolution)는 프로토콜 버퍼(Protocol Buffers)의 또 다른 중요한 특성이다. 기존 필드 번호(field number)는 안정적으로 유지해야 하며, 일반적으로 모든 구성요소를 동시에 업그레이드하지 않고도 새로운 필드를 추가할 수 있다. 이전 소프트웨어는 알 수 없는 필드(unknown field)의 의미를 반드시 이해하지 않아도 이를 처리할 수 있다. 개발자가 호환성 규칙(compatibility rule)을 준수하고 기존에 할당된 필드 번호를 부적절하게 재사용하지 않는다면 로봇 플릿(robot fleet), 엣지 컴퓨터(edge computer), 서버(server), 클라우드 서비스(cloud service)를 점진적으로 배포하고 업그레이드할 수 있다.

일반적인 gRPC의 전송 기반(transport foundation)은 HTTP/2이다. HTTP/2는 하나의 연결(connection)을 유지하면서 여러 개의 독립적인 스트림(stream)을 동시에 동작시킬 수 있다. 따라서 각각의 트랜잭션(transaction)에 별도의 연결을 생성하는 대신 여러 RPC 연산이 하나의 TCP 연결을 공유할 수 있다. 이러한 다중화 모델(multiplexing model)은 로봇이 명령 요청(command request), 상태 정보(status information), 진단 정보(diagnostics), 구성 데이터(configuration data), 임무 진행 상태(mission progress) 및 기타 서비스 트래픽(service traffic)을 동시에 통신할 때 유용하다.

HTTP/2는 gRPC 프로그래밍 모델(programming model) 아래에서 바이너리 프레이밍(binary framing)과 스트림 수준 구성(stream-level organization)을 제공한다. 요청(request)과 응답(response)은 프레임(frame)으로 나뉘며, 여러 스트림 사이에서 서로 교차하여 전송되더라도 논리적으로 독립성을 유지한다. 헤더 압축(header compression)은 반복되는 메타데이터 오버헤드(metadata overhead)를 줄이고, 지속 연결(persistent connection)은 불필요한 연결 설정을 방지한다. gRPC는 이러한 전송 기능을 활용하여 애플리케이션 개발자가 HTTP/2 프레임을 직접 조작하지 않고도 효율적인 통신 기반을 사용할 수 있도록 한다.

gRPC 실행 모델(execution model)은 일반적인 요청-응답(request-response) 통신 이상의 방식을 지원한다. 단항 RPC(unary RPC)는 하나의 요청에 하나의 응답을 반환하며, 서버 스트리밍(server streaming)은 하나의 요청으로부터 연속적인 응답을 생성한다. 클라이언트 스트리밍(client streaming)은 여러 클라이언트 메시지를 하나의 응답으로 처리할 수 있으며, 양방향 스트리밍(bidirectional streaming)은 양쪽 종단점(endpoint)이 서로 독립적으로 메시지 시퀀스(message sequence)를 교환할 수 있도록 한다. 이러한 방식은 다양한 로봇 통신 요구사항을 하나의 통합된 기반에서 지원한다.

로봇 아키텍처(robotics architecture)에서 단항 RPC(unary RPC)는 구성 조회(configuration query), 명령 제출(command submission), 상태 점검(health check), 입력과 출력의 범위가 명확한 AI 추론 요청(AI inference request)을 처리할 수 있다. 서버 스트리밍(server streaming)은 지속적으로 변화하는 로봇 상태(robot status)나 임무 진행 상태(mission progress)를 전달할 수 있으며, 클라이언트 스트리밍(client streaming)은 연속적인 측정값이나 누적 관측 데이터(observation data)를 전송하는 데 사용할 수 있다. 양방향 스트리밍(bidirectional streaming)은 명령, 승인(acknowledgement), 상태 전이(state transition), 운영 피드백(operational feedback)을 하나의 지속적인 세션(session)에서 교환해야 하는 상호작용형 협업을 지원할 수 있다.

생성된 스텁(generated stub)은 gRPC 프로그래밍 모델의 핵심 요소이다. 클라이언트 측(client side)에서 생성된 코드는 명확하게 정의된 API를 통해 서비스 메서드(service method)를 제공하고 애플리케이션 객체(application object)를 직렬화된 프로토버프 메시지(Protobuf message)로 변환한다. 서버는 메시지를 수신하여 해당 데이터 구조(data structure)를 복원하고 구현된 서비스 로직(service logic)을 호출한 후 응답을 직렬화한다. 이러한 생성형 인터페이스(generated interface)는 명시적인 기계 판독형 계약(machine-readable contract)을 유지하면서 반복적인 네트워크 프로그래밍 코드를 줄여준다.

전체 아키텍처는 여러 계층(layer)이 협력하는 구조로 이해할 수 있다. 애플리케이션 로직(application logic)은 생성된 클라이언트 또는 서버 인터페이스와 상호작용하고, 프로토버프(Protobuf)는 서비스 계약과 구조화된 메시지 인코딩을 정의한다. gRPC 런타임(runtime)은 RPC 의미체계(semantics), 메타데이터(metadata), 데드라인(deadline), 스트리밍(streaming), 상태 처리(status handling)를 관리하며, HTTP/2는 다중화된 전송(multiplexed transport)을 제공하고 TCP와 IP는 기본 네트워크 전달(network delivery)을 담당한다. 배포 요구사항에 따라 TLS 또는 상호 TLS(mutual TLS, mTLS)를 통해 보안(security)을 추가할 수 있다.

로봇 시스템에서 이러한 계층 분리(layer separation)는 구현이 독립적으로 발전하더라도 통신 계약(communication contract)을 안정적으로 유지할 수 있다는 점에서 중요하다. 엣지 GPU(edge GPU)에서 실행되는 인지 서비스(perception service), 산업용 컴퓨터(industrial computer)의 임무 관리자(mission manager), 온프레미스 서버(on-premise server)의 플릿 오케스트레이션 서비스(fleet orchestration service)가 서로 다른 프로그래밍 언어와 배포 환경을 사용하더라도 동일한 프로토버프 정의(Protobuf definition)를 공유할 수 있다. 이 계약은 독립적으로 개발되는 소프트웨어 구성요소 사이의 통제된 경계(controlled boundary)가 된다.

그러나 gRPC는 로컬 함수 호출(local function call)을 투명하게 대체하는 기술이 아니라 분산 통신(distributed communication)으로 다루어야 한다. 원격 호출(remote call)에서는 일반적인 프로세스 내부 호출에서는 발생하지 않는 지연시간(latency), 연결 실패(connection failure), 서비스 비가용성(service unavailability), 부분 실행(partial execution), 혼잡(congestion), 시간초과(timeout)가 발생할 수 있다. 따라서 로봇 소프트웨어는 각 연산의 중요도(criticality)에 따라 적절한 데드라인(deadline), 취소 동작(cancellation behavior), 재시도 정책(retry policy), 멱등성 가정(idempotency assumption), 오류 처리(error handling), 장애 격리(failure containment)를 정의해야 한다.

현대적인 로봇의 통신 아키텍처(communication architecture)에서 gRPC는 효율적인 직렬화(serialization), 강력한 계약(strong contract), 스트리밍 기능(streaming capability)이 필요한 구조화된 기계 간 인터페이스(machine-to-machine interface)에 특히 효과적이다. 프로토콜 버퍼(Protocol Buffers)는 공유 데이터 및 서비스 모델(shared data and service model)을 제공하고, HTTP/2는 다중화된 전송 기반(multiplexed transport foundation)을 제공하며, gRPC 런타임은 이들을 일관된 RPC 추상화(RPC abstraction)로 통합한다. 이러한 메커니즘은 로봇, 엣지 시스템(edge system), 플릿 플랫폼(fleet platform), AI 서비스, 클라우드 인프라(cloud infrastructure) 사이의 확장 가능한 통신을 지원한다.

## 02.02 Protobuf Message Definition and Code Generation [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

일반적으로 프로토버프(Protobuf)라고 줄여 부르는 프로토콜 버퍼(Protocol Buffers)는 분산 구성요소(distributed component) 사이의 구조화된 통신을 정의하기 위해 gRPC에서 사용하는 스키마(schema)와 직렬화(serialization)의 기반을 제공한다. 애플리케이션 코드(application code)에서 교환 데이터를 비공식적으로 정의하는 대신 개발자는 .proto 파일에 명시적인 메시지 정의(message definition)를 작성한다. 이러한 정의는 로봇, 엣지 컴퓨터(edge computer), 플릿 서버(fleet server), AI 서비스(AI service), 클라우드 애플리케이션(cloud application)이 일관되게 사용할 수 있는 공유 계약(shared contract)을 형성한다.

.proto 파일은 통신 애플리케이션을 구현하는 프로그래밍 언어와 독립적으로 메시지 구조(message structure)를 기술한다. 각 메시지는 명시적으로 선언된 데이터 타입(data type)과 고유한 숫자 식별자(numeric identifier)를 갖는 이름이 지정된 필드(field)로 구성된다. 기본 자료형(primitive type)에는 정수, 부동소수점 값, 불리언(Boolean) 값, 문자열, 바이트 시퀀스(byte sequence)가 포함되며, 다른 메시지 타입을 포함하거나 반복 컬렉션(repeated collection)을 정의하여 복잡한 구조도 구성할 수 있다.

각 프로토버프 필드(Protobuf field)에 할당되는 숫자 식별자(numeric identifier)는 바이너리 표현(binary representation)의 핵심 요소이다. 직렬화 과정에서는 텍스트 형태의 필드 이름 대신 필드 번호(field number)가 네트워크상의 정보 식별에 사용된다. 따라서 필드 번호는 지속적인 통신 계약(communication contract)의 일부가 되며 임의로 변경하거나 재사용해서는 안 된다. 여러 버전으로 배포된 로봇 소프트웨어 사이의 호환성을 유지하려면 신중한 필드 번호 관리(field-number management)가 필수적이다.

프로토버프(Protobuf)는 중첩 메시지(nested message), 열거형(enumeration), 반복 필드(repeated field), 맵(map), 선택적 필드(optional field), 다른 위치에서 정의된 메시지 타입에 대한 참조(reference)를 지원한다. 이러한 메커니즘을 통해 개발자는 특정 프로그래밍 언어의 클래스(class)에 의존하지 않고 복잡한 로봇 정보를 표현할 수 있다. 로봇 자세(robot pose), 속도(velocity), 배터리 상태(battery state), 센서 상태(sensor health), 임무 상태(mission status), 내비게이션 명령(navigation command), 진단 이벤트(diagnostic event), AI 추론 결과(AI inference result) 등을 명확한 의미를 갖는 구조화된 메시지로 모델링할 수 있다.

열거형(enumeration)은 하나의 필드가 제한된 상태 집합(controlled set of states) 가운데 하나의 값을 나타낼 때 유용하다. 예를 들어 로봇 동작 모드(robot operating mode)는 임의의 문자열 대신 대기(idle), 주행(navigating), 충전(charging), 일시정지(paused), 고장(fault)과 같은 기호형 열거 값(symbolic enumeration value)으로 표현할 수 있다. 이는 독립적으로 개발된 구성요소 사이의 모호성을 줄이고 생성된 소스 코드에서 해당 값을 프로그래밍 언어 고유의 구조로 사용할 수 있도록 한다.

반복 필드(repeated field)는 동일한 데이터 타입이 0개 이상 존재하는 구조를 표현하며 감지 객체(detected object), 웨이포인트(waypoint), 진단 기록(diagnostic record), 플릿 구성원(fleet member)과 같은 컬렉션에 유용하다. 맵 필드(map field)는 키-값 관계(key-value relationship)를 표현하고, 중첩 메시지 정의(nested message definition)는 논리적으로 관련된 구조를 하나의 그룹으로 구성할 수 있게 한다. 이러한 구조를 통해 프로토버프는 간결한 텔레메트리 메시지(telemetry message)부터 복잡한 임무 및 인지 데이터 모델(perception data model)까지 표현할 수 있다.

서비스 정의(service definition)는 메시지 정의와 동일한 .proto 계약(contract)에 포함할 수 있다. 서비스는 RPC 메서드(method)를 선언하고 각 연산과 연결되는 요청 메시지(request message)와 응답 메시지(response message)의 타입을 지정한다. 예를 들어 로봇 서비스(robot service)는 상태 조회, 임무 제출, 작업 취소, 진단 요청 등을 위한 메서드를 정의할 수 있다. 이러한 계약은 실제 비즈니스 로직(business logic)이나 제어 로직(control logic)을 포함하지 않으면서 인터페이스(interface)를 정의한다.

프로토버프 컴파일러(Protobuf compiler)인 protoc는 .proto 정의를 프로그래밍 언어별 소스 코드(programming-language-specific source code)로 변환한다. 개발자는 필요한 언어와 적절한 플러그인(plugin)을 선택하며, 컴파일러는 메시지를 표현하는 클래스(class) 또는 구조체(structure)를 생성한다. gRPC 워크플로(workflow)에서는 추가로 생성되는 코드가 선언된 RPC 서비스에 대응하는 클라이언트 스텁(client stub)과 서버 인터페이스(server interface)를 제공하여 인터페이스 정의와 애플리케이션 구현을 직접 연결한다.

생성된 메시지 클래스(generated message class)는 개발자가 바이너리 패킷(binary packet)을 직접 인코딩하지 않아도 직렬화(serialization)와 역직렬화(deserialization) 기능을 제공한다. 애플리케이션 소프트웨어는 생성된 메시지 객체(message object)를 만들고 필드에 값을 할당한 다음 통신 계층(communication layer)으로 전달한다. 프로토버프 런타임(Protobuf runtime)은 이를 네트워크 전송 표현(wire representation)으로 직렬화하고, 수신 애플리케이션은 전달받은 바이트로부터 대응하는 생성 객체를 복원한다.

생성된 gRPC 클라이언트 코드(client code)는 일반적으로 애플리케이션이 원격 서비스 메서드를 호출할 수 있는 스텁(stub)을 제공한다. 개발자는 HTTP 메시지나 바이너리 프레임(binary frame)을 직접 구성하는 대신 타입이 정의된 요청 및 응답 객체를 사용한다. 서버 측(server side)에서는 생성된 인터페이스가 애플리케이션 개발자가 실제 로봇, 플릿, AI 또는 클라우드 로직으로 구현해야 하는 메서드를 정의한다. 이러한 분리는 반복적인 통신 코드를 줄이고 인터페이스가 스키마와 일관성을 유지하도록 한다.

코드 생성(code generation)의 중요한 장점 가운데 하나는 언어 간 일관된 통신(cross-language communication)이다. C++로 구현된 로봇 제어기(robot controller)는 동일한 .proto 정의를 사용하여 Python 기반 AI 추론 서비스, Go 기반 플릿 백엔드(fleet backend), Java 기반 기업 애플리케이션과 통신할 수 있다. 각각의 구성요소는 해당 언어에 적합한 생성 타입(generated type)을 사용하지만 직렬화된 네트워크 형식(wire format)은 동일하게 유지되므로 이기종 소프트웨어 스택(heterogeneous software stack)이 하나의 통신 계약을 공유할 수 있다.

스키마 진화(schema evolution)는 프로토버프 인터페이스 설계 초기부터 고려해야 한다. 일반적으로 새로운 필드는 이전에 사용하지 않은 필드 번호를 이용하여 추가할 수 있으며, 이전 버전의 구성요소는 자신이 인식할 수 있는 필드를 계속 처리할 수 있다. 삭제된 필드의 숫자 식별자를 다른 의미로 다시 사용해서는 안 된다. 예약된 필드 번호 및 이름(reserved field number and name)을 사용하면 우발적인 재사용을 방지하고 진화하는 로봇 소프트웨어 릴리스 사이의 장기적인 호환성을 유지할 수 있다.

로봇 플릿(robot fleet)에서는 모든 로봇을 동시에 업그레이드하기 어려우므로 호환성(compatibility)이 특히 중요하다. 일부 로봇은 일정 기간 이전 버전의 소프트웨어를 실행하는 반면 서버나 엣지 서비스(edge service)는 이미 새로운 버전으로 전환되어 있을 수 있다. 신중하게 진화된 프로토버프 스키마(Protobuf schema)는 단계적 배포(staged deployment) 과정에서 서로 다른 버전이 공존할 수 있게 한다. 이를 통해 소프트웨어 업데이트와 관련된 운영 위험을 줄이고 분산 로봇 인프라(distributed robotic infrastructure)의 점진적인 마이그레이션(migration)을 가능하게 한다.

따라서 프로토버프 정의(Protobuf definition)는 일시적인 직렬화 파일이 아니라 관리되는 인터페이스 자산(managed interface asset)으로 취급해야 한다. 변경 사항은 검토되고 버전 관리(version control)되어야 하며, 호환성 테스트(compatibility test)를 거쳐 생성 산출물(generated artifact) 및 의존 서비스와 함께 관리되어야 한다. 대규모 로봇 플랫폼에서는 승인된 스키마가 변경될 때 자동화된 빌드 파이프라인(automated build pipeline)이 언어별 바인딩(language binding)을 다시 생성하고 클라이언트와 서버가 지정된 계약을 기준으로 계속 컴파일되는지 검증할 수 있다.

로봇 시스템에서 효과적인 메시지 설계(message design)를 위해서는 바이너리 효율성뿐만 아니라 의미체계(semantics)도 고려해야 한다. 단위(unit), 좌표 프레임(coordinate frame), 타임스탬프(timestamp), 식별자(identifier), 유효 조건(validity condition), 상태 의미(state meaning)를 명확하게 정의해야 한다. 기술적으로 유효한 메시지라도 좌표 프레임이 지정되지 않았거나 타임스탬프의 기준이 불명확하면 심각한 통합 오류(integration error)를 발생시킬 수 있다. 따라서 스키마는 안정적인 인터페이스 의미를 표현하고 필요한 경우 애플리케이션 문서가 운영상의 해석을 명확하게 정의해야 한다.

프로토버프(Protobuf)와 코드 생성 워크플로(code-generation workflow)는 궁극적으로 분산 로봇 소프트웨어 전반에 공통된 통신 언어를 제공한다. .proto 스키마는 구조화된 메시지와 RPC 인터페이스를 정의하고, protoc는 이러한 정의를 언어별 표현(language-specific representation)으로 변환하며, 생성된 코드는 직렬화와 통신에 필요한 많은 반복 작업을 처리한다. 이러한 계약 우선 접근법(contract-first approach)은 로봇, 엣지(edge), 플릿(fleet), AI 및 클라우드 서비스 사이의 상호운용 가능한 확장형 통신을 위한 기반을 제공한다.

## 02.03 gRPC Unary, Server, Client, Bidirectional Streaming [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC는 클라이언트(client)와 서버(server) 사이에서 요청과 응답이 어떻게 흐르는지를 결정하는 네 가지 기본 통신 패턴(communication pattern), 즉 단항 RPC(unary RPC), 서버 스트리밍(server streaming), 클라이언트 스트리밍(client streaming), 양방향 스트리밍(bidirectional streaming)을 제공한다. 이 모델들은 동일한 프로토콜 버퍼(Protocol Buffers)와 HTTP/2 기반을 공유하지만 서로 다른 상호작용 의미체계(interaction semantics)를 제공한다. 적절한 패턴을 선택하면 로봇 통신 인터페이스를 실제 운영 데이터의 시간 특성, 방향, 데이터 양에 맞게 설계할 수 있다.

단항 RPC(unary RPC)는 가장 단순한 통신 모델이며 일반적인 함수 호출(function call)과 가장 유사하다. 클라이언트는 하나의 요청 메시지(request message)를 전송하고, 서버는 해당 요청을 처리한 다음 정확히 하나의 응답 메시지(response message)를 반환한다. 이후 RPC가 종료된다. 이 모델은 입력과 출력의 범위가 명확한 연산에 적합하며 통신 장애가 발생했을 때 동작을 이해하고 구현하며 테스트, 모니터링, 복구하기가 비교적 쉽다.

로봇 시스템에서 단항 RPC(unary RPC)는 현재 로봇 구성(configuration) 요청, 진단 스냅샷(diagnostic snapshot) 조회, 개별 명령(discrete command) 제출, 임무 정보(mission information) 조회 또는 단일 입력에 대한 AI 추론 서비스(AI inference service) 호출 등에 적합하다. 예를 들어 플릿 관리자(fleet manager)는 임무 요청을 보내고 임무 식별자(mission identifier)를 포함하는 승인 응답을 받을 수 있으며, 이후의 진행 정보는 별도의 통신 메커니즘을 통해 전달할 수 있다.

서버 스트리밍(server streaming)은 하나의 클라이언트 요청으로 시간에 따라 여러 개의 응답 메시지를 생성할 수 있도록 요청-응답 모델(request-response model)을 확장한다. 초기 요청을 받은 후 서버는 RPC 스트림(stream)을 열린 상태로 유지하면서 처리가 완료되거나 클라이언트가 작업을 취소하거나 데드라인(deadline)이 만료되거나 오류로 스트림이 종료될 때까지 일련의 메시지를 전송한다. 따라서 클라이언트는 하나의 완성된 결과를 기다리지 않고 응답을 점진적으로 처리할 수 있다.

이 모델은 로봇이나 서버가 다른 구성요소에서 요청한 정보를 지속적으로 생성할 때 특히 유용하다. 모니터링 애플리케이션(monitoring application)은 로봇 상태 업데이트를 요청하고 새로운 상태 메시지가 생성될 때마다 이를 전달받을 수 있다. 이와 유사한 방식으로 임무 진행 상태(mission progress), 위치추정 업데이트(localization update), 진단 이벤트(diagnostic event), 인지 메타데이터(perception metadata), 지도 처리 결과(map-processing result) 등을 클라이언트의 반복적인 폴링(polling) 요청 없이 전달할 수 있다.

클라이언트 스트리밍(client streaming)은 스트리밍 관계의 방향을 반대로 구성한다. 클라이언트가 RPC를 열고 여러 개의 요청 메시지를 전송하는 동안 서버는 해당 메시지 시퀀스(message sequence)를 수신하고 처리한다. 클라이언트가 데이터 전송을 완료한 후 서버는 일반적으로 하나의 응답을 반환한다. 이 모델은 정보가 클라이언트 측에서 자연스럽게 누적되거나 점진적으로 생성되지만 서버는 전체 시퀀스에 대해 하나의 최종 결과를 생성해야 하는 경우에 적합하다.

로봇은 클라이언트 스트리밍(client streaming)을 사용하여 측정값(measurement), 관측 데이터(observation), 궤적 샘플(trajectory sample), 진단 기록(diagnostic record) 또는 기타 구조화된 데이터의 시퀀스를 엣지(edge) 또는 서버 애플리케이션으로 전송할 수 있다. 하나의 매우 큰 요청을 구성하는 대신 데이터를 스트림을 통해 점진적으로 전송할 수 있다. 서버는 수신 메시지를 순차적으로 처리하고 전송이 완료되면 요약(summary), 승인(acknowledgement), 집계 결과(aggregated result) 또는 최종 처리 상태를 반환할 수 있다.

양방향 스트리밍(bidirectional streaming)은 가장 유연한 gRPC 상호작용 모델을 제공한다. 클라이언트와 서버 모두 메시지 스트림(message stream)을 사용하여 RPC가 유지되는 동안 각각 독립적으로 여러 메시지를 전송할 수 있다. 한 방향으로 전송된 메시지가 반드시 반대 방향의 즉각적인 응답을 요구하지는 않는다. 이를 통해 강하게 정의된 프로토버프 서비스 계약(Protobuf service contract)을 유지하면서 비동기 대화형 통신(asynchronous conversational communication)을 구현할 수 있다.

로봇 시스템에서 양방향 스트리밍(bidirectional streaming)은 로봇과 플릿 제어기(fleet controller), 엣지 지능 서비스(edge intelligence service), 원격 운영 시스템(remote operation system) 사이의 상호작용형 협업을 지원할 수 있다. 명령(command), 승인(acknowledgement), 상태 업데이트(state update), 진행 이벤트(progress event), 제어 정보(control information)가 하나의 지속적인 통신 세션에 공존할 수 있다. 양쪽 종단점(endpoint)은 수신 정보에 반응하면서 애플리케이션 로직과 운영 조건에 따라 자체적인 송신 메시지를 계속 생성할 수 있다.

네 가지 통신 패턴은 gRPC 서비스 정의(service definition)에 직접 선언된다. 단항 메서드(unary method)는 스트림 수정자(stream modifier) 없이 하나의 요청과 하나의 응답 타입을 사용한다. 서버 스트리밍은 응답을 스트림으로 지정하고, 클라이언트 스트리밍은 요청을 스트림으로 지정하며, 양방향 스트리밍은 양쪽 모두를 스트림으로 지정한다. 이후 생성된 클라이언트 및 서버 인터페이스는 선언된 통신 패턴에 적합한 프로그래밍 구조를 제공한다.

HTTP/2는 이러한 스트리밍 모델을 실용적으로 구현할 수 있는 전송 기능(transport capability)을 제공한다. 여러 gRPC 호출은 하나의 지속 연결(persistent connection) 위에서 독립적인 HTTP/2 스트림으로 동시에 동작할 수 있다. 바이너리 프레이밍(binary framing)은 논리적인 스트림을 분리하면서 각각의 프레임이 기본 연결을 공유할 수 있도록 한다. 따라서 로봇은 각각의 논리적 통신 흐름마다 독립적인 TCP 연결을 생성하지 않고도 여러 서비스에 대한 RPC 상호작용을 동시에 유지할 수 있다.

스트리밍(streaming)을 사용한다고 해서 모든 로봇 데이터 경로를 자동으로 장시간 유지되는 gRPC 스트림으로 구성해야 하는 것은 아니다. 단항 RPC(unary RPC)는 생명주기(lifecycle)와 장애 의미체계(failure semantics)가 단순하기 때문에 개별적인 연산에 더 적합한 경우가 많다. 스트리밍은 정보가 자연스럽게 연속적으로 생성되거나 반복적인 폴링이 불필요한 오버헤드를 발생시키거나 양쪽 종단점 사이에 지속적인 상호작용이 필요한 경우에 유용하다. 따라서 인터페이스 설계는 단순히 성능을 이유로 스트리밍을 선택하기보다 통신 의미체계에 따라 결정해야 한다.

흐름 제어(flow control)는 생산자(producer)와 소비자(consumer)가 서로 다른 속도로 동작할 수 있기 때문에 스트리밍 애플리케이션에서 특히 중요하다. 로봇은 원격 서비스가 처리할 수 있는 속도보다 빠르게 텔레메트리(telemetry)나 인지 결과(perception result)를 생성할 수 있으며 네트워크 대역폭도 운영 과정에서 변할 수 있다. gRPC와 HTTP/2는 전송 수준의 흐름 제어 메커니즘을 제공하지만 애플리케이션 설계에서도 버퍼링(buffering), 큐 제한(queue limit), 메시지 주기(message frequency), 데이터 축소(data reduction), 지속적인 과부하 상태에서의 동작을 고려해야 한다.

데드라인(deadline), 취소(cancellation), 오류 처리(error handling) 역시 신중하게 다루어야 한다. 클라이언트는 RPC가 유지될 수 있는 시간을 지정할 수 있으며 어느 한쪽 종단점에서 스트림을 종료해야 하는 상황이 발생할 수 있다. 로봇 애플리케이션은 정상 완료(normal completion), 취소, 네트워크 중단(network interruption), 서버 장애(server failure), 애플리케이션 수준 오류(application-level fault)를 구분해야 한다. 장시간 유지되는 스트림은 통신이 중단되었을 때 재연결(reconnection) 및 상태 복구(state recovery) 전략도 추가로 필요하다.

스트리밍 인터페이스(streaming interface)를 결정론적 실시간 제어 버스(deterministic real-time control bus)와 혼동해서는 안 된다. gRPC는 로봇 컴퓨터, 엣지 서비스, 서버 사이에서 효율적인 저지연 통신(low-latency communication)을 제공할 수 있지만 운영체제, TCP, HTTP/2, 스케줄링(scheduling), 네트워크 상태의 영향을 받는다. 안전 필수 모터 루프(safety-critical motor loop)와 엄격한 시간 제한이 요구되는 하드 실시간 제어(hard real-time control)는 일반적인 네트워크 RPC보다 결정론적 실행을 위해 특별히 설계된 통신 메커니즘이 필요하다.

실제 로봇 아키텍처(robotics architecture)에서는 서비스 요구사항에 따라 네 가지 gRPC 모델을 조합할 수 있다. 단항 호출은 구성 및 개별 명령을 처리하고, 서버 스트림은 상태와 임무 진행 정보를 배포하며, 클라이언트 스트림은 누적 관측 데이터를 업로드하고, 양방향 스트림은 지속적인 협업을 지원할 수 있다. 이러한 조합을 통해 하나의 서비스 프레임워크에서 공통 스키마(schema), 생성 인터페이스(generated interface), 보안 메커니즘(security mechanism), 운영 도구(operational tooling)를 유지하면서 서로 다른 통신 동작을 표현할 수 있다.

따라서 gRPC 스트리밍의 핵심적인 아키텍처 장점은 단순히 데이터를 연속적으로 전송하는 것이 아니라 통신 방향(direction)과 생명주기(lifecycle)를 서비스 계약(service contract)에 명시적으로 표현할 수 있다는 점이다. 단항, 서버 스트리밍, 클라이언트 스트리밍, 양방향 메서드는 서로 다른 로봇 상호작용을 위한 명확한 추상화(abstraction)를 제공한다. 프로토콜 버퍼(Protocol Buffers) 및 HTTP/2와 결합하면 로봇, 엣지 컴퓨터, AI 서비스, 플릿 시스템, 클라우드 인프라 사이의 확장 가능한 통신을 위한 구조화된 기반을 제공한다.

## 02.04 gRPC Interceptors: Auth, Logging, Metrics [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC 인터셉터(gRPC interceptor)는 모든 서비스 메서드(service method)에 공통 로직을 직접 포함하지 않고도 원격 프로시저 호출(Remote Procedure Call, RPC)의 처리 전후에 공통 기능을 실행할 수 있도록 하는 구조화된 메커니즘이다. 인터셉터는 RPC 호출 경로(invocation path) 주변에서 동작하며 통신 활동을 관찰, 수정, 허용, 거부 또는 기록할 수 있다. 분산 로봇 시스템에서는 인증(authentication), 로깅(logging), 메트릭(metrics), 추적(tracing), 정책 적용(policy enforcement)과 같은 횡단 관심사(cross-cutting concern)를 서비스 전반에 일관되게 구현하는 데 특히 유용하다.

인터셉터(interceptor)는 개념적으로 애플리케이션 코드(application code)와 gRPC 런타임(runtime) 사이에 위치하는 미들웨어(middleware)로 이해할 수 있다. 클라이언트 측에서는 외부로 RPC가 전송되기 전이나 결과가 반환된 이후에 실행될 수 있다. 서버 측에서는 애플리케이션 서비스 로직이 실행되기 전에 수신 호출을 검사하고 실행 이후의 정보를 처리할 수 있다. 이를 통해 기본적인 프로토버프 서비스 계약(Protobuf service contract)을 변경하지 않고 RPC 트래픽을 제어하는 재사용 가능한 지점을 제공한다.

클라이언트 인터셉터(client interceptor)는 여러 외부 요청에 공통적인 메타데이터(metadata)나 운영 동작을 적용해야 할 때 유용하다. 모든 호출 지점에서 인증 자격증명(authentication credential), 상관관계 식별자(correlation identifier), 추적 정보(tracing information), 진단 메타데이터(diagnostic metadata)를 수동으로 추가하는 대신 인터셉터가 이를 자동으로 수행할 수 있다. 이를 통해 중복 코드를 줄이고 로봇, 엣지(edge), 플릿(fleet), 클라우드 애플리케이션 전반에 통신 정책을 일관되게 적용할 수 있다.

서버 인터셉터(server interceptor)는 수신되는 RPC 연산을 제어하기 위한 대응 메커니즘을 제공한다. 실제 서비스 구현으로 요청을 전달하기 전에 인터셉터는 메타데이터를 검사하고, 자격증명을 검증하며, 요청 컨텍스트(request context)를 설정하고, 시간 정보를 기록하거나 권한이 없는 연산을 거부할 수 있다. 서비스 실행 후에는 완료 상태(completion status)와 지연시간(latency)을 기록할 수 있다. 이러한 분리를 통해 서비스 메서드는 주로 로봇 또는 애플리케이션의 핵심 기능에 집중할 수 있다.

인증(authentication)은 인터셉터의 가장 중요한 활용 분야 가운데 하나이다. 클라이언트 인터셉터는 액세스 토큰(access token), API 자격증명(API credential) 또는 기타 인증 메타데이터를 외부 RPC에 추가할 수 있다. 서버 인터셉터는 이 정보를 추출하고 검증하여 요청과 연결된 신원(identity)을 확인하고 처리를 계속할 것인지 결정할 수 있다. TLS 또는 상호 TLS(mutual TLS, mTLS)와 같은 전송 보안(transport security)은 이러한 애플리케이션 수준 신원 확인 메커니즘과 함께 사용할 수 있다.

인증(authentication)과 인가(authorization)는 개념적으로 구분해야 한다. 인증은 누가 또는 어떤 시스템이 요청을 보내는지를 확인하는 과정이며, 인가는 인증된 신원이 특정 연산을 수행할 권한이 있는지를 판단하는 과정이다. 신원을 검증한 후 인터셉터 또는 관련 정책 계층(policy layer)은 명령, 임무 기능, 진단 또는 관리 인터페이스에 대한 접근을 허용하기 전에 역할(role), 권한(permission), 로봇 식별자(robot identifier), 서비스 범위(service scope), 운영 정책(operational policy)을 평가할 수 있다.

로봇 플릿(robot fleet)에서는 중앙화된 인가 동작(centralized authorization behavior)을 통해 개별 서비스 구현마다 접근 검사를 다르게 적용하는 위험을 줄일 수 있다. 유지보수 애플리케이션(maintenance application)은 진단 정보를 읽을 수 있지만 내비게이션 명령을 실행할 수 없도록 구성할 수 있으며, 플릿 제어기(fleet controller)는 지정된 로봇 그룹에 임무를 할당하기 위한 별도의 권한을 요구할 수 있다. 인터셉터는 운영 로직이 실행되기 전에 신원과 RPC 메서드 정보를 평가하는 공통 정책 적용 지점(enforcement point)을 제공한다.

로깅(logging)은 분산 로봇 시스템이 여러 프로세스와 컴퓨터 사이에서 상호작용하기 때문에 인터셉터의 또 다른 주요 활용 분야이다. 인터셉터는 RPC 서비스와 메서드, 요청 신원(request identity), 시작 시간, 완료 시간, 상태 코드(status code) 및 기타 관련 메타데이터를 기록할 수 있다. 중앙화된 로깅은 모든 서비스 메서드 내부에서 독립적으로 구현하는 방식보다 일관된 기록을 생성하며 분산 통신 경로에서 발생하는 문제 해결(troubleshooting)을 단순화한다.

로깅 설계(logging design)에서는 전체 요청 및 응답 페이로드(payload)를 무분별하게 기록하지 않아야 한다. 로봇 메시지에는 대용량 센서 데이터, 독점 정보(proprietary information), 자격증명, 토큰 또는 기타 민감한 내용이 포함될 수 있다. 따라서 로깅 정책은 운영 메타데이터와 페이로드 데이터를 구분하고 필요한 경우 필터링(filtering), 마스킹(redaction), 샘플링(sampling), 크기 제한(size limit)을 적용해야 한다. 목적은 과도한 저장 공간, 성능 오버헤드 또는 보안 노출을 발생시키지 않으면서 유용한 관측 가능성(observability)을 확보하는 것이다.

상관관계 식별자(correlation identifier)는 하나의 연산이 여러 분산 서비스를 통과하는 경우 특히 유용하다. 임무 요청은 플릿 애플리케이션에서 임무 서비스(mission service)로 전달된 후 엣지 컴퓨터를 거쳐 최종적으로 로봇 제어 구성요소에 도달할 수 있다. 요청 또는 추적 식별자(trace identifier)를 gRPC 메타데이터를 통해 전파하면 서로 다른 서비스에서 생성된 로그를 동일한 논리적 연산과 연결할 수 있으므로 종단 간 장애 분석(end-to-end fault investigation)이 크게 향상된다.

메트릭 인터셉터(metrics interceptor)는 RPC 활동을 정량적인 운영 측정값으로 변환한다. 일반적인 측정 항목에는 요청 횟수(request count), 성공 및 실패 호출, 응답 지연시간(response latency), 활성 스트림(active stream), 전송 메시지 양(message volume), 상태 코드 분포(status-code distribution) 등이 포함된다. 인터셉터는 공통 경계에서 호출을 관찰하기 때문에 각 비즈니스 기능이 자체적인 측정 로직을 구현하지 않아도 여러 서비스에서 이러한 메트릭을 체계적으로 수집할 수 있다.

지연시간 메트릭(latency metric)은 통신 성능이 임무 응답성(mission responsiveness)과 분산 협업(distributed coordination)에 직접적인 영향을 줄 수 있기 때문에 로봇 시스템에서 특히 중요하다. 평균 지연시간만으로는 간헐적으로 발생하는 느린 호출이 평균값에 가려질 수 있으므로 충분하지 않다. 운영 모니터링에서는 일반적으로 지연시간 분포(latency distribution)와 백분위수(percentile)를 오류율(error rate), 요청량(request volume)과 함께 분석한다. 이를 통해 과부하된 서비스, 네트워크 성능 저하, 자원 경합(resource contention), 비정상적인 통신 동작을 식별할 수 있다.

스트리밍 RPC(streaming RPC)는 단항 호출(unary call)과 다른 관측 가능성 요구사항을 가진다. 단항 RPC는 비교적 명확한 시작 및 완료 경계를 갖지만 서버, 클라이언트, 양방향 스트림은 장시간 활성 상태로 유지되면서 많은 메시지를 교환할 수 있다. 따라서 전체 스트림을 짧은 요청-응답 트랜잭션과 동일하게 처리하기보다 스트림 지속시간(stream duration), 활성 스트림 수, 송수신 메시지 수, 종료 원인(termination reason), 재연결 동작(reconnection behavior) 등을 메트릭으로 추적할 수 있다.

인터셉터는 gRPC 메타데이터를 통해 추적 컨텍스트(trace context)를 생성하거나 전파함으로써 분산 추적(distributed tracing)도 지원할 수 있다. 하나의 연산에 참여하는 각 서비스는 동일한 분산 추적에 시간 및 실행 정보를 추가할 수 있다. 로봇 명령이 플릿, 엣지, AI 및 제어 서비스를 통과할 때 추적 기능은 엔지니어가 어느 구간에서 시간이 소비되고 어디에서 장애가 발생했는지 파악하는 데 도움을 준다. 이러한 추적은 로그와 메트릭을 대체하는 것이 아니라 상호 보완한다.

여러 횡단 기능(cross-cutting function)을 조합하는 경우 인터셉터 실행 순서(interceptor ordering)를 신중하게 설계해야 한다. 인증은 애플리케이션별 로깅보다 먼저 실행되어야 할 수 있으며, 추적 컨텍스트는 인가와 서비스 처리를 모두 포함할 수 있도록 충분히 이른 단계에서 설정되어야 한다. 운영상 필요한 경우 메트릭은 성공한 호출뿐만 아니라 거부된 호출도 기록해야 한다. 명확하게 정의된 인터셉터 체인(interceptor chain)은 이러한 동작을 예측 가능하게 만들고 미들웨어 기능 사이의 숨겨진 의존성을 방지한다.

인터셉터는 이를 통과하는 모든 RPC가 계산 비용의 영향을 받을 수 있으므로 경량으로 유지해야 한다. 비용이 큰 동기식 로깅(synchronous logging), 원격 데이터베이스 접근, 복잡한 정책 평가, 과도한 페이로드 검사는 지연시간을 증가시키고 처리량(throughput)을 감소시킬 수 있다. 따라서 로봇 통신 인프라는 빈번하게 실행되는 인터셉터 로직을 제한적이고 효율적으로 유지하고, 비용이 큰 처리는 가능한 경우 비동기 파이프라인(asynchronous pipeline)이나 전문 관측 시스템(observability system)으로 이동해야 한다.

잘 설계된 gRPC 서비스 아키텍처(service architecture)는 생성된 서비스 인터페이스와 애플리케이션 로직을 둘러싸는 공통 운영 계층(operational layer)으로 인터셉터를 활용한다. 인증(authentication)은 신원을 확인하고, 인가(authorization)는 허용된 동작을 제어하며, 로깅(logging)은 중요한 이벤트를 기록하고, 메트릭(metrics)은 시스템 동작을 정량화하며, 추적(tracing)은 분산된 연산을 연결한다. 이러한 메커니즘은 보안과 관측 가능성을 향상시키면서 로봇, 엣지, 플릿, AI 및 클라우드 서비스 구현이 각각의 핵심 기능에 집중할 수 있도록 한다.

## 02.05 gRPC Error Handling and Status Codes [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC 오류 처리(error handling)는 원격 프로시저 호출(Remote Procedure Call, RPC)이 성공적으로 완료되었는지 또는 실패했는지, 그리고 실패한 이유가 무엇인지를 전달하기 위한 표준화된 메커니즘을 제공한다. 애플리케이션별 오류 문자열에만 의존하는 대신 gRPC는 서로 다른 프로그래밍 언어와 분산 서비스에서 일관되게 해석할 수 있는 공통 상태 모델(status model)을 정의한다. 이는 클라이언트, 서버, 네트워크, 엣지 컴퓨터(edge computer), 플릿 시스템(fleet system), 애플리케이션 로직(application logic) 등 다양한 위치에서 장애가 발생할 수 있는 로봇 시스템에서 중요하다.

완료된 모든 gRPC 연산(operation)은 RPC의 결과를 나타내는 상태(status)를 갖는다. 성공적인 실행은 정상(OK)으로 표현되며, 실패는 미리 정의된 상태 코드(status code)와 선택적인 설명 정보(descriptive information)를 통해 표현된다. 이를 통해 전송 및 서비스 수준의 장애 보고를 일반적인 응답 메시지와 분리할 수 있으며, 생성된 클라이언트 라이브러리(client library)는 언어에 적합한 예외(exception), 상태 객체(status object), 콜백(callback), 반환 메커니즘(return mechanism)을 통해 오류를 제공할 수 있다.

알 수 없음(UNKNOWN) 상태 코드는 정확하게 분류할 수 없는 오류를 나타내며, 내부 오류(INTERNAL)는 서비스 또는 내부 처리 과정에서 발생한 장애를 의미한다. 원인이 명확한 경우 이러한 코드를 더 구체적인 오류 대신 사용해서는 안 된다. 로봇 시스템에서는 의미 있는 오류 분류가 중요하다. 플릿 관리자(fleet manager)가 잘못된 명령과 일시적으로 사용할 수 없는 로봇 또는 원격 서비스 내부의 장애를 구분할 수 있어야 하기 때문이다.

잘못된 인수(INVALID_ARGUMENT)는 현재 시스템 상태와 관계없이 클라이언트가 제공한 인수가 유효하지 않음을 나타낸다. 예를 들어 음수 속도 제한, 잘못된 좌표 값, 지원되지 않는 운영 매개변수 또는 구조적으로 잘못된 임무 요청이 이 상태를 발생시킬 수 있다. 전제조건 실패(FAILED_PRECONDITION)는 요청 자체는 유효할 수 있지만 현재 시스템 상태가 적절하지 않아 실행할 수 없다는 점에서 잘못된 인수와 구별된다.

전제조건 실패(FAILED_PRECONDITION)는 물리적 로봇 동작에서 특히 중요하다. 주행 시작 요청 자체는 구조적으로 올바르지만 위치추정(localization)이 수렴하지 않았거나, 비상 상태가 활성화되어 있거나, 필요한 센서를 사용할 수 없거나, 로봇이 적절한 운영 모드(operating mode)에 진입하지 않은 경우 실행할 수 없다. 잘못된 입력과 충족되지 않은 운영 조건을 구분하면 클라이언트가 요청을 수정해야 하는지 또는 시스템 상태가 변경될 때까지 기다려야 하는지를 판단하는 데 도움이 된다.

찾을 수 없음(NOT_FOUND)은 요청된 엔터티(entity)가 존재하지 않는다는 것을 나타내며, 이미 존재함(ALREADY_EXISTS)은 클라이언트가 생성하려는 엔터티가 이미 존재한다는 것을 의미한다. 이러한 상태는 존재하지 않는 로봇 식별자(robot identifier), 임무 기록(mission record), 지도(map), 구성 프로파일(configuration profile), 플릿 서비스가 관리하는 자원(resource) 등에 사용할 수 있다. 이러한 코드를 일관되게 사용하면 서비스 동작을 쉽게 이해할 수 있고 유사한 상황마다 서로 다른 애플리케이션별 오류 규칙을 만드는 것을 방지할 수 있다.

권한 거부(PERMISSION_DENIED)는 호출자의 신원은 확인되었지만 요청된 연산을 수행할 권한이 없음을 나타낸다. 인증되지 않음(UNAUTHENTICATED)은 유효한 인증 자격증명(authentication credential)이 없거나 이를 통해 호출자의 신원을 확인할 수 없음을 의미한다. 인증 실패(authentication failure)와 인가 실패(authorization failure)는 서로 다른 보안 상태이며 서로 다른 운영 대응이 필요할 수 있으므로 안전한 로봇 API에서 이러한 구분을 유지하는 것이 중요하다.

자원 고갈(RESOURCE_EXHAUSTED)은 필요한 자원의 한계에 도달한 상황을 나타낸다. 서비스가 큐 용량(queue capacity), 메모리 압박(memory pressure), 처리 할당량(processing quota), 연결 제한(connection limit) 또는 기타 제한된 자원 때문에 추가 요청을 처리하지 못할 수 있다. 플릿 환경에서는 이 상태가 영구적인 애플리케이션 장애가 아니라 일시적인 포화 상태(saturation)를 의미할 수 있으므로 클라이언트와 모니터링 시스템이 용량 관련 문제를 식별하는 데 활용할 수 있다.

사용 불가(UNAVAILABLE)는 일반적으로 서비스가 현재 요청을 처리할 수 없지만 이후 다시 사용 가능해질 수 있는 상태를 나타낸다. 서비스 재시작, 일시적인 네트워크 장애, 로드 밸런서 전환(load-balancer transition), 종속 구성요소와의 연결 손실 등에서 발생할 수 있다. 이러한 상태는 일시적일 수 있으므로 클라이언트가 재시도(retry)할 수 있지만, 물리적 동작의 중복 실행, 과도한 트래픽 또는 동기화된 재시도 폭주(retry storm)를 방지하도록 신중하게 설계해야 한다.

데드라인 초과(DEADLINE_EXCEEDED)는 RPC가 허용된 데드라인(deadline) 안에 완료되지 않았음을 의미한다. 서버가 최종적으로 작업을 완료하더라도 너무 늦게 도착한 응답은 운영상 가치가 없을 수 있기 때문에 데드라인은 중요하다. 따라서 로봇 클라이언트는 구성 조회, 임무 요청, AI 추론(AI inference), 진단, 상호작용형 제어 등 모든 작업에 하나의 공통 시간초과(timeout)를 사용하는 대신 각 연산의 의미에 따라 적절한 데드라인을 정의해야 한다.

취소됨(CANCELLED)은 연산이 취소되었음을 의미하며, 일반적으로 클라이언트가 더 이상 결과를 필요로 하지 않거나 관련 작업이 종료되었을 때 발생한다. 취소는 분산 서비스 체인(distributed service chain)을 따라 전파되어 하위 단계의 불필요한 작업이 계속 실행되는 것을 방지할 수 있다. 로봇 시스템에서는 임무가 교체되거나 운영자가 작업을 중단하거나 상위 수준 플래너(planner)가 아직 처리 중인 요청을 무효화했을 때 유용하다.

중단됨(ABORTED)은 일반적으로 동시성(concurrency) 또는 트랜잭션과 유사한 충돌 때문에 연산을 완료할 수 없는 경우와 관련되며, 범위 초과(OUT_OF_RANGE)는 허용된 범위를 벗어난 연산이 시도되었음을 나타낸다. 구현되지 않음(UNIMPLEMENTED)은 해당 서비스가 요청된 연산 또는 기능을 지원하지 않는다는 의미이다. 데이터 손실(DATA_LOSS)은 복구할 수 없는 데이터 손실 또는 손상을 나타내며 일반적인 일시적 통신 장애가 아니라 심각한 상태로 처리해야 한다.

오류 처리(error handling)는 표준화된 gRPC 상태 코드와 의미 있는 애플리케이션 컨텍스트(application context)를 함께 사용해야 한다. 전제조건 실패(FAILED_PRECONDITION)와 같은 상태는 일반적인 실패 범주를 클라이언트에 알려주며, 추가 오류 정보(error detail)는 위치추정을 사용할 수 없거나 안전 상태(safety state)가 이동을 차단하고 있거나 충전을 먼저 완료해야 한다는 구체적인 원인을 설명할 수 있다. 클라이언트가 복구 동작을 자동으로 결정해야 한다면 기계 판독형 세부 정보(machine-readable detail)가 유용하며, 사람이 읽을 수 있는 설명도 진단 과정에서 중요한 역할을 한다.

재시도 동작(retry behavior)은 모든 실패한 RPC에 동일하게 적용하기보다 오류의 의미체계(error semantics)를 기반으로 결정해야 한다. 잘못된 인수(INVALID_ARGUMENT), 권한 거부(PERMISSION_DENIED), 인증되지 않음(UNAUTHENTICATED)은 일반적으로 즉시 반복하기보다 수정 또는 개입이 필요하다. 사용 불가(UNAVAILABLE) 또는 일부 자원 관련 장애는 통제된 재시도가 적절할 수 있다. 지수 백오프(exponential backoff), 지터(jitter), 재시도 제한(retry limit), 데드라인을 사용하면 이미 성능이 저하된 상황에서 반복 요청이 부하를 더욱 증가시키는 것을 방지할 수 있다.

멱등성(idempotency)은 재시도가 물리적 세계를 변화시키는 명령과 관련될 때 매우 중요하다. 클라이언트가 명령을 전송한 후 응답을 받지 못하면 통신이 실패하기 전에 서버가 해당 작업을 실행했는지 알 수 없을 수 있다. 멱등성이 없는 요청을 반복하면 임무 또는 동작이 중복 실행될 수 있다. 명령 식별자(command identifier), 요청 중복 제거(request deduplication), 명시적인 상태 확인(state check), 신중하게 설계된 멱등 API(idempotent API)를 사용하면 이러한 위험을 줄일 수 있다.

스트리밍 RPC(streaming RPC)는 많은 메시지가 이미 교환된 이후에도 오류가 발생할 수 있기 때문에 추가적인 장애 고려사항이 필요하다. 스트림이 종료되면 애플리케이션은 어떤 정보까지 성공적으로 처리되었는지 판단하고 재연결 시 작업을 이어서 수행할지, 처음부터 다시 시작할지 또는 중단할지를 결정해야 한다. 중단된 로봇 텔레메트리(telemetry) 또는 협업 스트림(coordination stream)의 안정적인 복구가 필요하다면 시퀀스 번호(sequence number), 승인(acknowledgement), 체크포인트(checkpoint), 애플리케이션 수준 상태(application-level state)가 필요할 수 있다.

오류 처리는 관측 가능성(observability)과도 통합되어야 한다. 인터셉터(interceptor)는 상태 코드를 기록하고 서비스, 메서드, 로봇 또는 오류 범주별 실패율을 보여주는 메트릭(metrics)으로 집계할 수 있다. 로그(log)는 관련 컨텍스트를 보존하고 분산 추적(distributed tracing)은 플릿, 엣지, AI, 클라우드 서비스 전반에서 장애가 시작된 위치를 식별할 수 있다. 따라서 사용 불가(UNAVAILABLE) 또는 데드라인 초과(DEADLINE_EXCEEDED)가 갑자기 증가하는 현상은 개별적인 애플리케이션 오류가 아니라 운영 상태를 나타내는 신호로 활용할 수 있다.

견고한 로봇 아키텍처(robotics architecture)에서 gRPC 상태 코드(status code)는 분산 구성요소 사이에서 장애를 표현하는 공통 언어 역할을 한다. 표준화된 범주는 장애의 특성을 식별하고, 데드라인은 연산이 유효하게 유지되는 시간을 제한하며, 취소는 불필요한 작업을 방지하고, 재시도 정책은 선택된 일시적 장애를 처리하며, 애플리케이션별 세부 정보는 운영상의 원인을 설명한다. 이러한 메커니즘을 멱등 명령 설계(idempotent command design) 및 관측 가능성과 결합하면 로봇, 엣지, 플릿, AI, 클라우드 서비스 전반에서 예측 가능한 장애 복구(failure recovery)를 구현할 수 있다.

## 02.06 gRPC Load Balancing and Service Discovery [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC 기반 시스템이 단일 클라이언트(client)와 서버(server) 구조에서 분산 배포(distributed deployment) 환경으로 확장되면 확장성(scalability)과 가용성(availability)을 확보하기 위해 동일한 서비스의 여러 인스턴스(instance)가 필요한 경우가 많다. 부하 분산(load balancing)은 이러한 인스턴스 사이에 RPC 트래픽을 분배하고, 서비스 디스커버리(service discovery)는 현재 사용 가능한 인스턴스와 해당 인스턴스의 접근 위치를 결정한다. 이 두 메커니즘을 결합하면 로봇, 엣지(edge), 플릿(fleet), AI, 클라우드 서비스가 하나의 고정된 서버 종단점(endpoint)에 의존하지 않고 동작할 수 있다.

단순한 gRPC 배포 환경에서는 클라이언트에 정적인 서버 주소(static server address)를 설정할 수 있다. 이러한 방식은 소규모 시스템에서는 실용적이지만 서비스가 복제(replication), 재시작, 이동 또는 동적 확장(dynamic scaling)되는 환경에서는 관리하기 어려워진다. 대규모 로봇 플랫폼에서는 클라이언트가 논리적 서비스 식별자(logical service identity)를 하나 이상의 접근 가능한 종단점으로 해석하고, 정상적인 서비스 인스턴스 집합이 변경됨에 따라 지속적으로 적응할 수 있는 메커니즘이 필요하다.

서비스 디스커버리(service discovery)는 서비스 이름(service name)과 활성 네트워크 위치(active network location) 사이의 동적인 매핑을 제공한다. 디스커버리 메커니즘은 DNS, 서비스 레지스트리(service registry), 오케스트레이션 플랫폼(orchestration platform) 또는 기타 이름 지정 시스템(naming system)을 통해 종단점을 얻을 수 있다. 개별 서버 주소를 로봇 애플리케이션에 직접 포함하는 대신 클라이언트는 안정적인 논리적 서비스 이름을 참조하고, 인프라가 해당 서비스를 구현하는 현재 인스턴스의 주소를 결정하도록 할 수 있다.

DNS 기반 디스커버리(DNS-based discovery)는 가장 단순한 접근 방식 가운데 하나이다. 하나의 서비스 이름에 여러 주소를 연결하여 클라이언트 또는 지원 인프라가 여러 백엔드 인스턴스(backend instance)를 검색할 수 있도록 한다. 종단점 변경이 비교적 관리 가능한 환경에서는 효과적이지만 DNS 캐싱(DNS caching), 레코드 갱신 지연(record update delay), 유효시간(time-to-live, TTL) 설정, 장애가 발생했거나 새로 생성된 서비스 인스턴스가 클라이언트에 얼마나 빠르게 반영되는지를 고려해야 한다.

서비스 레지스트리(service registry)는 보다 명시적인 디스커버리 동작을 제공한다. 서비스 인스턴스는 자신의 네트워크 위치와 관련 메타데이터(metadata)를 등록하고, 클라이언트 또는 인프라는 레지스트리를 조회하여 사용 가능한 종단점을 확인한다. 레지스트리를 상태 정보(health information)와 결합하면 장애가 발생한 인스턴스를 사용 가능한 종단점 집합에서 제거할 수 있다. 이러한 모델은 엣지 서비스, 플릿 백엔드(fleet backend), AI 추론 작업자(AI inference worker)가 운영 중 동적으로 생성되거나 제거되는 환경에 유용하다.

컨테이너 오케스트레이션 플랫폼(container orchestration platform)은 서비스 디스커버리를 배포 인프라에 직접 통합할 수 있다. gRPC 서비스가 복제된 컨테이너(container)로 실행되는 경우 오케스트레이션 환경은 논리적인 서비스 식별자를 유지하고 이를 현재 실행 중인 인스턴스에 매핑할 수 있다. 이를 통해 개별 컨테이너가 교체, 재배치(rescheduling), 확장되더라도 애플리케이션 수준의 인터페이스를 안정적으로 유지할 수 있으며, 온프레미스 플릿 서버(on-premise fleet server)와 클라우드 로보틱스 백엔드(cloud robotics backend)에 특히 유용하다.

부하 분산(load balancing)은 여러 서비스 종단점이 검색된 이후 요청을 어떻게 분배할지를 결정한다. 일반적인 접근 방식 가운데 하나는 프록시 기반 부하 분산(proxy-based load balancing)으로, 클라이언트가 중간의 로드 밸런서(load balancer)에 연결하면 로드 밸런서가 적절한 백엔드 서버를 선택한다. 프록시는 라우팅 결정(routing decision)을 중앙화하고 상태 확인(health checking), 트래픽 정책(traffic policy), 보안 통합, 관측 가능성(observability)을 제공할 수 있지만 통신 경로에 추가적인 네트워크 홉(network hop)과 인프라 구성요소가 포함된다.

클라이언트 측 부하 분산(client-side load balancing)은 종단점 선택 기능을 gRPC 클라이언트 또는 이를 지원하는 리졸버(resolver)와 밸런싱 구성요소로 이동시킨다. 클라이언트는 여러 백엔드 주소를 검색하고 설정된 정책에 따라 각 RPC에 적절한 인스턴스를 선택한다. 모든 요청을 중앙화된 애플리케이션 프록시(application proxy)를 통해 전달할 필요가 없어 효율적인 통신이 가능하지만, 효과적인 결정을 위해 클라이언트가 정확한 종단점 및 상태 정보를 제공받아야 한다.

기본적인 선택 전략으로는 우선 선택(pick-first)이 있으며, 클라이언트가 검색된 주소 집합에서 적절한 하나의 종단점을 사용하고 해당 연결을 사용할 수 있는 동안 계속 유지하는 방식이다. 또 다른 일반적인 전략은 라운드 로빈(round-robin)으로, 여러 사용 가능한 백엔드에 호출을 순차적으로 분배한다. 보다 발전된 인프라는 목적지를 선택할 때 백엔드 부하, 지역성(locality), 상태(health), 지연시간(latency), 용량(capacity), 트래픽 관리 정책 등을 고려할 수 있다.

상태 확인(health checking)은 검색된 종단점이 실제로 요청을 정상 처리할 수 있다고 보장할 수 없기 때문에 필수적이다. 프로세스(process)가 실행 중이더라도 데이터베이스, AI 가속기(AI accelerator), 로봇 게이트웨이(robot gateway) 또는 필요한 다른 종속 구성요소에 접근하지 못할 수 있다. 따라서 상태 메커니즘은 필요한 경우 단순한 프로세스 가용성(process availability)과 서비스 준비 상태(service readiness)를 구분하여 실행 중이지만 운영 준비가 되지 않은 인스턴스로 트래픽이 전달되는 것을 방지해야 한다.

gRPC 연결은 지속 연결(persistent connection)이므로 부하 분산 동작에도 영향을 준다. HTTP/2에서는 많은 RPC가 하나의 연결을 공유할 수 있기 때문에 여러 서버 앞에 일반적인 연결 수준 로드 밸런서(connection-level load balancer)를 배치한다고 해서 항상 RPC가 균등하게 분배되는 것은 아니다. 장시간 유지되는 클라이언트 연결은 동일한 백엔드로 많은 호출을 계속 전송할 수 있다. 따라서 부하 분산 아키텍처는 TCP 연결 분배만으로 애플리케이션 트래픽이 자동으로 균형을 이룬다고 가정하지 말고 RPC 수준의 동작을 고려해야 한다.

스트리밍 RPC(streaming RPC)는 장시간 유지되는 스트림이 일반적으로 해당 수명 동안 특정 백엔드와 연결되기 때문에 추가적인 고려가 필요하다. 활성화된 양방향 스트림(bidirectional stream)이나 서버 스트림(server stream)을 서비스 인스턴스 사이에서 이동시키는 것은 독립적인 단항 호출(unary call)을 라우팅하는 것과 근본적으로 다르다. 백엔드에 장애가 발생하면 기존 스트림이 자동으로 다른 서버로 이동한다고 가정하기보다 애플리케이션이 재연결하고 종단점을 다시 검색하며 스트림을 재생성하고 애플리케이션 상태를 복원해야 할 수 있다.

상태 유지 서비스(stateful service) 역시 부하 분산을 복잡하게 만든다. 서비스가 임무 컨텍스트(mission context), 로봇 세션 상태(robot session state), 스트림별 정보를 로컬 메모리에만 유지한다면 이후 요청을 다른 인스턴스로 전달했을 때 일관되지 않은 동작이 발생할 수 있다. 분산 로봇 시스템에서는 외부화된 공유 상태(externalized shared state), 명시적인 세션 소유권(session ownership), 어피니티 메커니즘(affinity mechanism) 또는 서비스 인스턴스를 충분히 무상태(stateless)로 유지하는 API 설계를 통해 이러한 문제를 해결할 수 있다.

장애 처리(failure handling)는 서비스 디스커버리 및 부하 분산을 gRPC 상태 의미체계(status semantics)와 연결한다. 종단점을 사용할 수 없게 되면 클라이언트는 사용 불가(UNAVAILABLE)와 같은 오류를 수신할 수 있으며, 재시도 정책(retry policy)과 연산의 의미가 허용한다면 다른 정상 종단점을 시도할 수 있다. 그러나 자동 종단점 장애조치(endpoint failover)에서도 데드라인(deadline)과 멱등성(idempotency)을 준수해야 한다. 원래 서버가 연결 실패 전에 명령을 실행했을 가능성이 있다면 물리적 로봇 명령을 자동으로 반복하는 것은 위험할 수 있다.

부하 분산은 엣지 로보틱스(edge robotics)에서 지역성(locality)도 고려해야 한다. 로봇은 로컬 엣지 AI 서비스(local edge AI service), 온프레미스 서버(on-premise server), 원격 클라우드 서비스(remote cloud service)에 접근할 수 있으며, 이들은 유사한 기능을 제공하더라도 지연시간과 연결 특성이 크게 다를 수 있다. 디스커버리 메타데이터와 라우팅 정책(routing policy)을 이용하면 모든 종단점을 운영상 동일하게 취급하는 대신 사이트(site), 네트워크 영역(network zone), 하드웨어 성능(hardware capability), 배포 계층(deployment tier)에 따라 서비스를 선택할 수 있다.

관측 가능성(observability)은 트래픽 분배가 의도한 대로 동작하는지를 검증하기 위해 필요하다. 메트릭(metrics)은 요청량, 활성 연결(active connection), RPC 지연시간, 오류율(error rate), 백엔드 사용률(backend utilization), 종단점 상태, 장애조치 이벤트(failover event)를 추적할 수 있다. 분산 추적(distributed tracing)은 각 요청을 처리한 백엔드를 보여주며, 로그는 디스커버리 변경 및 밸런싱 결정을 기록할 수 있다. 이를 통해 과부하된 인스턴스, 비정상 종단점, 불균형한 트래픽, 반복적인 재연결 동작을 식별할 수 있다.

종단점이 동적으로 변경되더라도 보안(security)은 일관되게 유지되어야 한다. 서비스 디스커버리 때문에 백엔드 주소가 일시적이라는 이유로 클라이언트가 신원 검증(identity verification)을 우회해서는 안 된다. TLS 또는 상호 TLS(mutual TLS, mTLS)는 통신 종단점을 인증할 수 있으며, 서비스 신원(service identity)과 인가 정책(authorization policy)은 변경되기 쉬운 IP 주소가 아니라 논리적인 서비스와 연결되어야 한다. 트래픽이 승인되지 않은 서비스로 리디렉션되는 것을 방지하기 위해 디스커버리 정보 자체도 신뢰할 수 있는 인프라에서 제공되어야 한다.

대규모 로봇 플릿(robot fleet)에서 이러한 메커니즘을 사용하면 통신 인프라가 개별 로봇과 독립적으로 발전할 수 있다. 플릿 API(fleet API), 임무 서비스(mission service), 지도 서버(map server), 진단 서비스(diagnostics service), AI 추론 백엔드(AI inference backend)를 용량과 복원력(resilience)을 위해 복제할 수 있다. 클라이언트는 계속 논리적 서비스를 지정하고, 디스커버리가 사용 가능한 인스턴스를 식별하며, 부하 분산이 RPC를 여러 인스턴스에 분배함으로써 수동으로 설정된 서버 주소에 대한 의존성을 줄일 수 있다.

견고한 gRPC 아키텍처에서는 부하 분산(load balancing)과 서비스 디스커버리(service discovery)를 서로 연계된 기능으로 다룬다. 디스커버리는 사용 가능한 서비스 종단점의 정확한 정보를 유지하고, 상태 확인 메커니즘은 어떤 종단점이 요청을 처리할 준비가 되었는지를 판단하며, 밸런싱 정책(balancing policy)은 RPC 트래픽을 전달할 적절한 목적지를 선택한다. 이러한 메커니즘을 데드라인, 재시도, 보안, 상태 관리(state management), 관측 가능성과 결합하면 분산된 로봇, 엣지, 플릿, AI 및 클라우드 시스템 전반에서 확장성과 복원력을 갖춘 통신을 지원할 수 있다.

## 02.07 gRPC Web: Browser-Based Robot Interface [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

웹 브라우저(web browser)는 전용 데스크톱 소프트웨어를 설치하지 않고도 플랫폼 독립적인 접근(platform-independent access)을 제공할 수 있기 때문에 로봇 운영자 인터페이스(operator interface)로 점점 더 많이 활용되고 있다. 브라우저 기반 인터페이스(browser-based interface)는 노트북, 태블릿 또는 산업용 터미널(industrial terminal)에서 로봇 상태, 임무 진행 상황, 진단 정보, 지도, 경보 및 제어 기능을 표시할 수 있다. 그러나 일반적인 네이티브 gRPC(native gRPC) 통신은 네이티브 gRPC 클라이언트와 동일한 방식으로 브라우저 자바스크립트(JavaScript)에서 직접 사용할 수 없다.

이러한 제한은 브라우저 네트워킹 환경(browser networking environment)과 네이티브 gRPC가 요구하는 전송 동작(transport behavior)의 차이에서 발생한다. 표준 gRPC는 브라우저가 자바스크립트 애플리케이션에 직접 제공하지 않는 HTTP/2 기능과 프레이밍(framing)에 의존한다. 따라서 브라우저 애플리케이션에는 gRPC의 서비스 지향 장점을 유지하면서 웹 플랫폼의 제약에 맞게 전송 동작을 조정하는 브라우저 호환 통신 메커니즘(browser-compatible communication mechanism)이 필요하다.

gRPC-Web은 브라우저 기반 애플리케이션을 위한 이러한 호환 계층(compatibility layer)을 제공한다. 웹 클라이언트가 프로토콜 버퍼(Protocol Buffers)에서 파생된 인터페이스를 사용하여 백엔드 서비스(backend service)와 통신하면서 gRPC 통신을 브라우저가 지원하는 HTTP 메커니즘에 맞게 조정할 수 있도록 한다. 애플리케이션 개발자 관점에서는 브라우저가 느슨하게 구조화된 네트워크 요청을 수동으로 생성하는 대신 강하게 정의된 메시지와 생성된 서비스 인터페이스(generated service interface)를 계속 사용할 수 있다.

일반적인 아키텍처에서는 웹 브라우저와 네이티브 gRPC 서비스(native gRPC service)를 양쪽에 배치하고, 배포 환경에 따라 그 사이에 호환 가능한 프록시(proxy) 또는 게이트웨이(gateway)를 배치한다. 브라우저는 gRPC-Web 요청을 전송하고 중간 구성요소는 이를 백엔드 gRPC 서비스로 변환하거나 전달한다. 이를 통해 기존 로봇, 엣지(edge), 플릿(fleet), 클라우드 서비스는 네이티브 gRPC 기반을 유지하면서 브라우저 애플리케이션에는 적절한 웹 호환 인터페이스를 제공할 수 있다.

프로토콜 버퍼(Protocol Buffers)는 동일한 개념적 서비스 계약(service contract)을 백엔드와 프론트엔드(frontend) 구성요소에서 공유할 수 있기 때문에 이러한 아키텍처에서도 중요하다. 개발자는 .proto 파일에 요청 메시지(request message), 응답 메시지(response message), RPC 메서드를 정의하고 웹 애플리케이션에 적합한 클라이언트 코드를 생성한다. 따라서 브라우저 애플리케이션은 타입이 정의된 객체와 사전에 정의된 서비스 메서드를 사용할 수 있으며, 백엔드 구현 역시 해당 프로그래밍 언어에 맞는 생성 인터페이스를 사용할 수 있다.

브라우저 기반 로봇 인터페이스는 일반적인 운영자 기능에 단항 RPC(unary RPC)를 사용할 수 있다. 인터페이스는 현재 로봇 구성(configuration)을 요청하거나, 임무 정보를 조회하거나, 상위 수준 명령(high-level command)을 제출하거나, 경보를 확인하거나, 진단 스냅샷(diagnostic snapshot)을 요청할 수 있다. 각각의 브라우저 동작은 구조화된 요청을 생성하고 구조화된 응답을 수신하므로 문서화되지 않은 메시지 형식을 사용하는 인터페이스보다 검증과 유지보수가 용이하다.

로봇 인터페이스에서는 지속적으로 변화하는 정보를 표시해야 하므로 서버에서 시작되는 업데이트(server-originated update)도 중요하다. 임무 상태, 배터리 수준, 내비게이션 진행 상태, 건전성 정보(health information), 이벤트 알림(event notification) 등을 반복적인 수동 새로고침 없이 표시해야 할 수 있다. gRPC-Web 구현과 주변 아키텍처에 따라 지원되는 스트리밍 메커니즘을 사용하여 연속적인 서버 응답을 전달할 수 있지만 브라우저 지향 스트리밍 기능은 제한이 없는 네이티브 gRPC 스트리밍과 완전히 동일하지는 않다.

이러한 차이는 클라이언트 스트리밍(client streaming)과 양방향 스트리밍(bidirectional streaming)에서 특히 중요하다. 네이티브 gRPC는 두 패턴을 직접 지원하지만 브라우저 기반 gRPC-Web 환경에서는 구현 방식과 전송 모드에 따라 제한이 발생할 수 있다. 따라서 지속적인 양방향 상호작용이 필요한 애플리케이션은 gRPC-Web을 다른 브라우저 호환 메커니즘과 결합하거나, 브라우저 명령은 개별 RPC 호출을 사용하고 서버 업데이트는 지원되는 스트리밍 방식을 사용하도록 인터페이스를 재설계할 수 있다.

브라우저 인터페이스는 일반적으로 안전 필수 실시간 제어 루프(safety-critical real-time control loop)를 직접 구성하기보다 감독 또는 임무 수준(supervisory or mission level)에서 동작해야 한다. 운영자는 웹 인터페이스를 통해 목적지 주행을 요청하거나, 임무를 일시정지하거나, 운영 모드를 선택하거나, 진단 정보를 확인할 수 있다. 저수준 모터 제어, 비상 정지 로직(emergency stopping logic), 결정론적 안전 기능(deterministic safety function), 엄격한 시간 제약을 갖는 제어 루프는 해당 시간 및 안전 요구사항을 위해 설계된 로봇 제어기와 통신 시스템 내부에 유지해야 한다.

브라우저 인터페이스는 단순한 모니터링을 넘어 운영 기능을 노출할 수 있으므로 인증(authentication)이 필수적이다. 애플리케이션은 사용자 신원(user identity)을 확인하고 서비스 요청에 인증 정보를 첨부해야 할 수 있다. 이후 백엔드 서비스 또는 게이트웨이는 사용자 역할(user role), 로봇 식별자(robot identity), 사이트(site), 임무(mission), 요청된 메서드에 따라 인가 정책(authorization policy)을 적용할 수 있다. 이를 통해 읽기 전용 모니터링 기능과 실제 로봇 동작을 변경하는 명령을 분리할 수 있다.

전송 보안(transport security)은 브라우저와 서비스 인프라 사이의 통신을 보호해야 한다. HTTPS와 TLS는 기밀성(confidentiality)과 서버 인증(server authentication)을 제공하며, 배포 아키텍처에서는 게이트웨이와 내부 gRPC 서비스 사이에 추가적인 신원 확인 메커니즘을 사용할 수 있다. 브라우저 인터페이스를 외부에 제공한다는 이유로 모든 내부 로봇 서비스를 외부 네트워크나 신뢰할 수 없는 클라이언트 환경에 직접 노출하지 않도록 보안 경계(security boundary)를 설계해야 한다.

브라우저 애플리케이션과 gRPC-Web 종단점(endpoint)이 서로 다른 오리진(origin)에서 제공되는 경우 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)도 고려해야 한다. 브라우저는 네이티브 gRPC 클라이언트가 동일한 방식으로 경험하지 않는 오리진 기반 보안 정책(origin-based security policy)을 적용한다. 따라서 게이트웨이 또는 서비스 인프라는 적절한 CORS 설정을 제공해야 하며, 로봇 API를 승인되지 않은 웹 오리진에 불필요하게 노출하는 지나치게 광범위한 정책은 피해야 한다.

게이트웨이(gateway)는 단순한 프로토콜 변환(protocol translation) 이상의 역할을 수행할 수 있다. 브라우저 트래픽이 내부 로봇 서비스에 도달하기 전에 인증 검사, 인가, 속도 제한(rate limiting), 요청 검증(request validation), 로깅(logging), 메트릭(metrics), 라우팅(routing)을 중앙에서 처리할 수 있다. 이를 통해 웹에 노출되는 환경과 운영 인프라 사이에 통제된 경계(controlled boundary)를 형성할 수 있으며, 내부 서비스는 로봇 기능에 집중하고 게이트웨이는 브라우저 접근과 외부 통신에 관련된 기능을 담당할 수 있다.

사용자의 하나의 동작이 로봇에 영향을 주기 전에 여러 분산 구성요소를 통과할 수 있기 때문에 운영자 인터페이스에서는 관측 가능성(observability)이 특히 중요하다. 명령은 브라우저에서 게이트웨이를 거쳐 플릿 서비스로 전달된 다음 엣지 컴퓨터를 거쳐 최종적으로 로봇에 도달할 수 있다. 상관관계 식별자(correlation identifier)와 분산 추적(distributed tracing)은 이러한 단계를 연결할 수 있으며, 로그와 메트릭은 요청 지연시간, 장애, 인가 결정, 백엔드 서비스 동작을 보여줄 수 있다.

연결 장애(connection failure)와 일시적인 서비스 비가용성(service unavailability)은 그래픽 인터페이스 뒤에 숨기지 않고 브라우저 애플리케이션에서 명확하게 표시해야 한다. 사용자 인터페이스는 확인된 로봇 상태(confirmed robot state)와 오래되었거나 사용할 수 없는 정보(stale or unavailable information)를 구분해야 하며, 브라우저가 명령을 전송했다는 이유만으로 해당 명령이 성공적으로 실행되었다고 표시해서는 안 된다. gRPC 상태 정보(status information)를 이용하면 거부된 요청, 데드라인(deadline), 사용 불가능한 서비스 및 기타 장애를 명시적으로 표현할 수 있다.

많은 운영자 또는 브라우저 세션이 대규모 플릿을 동시에 모니터링하면 확장성(scalability)이 중요해진다. 각 브라우저가 반복적으로 고주파 요청을 전송하면 플릿 및 로봇 서비스에 불필요한 부하가 발생할 수 있다. 백엔드 집계(backend aggregation), 제어된 업데이트 주기(controlled update rate), 캐싱(caching), 적절한 스트리밍, 선택적 구독(selective subscription)을 사용하면 트래픽을 줄일 수 있다. 웹 인터페이스는 모든 내부 로봇 데이터 스트림을 그대로 복제하기보다 운영 상황 인식(operational awareness)에 필요한 정보를 선별적으로 전달받아야 한다.

실용적인 로봇 웹 아키텍처(robot web architecture)는 프레젠테이션(presentation), 통신(communication), 오케스트레이션(orchestration), 물리적 제어(physical control)의 책임을 분리한다. 브라우저는 시각화와 운영자 상호작용을 제공하고, gRPC-Web은 구조화된 서비스 접근을 제공하며, 게이트웨이는 브라우저 지향 통신과 보안을 관리하고, 네이티브 gRPC는 분산 백엔드 서비스를 연결하며, 로봇 제어기는 실제 운영 동작을 실행한다. 이러한 분리를 통해 브라우저 기술을 저수준 로봇 제어에 강제로 적용하지 않고도 각 계층을 독립적으로 발전시킬 수 있다.

궁극적으로 gRPC-Web은 계약 기반 gRPC 서비스 설계(contract-based gRPC service design)를 브라우저에서 접근 가능한 로봇 애플리케이션으로 확장한다. 프로토콜 버퍼(Protocol Buffers)는 구조화된 인터페이스를 유지하고, 생성된 클라이언트(generated client)는 수동 통신 코드를 줄이며, 게이트웨이는 브라우저와 백엔드 환경을 연결하고, 보안 및 관측 가능성 메커니즘은 통제된 운영 접근을 제공한다. 적절한 아키텍처 계층에서 사용하면 로봇, 엣지 시스템, 플릿 플랫폼, 클라우드 인프라 전반에 확장 가능한 웹 대시보드(web dashboard)와 감독 인터페이스(supervisory interface)를 구축할 수 있다.

## 02.08 gRPC vs REST Performance Comparison: Robot Scenario

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC와 REST는 모두 분산 로봇 시스템(distributed robot system)의 동기 통신(synchronous communication)에 폭넓게 적용할 수 있지만, 서로 다른 인터페이스 및 전송 설계 철학(interface and transport design philosophy)을 가진다. REST는 일반적으로 HTTP와 JSON을 사용하여 자원 지향 API(resource-oriented API)를 제공하는 반면, gRPC는 프로토콜 버퍼(Protocol Buffers)를 통해 호출 가능한 서비스와 강타입 메시지(strongly typed message)를 정의한다. 이 장의 구조에서는 어느 하나를 다른 하나의 보편적인 대체 기술로 취급하기보다 두 기술 모두를 중요한 통신 메커니즘으로 다룬다.

REST는 HTTP 기반 API가 브라우저(browser), 기업 시스템(enterprise system), 개발 도구(development tool), 클라우드 플랫폼(cloud platform)에서 광범위하게 지원되기 때문에 로봇 인터페이스에 유용하다. JSON 메시지는 사람이 읽을 수 있으며 통합과 디버깅 과정에서 편리하다. 플릿 애플리케이션(fleet application)은 로봇, 임무, 지도, 경보 또는 구성(configuration)과 같은 자원을 일반적인 종단점(endpoint)을 통해 제공할 수 있으므로 REST는 상호운용성(interoperability)과 외부 접근성이 중요한 경우 특히 실용적이다.

gRPC는 효율적인 기계 간 통신(machine-to-machine communication)에 중점을 둔다. 프로토콜 버퍼(Protocol Buffers)는 구조화된 정보를 압축된 바이너리 표현(binary representation)으로 인코딩하며, 생성된 클라이언트 및 서버 인터페이스는 .proto 스키마에 정의된 계약을 적용한다. HTTP/2는 지속 연결(persistent connection)과 다중화 스트림(multiplexed stream)을 제공한다. 이러한 특성으로 인해 gRPC는 구조화된 고빈도 상호작용이 필요한 로봇 컴퓨터, 엣지 AI 서비스(edge AI service), 플릿 서버(fleet server), 내부 백엔드 서비스(backend service) 사이의 통신에 특히 적합하다.

페이로드 표현(payload representation)은 성능 차이가 발생하는 원인 가운데 하나이다. JSON은 필드 이름과 값을 텍스트 형태로 표현하므로 메시지를 쉽게 읽을 수 있지만 전송 크기와 파싱 작업이 증가할 수 있다. 프로토버프(Protobuf)는 숫자 태그(numeric tag)를 사용하여 필드를 식별하고 값을 바이너리 형식으로 인코딩한다. 반복적으로 전송되는 로봇 상태, 임무 상태, 위치추정 메타데이터(localization metadata), AI 추론 메시지에서는 이러한 압축된 표현을 통해 동일한 의미를 갖는 장황한 JSON 구조보다 통신 오버헤드를 줄일 수 있다.

직렬화 비용(serialization cost)도 시스템 동작에 영향을 미친다. JSON을 사용하는 REST 애플리케이션은 애플리케이션 객체를 텍스트 JSON으로 변환하고 수신된 텍스트를 다시 내부 구조로 파싱해야 한다. 프로토버프 구현은 생성된 메시지 표현과 바이너리 직렬화(binary serialization)를 사용한다. 실제 성능은 프로그래밍 언어, 라이브러리, 메시지 구조, 하드웨어, 워크로드에 따라 달라지므로 보편적인 수치상의 성능 향상을 가정하기보다 실제 로봇 시나리오를 측정하여 비교해야 한다.

전송 동작(transport behavior)은 또 다른 중요한 차이를 만든다. REST 배포 환경에서는 구현 방식에 따라 HTTP/1.1, HTTP/2 또는 새로운 HTTP 인프라를 사용할 수 있는 반면 일반적인 gRPC는 HTTP/2와 밀접하게 연관되어 있다. HTTP/2 다중화(multiplexing)는 여러 논리적 RPC 스트림이 하나의 지속 연결을 공유할 수 있도록 한다. 이는 하나의 로봇이나 엣지 컴퓨터가 임무, 진단, 인지(perception), 구성 및 AI 서비스와 동시에 통신하는 환경에서 유용하다.

스트리밍(streaming)은 두 방식 사이의 중요한 아키텍처 차이를 제공한다. REST는 기본적으로 개별적인 요청-응답(request-response) 상호작용에 적합하지만 추가적인 HTTP 기술을 이용하여 스트리밍을 구현할 수도 있다. gRPC는 단항(unary), 서버 스트리밍(server streaming), 클라이언트 스트리밍(client streaming), 양방향 스트리밍(bidirectional streaming)을 서비스 모델에 직접 정의한다. 지속적으로 변화하는 로봇 상태나 서비스 간 지속적인 협업에서는 이러한 명시적인 스트리밍 추상화(streaming abstraction)를 통해 별도의 폴링(polling)이나 사용자 정의 통신 규칙을 설계해야 하는 필요성을 줄일 수 있다.

많은 자율이동로봇(Autonomous Mobile Robot, AMR)이 배터리 수준, 자세(pose), 속도, 임무 상태, 건전성 정보(health information), 진단 이벤트를 온프레미스 플릿 서버(on-premise fleet server)에 주기적으로 보고하는 로봇 플릿 시나리오를 생각할 수 있다. REST 구현에서는 JSON 문서를 포함하는 HTTP 요청을 반복적으로 전송할 수 있다. 이러한 방식은 단순하고 투명하지만 로봇 수와 업데이트 빈도가 증가하면 직렬화, 페이로드, 요청 처리, 연결 관리(connection management) 오버헤드도 증가할 수 있다.

gRPC 구현에서는 동일한 로봇 상태를 프로토버프 메시지(Protobuf message)로 표현하고 HTTP/2를 통해 효율적인 RPC 통신을 유지할 수 있다. 상태 정보는 애플리케이션 아키텍처에 따라 단항 호출(unary call) 또는 적절한 스트리밍 메서드를 통해 전송할 수 있다. 업데이트 빈도와 동시 서비스 상호작용이 증가할수록 압축된 직렬화와 지속적인 다중화 통신의 가치가 커질 수 있으며, 특히 통제된 로봇-엣지 또는 서비스 간 네트워크에서 유용하다.

인터페이스가 브라우저 기반 운영자 대시보드(browser-based operator dashboard)를 대상으로 하면 비교 조건이 달라진다. REST는 브라우저 HTTP API와 자연스럽게 통합되며 개발자가 JSON을 직접 확인할 수 있다. 네이티브 gRPC(native gRPC)는 일반적으로 브라우저 자바스크립트에서 같은 방식으로 직접 호출할 수 없으므로 gRPC-Web과 경우에 따라 게이트웨이(gateway) 또는 프록시(proxy)가 필요하다. 따라서 내부 기계 통신에서는 gRPC가 장점을 제공하더라도 브라우저 통합에서는 REST가 더 낮은 통합 복잡성(integration complexity)을 제공할 수 있다.

명령 의미체계(command semantics) 역시 프로토콜 선택에 영향을 미친다. 플릿 관리 시스템은 임무 자원을 생성, 조회, 갱신 또는 취소하는 작업이 자원 지향 API와 자연스럽게 대응하므로 REST를 사용할 수 있다. 이후 내부 실행 서비스는 gRPC를 사용하여 구조화된 임무 명령을 전송하고, AI 처리를 요청하며, 상태를 교환하거나 분산 구성요소를 협업시킬 수 있다. 따라서 실용적인 로봇 아키텍처에서는 서로 다른 통신 경계에서 두 기술을 함께 사용할 수 있다.

성능은 하나의 지연시간(latency) 수치가 아니라 여러 측정값을 이용하여 평가해야 한다. 종단 간 지연시간(end-to-end latency), 초당 요청 수(requests per second), CPU 사용률, 메모리 사용량, 직렬화된 메시지 크기, 네트워크 대역폭, 연결 수, 동시성(concurrency) 환경에서의 동작 등이 결과에 영향을 미칠 수 있다. 스트리밍 워크로드에서는 메시지 전송률, 스트림 지속시간, 재연결 동작, 역압(backpressure)도 측정해야 단순한 요청-응답 벤치마크에서 확인하기 어려운 특성을 평가할 수 있다.

메시지 크기(message size)는 직렬화 효율성이 얼마나 중요한지를 결정하는 요소이다. 요청이 작고 빈도가 낮다면 JSON과 프로토버프의 실제 차이는 네트워크 지연, 애플리케이션 처리, 데이터베이스 접근 또는 로봇 제어 로직에 비해 중요하지 않을 수 있다. 반면 많은 로봇이 구조화된 메시지를 빈번하게 교환하는 경우 메시지 하나에서 발생하는 작은 절감 효과도 수천 또는 수백만 건의 트랜잭션에 누적되어 대역폭과 처리 요구사항에 실질적인 영향을 미칠 수 있다.

지연시간 측정(latency measurement)에서는 평균 동작과 꼬리 지연시간(tail latency)을 구분해야 한다. 평균 지연시간이 허용 가능한 시스템에서도 간헐적인 느린 응답이 임무 협업에 영향을 줄 수 있다. p95 또는 p99와 같은 백분위수(percentile) 측정은 이러한 지연을 확인하는 데 도움이 된다. 또한 직렬화 시간, 네트워크 전송, 서버 처리, 큐잉(queueing), 애플리케이션 실행을 구분하여 관찰된 성능 차이가 REST 또는 gRPC 자체에만 잘못 귀속되지 않도록 해야 한다.

아키텍처 선택을 위한 비교라면 벤치마크 조건(benchmark condition)을 통제해야 한다. 두 구현은 동일한 애플리케이션 의미체계, 보안 설정, 페이로드 정보, 서버 로직, 네트워크 조건, 동시성을 사용해야 한다. 최적화된 gRPC 구현과 최적화되지 않은 REST 서버를 비교하면 프로토콜 특성이 아니라 구현 품질을 측정하게 된다. 운영상 중요하다면 연결이 이미 형성된 웜 연결(warm connection)과 초기 연결을 설정하는 콜드 연결(cold connection)도 별도로 평가해야 한다.

신뢰성(reliability)과 유지보수성(maintainability)은 순수한 처리량 메트릭은 아니지만 엔지니어링 비교에서 중요한 요소이다. REST는 성숙한 HTTP 생태계, 쉬운 메시지 확인, 광범위한 상호운용성이라는 장점을 가진다. gRPC는 강하게 정의된 계약, 생성 코드(generated code), 표준화된 상태 처리(status handling), 명시적인 스트리밍 모델을 제공한다. 이러한 특성은 특정 유형의 통합 오류를 줄일 수 있으며 장기간 운영되는 로봇 플랫폼에서는 작은 벤치마크 성능 차이보다 더 중요할 수 있다.

따라서 분산 로봇 플랫폼에서는 하이브리드 아키텍처(hybrid architecture)가 적합한 경우가 많다. REST는 브라우저 애플리케이션, 기업 시스템 통합, 외부 공개 관리 API(public-facing management API), 자원 지향 연산을 담당하고, gRPC는 압축된 구조화 메시지, 낮은 통신 오버헤드 또는 스트리밍이 필요한 내부 서비스를 연결할 수 있다. 필요한 경계에는 게이트웨이를 배치하여 외부 접근성과 내부 통신 효율성을 동일한 시스템 아키텍처에서 함께 확보할 수 있다.

궁극적으로 의미 있는 gRPC와 REST의 비교는 시나리오에 따라 달라진다. 로봇 수, 메시지 빈도, 페이로드 크기, 브라우저 요구사항, 네트워크 품질, 스트리밍 요구사항, 개발 생태계, 운영 제약조건에 따라 중요하게 평가해야 할 특성이 달라진다. 일반적인 벤치마크 주장만으로 프로토콜을 선택하기보다 로봇 공학 엔지니어는 대표적인 실제 워크로드를 벤치마크하고 REST와 gRPC 각각의 특성이 시스템 요구사항에 가장 적합한 통신 경계에 두 기술을 배치해야 한다.

## 02.09 Robot Fleet Control gRPC API Design Case [w/Code]

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 플릿 제어 시스템(robot fleet control system)은 공통 통신 및 오케스트레이션 계층(communication and orchestration layer)을 통해 다수의 자율 로봇을 조정한다. gRPC API는 안정적인 서비스 계약(service contract)을 통해 로봇 식별자, 운영 상태, 임무, 명령, 이벤트, 진단 및 플릿 수준 협업을 표현해야 한다. 내부 구현 세부사항을 직접 노출하기보다 플릿 관리, 로봇 실행, 모니터링 및 지원 서비스 사이에 명확한 경계를 정의해야 한다.

실용적인 아키텍처는 플릿 수준 서비스(fleet-level service)와 로봇 수준 서비스(robot-level service)를 분리한다. 플릿 서버(fleet server)는 로봇, 임무, 지도, 자원 및 운영 정책에 대한 전역 관점(global view)을 유지하며, 각 로봇 또는 엣지 컴퓨터(edge computer)는 로컬 실행에 필요한 기능을 제공한다. gRPC는 이러한 구성요소 사이에 강타입 서비스 인터페이스(strongly typed service interface)를 제공하여 공유 프로토버프 계약(Protobuf contract)을 유지하면서 플릿 소프트웨어와 로봇 소프트웨어가 독립적으로 발전할 수 있도록 한다.

모든 명령, 상태 업데이트 및 임무가 올바른 물리적 시스템과 연결되어야 하므로 로봇 식별자(robot identity)는 API에 명시적으로 정의되어야 한다. RobotId 또는 이에 상응하는 식별자를 요청 및 상태 메시지에 포함할 수 있으며, 추가 메타데이터(metadata)를 통해 로봇 유형, 기능, 소프트웨어 버전, 운영 사이트 또는 소속 플릿을 설명할 수 있다. 안정적인 식별자를 사용하면 네트워크 주소가 영구적인 애플리케이션 식별자로 사용되는 것을 방지할 수 있다.

로봇 상태 메시지(robot state message)는 모든 내부 센서 신호를 불필요하게 노출하지 않으면서 플릿 의사결정에 필요한 정보를 표현해야 한다. 일반적인 필드는 운영 모드, 임무 상태, 자세(pose), 속도, 배터리 상태, 위치추정 품질(localization quality), 연결 상태(connectivity), 안전 상태(safety state), 건전성 요약(health summary)을 나타낼 수 있다. 상세 인지(perception) 또는 제어기 데이터는 특화된 인터페이스에 유지하여 일반적인 플릿 통신을 간결하고 의미적으로 안정되게 유지할 수 있다.

임무 관리(mission management)는 API의 핵심적인 책임이다. 임무 서비스(mission service)는 대상 로봇, 임무 식별자, 작업 설명, 목적지, 우선순위 및 필요한 매개변수를 포함하는 구조화된 임무 요청을 수신할 수 있다. 서버는 요청을 검증하고 운영 제약조건을 확인하며 임무를 기록한 후 승인 응답(acknowledgement)을 반환한다. 이후 물리적인 완료 시점까지 원래 RPC를 유지하는 대신 임무 실행을 비동기적(asynchronous)으로 진행할 수 있다.

따라서 임무 상태(mission status)는 임무 제출(mission submission)과 분리해야 한다. 임무를 수락한 후 플릿 시스템은 조회 메서드(query method) 또는 서버 스트리밍(server streaming) 업데이트를 통해 상태를 제공할 수 있다. 대기(queued), 할당(assigned), 실행(executing), 일시정지(paused), 완료(completed), 취소(cancelled), 실패(failed)와 같은 상태를 사용하면 클라이언트가 임무 생명주기(mission lifecycle)를 추적할 수 있다. 추가 필드를 통해 원래 임무 요청을 변경하지 않고 진행률, 타임스탬프, 실패 원인 또는 현재 실행 단계를 보고할 수 있다.

명령 API(command API)는 상위 수준 운영 의도(high-level operational intent)와 저수준 액추에이터 제어(low-level actuator control)를 구분해야 한다. 플릿 서비스는 일시정지, 재개, 임무 취소, 스테이션 복귀, 도킹 요청 또는 승인된 운영 모드 변경과 같은 명령을 제공할 수 있다. 직접적인 휠 속도, 모터 토크, 비상 안전 로직 및 엄격한 시간 제약을 갖는 기타 제어 기능은 일반적인 플릿 수준 gRPC 호출로 실행하지 않고 로봇 제어 시스템 내부에 유지해야 한다.

멱등성(idempotency)은 네트워크 장애로 인해 클라이언트가 요청 실행 여부를 확신할 수 없는 상황이 발생할 수 있으므로 플릿 명령에서 특히 중요하다. 상태를 변경하는 각 요청은 서버가 기록하는 고유한 명령 또는 요청 식별자(command or request identifier)를 포함할 수 있다. 동일한 식별자가 다시 수신되면 서비스는 물리적 동작을 반복하지 않고 이전 결과를 반환할 수 있다. 이를 통해 재시도로 인해 임무나 명령이 의도하지 않게 중복 실행되는 것을 방지할 수 있다.

플릿 애플리케이션이 지속적인 로봇 업데이트를 필요로 하는 경우 상태 배포(status distribution)는 서버 스트리밍(server streaming)에 적합하다. 클라이언트는 하나의 로봇, 로봇 그룹 또는 전체 플릿을 구독하고 상태가 변경될 때 구조화된 상태 메시지를 수신할 수 있다. 모든 내부 제어 주기의 값을 전달하면 불필요한 네트워크 및 처리 부하가 발생하므로 업데이트 주기는 운영상의 필요에 따라 제어해야 한다. 플릿 텔레메트리(fleet telemetry)는 감독에 유용한 정보를 표현해야 한다.

지속적인 협업 세션(persistent coordination session)이 필요한 경우 양방향 스트리밍(bidirectional streaming)을 사용할 수 있다. 로봇과 플릿 서비스는 장시간 유지되는 스트림을 통해 상태 전환, 승인, 이벤트 및 협업 정보를 교환할 수 있다. 그러나 설계에서는 재연결 동작(reconnection behavior), 시퀀스 처리(sequence handling), 세션 소유권(session ownership), 상태 복구(state recovery)를 명확하게 정의해야 한다. 지속 스트림이 명확한 의미체계 없이 서로 관련 없는 메시지를 전달하는 비정형 통신 채널이 되어서는 안 된다.

플릿 이벤트 API(fleet event API)는 일반적인 상태 변수가 아니지만 주의가 필요한 비동기 상태(asynchronous condition)를 표현할 수 있다. 예를 들어 장애물로 인한 임무 중단, 위치추정 손실(localization loss), 도킹 실패, 배터리 경고, 안전 이벤트, 하드웨어 장애, 연결 상태 변경 등이 포함될 수 있다. 이벤트에는 식별자, 타임스탬프, 심각도 또는 범주 정보, 발생원(source information), 구조화된 세부 정보를 포함하여 모니터링 및 자동화 시스템이 이를 일관되게 해석할 수 있도록 해야 한다.

플릿 의사결정은 여러 로봇과 컴퓨터에서 생성된 관측 정보를 결합할 수 있기 때문에 시간 정보(time information)를 신중하게 설계해야 한다. 메시지는 측정 시간(measurement time), 이벤트 발생 시간(event occurrence time), 서버 수신 시간(server receipt time), 명령 생성 시간(command creation time) 등 타임스탬프의 출처와 의미를 명확하게 정의해야 한다. 시간 협조가 중요한 경우 분산 장치의 모든 타임스탬프가 자동으로 동일하다고 가정하지 말고 아키텍처에서 클록 동기화(clock synchronization)의 전제조건을 정의해야 한다.

오류 처리(error handling)는 표준 gRPC 상태 코드(status code)와 로봇별 구조화된 세부 정보(structured detail)를 함께 사용해야 한다. 잘못된 인수(INVALID_ARGUMENT)는 잘못된 임무 매개변수를 나타내고, 찾을 수 없음(NOT_FOUND)은 알 수 없는 로봇을 나타낼 수 있으며, 전제조건 실패(FAILED_PRECONDITION)는 로봇이 현재 임무를 실행할 수 없는 상태를 표현할 수 있다. 사용 불가(UNAVAILABLE)는 필요한 서비스의 일시적인 손실을 나타낼 수 있으며, 애플리케이션 세부 정보를 통해 각 메서드마다 호환되지 않는 별도의 오류 규칙을 만들지 않고도 구체적인 운영 원인을 설명할 수 있다.

데드라인(deadline)은 각 RPC의 예상 생명주기에 맞게 정의해야 한다. 로봇 정보 조회에는 짧은 데드라인이 적합할 수 있지만 임무 승인에는 더 긴 처리 시간을 허용할 수 있다. 명령을 수락하기 위한 데드라인과 로봇이 해당 명령을 물리적으로 완료하는 데 필요한 시간을 혼동해서는 안 된다. 장시간 수행되는 물리적 작업은 일반적으로 전체 실행 시간 동안 RPC 호출을 차단하는 대신 비동기 상태 전환(asynchronous state transition)으로 표현해야 한다.

인증(authentication)과 인가(authorization)는 서비스 경계(service boundary)에 통합되어야 한다. 로봇 식별자, 플릿 서비스, 운영자 및 유지보수 애플리케이션에는 서로 다른 자격증명(credential)과 권한(permission)이 필요할 수 있다. 인터셉터(interceptor)는 요청이 애플리케이션 로직에 도달하기 전에 인증 메타데이터를 검증하고 메서드 수준 정책(method-level policy)을 적용할 수 있다. 예를 들어 모니터링 클라이언트는 플릿 상태를 읽을 수 있지만 승인된 오케스트레이션 서비스만 임무를 할당하거나 취소할 수 있도록 구성할 수 있다.

플릿 백엔드 서비스(fleet backend service)가 복제되는 경우 서비스 디스커버리(service discovery)와 부하 분산(load balancing)이 중요해진다. 클라이언트는 고정된 서버 주소가 아니라 논리적인 서비스 식별자를 사용하고, 디스커버리 메커니즘은 사용 가능한 인스턴스를 식별해야 한다. 무상태(stateless) 또는 외부에서 상태가 조정되는 백엔드 서비스는 단항 RPC 트래픽을 여러 복제본에 분배할 수 있다. 상태를 유지하는 스트림과 로봇 세션에는 애플리케이션 컨텍스트 없이 활성 연결을 단순히 이동시킬 수 없으므로 명확한 소유권 및 복구 전략이 필요하다.

관측 가능성(observability)은 배포 이후에 추가하기보다 API 설계 단계부터 포함해야 한다. RPC 메타데이터는 플릿, 엣지, AI 및 로봇 서비스 전반에 상관관계 식별자(correlation identifier)와 추적 식별자(trace identifier)를 전파할 수 있다. 인터셉터는 요청 횟수, 상태 코드, 지연시간, 스트림 지속시간, 인증 결과를 수집할 수 있다. 로그는 중요한 작업을 로봇, 임무 및 명령 식별자와 연결하여 엔지니어가 플릿 수준 요청이 시스템을 통해 어떻게 전달되었는지 재구성할 수 있도록 해야 한다.

확장 가능한 플릿 API(scalable fleet API)는 모든 소비자(consumer)를 모든 로봇에 직접 연결하는 구조도 피해야 한다. 운영자 대시보드, 분석 시스템, 기업 애플리케이션 및 외부 서비스는 플릿 수준 API와 통신하고 플릿 백엔드가 로봇 및 엣지 서비스와의 통신을 관리할 수 있다. 이러한 계층적 구조(hierarchical structure)는 연결 복잡성을 줄이고 내부 로봇 인터페이스를 보호하며 집계(aggregation), 인가, 라우팅 및 운영 정책을 적용할 수 있는 통제된 지점을 제공한다.

결과적으로 gRPC API는 플릿 지능(fleet intelligence)과 분산 로봇 실행(distributed robot execution) 사이의 계약 역할을 한다. 프로토버프(Protobuf)는 안정적인 메시지를 정의하고, 단항 RPC(unary RPC)는 범위가 명확한 연산을 처리하며, 스트리밍(streaming)은 변화하는 상태를 전달하고, 상태 코드(status code)는 장애를 표현하며, 보안 메커니즘은 서비스 접근을 제어한다. 이러한 구조를 멱등 명령, 명확한 임무 생명주기, 관측 가능성, 디스커버리 및 복구 메커니즘과 결합하면 협업형 다중 로봇 플릿 운영(coordinated multi-robot fleet operation)을 위한 확장 가능한 기반을 구축할 수 있다.

## 02.10 gRPC Debugging Tools: grpcurl / grpc.gateway

![](images/image11.png){width="7.268055555555556in" height="7.268055555555556in"}

gRPC 서비스 디버깅(gRPC service debugging)을 위해서는 서비스 정의(service definition), 요청 메시지(request message), 메타데이터(metadata), 상태 코드(status code), 전송 보안(transport security), 스트리밍 동작(streaming behavior)을 확인할 수 있어야 한다. 분산 로봇 시스템(distributed robot system)에서는 로봇, 엣지 컴퓨터(edge computer), 플릿 서버(fleet server), AI 서비스 또는 클라우드 구성요소 사이에서 장애가 발생할 수 있다. grpcurl과 grpc-gateway 같은 도구를 사용하면 각 서비스마다 별도의 진단 클라이언트(diagnostic client)를 개발하지 않고도 이러한 인터페이스를 검사할 수 있다.

grpcurl은 HTTP 서비스에서 curl을 사용하는 것과 개념적으로 유사한 방식으로 gRPC 서버와 상호작용하기 위해 설계된 명령줄 도구(command-line tool)이다. 엔지니어는 터미널에서 직접 RPC 메서드를 호출하고, 요청 데이터를 전달하며, 응답을 확인하고, 메타데이터를 추가하고, 보안이 적용된 종단점(endpoint)을 테스트할 수 있다. 따라서 개발, 시스템 통합, 배포 검증 및 로봇 통신 서비스의 현장 문제 해결 과정에서 유용하다.

gRPC 서버는 grpcurl이 사용 가능한 서비스와 메서드를 동적으로 검색할 수 있도록 리플렉션 정보(reflection information)를 제공할 수 있다. 서버 리플렉션(server reflection)이 활성화되어 있으면 엔지니어는 모든 .proto 파일을 직접 찾지 않고도 API를 검사할 수 있다. 서비스 목록과 메서드 정의를 확인하고 메시지 구조를 이해할 수 있으며, 리플렉션은 인터페이스 정보 노출을 최소화하는 것보다 빠른 검사가 중요한 개발 및 통제된 진단 환경에서 특히 편리하다.

리플렉션을 사용할 수 없는 경우 grpcurl은 프로토콜 버퍼 소스 파일(Protocol Buffer source file) 또는 컴파일된 디스크립터 세트(compiled descriptor set)를 사용할 수 있다. 이 방식은 운영 시스템에서 의도적으로 리플렉션을 비활성화했거나 특정 버전의 API 계약을 테스트해야 할 때 유용하다. 이 경우 디버깅 클라이언트는 배포된 서비스와 일치하는 스키마 정보를 사용해야 하므로 API 버전 관리와 디스크립터 추적성(descriptor traceability)이 진단 과정의 중요한 요소가 된다.

단항 RPC 테스트(unary RPC testing)는 가장 단순한 grpcurl 작업 방식 중 하나이다. 엔지니어는 구조화된 요청을 구성하고 메서드를 호출한 후 반환된 메시지와 gRPC 상태를 확인할 수 있다. 로봇 플릿 서비스에서는 RobotId 조회, 임무 정보 검색, 진단 요청 또는 명령 검증 메서드 테스트 등이 이에 해당할 수 있다. 이러한 직접 호출은 서버 측 문제와 일반 플릿 애플리케이션 또는 운영자 인터페이스의 결함을 분리하여 분석하는 데 도움이 된다.

많은 gRPC 문제가 프로토버프 페이로드(Protobuf payload) 자체에서 발생하는 것이 아니므로 메타데이터 검사(metadata inspection)가 중요하다. 인증 토큰(authentication token), 상관관계 식별자(correlation identifier), 테넌트 정보(tenant information), API 버전 또는 기타 문맥 정보가 RPC 메타데이터로 전달될 수 있다. grpcurl은 테스트 요청에 적절한 메타데이터를 추가할 수 있으므로 엔지니어는 장애가 인증, 인가(authorization), 라우팅, 애플리케이션 검증 또는 요청된 작업 자체에서 발생하는지를 판단할 수 있다.

TLS 구성(TLS configuration)은 또 다른 일반적인 디버깅 경계이다. 로봇 및 플릿 서비스는 암호화된 TLS 연결이나 양쪽 종단점이 모두 자격증명을 제공하는 상호 TLS(mutual TLS)를 사용할 수 있다. 따라서 진단 테스트에서는 인증서 검증 실패, 호스트 이름 문제, 신뢰 루트(trust root) 부재, 인증 거부 및 애플리케이션 수준 RPC 오류를 구분해야 한다. 검증을 일시적으로 우회하는 방식이 통제된 진단에 도움이 될 수 있지만 정상적인 운영 환경의 기본 설정으로 사용해서는 안 된다.

RPC 호출이 실패할 경우 gRPC 상태 코드(status code)는 구조화된 정보를 제공한다. 잘못된 인수(INVALID_ARGUMENT)는 잘못 구성된 요청 데이터를 나타낼 수 있고, 인증되지 않음(UNAUTHENTICATED)은 누락되거나 유효하지 않은 자격증명을 나타낼 수 있으며, 권한 거부(PERMISSION_DENIED)는 부족한 인가 권한을 의미할 수 있다. 사용 불가(UNAVAILABLE)는 서비스 또는 네트워크 문제를, 데드라인 초과(DEADLINE_EXCEEDED)는 시간 제약 또는 부하 문제를 나타낼 수 있다. 디버깅에서는 이러한 코드를 서버 로그 및 구조화된 오류 세부 정보와 함께 해석해야 한다.

스트리밍 RPC(streaming RPC)는 연결이 성공했다고 해서 스트림이 계속 정상적으로 유지되는 것은 아니므로 추가적인 주의가 필요하다. 엔지니어는 서버 스트리밍(server streaming) 메서드가 업데이트를 지속적으로 전달하는지, 스트림이 오류와 함께 종료되는지, 데드라인이나 네트워크 중단이 세션에 어떤 영향을 주는지 관찰해야 한다. 따라서 장시간 유지되는 로봇 텔레메트리(robot telemetry) 및 플릿 상태 스트림은 정상 운영뿐만 아니라 통신 장애 조건에서도 테스트해야 한다.

grpc-gateway는 서로 다르지만 상호보완적인 문제를 해결한다. grpc-gateway는 API 정의와 HTTP 매핑 규칙(mapping rule)에 따라 REST 방식의 HTTP 요청을 gRPC 호출로 변환하는 역방향 프록시 계층(reverse-proxy layer)을 생성할 수 있다. 따라서 외부 애플리케이션은 익숙한 HTTP/JSON 인터페이스를 통해 상호작용하면서 내부 백엔드 서비스는 강타입 gRPC API를 유지할 수 있다. 이는 로봇 플랫폼에서 내부 서비스 통신과 일반적인 웹 통합을 동시에 지원해야 하는 경우 유용하다.

게이트웨이(gateway)는 브라우저, 기업 시스템 또는 서드파티 클라이언트(third-party client)와 내부 로봇 서비스 사이에 배치할 수 있다. 게이트웨이가 수신한 HTTP 요청은 대응되는 gRPC 메서드로 매핑되고, JSON 데이터는 프로토버프 호환 메시지(Protobuf-compatible message)로 변환되며, gRPC 응답은 다시 HTTP 클라이언트에 적합한 형태로 변환된다. 이를 통해 모든 내부 로봇 서비스가 REST와 gRPC용 비즈니스 로직을 각각 구현하지 않고 통제된 프로토콜 경계(protocol boundary)를 구축할 수 있다.

게이트웨이 아키텍처(gateway architecture)를 디버깅하려면 변환 경계의 양쪽을 모두 확인해야 한다. 엔지니어는 실패한 작업이 HTTP 클라이언트, 게이트웨이 라우팅, JSON-프로토버프 변환(JSON-to-Protobuf conversion), 메타데이터 전달, gRPC 백엔드 또는 로봇 애플리케이션 중 어디에서 발생했는지를 판단해야 한다. grpcurl로 백엔드를 직접 테스트한 다음 대응되는 HTTP 종단점을 테스트하면 장애가 네이티브 서비스(native service)에 있는지 게이트웨이 통합 계층에 있는지를 분리할 수 있다.

HTTP 매핑(HTTP mapping)은 RPC 메서드와 REST 자원이 항상 동일한 의미체계를 갖는 것은 아니므로 신중하게 설계해야 한다. 임무 생성 RPC는 HTTP POST 작업과 자연스럽게 매핑될 수 있으며, 로봇 상태 조회는 GET에 대응할 수 있다. 그러나 복잡한 스트리밍 또는 양방향 상호작용은 일반적인 REST 동작으로 자연스럽게 변환되지 않을 수 있다. 따라서 게이트웨이는 모든 RPC를 기계적으로 변환하기보다 의미 있는 애플리케이션 의미체계(application semantics)를 유지하는 매핑을 제공해야 한다.

요청이 게이트웨이를 통과하는 환경에서는 로깅(logging)과 추적(tracing)이 특히 중요하다. 상관관계 식별자 또는 추적 식별자(trace identifier)는 HTTP 요청에서 시작하여 게이트웨이, gRPC 백엔드 및 하위 로봇 서비스까지 전달될 수 있다. 이를 통해 엔지니어는 클라이언트에서 확인된 오류를 게이트웨이 로그, RPC 상태 코드, 서비스 지연시간 및 로봇 측 이벤트와 연결할 수 있다. 일관된 식별자가 없다면 여러 분산 구성요소에 걸쳐 하나의 작업을 진단하는 과정이 훨씬 어려워진다.

체계적인 디버깅 작업 흐름(systematic debugging workflow)은 애플리케이션 의미체계를 검사하기 전에 연결성(connectivity)과 서비스 디스커버리(service discovery)를 확인하는 것에서 시작해야 한다. 엔지니어는 서버 접근 가능 여부를 확인하고, 예상 서비스를 검사하고, 최소 RPC를 호출하고, 인증을 검증한 다음 실제적인 메시지를 테스트할 수 있다. 게이트웨이가 존재한다면 동일한 논리적 작업을 네이티브 gRPC 경로와 HTTP 경로 모두에서 실행하여 동작이 달라지기 시작하는 계층을 식별할 수 있다.

개발 환경(development environment)과 운영 환경(production environment)에는 서로 다른 디버깅 정책이 필요할 수 있다. 리플렉션, 상세 로그(verbose log), 진단 종단점 및 완화된 테스트 자격증명은 개발 속도를 높일 수 있지만 외부에 노출되면 불필요한 정보를 제공할 수 있다. 운영 환경에서는 진단 접근을 제한하고 자격증명을 보호하며 로그를 정제하고 적절한 TLS 검증을 유지해야 한다. 디버깅 기능은 운영 보안 통제를 약화시키는 방식이 아니라 의도적으로 설계되어야 한다.

로봇 플릿 운영(robot fleet operation)에서는 반복 가능한 진단 명령(repeatable diagnostic command)과 알려진 테스트 요청을 사용하는 것이 유용하다. 팀은 로봇 상태 조회, 임무 제출, 상태 점검(health check), 서비스 검증을 위한 승인된 예제를 관리하여 개발자와 운영자가 동일한 API 동작을 테스트하도록 할 수 있다. 이러한 테스트는 배포 검증(deployment validation) 또는 지속적 통합(continuous integration)에 포함하여 새로운 서비스 버전이 실제 로봇 플릿에 적용되기 전에 통신 문제를 발견하는 데 활용할 수 있다.

따라서 grpcurl과 grpc-gateway는 동일한 gRPC 엔지니어링 작업 흐름에서 서로 다른 역할을 수행한다. grpcurl은 네이티브 gRPC 서비스를 직접 검사하고 호출하며 문제를 해결할 수 있는 가시성을 제공하고, grpc-gateway는 선택된 gRPC 기능을 HTTP/JSON 인터페이스를 통해 제공한다. 리플렉션 또는 디스크립터, 구조화된 상태 처리, TLS, 로깅, 추적 및 반복 가능한 테스트와 함께 사용하면 분산 로봇 API(distributed robot API)를 위한 실용적인 디버깅 및 통합 환경을 구축할 수 있다.
