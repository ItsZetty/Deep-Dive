# Equals and HashCode
`equals`와 `hashCode` 메서드는 객체의 동등성 비교와 해시값 생성을 위해서 사용할 수 있습니다. 

하지만 함께 재정의하지 않는다면, 예상치 못한 결과를 만들 수 있습니다. 예를 들어, 해시값을 사용하는 자료구조(HashSet, HashMap..)를 사용할 때 문제가 발생할 수 있습니다.

## 🤷🏻‍♂️ 왜 이런 현상이 발생하나요?

```java
class EqualsHashCodeTest {

    @Test
    @DisplayName("equals만 정의하면 HashSet이 제대로 동작하지 않는다.")
    void test() {
        // 아래 2개는 같은 구독자
        Subscribe subscribe1 = new Subscribe("team.maeilmail@gmail.com", "backend");
        Subscribe subscribe2 = new Subscribe("team.maeilmail@gmail.com", "backend");
        HashSet<Subscribe> subscribes = new HashSet<>(List.of(subscribe1, subscribe2));

        // 결과는 1개여야하는데..? 2개가 나온다.
        System.out.println(subscribes.size());
    }

    class Subscribe {

        private final String email;
        private final String category;

        public Subscribe(String email, String category) {
            this.email = email;
            this.category = category;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Subscribe subscribe = (Subscribe) o;
            return Objects.equals(email, subscribe.email) && Objects.equals(category, subscribe.category);
        }
    }
}
```

해시값을 사용하는 자료구조는 `hashCode` 메서드의 반환값을 사용합니다. `hashCode` 메서드의 반환 값이 일치한 이후, `equals` 메서드의 반환값 참일 때만 논리적으로 같은 객체라고 판단합니다. 

위 예제에서 `Subscribe` 클래스는 `hashCode` 메서드를 재정의하지 않았기 때문에 `Object` 클래스의 기본 `hashCode` 메서드를 사용합니다. `Object` 클래스의 기본 `hashCode` 메서드는 객체의 고유한 주소를 사용하기 때문에 객체마다 다른 값을 반환합니다. 

따라서 2개의 `Subscribe` 객체는 다른 객체로 판단되었고, `HashSet`에서 중복 처리가 되지 않았습니다.

## 🤷🏻‍♂️ 그럼 일치시키려면 어떻게 해야 하나요?

`hashCode` 메서드도 함께 재정의해야 합니다.

```java
class Subscribe {
    private final String email;
    private final String category;

    public Subscribe(String email, String category) {
        this.email = email;
        this.category = category;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Subscribe subscribe = (Subscribe) o;
        return Objects.equals(email, subscribe.email) && Objects.equals(category, subscribe.category);
    }

    @Override
    public int hashCode() {
        return Objects.hash(email, category);
    }
}
```

`equals`에서 비교하는 필드와 동일한 필드를 `hashCode`에서도 사용해야 합니다.

이제 `HashSet`에서 중복 처리가 정상적으로 동작합니다.

## 🤷🏻‍♂️ 그럼 모든 코드에 `equals`와 `hashCode`를 재정의하면 되나요?

꼭 그렇지는 않습니다. 상황에 따라 다릅니다.

### JPA 엔티티
```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    
    // equals/hashCode 재정의 권장하지 않음
    // 프록시 객체, 지연 로딩 문제 발생 가능
}
```

JPA 엔티티에서 `equals/hashCode` 재정의를 피해야 하는 이유는 아래와 같습니다.

### **1. 프록시 객체 문제**
JPA는 지연 로딩을 위해 프록시 객체를 사용합니다. 프록시 객체는 실제 데이터가 로딩되지 않은 상태에서 `equals` 비교를 하면 예상과 다른 결과가 발생할 수 있습니다. `em.getReference()`로 가져온 프록시 객체와 실제 엔티티를 비교할 때 문제가 됩니다.

### **2. 지연 로딩(Lazy Loading) 문제**
연관관계 필드가 지연 로딩 상태일 때 `equals` 메서드가 호출되면, 아직 로딩되지 않은 연관 객체를 강제로 로딩하려고 시도하면서 `LazyInitializationException`이 발생할 수 있습니다. 또 불필요한 데이터베이스 쿼리가 발생하여 성능에 영향을 줄 수 있습니다.

### **3. 세션 간 엔티티 비교 문제**
서로 다른 `EntityManager`에서 로딩된 엔티티들을 비교할 때 문제가 발생할 수 있습니다. 세션이 닫힌 후 equals 비교를 하거나, 다른 세션에서 로딩된 같은 ID의 엔티티를 비교할 때 예상과 다른 결과가 나올 수 있습니다.

### **4. 데이터베이스 상태와 불일치**
아직 데이터베이스에 저장되지 않은 엔티티의 `equals` 비교 시, 데이터베이스에 저장된 값과 다를 수 있습니다. 특히 ID가 자동 생성되는 경우, 저장 전후로 `equals` 결과가 달라질 수 있습니다.

### **5. 성능 문제**
`equals/hashCode`에서 모든 필드를 비교하면, 연관관계가 많은 엔티티의 경우 불필요한 데이터베이스 쿼리가 발생할 수 있습니다. 특히 컬렉션에 엔티티를 저장할 때 `hashCode`가 호출되면서 예상치 못한 쿼리가 실행될 수 있습니다.

### **권장사항**

**JPA 엔티티의 경우** : `equals/hashCode`를 재정의하지 않고 `Object` 클래스의 기본 구현을 사용합니다.

**ID 기반 비교가 필요한 경우** : 별도의 메서드를 제공합니다. 예를 들어 `hasSameId()` 같은 메서드를 만들어 ID만으로 비교합니다.

**DTO, Value Object의 경우** : `Lombok`의 `@EqualsAndHashCode` 어노테이션이나 `Java 14+`의 `Record`를 사용하여 자동으로 `equals`와 `hashCode`를 생성합니다.


JPA 엔티티는 프록시 객체, 지연 로딩, 세션 관리 등 JPA의 특수한 특성 때문에 `equals/hashCode` 재정의를 피해야 합니다. 대신 ID 기반 비교가 필요하다면 별도 메서드를 제공하고, `DTO`나 `Value Object`에서는 `Lombok`이나 `Record`를 활용하는 것이 안전합니다.