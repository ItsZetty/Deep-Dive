# First-Class Collection
일급 컬렉션(First-Class Collection)은 하나의 컬렉션을 감싸는 클래스를 만들고, 해당 클래스에서 컬렉션과 관련된 비즈니스 로직을 관리하는 패턴을 말합니다. 

아래 코드 중에서 Order의 List 자료구조를 감싼 Orders가 일급 컬렉션의 예시입니다.

```java
// 일급 컬렉션
public class Orders {

    private final List<Order> orders;

    public Orders(List<Order> orders) {
        validate(orders); // 검증 수행
        ...
    }

    public void add(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        orders.add(order);
    }

    public List<Order> getAll() {
        return Collections.unmodifiableList(orders);
    }

    public double getTotalAmount() {
        return orders.stream()
                     .mapToDouble(Order::getAmount)
                     .sum();
    }
}
```

```java
public class OrderService {
  
    private final Orders orders = new Orders();

    public void addOrder(Order order) {
        orders.add(order);
    }

    public Orders getOrders() {
        return orders;
    }

    // 추가 비즈니스 로직...
}
```

## 🤷🏻‍♂️ 흠.. 불편한데 왜 일급 컬렉션을 사용해야 하나요?
일급 컬렉션 클래스에 로직을 포함하거나 비즈니스에 특화된 명확한 이름을 부여할 수 있습니다.

또 불필요한 컬렉션 API를 외부로 노출하지 않도록 할 수 있으며, 컬렉션을 변경할 수 없도록 만든다면 예기치 않은 변경으로부터 데이터를 보호할 수 있습니다.

## 💡 결론
**👍🏻 장점**
- 응집도 향상: 컬렉션과 관련된 상태와 행위를 한 클래스 안에 모을 수 있습니다.
- 불변성 보장 용이: 외부에서 내부 컬렉션을 수정하지 않도록 캡슐화하기 좋습니다.
- 비즈니스 규칙 캡슐화: 중복 검사, 정렬, 유효성 검사 등 도메인 규칙을 한 곳에서 관리할 수 있습니다.
- 가독성 향상: 의미 있는 이름을 부여해 도메인 표현력이 좋아집니다.
- 재사용성 증가: 같은 성격의 컬렉션이 여러 곳에서 쓰일 경우, 한 번 정의해 두면 편리합니다.

**👎🏻 단점**
- 클래스 수 증가: 단순한 경우에도 별도의 클래스를 정의해야 하므로 구조가 복잡해질 수 있습니다.
- 과도한 래핑 가능성: 특별한 행위나 상태 없이 단순 래핑만 한다면 오히려 오버엔지니어링이 됩니다.
- JPA 연동 시 신경 쓸 부분 증가: 엔티티 내부에서 일급 컬렉션을 사용하면 별도의 컨버터나 매핑이 필요할 수 있습니다.