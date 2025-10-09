# Micrometer

Micrometer는 벤더 중립적인 메트릭 계측 라이브러리로, 애플리케이션에서 발생하는 다양한 지표(예: CPU 사용량, 메모리 소비, HTTP 요청 및 커스텀 이벤트)를 수집합니다. 이 라이브러리는 Prometheus, Datadog, Graphite 등 여러 모니터링 시스템에 메트릭을 전송할 수 있도록 단순하고 일관된 API(파사드)를 제공하여, 각 백엔드 클라이언트의 복잡한 세부 구현을 감춥니다. 특히 Spring Boot Actuator와 깊이 통합되어, 기본 메트릭을 자동으로 수집하고 노출할 수 있습니다.

## 🤷🏻‍♂️ Spring Boot Actuator와 Micrometer의 관계는 무엇인가요?

Spring Boot Actuator는 애플리케이션의 상태, 헬스 체크, 환경, 로그 등 여러 운영 정보를 노출하는 관리 엔드포인트를 제공합니다. 내부적으로 Actuator는 Micrometer를 사용하여 JVM, HTTP, 데이터베이스 등 다양한 메트릭을 수집합니다. 즉, Actuator는 모니터링 및 관리 인터페이스를 제공하고, Micrometer는 그 밑에서 실제 메트릭 데이터를 계측하고 여러 모니터링 시스템으로 전송하는 역할을 담당합니다.

## 🤷🏻‍♂️ Micrometer를 사용하여 커스텀 메트릭을 생성하는 방법을 설명해주세요.

아래는 Micrometer를 활용하여 커스텀 메트릭(카운터, 타이머, 게이지)을 생성하고 업데이트하는 예제 코드입니다. 이 코드는 HTTP 요청의 건수와 처리 시간, 그리고 현재 활성 세션 수를 측정하는 예제입니다.

```java
@Service
public class CustomMetricsService {

    private final Counter requestCounter;
    private final Timer requestTimer;
    private final CustomGauge customGauge;

    // 생성자에서 MeterRegistry를 주입받아 필요한 메트릭을 등록합니다.
    public CustomMetricsService(MeterRegistry meterRegistry) {
        // HTTP 요청 총 건수를 세는 Counter (태그로 엔드포인트 구분)
        this.requestCounter = meterRegistry.counter("custom.requests.total", "endpoint", "/api/test");

        // HTTP 요청 처리 시간을 측정하는 Timer (태그로 엔드포인트 구분)
        this.requestTimer = meterRegistry.timer("custom.request.duration", "endpoint", "/api/test");

        // Gauge: 예를 들어, 현재 활성 세션 수를 측정하기 위한 커스텀 객체를 등록
        this.customGauge = new CustomGauge();
        Gauge.builder("custom.active.sessions", customGauge, CustomGauge::getActiveSessions)
                .tag("region", "us-east")
                .register(meterRegistry);
    }

    /**
     * 실제 비즈니스 로직을 실행할 때 요청 카운트와 처리 시간을 측정합니다.
     * @param requestLogic 실제 처리할 로직 (예: HTTP 요청 처리)
     */
    public void processRequest(Runnable requestLogic) {
        // 요청 수 증가
        requestCounter.increment();
        // 요청 처리 시간 측정
        requestTimer.record(requestLogic);
    }

    /**
     * 활성 세션 수 업데이트 (예를 들어, 로그인/로그아웃 이벤트에서 호출)
     * @param activeSessions 현재 활성 세션 수
     */
    public void updateActiveSessions(int activeSessions) {
        customGauge.setActiveSessions(activeSessions);
    }

    /**
     * 커스텀 Gauge의 값을 저장하는 내부 클래스.
     */
    private static class CustomGauge {
        // 현재 활성 세션 수를 저장 (volatile을 사용해 스레드 안정성 확보)
        private volatile double activeSessions = 0;

        public double getActiveSessions() {
            return activeSessions;
        }

        public void setActiveSessions(double activeSessions) {
            this.activeSessions = activeSessions;
        }
    }
}
```

### MeterRegistry

- 생성자에서 MeterRegistry를 주입받아 애플리케이션의 모든 메트릭을 중앙에서 관리하고, 설정된 모니터링 백엔드로 주기적으로 전송합니다.

### Counter

- requestCounter는 /api/test 엔드포인트에 대한 요청 건수를 카운트합니다. 매 HTTP 요청마다 increment() 호출로 증가시킵니다.

### Timer

- requestTimer는 HTTP 요청 처리 시간을 측정합니다. record() 메서드를 사용해 실제 로직 실행 시간을 기록합니다.

### Gauge

- customGauge는 현재 활성 세션 수를 측정하는 데 사용됩니다. Gauge는 항상 현재 상태를 조회하는 함수(getActiveSessions())를 호출하여 실시간 값을 반영합니다.