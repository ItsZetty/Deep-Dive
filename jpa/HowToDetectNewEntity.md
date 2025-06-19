# How to detect new entity

Spring Data JPA에서 새로운 Entity인지 판단하는 방법은 무엇일까요?

Spring Data JPA에서 새로운 Entity 판단은 `JpaEntityInformation.isNew(T entity)`로 결정됩니다.
기본적으로 `JpaMetamodelEntityInformation`이 사용되며, `@Version` 필드 유무와 타입에 따라 판단 로직이 달라집니다.

## 🤷🏻‍♂️ 그럼 판단 로직은 어떻게 될까요?

### 1. @Version 필드가 없는 경우
`AbstractEntityInformation.isNew()` 호출합니다.

```java
public boolean isNew(T entity) {
    Id id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;  // ID가 null이면 새로운 엔티티
    }

    if (id instanceof Number) {
        return ((Number) id).longValue() == 0L;  // ID가 0이면 새로운 엔티티
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s", idType));
}
```

### 2. @Version 필드가 있는 경우
- **Primitive 타입**: `AbstractEntityInformation.isNew()` 호출합니다.
- **Wrapper class**: 해당 필드의 null 여부를 확인합니다.

```java
@Override
public boolean isNew(T entity) {
    if(versionAttribute.isEmpty()
          || versionAttribute.map(Attribute::getJavaType).map(Class::isPrimitive).orElse(false)) {
        return super.isNew(entity);
    }

    BeanWrapper wrapper = new DirectFieldAccessFallbackBeanWrapper(entity);
    return versionAttribute.map(it -> wrapper.getPropertyValue(it.getName()) == null).orElse(true);
}
```

###  3. save() 메서드에서의 활용

```java
@Override
@Transactional
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {
        entityManager.persist(entity);  // INSERT
        return entity;
    } else {
        return entityManager.merge(entity);  // SELECT 후 UPDATE
    }
}
```

## 🤷🏻‍♂️ 직접 ID를 할당하는 경우에는 어떻게 동작하나요?

직접 ID를 할당하는 경우 `Persistable<T>` 인터페이스를 구현하여 `JpaPersistableEntityInformation` 사용합니다.

```java
public class JpaPersistableEntityInformation<T extends Persistable<ID>, ID> 
        extends JpaMetamodelEntityInformation<T, ID> {

    @Override
    public boolean isNew(T entity) {
        return entity.isNew();  // 엔티티가 직접 판단
    }
}
```

## 🤷🏻‍♂️ 새로운 Entity인지 판단하는게 왜 중요할까요?

`SimpleJpaRepository`의 `save()` 메서드에서 `isNew()`를 사용하여 **persist**를 수행할지 **merge**를 수행할지 결정합니다.

### 성능 차이
- **persist()**: INSERT 쿼리만 실행 (효율적)
- **merge()**: SELECT 쿼리 후 UPDATE 쿼리 실행 (비효율적)

### 문제 상황
직접 ID를 할당하는 경우 새로운 엔티티로 판단되지 않아 `merge()`가 호출됩니다. 이때 신규 엔티티임에도 불구하고 DB를 조회하므로 성능상 비효율적입니다.

### 해결책
`Persistable<T>` 인터페이스를 구현하여 엔티티가 직접 새로운 엔티티 여부를 판단하도록 합니다.

## 💡 결론

- **성능 영향**: 새로운 엔티티 판단은 persist/merge 선택에 직접적 영향입니다.
- **@GeneratedValue**: 자동으로 효율적인 persist() 호출합니다.
- **직접 ID 할당**: Persistable 인터페이스 구현 필요합니다.
-  **@Version 필드**: 타입에 따라 판단 로직 변경합니다.