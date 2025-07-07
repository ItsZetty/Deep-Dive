# Fetch Join
Fetch Join은 JPA에서 연관된 엔티티나 컬렉션을 한 번의 SQL로 즉시 함께 가져오기 위해 사용하는 JPQL의 특별한 JOIN 방식입니다.

```sql 
select p from Parent p join fetch p.children
```

이렇게 쓰면 Parent와 연관된 Children을 한 쿼리로 미리 당겨와(N+1 문제 예방), 영속성 컨텍스트에 적재해 둡니다.

## 🤷🏻‍♂️ 그럼 LAZY와 Fetch Join의 동작 차이는 어떻게 되나요?
LAZY는 조회 시점에 일단 Parent만 조회를 하고 Children은 프록시로 세팅합니다. 그리고 Children 필드 접근할 때마다 N+1 쿼리를 발생시킵니다. 

하지만 Fetch Join은 Parent를 조회할 때, SQL 조인을 해서 Children까지 한 번에 조회를 합니다. 그리고 추후 Children 필드를 접근하더라도 이미 조회된 상태이므로 추가 쿼리는 발생하지 않습니다.

즉, Fetch Join을 쓰면 Parent를 가져올 때 Children을 미리 끌고 오며, LAZY를 쓰면 Children은 호출할 때마다 쿼리를 날립니다.

## 🤷🏻‍♂️ 그럼 EAGER와 Fetch Join의 동작이 똑같은 거 아니야?
EAGER와 Fetch Join은 모두 연관 엔티티를 한 번에 가져오는 방식이지만, 동작 방식과 유연성에서 큰 차이가 있습니다.

EAGER는 엔티티에 `fetch = FetchType.EAGER`를 지정해 두면, 엔티티를 조회할 때마다 연관 엔티티를 항상 즉시 함께 조회합니다. 

이 방식은 개발자가 별도로 컨트롤할 수 있는 여지가 적고, 대부분의 경우 불필요한 데이터를 너무 일찍 조회하게 되므로 성능 문제가 생기기 쉽습니다.

하지만 Fetch Join은 JPQL에서 `JOIN FETCH`를 명시적으로 사용해, 원하는 조회 쿼리에서만 연관 엔티티를 함께 가져오는 방식입니다. 

호출 시점과 조회 대상을 유연하게 조절할 수 있어, 꼭 필요할 때만 조인을 수행하고 성능을 최적화할 수 있습니다. 

## 🤷🏻‍♂️ Fetch Join의 동작 방식에 대해 자세히 알고 싶습니다.
Fetch Join을 사용하면 JPA가 조인을 수행하고, 결과를 파싱해서 연관 엔티티를 부모 엔티티 안에 채웁니다. 그리고 연관 엔티티는 이미 영속 상태이므로 이후 호출할 때 쿼리가 발생하지 않게 됩니다.

## 🤷🏻‍♂️ 그럼 Fetch Join이 좋은 기능인데, 주의해야 할 부분이 있을까요?
먼저 Fetch Join의 장점은 위에서도 쭉 말해왔지만, N+1 문제를 해결합니다. 그리고 트랜잭션 내부에서 연관된 엔티티를 미리 조회하기 때문에 지연 초기화 문제가 없습니다.

하지만 아래와 같은 문제점도 분명히 있습니다.

- `@OneToMany` 컬렉션 Fetch Join + 페이징(setFirstResult, setMaxResults) -> DB 레벨 페이징 불가 (중복 Row 때문)
- Fetch Join은 한 쿼리에서 한 번만 사용할 수 있는 경우가 많음(Hibernate 한계)
- Fetch Join 남발하면 조인 수 증가 -> 쿼리 복잡, 성능 저하