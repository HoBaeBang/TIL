### Context
Spring Boot 기반 서버 개발 시 로컬, 개발(Dev), 운영(Prod) 등 다양한 환경(Environment)에 따라 다른 로직을 수행해야 하는 경우가 빈번합니다. 단순한 `if-else` 분기 처리는 코드 복잡도를 높이고 유지보수성을 저하시킵니다. 이를 해결하기 위해 인터페이스와 추상 클래스를 활용하여 공통 로직을 템플릿화하고, Spring Profile 기능을 통해 환경별로 최적화된 구현체를 주입받는 확장성 있는 아키텍처 설계가 필요합니다.

### Core
전략 패턴(Strategy Pattern)과 템플릿 메서드 패턴(Template Method Pattern)을 결합하여 환경별 구현체를 분리하는 예시 코드입니다.

*   **Step 1: Interface 정의** (계약 정의)
```java
public interface PaymentService {
    void processPayment(long amount);
}
```

*   **Step 2: Abstract Class 설계** (공통 로직 템플릿화)
```java
public abstract class AbstractPaymentService implements PaymentService {
    @Override
    public void processPayment(long amount) {
        validate(amount); // 공통 검증 로직
        doProcess(amount); // 환경별 특화 로직 (추상 메서드)
        logSuccess(amount); // 공통 로깅
    }

    protected void validate(long amount) {
        if (amount <= 0) throw new IllegalArgumentException("Invalid amount");
    }

    protected abstract void doProcess(long amount);

    private void logSuccess(long amount) {
        System.out.println("Payment processed successfully: " + amount);
    }
}
```

*   **Step 3: Environment별 구현체** (@Profile 활용)
```java
@Component
@Profile("dev")
public class DevPaymentService extends AbstractPaymentService {
    @Override
    protected void doProcess(long amount) {
        // 개발 환경: 가상 게이트웨이 호출 또는 로그 처리
        System.out.println("[DEV] Mock Payment API Call for " + amount);
    }
}

@Component
@Profile("prod")
public class ProdPaymentService extends AbstractPaymentService {
    @Override
    protected void doProcess(long amount) {
        // 운영 환경: 실제 PG사 API 연동
        System.out.println("[PROD] Real PG API Call for " + amount);
    }
}
```

### Insight
*   **개방-폐쇄 원칙(OCP) 준수**: 새로운 환경(예: QA, Staging)이 추가되더라도 기존 코드를 수정하지 않고 새로운 클래스를 확장하여 대응할 수 있습니다.
*   **템플릿 메서드 패턴의 이점**: 추상 클래스에서 `processPayment`의 전체 흐름을 제어함으로써, 구현체는 핵심 비즈니스 로직인 `doProcess`에만 집중할 수 있어 코드 중복을 방지합니다.
*   **Spring Profile 기반의 결합도 낮추기**: 클라이언트 코드(Service를 사용하는 쪽)는 구체적인 클래스가 아닌 `PaymentService` 인터페이스에만 의존하며, Spring 컨테이너가 런타임에 활성화된 프로파일에 맞춰 적절한 Bean을 주입(Dependency Injection)하므로 결합도가 낮아집니다.
*   **기술적 보강**: 복잡한 조건이 필요한 경우 `@Profile` 외에 `@ConditionalOnProperty`를 사용하여 특정 설정값의 유무에 따라 구현체를 선택적으로 활성화할 수도 있습니다.

**출처:**
*   [Spring @Profile Annotation - Baeldung](https://www.baeldung.com/spring-profiles)
*   [How @Profile Powers Multi-Environment Spring Boot Apps - Medium](https://medium.com/@gaddamnaveen192/how-profile-powers-multi-environment-spring-boot-apps-8128d3bf2f41)
*   [Core Technologies: Environment Abstraction - Spring Docs](https://docs.spring.io/spring-framework/reference/core/beans/environment.html)