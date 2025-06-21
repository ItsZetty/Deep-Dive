# ScanBase
ScanBase에 대해 알아보기 전에 컴포넌트 스캔의 정의에 대해 먼저 알아보겠습니다.

컴포넌트 스캔은 스프링이 `@Component`, `@Service`, `@Repository`, `@Controller` 등의 어노테이션이 붙은 클래스들을 자동으로 찾아서 스프링 컨테이너에 빈으로 등록하는 기능입니다.

스프링은 기본적으로 `@SpringBootApplication`이 있는 클래스의 패키지와 그 하위 패키지들을 스캔 범위로 설정합니다.

하지만 `scanBasePackages`를 사용하게 된다면, 달라집니다.

ScanBase는 컴포넌트 스캔의 기준이 되는 패키지를 의미합니다. `@SpringBootApplication(scanBasePackages = "패키지명")`으로 설정하면, 해당 패키지와 그 하위 패키지들만 스캔 대상으로 지정됩니다.

## 🤷🏻‍♂️ 아직 이해가 어려운데, 예시 없을까요?

평소 프로젝트를 생성하게 별다른 설정을 하지 않았다면, 아래와 같은 코드를 볼 수 있습니다.

```java
@SpringBootApplication
public class ItemServiceApplication {
    // 기본적으로 ItemServiceApplication이 있는 패키지와 하위 패키지들을 스캔
}
```
이렇게 된다면, `ItemServiceApplication`이 위치한 패키지(예: com.example)와 그 하위 패키지들 전체를 스캔하게 됩니다. 

하지만.

아래 코드를 보면, "hello.itemservice.web"을 scanBasePackages로 지정하였습니다. 이렇게 된다면, "hello.itemservice.web" 패키지와 그 하위 패키지만 스캔을 하게 됩니다.

```java
@Import(MemoryConfig.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
public class ItemServiceApplication {
public static void main(String[] args) {
    SpringApplication.run(ItemServiceApplication.class, args);
    }
    @Bean
    @Profile("local")
    public TestDataInit testDataInit(ItemRepository itemRepository) {
    return new TestDataInit(itemRepository);
    }
}
```

## 💡 결론
scanBase를 활용한다면 아래와 같은 효과를 얻을 수 있습니다.

- 스캔 범위 제한: 불필요한 패키지 스캔을 방지하여 애플리케이션 시작 시간을 단축할 수 있습니다.
- 명확한 의존성: 어떤 패키지들이 스캔 대상인지 명시적으로 표현됩니다.
- 성능 최적화: 큰 프로젝트에서 특정 모듈만 스캔하고 싶을 때 유용합니다.
- 모듈화: 각 모듈별로 스캔 범위를 분리하여 관리할 수 있습니다.