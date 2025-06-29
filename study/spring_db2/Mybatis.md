# Mybatis
MyBatis는 앞서 설명한 JdbcTemplate보다 더 많은 기능을 제공하는 SQL Mapper입니다.
기본적으로 JdbcTemplate이 제공하는 대부분의 기능을 제공합니다. 또 SQL을 XML에 편리하게 작성할 수 있습니다.

그리고 JdbcTemplate에서 발생한 단점인 "동적 쿼리"를 해결할 수 있습니다.

JdbcTemplate와 Mybatis 코드를 비교해서 확인해 보겠습니다.

**JdbcTemplate**
```java
String sql = "update item " +
    "set item_name=:itemName, price=:price, quantity=:quantity " +
    "where id=:id";
```

**Mybatis**
```xml
<update id="update">
    update item
    set item_name=#{itemName},
    price=#{price},
    quantity=#{quantity}
    where id = #{id}
</update>
```

위에서 보면 알 수 있듯이 MyBatis는 XML에 작성하기 때문에 라인이 길어져도 문자 더하기에 대한 불편함이 없습니다.

## 🤷🏻‍♂️ 동적 쿼리는 어떻게 해결하나요?

```xml
<select id="findAll" resultType="Item">
    select id, item_name, price, quantity
    from item
    <where>
        <if test="itemName != null and itemName != ''">
        and item_name like concat('%',#{itemName},'%')
        </if>
        <if test="maxPrice != null">
        and price &lt;= #{maxPrice}
        </if>
    </where>
</select>
```

동적 쿼리는 위 코드처럼 if를 통해 작성할 수 있습니다.

## 🤷🏻‍♂️ 그럼 Mybatis는 JdbcTemplate에 비해 장점만 존재하나요?
단점도 물론 존재합니다. JdbcTemplate과 비교했을 때, 설정 부분에서 불편함이 있습니다.

우선 의존성을 `build.gradle`에 추가해 줘야 합니다.

그리고 설정 파일에 아래와 같이 코드를 추가해 주어야 사용이 가능합니다.

**`main - application.properties`**
```properties
#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

`test - application.properties`
```properties
#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

`mybatis.type-aliases-package` : 마이바티스에서 타입 정보를 사용할 때는 패키지 이름을 적어주어야 하는데, 여기에 명시하면 패키지 이름을 생략할 수 있습니다.

`mybatis.configuration.map-underscore-to-camel-case` : 언더바를 카멜로 자동 변경해주는 기능을 활성해 줍니다.

`logging.level.hello.itemservice.repository.mybatis=trace` : MyBatis에서 실행되는 쿼리 로그를 확인할 수 있습니다.

## 🤷🏻‍♂️ 어떻게 Mybatis를 사용하나요?
Mybatis 매핑 XML을 호출해주는 Mapper Interface를 구현해야 합니다.

**ItemMapper**
```java
@Mapper
public interface ItemMapper {
    void save(Item item);
    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto
    updateParam);
    Optional<Item> findById(Long id);
    List<Item> findAll(ItemSearchCond itemSearch);
}
```
그리고 `src/main/resources` 하위 패키지에 XML 파일을 생성하면 됩니다.

**ItemMapper.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">
    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        insert into item (item_name, price, quantity)
        values (#{itemName}, #{price}, #{quantity})
    </insert>

    <update id="update">
        update item
        set item_name=#{updateParam.itemName},
        price=#{updateParam.price},
        quantity=#{updateParam.quantity}
        where id = #{id}
    </update>

    <select id="findById" resultType="Item">
        select id, item_name, price, quantity
        from item
        where id = #{id}
    </select>

    <select id="findAll" resultType="Item">
        select id, item_name, price, quantity
        from item
        <where>
            <if test="itemName != null and itemName != ''">
                and item_name like concat('%',#{itemName},'%')
            </if>
            <if test="maxPrice != null">
                and price &lt;= #{maxPrice}
            </if>
        </where>
    </select>
</mapper>
```

여기서 주의해야 할 점은 아래와 같습니다.

>- Insert SQL은 `<insert>`를 사용합니다.
>- `id`에는 매퍼 인터페이스의 메서드 이름을 지정합니다.
>- 파라미터는 `#{}` 문법을 사용하며, 객체의 프로퍼티 이름을 지정합니다.
>- `#{}`는 `PreparedStatement`를 사용하여 SQL 인젝션을 방지합니다.
>- `useGeneratedKeys`는 `IDENTITY` 전략에서 자동 생성된 키를 반환받을 때 사용합니다.
>- Update SQL은 `<update>`를 사용합니다.
>- 파라미터가 2개 이상이면 `@Param`으로 이름을 지정해야 합니다.
>- Select SQL은 `<select>`를 사용합니다.
>- `resultType`은 반환 타입을 명시합니다.
>- Mybatis는 `<where>`, `<if>` 등으로 동적 쿼리를 지원합니다.
>- XML 특수 문자는 `&lt;`, `&gt;`, `&amp;`로 사용해야 합니다.

## 🤷🏻‍♂️ ItemMapper Mapper Interface의 구현체가 없는데 어떻게 동작하나요?
동작은 아래 순서로 진행됩니다.

1. 애플리케이션 로딩 시점에 MyBatis 스프링 연동 모듈은 `@Mapper` 가 붙어있는 인터페이스를 조사합니다.
2. 해당 인터페이스가 발견되면 동적 프록시 기술을 사용해서 `ItemMapper` 인터페이스의 구현체를 만듭니다.
3. 생성된 구현체를 스프링 빈으로 등록합니다.

## 🤷🏻‍♂️ 그럼 Mybatis는 애노테이션으로 사용 불가능한가요?
가능은 합니다. 아래처럼 사용을 합니다.

```java
@Select("select id, item_name, price, quantity from item where id=#{id}")
Optional<Item> findById(Long id);
```

하지만 동적 SQL이 해결되지 않으므로 간단한 경우에만 사용합니다. 또 `${}` 를 사용하면 SQL 인젝션 공격을 당할 수 있습니다.

그래서 가급적이면 XML을 통해 사용하는 것이 좋습니다.