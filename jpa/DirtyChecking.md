# Dirty Checking

더티 체킹(Dirty Checking)은 영속성 컨텍스트가 관리 중인 엔티티의 변경을 자동으로 감지하고, 트랜잭션 커밋 또는 flush 시점에 데이터베이스에 SQL UPDATE를 전송하는 기능입니다.

개발자는 update()나 별도의 호출 없이, 단순히 엔티티 필드를 수정하기만 하면 됩니다.

## 🤷🏻‍♂️ 어떻게 동작하나요?
영속성 컨텍스트에 등록된 엔티티를 스냅샷(원본 상태)과 비교하여 바뀐 부분을 추적합니다.
그리고 트랜잭션이 커밋되거나 수동 flush()가 호출될 때 변경 내역을 SQL로 생성해 DB에 반영합니다.

## 🤷🏻‍♂️ 트리거되는 시점은 어떻게 되나요?
더티 체킹 자체가 SQL을 보내는 건 아니며, flush가 호출될 때 트리거됩니다.

**flush 호출되는 경우**
- 트랜잭션 커밋 직전에 자동 호출
- JPQL 또는 Native Query 실행 전
- 개발자가 명시적으로 entityManager.flush() 호출

```java 
// 트랜잭션 시작
Member member = em.find(Member.class, 1L); // 영속 상태
member.setName("홍길동"); // 엔티티 상태만 수정

// 트랜잭션 커밋 시, flush 호출 → UPDATE SQL 실행
transaction.commit();
```

## 💡 결론
트랜잭션 커밋 시 JPA는 무조건 flush를 호출하고, flush 과정에서 더티체킹(스냅샷과 엔티티 상태 비교)을 통해 변경된 엔티티만 DB에 반영한다.

flush 호출 → 더티 체킹 수행 → 변경된 내용 있으면 SQL 실행 → 트랜잭션 커밋 완료