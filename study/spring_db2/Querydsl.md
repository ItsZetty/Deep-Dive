# Querydsl

Querydsl은 정적 타입을 사용하여 SQL과 같은 쿼리를 생성할 수 있도록 지원하는 자바 프레임워크입니다.

- Querydsl을 사용하려면 `JPAQueryFactory`가 필요합니다. `JPAQueryFactory`는 JPA 쿼리인 JPQL을 만들기 때문에 `EntityManager`가 필요합니다.
- 설정 방식은 `JdbcTemplate`을 설정하는 것과 유사합니다.
- 참고로 `JPAQueryFactory`를 스프링 빈으로 등록해서 사용해도 됩니다.

`save()`, `update()`, `findById()`와 같은 기본 기능들은 JPA가 제공하는 기본 기능을 사용하면 됩니다.

하지만 동적 쿼리를 해결해야 할 때에는 JPA만으로는 어려울 수 있습니다. 그래서 Querydsl을 활용해서 해결하면 됩니다.

`BooleanBuilder`를 활용해서 `where` 조건들을 넣어주면 이를 해결할 수 있습니다. 이 모든 것이 자바 코드로 작성할 수 있기 때문에 동적 쿼리를 매우 편리하게 작성할 수 있습니다.

```java
List<Item> result = query
    .select(item)
    .from(item)
    .where(likeItemName(itemName), maxPrice(maxPrice))
    .fetch();
```

- 쿼리 문장에 오타가 있어도 컴파일 시점에 오류를 막을 수 있습니다.
- 메서드 추출을 통해서 코드를 재사용할 수 있습니다. 예를 들어서 여기서 만든 `likeItemName(itemName)` , `maxPrice(maxPrice)` 메서드를 다른 쿼리에서도 함께 사용할 수 있습니다.