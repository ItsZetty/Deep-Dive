# Event-Driven Architecture

이벤트(Event)란 시스템 내에서 발생한 사건 또는 변화를 의미합니다. 그리고 이벤트 처리는 이 사건이 발생했을 때, 그에 대응하는 비동기 처리 로직을 실행하는 구조를 말합니다.

즉, "주문이 생성되었을 때, 결제 처리하고 알림 보내고 포인트 적립도 하고 싶어!"라는 설계를 하게 된다면, 이벤트 기반 아키텍처가 적합합니다.

## 🤷🏻‍♂️ 이벤트 기반 아키텍처가 무엇인가요?

이벤트 기반 아키텍처란, 이벤트를 중심으로 구성된 시스템 설계 방식입니다. 구성 요소들은 이벤트를 발행(Publish)하고 구독(Subscribe)하며, 서로 직접적으로 의존하지 않고 메시지를 통해 느슨하게 연결됩니다.

## 🤷🏻‍♂️ 이벤트 기반 아키텍처의 구성 요소는 무엇인가요?

### 1. **이벤트 발행자 (Event Publisher)**
- 이벤트를 생성하고 발행하는 컴포넌트
- 예: 주문 서비스가 "주문 생성" 이벤트 발행

### 2. **이벤트 구독자 (Event Subscriber)**
- 이벤트를 수신하고 처리하는 컴포넌트
- 예: 결제 서비스, 알림 서비스, 포인트 서비스

### 3. **이벤트 브로커 (Event Broker)**
- 이벤트를 중계하는 메시지 큐 시스템
- 예: Kafka, RabbitMQ, Redis Pub/Sub

### 4. **이벤트 스토어 (Event Store)**
- 이벤트를 영구 저장하는 저장소
- 예: Event Sourcing 패턴에서 사용

## 🤷🏻‍♂️ 이벤트 기반 아키텍처의 장점은 무엇인가요?

### 1. **느슨한 결합 (Loose Coupling)**
- 서비스 간 직접적인 의존성 제거
- 독립적인 개발과 배포 가능

### 2. **확장성 (Scalability)**
- 각 서비스를 독립적으로 확장 가능
- 부하 분산 효과

### 3. **비동기 처리**
- 응답 시간 단축
- 시스템 안정성 향상

### 4. **유연성**
- 새로운 기능 추가 시 기존 코드 변경 없음
- 마이크로서비스 아키텍처에 적합

## 🤷🏻‍♂️ Spring에서 이벤트 기반 아키텍처를 어떻게 구현하나요?

### 1. **Spring Events (ApplicationEvent)**
```java
// 이벤트 클래스
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;
    
    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }
    
    public Order getOrder() {
        return order;
    }
}

// 이벤트 발행
@Component
public class OrderService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void createOrder(Order order) {
        // 주문 생성 로직
        eventPublisher.publishEvent(new OrderCreatedEvent(this, order));
    }
}

// 이벤트 구독
@Component
public class PaymentService {
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        // 결제 처리 로직
    }
}
```

### 2. **Spring Cloud Stream + Kafka**
```java
// 이벤트 발행
@Component
public class OrderEventPublisher {
    @Autowired
    private StreamBridge streamBridge;
    
    public void publishOrderCreated(Order order) {
        streamBridge.send("order-created", order);
    }
}

// 이벤트 구독
@Component
public class PaymentEventHandler {
    @Bean
    public Consumer<Order> handleOrderCreated() {
        return order -> {
            // 결제 처리 로직
        };
    }
}
```

## 🤷🏻‍♂️ 실제 사용 예시는 무엇인가요?

### 1. **E-commerce 시스템**
- 주문 생성 → 결제, 재고 감소, 알림 발송
- 결제 완료 → 배송 준비, 포인트 적립

### 2. **소셜 미디어**
- 게시물 작성 → 알림 발송, 피드 업데이트
- 댓글 작성 → 알림 발송, 통계 업데이트

### 3. **IoT 시스템**
- 센서 데이터 수신 → 분석, 알림, 저장
- 디바이스 상태 변경 → 모니터링, 로그 기록

## 🤷🏻‍♂️ 이벤트 기반 아키텍처의 단점은 무엇인가요?

### 1. **복잡성 증가**
- 이벤트 흐름 추적 어려움
- 디버깅 복잡성

### 2. **일관성 보장 어려움**
- 분산 트랜잭션 관리 복잡
- 이벤트 순서 보장 어려움

### 3. **성능 오버헤드**
- 메시지 브로커 추가로 인한 지연
- 네트워크 오버헤드

### 4. **모니터링 어려움**
- 분산 환경에서 추적 복잡
- 장애 지점 파악 어려움

## 🤷🏻‍♂️ 여러 이벤트 브로커들을 상황에 따라 비교해 주세요.

| 브로커 | 성능 | 순서 보장 | 내구성 | 사용 시나리오 |
|--------|------|-----------|--------|---------------|
| **Kafka** | 매우 높음 | ✅ | 매우 높음 | 대용량 실시간 스트리밍, 로그 수집 |
| **RabbitMQ** | 높음 | ✅ | 높음 | 메시지 큐, 워크로드 분산 |
| **Redis Pub/Sub** | 매우 높음 | ❌ | 낮음 | 실시간 알림, 임시 이벤트 |
| **Apache Pulsar** | 높음 | ✅ | 높음 | 멀티테넌트, 클라우드 네이티브 |
| **Amazon SQS** | 중간 | ❌ | 높음 | AWS 환경, 간단한 메시징 |

**선택 가이드:**
- **대용량 실시간**: Kafka
- **안정적인 메시징**: RabbitMQ  
- **간단한 실시간**: Redis Pub/Sub
- **클라우드 환경**: Apache Pulsar
- **AWS 서비스**: Amazon SQS