# Reflection

**Reflection**이란, 런타임에 클래스의 메타 정보를 조회하고 조작할 수 있는 기능입니다.

예를 들어 클래스 이름, 필드, 메서드, 생성자 정보를 가져오거나, 접근 제어자를 무시하고 private 필드나 메서드에도 접근할 수 있습니다. 

> **💡 핵심 포인트**: Reflection은 컴파일 타임이 아닌 **런타임**에 동적으로 클래스 정보를 다루는 기능입니다.

---

## 🤷🏻‍♂️ 언제 Reflection이 사용될까요?

### 1. JPA 엔티티 생성할 때, 사용됩니다.
JPA는 리플렉션으로 new 엔티티()를 호출해서 빈 객체를 만든 후, DB에서 조회한 필드 값을 해당 엔티티에 주입합니다.

JPA는 기본 생성자(no-args constructor)를 리플렉션으로 호출하여 엔티티 객체를 생성하고, `setter` 메서드나 필드에 직접 접근하여 데이터베이스에서 조회한 값을 주입합니다.

```java
// JPA 내부에서 일어나는 일 (의사 코드)
Class<?> entityClass = Class.forName("com.example.User");
Constructor<?> constructor = entityClass.getDeclaredConstructor();
Object entity = constructor.newInstance(); // 기본 생성자 호출

// 필드에 값 주입
Field idField = entityClass.getDeclaredField("id");
idField.setAccessible(true); // private 필드 접근 허용
idField.set(entity, 1L);
```

### 2. 프레임워크나 라이브러리에서 객체를 동적으로 생성할 때, 사용됩니다.

Spring Framework, Jackson, Gson 등 많은 프레임워크들이 런타임에 객체를 생성하거나 직렬화/역직렬화할 때 리플렉션을 사용합니다.

```java
// Spring의 Bean 생성 예시
Class<?> beanClass = Class.forName("com.example.Service");
Object bean = beanClass.getDeclaredConstructor().newInstance();

// Jackson JSON 역직렬화 예시
ObjectMapper mapper = new ObjectMapper();
User user = mapper.readValue(jsonString, User.class); // 내부적으로 리플렉션 사용
```

### 3. DI를 구현할 때, 사용됩니다.

의존성 주입 컨테이너는 리플렉션을 사용하여 클래스의 생성자나 필드에 `@Autowired` 같은 어노테이션이 붙은 의존성을 자동으로 주입합니다.

```java
// Spring DI 내부 동작 (의사 코드)
Class<?> serviceClass = Class.forName("com.example.UserService");
Constructor<?> constructor = serviceClass.getDeclaredConstructors()[0];
Parameter[] parameters = constructor.getParameters();

// 각 파라미터에 대해 적절한 의존성 주입
Object[] args = new Object[parameters.length];
for (int i = 0; i < parameters.length; i++) {
    Class<?> paramType = parameters[i].getType();
    args[i] = getBean(paramType); // 컨테이너에서 해당 타입의 빈 찾기
}

Object service = constructor.newInstance(args);
```

### 4. 객체의 타입을 모를 때, 사용됩니다.
객체의 타입을 모를 때, 클래스 정보 확인 / 동적으로 메서드 호출이 가능하며, 만약 클래스 이름만 알고 있다면 인스턴트 생성이 가능합니다.

```java
// 클래스 이름만 알고 있을 때 인스턴스 생성
String className = "com.example.UserService";
Class<?> clazz = Class.forName(className);
Object instance = clazz.getDeclaredConstructor().newInstance();

// 동적으로 메서드 호출
Method method = clazz.getDeclaredMethod("getUser", Long.class);
Object result = method.invoke(instance, 1L);
```

---

## 🤷🏻‍♂️ 단점은 없을까요?

### 1. 성능 오버헤드
- 리플렉션은 일반적인 메서드 호출보다 **느립니다**
- JVM 최적화가 어려워 성능에 영향을 줄 수 있습니다

### 2. 보안 위험
- `setAccessible(true)`로 private 멤버에 접근할 수 있어 캡슐화를 깨뜨릴 수 있습니다
- 보안 관리자(SecurityManager)가 이를 제한할 수 있습니다

### 3. 컴파일 타임 검증 불가
- 런타임에만 오류를 발견할 수 있어 디버깅이 어려울 수 있습니다

---

## 📚 추가 내용

### Reflection Methods
| 클래스 | 설명 |
|--------|------|
| **Class** | 클래스 정보를 담는 클래스 |
| **Field** | 필드 정보를 담는 클래스 |
| **Method** | 메서드 정보를 담는 클래스 |
| **Constructor** | 생성자 정보를 담는 클래스 |
| **Modifier** | 접근 제어자 정보를 담는 클래스 |

### JPA Entity 필드에 final를 쓰면 안 되는 이유
엔티티 필드를 `final`로 구성하면 JPA 동작 과정에서 문제가 발생합니다.

**문제점**
- Java에서 `final`은 불변을 의미합니다
- `final`로 구성된 필드는 생성 이후 값을 변경할 수 없습니다
- JPA는 리플렉션으로 빈 객체를 만든 후, DB에서 조회한 필드 값을 주입합니다
- `final` 필드는 값 변경이 불가능하여 JPA가 값을 주입하는 과정에서 예외가 발생하거나, 값이 정상적으로 초기화되지 않습니다

**결론** 
JPA 엔티티의 필드는 `final`로 선언하지 않는 것이 원칙입니다.

### 스프링 애플리케이션 동작 과정

1. **컴포넌트 스캔** → 리플렉션으로 어노테이션 분석
2. **빈 정의 생성** → 클래스 메타데이터 수집
3. **의존성 분석** → 생성자, 필드, 메서드 파라미터 분석
4. **빈 인스턴스 생성** → 리플렉션으로 객체 생성
5. **의존성 주입** → 리플렉션으로 값 주입
6. **초기화 콜백** → `@PostConstruct` 등 실행
7. **애플리케이션 시작** → 모든 빈이 준비된 상태

> **💡 성능 고려사항**: 스프링 부트 애플리케이션 시작이 상대적으로 느린 이유 중 하나가 바로 이 모든 과정에서 리플렉션을 사용하기 때문입니다.