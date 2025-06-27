# EntityManager

EntityManger 개념을 알기 전에 영속성 컨텍스트에 대해 짚고 넘어가야 합니다. 

## 🤷🏻‍♂️ 영속성 컨텍스트가 무엇인가요?
영속성 컨텍스트는 엔티티를 영구 저장하는 환경으로 1차 캐싱, 쓰기 지연, 변경 감지를 통해 영속 로직을 효율적으로 할 수 있게 해줍니다. 이러한 효율적인 영속 로직 수행을 위해서 엔티티는 영속성 컨텍스트에 관리되어야 합니다. 

## 🤷🏻‍♂️ 그럼 영속성 컨텍스트 안에 엔티티는 영구적으로 계속 남아있나요?
영속성 컨텍스트 안의 엔티티는 컨텍스트 수명 주기에 따라 존재하고 사라집니다. 무한히 남아있지 않고, 트랜잭션의 수명과 함께 끝난다고 생각하면 됩니다.

- EntityManager의 수명에 종속
    - 트랜잭션 범위(@Transactional)가 끝나면 트랜잭션을 관리하던 EntityManager가 닫히며 → 내부 컨텍스트가 비워짐
- 개발자가 명시적으로 정리
    - entityManager.clear() 호출 → 컨텍스트 비움
    - entityManager.detach(entity) 호출 → 특정 엔티티를 컨텍스트에서 분리

## 🤷🏻‍♂️ 그럼 EntityManger랑은 무슨 연관이 있나요?
이런 작업을 도와주는 것이 바로 EntityManger입니다. EntityManger는 엔티티의 상태를 변경하고, 영속성 컨텍스트와 상호작용함으로써 영속 로직을 수행하는 역할을 가지고 있습니다.

엔티티는 영속성 컨텍스트와 관련하여 4가지 상태를 가집니다.
1. 비영속 
2. 영속
3. 준영속
4. 삭제

이에 대해 EntityManger는 `persist`, `merge`, `remove`, `close` 메서드를 이용하여 엔티티의 상태를 변경할 수 있습니다. 

그리고 엔티티 매니저는 영속성 컨텍스트의 1차 캐시로부터 엔티티를 조회할 수 있으며, 쓰기 지연 저장소에 있는 쿼리들을 flush하여 DB와 동기화시킬 수 있습니다. 또한 JPQL이나 Native Query를 이용해 직접 DB로부터 데이터를 불러올 수도 있습니다.

## 🤷🏻‍♂️ EntityManger의 각 메서드에 대해서 설명해주세요.
**1️⃣ persist(entity)**
```java 
entityManager.persist(member); // 컨텍스트에 등록
```
비영속(new) 상태의 엔티티를 영속(managed) 상태로 전환합니다.

**2️⃣ find(entityClass, id)**
```java 
Member member = entityManager.find(Member.class, 1L);
```
엔티티를 PK로 조회합니다.

**3️⃣ getReference(entityClass, id)**
```java 
Member member = entityManager.getReference(Member.class, 1L); // 아직 쿼리 안 나감
```
프록시(지연 로딩용 엔티티) 반환합니다.

**4️⃣ remove(entity)**
```java 
entityManager.remove(member);
```
영속 상태 엔티티를 삭제(removed) 상태로 전환합니다.

**5️⃣ merge(entity)**
```java 
Member managed = entityManager.merge(detachedMember);
```
준영속(detached) 상태의 엔티티를 새 영속 상태 엔티티로 복사합니다.

**6️⃣ detach(entity)**
```java 
entityManager.detach(member); // 이 엔티티만 관리 X
```
특정 엔티티를 컨텍스트에서 분리합니다.

**7️⃣ clear()**
```java 
entityManager.clear(); // 컨텍스트 초기화
```
컨텍스트를 완전히 초기화 합니다.

**8️⃣ flush()**
```java 
entityManager.flush();
```
컨텍스트의 변경 내용을 DB에 즉시 반영합니다.

## 🤷🏻‍♂️ 엔티티의 각 상태에 대해서 설명해주세요.

**1️⃣ 비영속 상태**
```java 
Member member = new Member("산초");
```

비영속 상태는 엔티티 객체가 새로 생성되었지만, 아직 영속성 컨텍스트와 연관되지 않은 상태입니다. 이 상태에서는 데이터베이스와 전혀 관련이 없으며, 엔티티 객체는 메모리 상에만 존재합니다.

**2️⃣ 영속 상태**
```java 
em.persist(member);
em.merge(detagedMember);
em.find(Member.class, 1L);
```

영속 상태는 엔티티 객체가 영속성 컨텍스트에 관리되고 있는 상태입니다. 이 상태에서는 엔티티의 변경 사항이 자동으로 데이터베이스에 반영됩니다.

**3️⃣ 준영속 상태**
```java
em.detach(member);
em.clear();
em.close();
```

준영속 상태는 엔티티 객체가 한 번 영속성 컨텍스트에 의해 관리되었지만, 현재는 영속성 컨텍스트와 분리된 상태입니다. 이 상태에서는 엔티티 객체의 변경 사항이 더 이상 데이터베이스에 반영되지 않습니다. 영속성 컨텍스트 종료, 트랜잭션 종료 등으로도 준영속 상태로 전환됩니다.

**4️⃣ 삭제 상태**
```java 
em.remove(member);
```

삭제 상태는 엔티티 객체가 영속성 컨텍스트에서 제거된 상태입니다. 이 상태에서는 엔티티 객체가 데이터베이스에서 삭제됩니다.