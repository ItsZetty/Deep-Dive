# ID Generation Strategy In JPA

JPA에서는 ID를 생성하기 위해서 직접 할당과 자동 할당을 사용할 수 있습니다. 직접 할당은 `@Id`와 `@GeneratedValue`를 함께 사용해서 원하는 키 생성 전략을 선택하는 방식입니다.

`@GeneratedValue`의 strategy 옵션을 통해 생성 전략을 설정할 수 있는데, 여기에 올 수 있는 값인 GenerationType는 다음과 같습니다.

```java
@Target({ElementType.METHOD, ElementType.FIELD})  
@Retention(RetentionPolicy.RUNTIME)  
public @interface GeneratedValue {  
    GenerationType strategy() default GenerationType.AUTO;  
  
    String generator() default "";  
}

public enum GenerationType { 
	AUTO,
	IDENTITY,
	SEQUENCE, 
	TABLE
}
```

## 🤷🏻‍♂️ 자동 생성 방식을 사용할 때 각 전략에 대해서 설명해주세요.
**1️⃣ IDENTITY 전략**
IDENTITY 전략은 기본 키 생성을 DB에 위임하는 전략입니다. 

이 전략을 사용하면 엔티티를 생성할 때, 쓰기 지연이 적용되지 않습니다. 왜냐하면 JPA에서 엔티티를 영속하기 위해서는 식별자가 필요한데, IDENTITY 전략에서는 이 식별자가 데이터베이스에 저장되어야 할당되기 때문입니다.

따라서 엔티티를 생성할 때, 즉시 INSERT 쿼리가 실행되어야 합니다. 이때 하이버네이트를 사용하는 경우에는 INSERT 쿼리의 결과를 재조회하지 않기 위해서 `Statement.getGeneratedKeys`를 사용합니다.

>💡 **IDENTITY 사용하면, Batch INSERT는 불가능합니다.** <br>
> 각 INSERT 실행 시점에 즉시 PK를 생성해야 하기 때문에, 여러 INSERT 문을 한꺼번에 묶어서 처리하는 배치 INSERT가 불가능합니다.

**2️⃣ SEQUENCE 전략**
우선 SEQUENCE 전략은 SEQUENCE 키 생성 전략을 지원하는 DB에서 사용할 수 있습니다. 데이터베이스 시퀀스란, 유일한 값을 자동으로 생성하게 하는 객체입니다.

`auto_increment`와 달리 초기 값과 한번에 증가할 크기를 설정할 수 있습니다. 해당 시퀀스를 키 생성 전략으로 갖는 데이터베이스에 대해 SEQUENCE 전략을 사용할 수 있습니다.

어떤 SEQUENCE를 사용할 것인지를 `@SequenceGenerator`로 설정할 수 있습니다. SEQUENCE 전략은 `em.persist()`를 호출하는 경우 먼저 데이터베이스 시퀀스를 이용하여 식별자를 조회합니다. 이후 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장합니다.

트랜잭션을 커밋하여 플러시가 일어나면 엔티티를 저장한다는 점에서 IDENTITY 전략과 차이가 있습니다.

**3️⃣ TABLE 전략**
TABLE 전략은 키 생성 전용 테이블을 만들어 SEQUENCE를 흉내내는 전략입니다. 

어떤 테이블을 사용할 것인지를 `@TableGenerator`로 설정할 수 있습니다. TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하며 증가를 위해 UPDATE 쿼리를 사용합니다. SEQUENCE 전략보다 DB와 한번 더 통신한다는 점에서 성능이 안좋다는 단점이 있지만, 모든 데이터베이스에 적용할 수 있다는 장점이 있습니다.

**4️⃣ AUTO 전략**
AUTO 전략은 데이터베이스 방언에 따라서 IDENTITY, SEQUENCE, TABLE 중 하나를 자동으로 선택합니다. 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 장점이 있습니다.

>💡 **IDENTITY, SEQUENCE, TABLE 중 선택하는 기준** <br>
> 각 데이터베이스의 "가장 효율적이고 표준적인 ID 생성 방식"을 선택합니다. <br>
> 
> AUTO_INCREMENT 지원: IDENTITY 선택 <br>
> SEQUENCE 지원: SEQUENCE 선택 <br>
> 둘 다 지원: 해당 DB의 표준 방식 선택 <br>