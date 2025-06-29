# Test

데이터베이스 접근과 관련된 테스트 코드는 어떤 식으로 세팅을 해야 하는지 알아보겠습니다.

우선 테스트는  `src/test` 에 있기 때문에, 실행하면 `src/test` 에 있는 `application.properties` 파일이 우선순위를 가지고 실행됩니다. 

그런데 문제는 테스트용 설정에는 `spring.datasource.url` 과 같은 데이터베이스 연결 설정이 없다는 점입니다.

그렇기 떄문에 테스트도 DB에 접근할 수 있게 세팅해 주어야 합니다.

```java
@SpringBootTest
class ItemRepositoryTest {}
```

위 코드에서 `@SpringBootTest`는 `@SpringBootApplication` 를 찾아서 설정으로 사용합니다.

이렇게 되면, 기본 SpringBootApplication에서 사용하는 DB에 접근할 수 있습니다.

하지만 이렇게 되면 실제 Local 환경 DB 데이터와 Test 환경 데이터가 합쳐지게 됩니다. 그래서 조회와 같은 테스트를 했을 때, 원하는 결과값이 나오지 않게 됩니다.

## 🤷🏻‍♂️ 그럼 Local 환경과 Test 환경 각각 분리를 해야 하나요?
분리를 하는 것이 좋습니다.

H2 데이터베이스를 2가지로 구분하면 됩니다.
- `jdbc:h2:tcp://localhost/~/test` : local에서 접근하는 서버 전용 데이터베이스
- `jdbc:h2:tcp://localhost/~/testcase` : test 케이스에서 사용하는 전용 데이터베이스

**main - application.properties**
```properties
spring.profiles.active=local
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
```

**test - application.properties**
```properties
spring.profiles.active=test
spring.datasource.url=jdbc:h2:tcp://localhost/~/testcase
spring.datasource.username=sa
```

하지만, 또 이렇게 진행을 하면 각각의 테스트에서 발생하는 데이터 저장이 쌓이게 되어 서로 테스트마다 영향을 받게 됩니다.

그래서 아래 2가지의 원칙을 세울 수 있습니다.

>**1. 테스트는 다른 테스트와 격리해야 한다.**<br>
>**2. 테스트는 반복해서 실행할 수 있어야 한다.**

## 🤷🏻‍♂️ 2가지 원칙을 어떻게 지킬 수 있나요?
이를 해결하기 위해 **"트랜잭션과 롤백 전략"**을 사용할 수 있습니다.

```java
@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository itemRepository;

    //트랜잭션 관련 코드
    @Autowired
    PlatformTransactionManager transactionManager;
    TransactionStatus status;

    @BeforeEach
    void beforeEach() {
        //트랜잭션 시작
        status = transactionManager.getTransaction(new
        DefaultTransactionDefinition());
    }
    @AfterEach
    void afterEach() {
        //MemoryItemRepository 의 경우 제한적으로 사용
        if (itemRepository instanceof MemoryItemRepository) {
        ((MemoryItemRepository) itemRepository).clearStore();
        }
        //트랜잭션 롤백
        transactionManager.rollback(status);
    }
    //...
}
```

각각의 테스트를 실행할 때, 트랜잭션이 생성되며 종료 시에는 롤백이 되는 형태로 유지할 수 있습니다.

하지만 너무 코드가 복잡해 보입니다.

## 🤷🏻‍♂️ 코드를 깔끔하게 할 수 없을까요?
SpringBoot에서  `@Transactional` 애노테이션 하나로 이를 깔끔하게 해결해줍니다.

```java
@Transactional
@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository itemRepository;

    @AfterEach
    void afterEach() {
        //MemoryItemRepository 의 경우 제한적으로 사용
        if (itemRepository instanceof MemoryItemRepository) {
        ((MemoryItemRepository) itemRepository).clearStore();
        }
    }
    //...
}
```

## 🤷🏻‍♂️ `@Transactional`의 원리가 어떻게 되나요?
스프링이 제공하는 `@Transactional` 애노테이션은 로직이 성공적으로 수행되면 커밋하도록 동작합니다. 그런데 `@Transactional` 애노테이션을 테스트에서 사용하면 아주 특별하게 동작합니다.

`@Transactional` 이 테스트에 있으면 스프링은 테스트를 트랜잭션 안에서 실행하고, 테스트가 끝나면 트랜잭션을 자동으로 롤백시켜 버립니다.

## 🤷🏻‍♂️ 그럼 테스트를 진행하면서 데이터를 직접 확인하고 싶다면 어떻게 해야 하나요?
`@Transactional`을 사용하면 롤백이 되어버리기 때문에 눈으로 확인이 어렵습니다. 확인을 위해서는 commit을 시켜버려야 합니다.

```java
@Commit
@Transactional
@SpringBootTest
class ItemRepositoryTest {}
```

`@Commit` 말고, `@Rollback(value = false)`를 사용해도 됩니다.

## 🤷🏻‍♂️ 테스트 작업을 위해서 데이터베이스를 세팅하는 데에 너무 많은 시간을 소모하는 거 아닌가요?
그래서 "임베디드 모드 DB"라는 개념이 존재합니다.

H2 데이터베이스는 자바로 개발되어 있고, JVM안에서 메모리 모드로 동작하는 특별한 기능을 제공합니다. 그래서 애플리케이션을 실행할 때 H2 데이터베이스도 해당 JVM 메모리에 포함해서 함께 실행할 수 있습니다. 

DB를 애플리케이션에 내장해서 함께 실행한다고 해서 임베디드 모드(Embedded mode)라 합니다.

```java
@Slf4j
//@Import(MemoryConfig.class)
//@Import(JdbcTemplateV1Config.class)
//@Import(JdbcTemplateV2Config.class)
@Import(JdbcTemplateV3Config.class)
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

    @Bean
    @Profile("test")
    public DataSource dataSource() {
        log.info("메모리 데이터베이스 초기화");
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }
}
```

그리고 테이블 생성을 위해 아래 스크립트를 추가합니다.

```sql
drop table if exists item CASCADE;
create table item
(
    id bigint generated by default as identity,
    item_name varchar(10),
    price integer,
    quantity integer,
    primary key (id)
);
```

## 🤷🏻‍♂️ 그런데 너무 복잡한 것 같습니다. 편리하게 사용 불가능할까요?
SpringBoot는 개발자에게 정말 많은 편리함을 제공하는데, 임베디드 데이터베이스에 대한 설정도 기본으로 제공합니다.

SpringBoot는 데이터베이스에 대한 별다른 설정이 없으면 임베디드 데이터베이스를 사용합니다.